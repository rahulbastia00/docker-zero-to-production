FROM mcr.microsoft.com/dotnet/sdk:7.0
WORKDIR /app
COPY Worker.csproj .
RUN dotnet restore
COPY . .
RUN dotnet publish -c Release -o /out
CMD ["dotnet", "/out/Worker.dll"]