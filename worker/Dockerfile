FROM mcr.microsoft.com/dotnet/sdk:7.0 AS builder

WORKDIR /src
COPY Worker.csproj .
RUN dotnet restore

COPY . .
RUN dotnet publish -c Release -o /app/publish

FROM mcr.microsoft.com/dotnet/runtime:7.0-alpine
WORKDIR /app

COPY --from=builder /app/publish .

CMD ["dotnet", "Worker.dll"]