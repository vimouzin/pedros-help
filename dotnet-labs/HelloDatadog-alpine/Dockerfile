# Build stage
FROM mcr.microsoft.com/dotnet/sdk:8.0-alpine AS build-stage
ARG TAG_VERSION
WORKDIR /app
COPY . ./

# Add the Datadog.Trace NuGet package
RUN dotnet add package Datadog.Trace --version 3.6.1


# Restore dependencies
RUN dotnet restore

# Configurações de Datadog
ENV TRACER_VERSION=3.6.1
RUN mkdir -p /opt/datadog
RUN apk add --no-cache curl
RUN curl -LO https://github.com/DataDog/dd-trace-dotnet/releases/download/v${TRACER_VERSION}/datadog-dotnet-apm-${TRACER_VERSION}-musl.tar.gz
RUN tar -C /opt/datadog -xzf datadog-dotnet-apm-${TRACER_VERSION}-musl.tar.gz && sh /opt/datadog/createLogPath.sh
ENV CORECLR_ENABLE_PROFILING=1
ENV CORECLR_PROFILER={846F5F1C-F9AE-4B07-969E-05C26BC060D8}
ENV CORECLR_PROFILER_PATH=/opt/datadog/linux-musl-x64/Datadog.Trace.ClrProfiler.Native.so
ENV DD_DOTNET_TRACER_HOME=/opt/datadog

# Additional Datadog environment variables
ENV DD_RUNTIME_METRICS_ENABLED=true
ENV DD_TRACE_DEBUG=true

ENV DD_SERVICE=log-datadog
ENV DD_ENV=development
ENV DD_VERSION=5.0.0
ENV DD_SOURCE=.NET
ENV DD_LOGS_INJECTION=true
ENV DD_TRACE_ENABLED=true

# Compile and publish the project in Release mode
RUN dotnet publish -c Release -o out -p:InformationalVersion=${TAG_VERSION}

# Runtime stage
FROM mcr.microsoft.com/dotnet/aspnet:8.0-alpine AS production-stage
WORKDIR /app
# Copy the published files from the build stage
COPY --from=build-stage /app/out .
# Configure environment variables
ENV ASPNETCORE_ENVIRONMENT=Development
ENV DOTNET_LOGGING__CONSOLE__LOGLEVEL__DEFAULT=Debug
ENV ASPNETCORE_URLS=http://+:5000
# Expose port 5000
EXPOSE 5000
# Set the entry point for the application
ENTRYPOINT ["dotnet", "HelloDatadog.dll"]