FROM mcr.microsoft.com/dotnet/sdk:8.0-alpine AS build-stage
ARG TAG_VERSION
WORKDIR /app

# Nuget source authentication
ARG NUGET_SOURCE
ARG NUGET_PAT
ENV VSS_NUGET_EXTERNAL_FEED_ENDPOINTS \
  "{\"endpointCredentials\": [{\"endpoint\":\"${NUGET_SOURCE}\", \"username\":\"docker\", \"password\":\"${NUGET_PAT}\"}]}"
RUN wget https://raw.githubusercontent.com/Microsoft/artifacts-credprovider/master/helpers/installcredprovider.sh && sh installcredprovider.sh

FROM mcr.microsoft.com/dotnet/aspnet:8.0-alpine AS production-stage
EXPOSE 5000
ENV ASPNETCORE_URLS=http://*:8080
WORKDIR /app
COPY . ./
ENV COMPlus_EnableDiagnostics=0
# Configuring timezone America/Sao_Paulo
RUN apk add --no-cache tzdata && \
    ln -sf /usr/share/zoneinfo/America/Sao_Paulo /etc/localtime && \
    echo "America/Sao_Paulo" > /etc/timezone

ENV LANG pt_BR.UTF-8
ENV LANGUAGE pt_BR:en
ENV LC_ALL pt_BR.UTF-8
# Allow TLSv1.2 ou v1
RUN sed -i 's/providers = provider_sect/providers = provider_sect\n\
ssl_conf = ssl_sect\n\
\n\
[ssl_sect]\n\
system_default = system_default_sect\n\
\n\
[system_default_sect]\n\
Options = UnsafeLegacyRenegotiation/' /etc/ssl/openssl.cnf

HEALTHCHECK --interval=1m --timeout=3s \
  CMD wget --no-verbose --tries=1 --spider http://localhost:8080/health-check || exit 1
RUN apk update && apk add --no-cache curl
# Datadog Setup
ENV TRACER_VERSION=3.6.1
RUN mkdir -p /opt/datadog
RUN curl -LO https://github.com/DataDog/dd-trace-dotnet/releases/download/v${TRACER_VERSION}/datadog-dotnet-apm-${TRACER_VERSION}-musl.tar.gz
RUN tar -C /opt/datadog -xzf datadog-dotnet-apm-${TRACER_VERSION}-musl.tar.gz && sh /opt/datadog/createLogPath.sh
ENV CORECLR_ENABLE_PROFILING=1
ENV CORECLR_PROFILER={846F5F1C-F9AE-4B07-969E-05C26BC060D8}
ENV CORECLR_PROFILER_PATH=/opt/datadog/linux-musl-x64/Datadog.Trace.ClrProfiler.Native.so
ENV DD_DOTNET_TRACER_HOME=/opt/datadog
ENV DD_LOGS_INJECTION=true
ENV DD_RUNTIME_METRICS_ENABLED=true
ENV DD_TRACE_DEBUG=true
USER $APP_UID
ENTRYPOINT ["dotnet", "HelloDatadog.dll"]