#See https://aka.ms/customizecontainer to learn how to customize your debug container and how Visual Studio uses this Dockerfile to build your images for faster debugging.

FROM mcr.microsoft.com/dotnet/aspnet:8.0 AS base
USER app
WORKDIR /app
EXPOSE 8080

FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
ARG BUILD_CONFIGURATION=Release
WORKDIR /src
COPY ["diceroll-app.csproj", "diceroll-app/"]
RUN dotnet restore "./diceroll-app/./diceroll-app.csproj"
WORKDIR "/src/diceroll-app"
COPY . .
RUN dotnet build "./diceroll-app.csproj" -c $BUILD_CONFIGURATION -o /app/build

# Add dependencies for splunk-otel-dotnet-install.sh
RUN apt-get update && \
	apt-get install -y unzip

# Download Splunk OTel .NET installer
RUN curl -sSfL https://github.com/signalfx/splunk-otel-dotnet/releases/latest/download/splunk-otel-dotnet-install.sh -O

# Install the distribution
RUN sh ./splunk-otel-dotnet-install.sh

FROM build AS publish
ARG BUILD_CONFIGURATION=Release
RUN dotnet publish "./diceroll-app.csproj" -c $BUILD_CONFIGURATION -o /app/publish /p:UseAppHost=false

FROM base AS final

# Copy instrumentation file tree
WORKDIR "//home/app/.splunk-otel-dotnet"
COPY --from=build /root/.splunk-otel-dotnet/ .

WORKDIR /app
COPY --from=publish /app/publish .
COPY entrypoint.sh .

ENV OTEL_SERVICE_NAME=diceroll-app
ENV OTEL_RESOURCE_ATTRIBUTES='deployment.environment=otel-yourname'

ENTRYPOINT ["sh", "entrypoint.sh"]
CMD ["dotnet", "diceroll-app.dll"]