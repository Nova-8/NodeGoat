FROM mcr.microsoft.com/dotnet/runtime:7.0 AS base
WORKDIR /app
COPY ./src/services/Check.Integration.LegacyWorker/entrypoint.sh /app/entrypoint.sh
RUN chmod +x /app/entrypoint.sh 

FROM mcr.microsoft.com/dotnet/sdk:7.0 AS build
WORKDIR /
COPY ["src/services/Check.Integration.LegacyWorker/Check.Integration.LegacyWorker.csproj", "src/services/Check.Integration.LegacyWorker/"]
COPY . .
RUN dotnet restore "src/services/Check.Integration.LegacyWorker/Check.Integration.LegacyWorker.csproj"
WORKDIR "/src/services/Check.Integration.LegacyWorker" 
RUN dotnet build "Check.Integration.LegacyWorker.csproj" -c Release -o /app/build

FROM build AS publish
RUN dotnet publish "Check.Integration.LegacyWorker.csproj" -c Release -o /app/publish


FROM base AS final
ENV DOTNET_SYSTEM_GLOBALIZATION_INVARIANT=false

RUN apt-get update && apt-get install -y wget ca-certificates gnupg \
&& echo 'deb [trusted=yes] http://apt.newrelic.com/debian/ newrelic non-free' | tee /etc/apt/sources.list.d/newrelic.list \
&& wget --no-check-certificate https://download.newrelic.com/548C16BF.gpg \
&& apt-key add 548C16BF.gpg \
&& apt-get update \
&& apt-get install -y newrelic-netcore20-agent

RUN groupadd --system --gid 1001 newProcelegacySyncUserssUser \
    && useradd --uid 1001 --gid 1001 legacySyncUser

USER legacySyncUser
HEALTHCHECK CMD curl --fail http://localhost:5001/healthz/live
WORKDIR /app
COPY --from=publish /app/publish .
ENTRYPOINT [ "/app/entrypoint.sh" ]
