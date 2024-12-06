FROM mcr.microsoft.com/dotnet/sdk:8.0-alpine AS build-stage
ARG TAG_VERSION
WORKDIR /app
COPY . ./
# Nuget source authentication
ARG NUGET_SOURCE
ARG NUGET_PAT
ENV VSS_NUGET_EXTERNAL_FEED_ENDPOINTS \
  "{\"endpointCredentials\": [{\"endpoint\":\"${NUGET_SOURCE}\", \"username\":\"docker\", \"password\":\"${NUGET_PAT}\"}]}"
RUN wget https://raw.githubusercontent.com/Microsoft/artifacts-credprovider/master/helpers/installcredprovider.sh && sh installcredprovider.sh
# end: Nuget source authentication
RUN dotnet publish -c Release -o out HelloDatadog.csproj \
  -p:InformationalVersion=${TAG_VERSION}
# Build runtime image

FROM mcr.microsoft.com/dotnet/aspnet:8.0-alpine AS production-stage
WORKDIR /app


ENV COMPlus_EnableDiagnostics=1
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
# HEALTHCHECK --interval=1m --timeout=3s \
#   CMD wget --no-verbose --tries=1 --spider http://localhost:8080/health-check || exit 1
# RUN apk update && apk add --no-cache curl
# Datadog Setup

COPY --from=build-stage /app/out/ .

ENV ASPNETCORE_ENVIRONMENT=Development
ENV DOTNET_LOGGING__CONSOLE__LOGLEVEL__DEFAULT=Debug
ENV ASPNETCORE_URLS=http://*:8080

EXPOSE 8080
USER $APP_UID
ENTRYPOINT ["dotnet", "HelloDatadog.dll"]