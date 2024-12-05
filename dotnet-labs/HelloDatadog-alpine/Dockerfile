FROM mcr.microsoft.com/dotnet/sdk:8.0-alpine AS build-stage
ARG TAG_VERSION
WORKDIR /app
# Copie todo o conteúdo do projeto
COPY . ./
# Restaurar dependências
RUN dotnet restore HelloDatadog.csproj
# Publicar o projeto
RUN dotnet publish -c Release -o out HelloDatadog.csproj \
  -p:InformationalVersion=${TAG_VERSION}
# Build runtime image
FROM mcr.microsoft.com/dotnet/aspnet:8.0-alpine AS production-stage
EXPOSE 8080
ENV ASPNETCORE_URLS=http://*:8080
WORKDIR /app
# Copie os arquivos publicados do estágio de build
COPY --from=build-stage /app/out/ .
ENV COMPlus_EnableDiagnostics=0
# Configurar o timezone para America/Sao_Paulo
RUN apk add --no-cache tzdata && \
    ln -sf /usr/share/zoneinfo/America/Sao_Paulo /etc/localtime && \
    echo "America/Sao_Paulo" > /etc/timezone

# Ensure directories exist before applying chown and chmod
RUN mkdir -p /app /opt/datadog /tmp /var/tmp /usr/bin /etc/ssl && \
    touch /usr/bin/wget /etc/ssl/openssl.cnf && \
    chown -R 1654:1654 /app /opt/datadog /tmp /var/tmp /usr/bin/wget /etc/ssl/openssl.cnf && \
    chmod -R u+rwX,g+rX,o+rX /app /opt/datadog /tmp /var/tmp /usr/bin/wget /etc/ssl/openssl.cnf

    
ENV LANG pt_BR.UTF-8
ENV LANGUAGE pt_BR:en
ENV LC_ALL pt_BR.UTF-8
# Permitir TLSv1.2 ou v1
RUN sed -i 's/providers = provider_sect/providers = provider_sect\n\
ssl_conf = ssl_sect\n\
\n\
[ssl_sect]\n\
system_default = system_default_sect\n\
\n\
[system_default_sect]\n\
Options = UnsafeLegacyRenegotiation/' /etc/ssl/openssl.cnf
# Configurar o usuário (se necessário)
USER $APP_UID
# Ponto de entrada
ENTRYPOINT ["dotnet", "HelloDatadog.dll"]