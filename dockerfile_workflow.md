# Docker Workflow and Compose Setup Guide

## Complete Workflow Overview

This guide covers the entire workflow from Dockerfile creation to Docker Compose orchestration with a SQL Server database.

## Step 1: Project Structure

```
my-web-api/
├── src/
│   ├── MyWebApi/
│   │   ├── MyWebApi.csproj
│   │   ├── Program.cs
│   │   ├── appsettings.json
│   │   ├── appsettings.Development.json
│   │   ├── appsettings.Production.json
│   │   └── Controllers/
│   └── MyWebApi.sln
├── Dockerfile
├── docker-compose.yml
├── docker-compose.override.yml
└── .dockerignore
```

## Step 2: Enhanced Dockerfile

```dockerfile
# Stage 1: Build the app
FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
WORKDIR /src

# Copy solution file and project files
COPY *.sln ./
COPY src/MyWebApi/*.csproj ./src/MyWebApi/

# Restore dependencies
RUN dotnet restore

# Copy source code and build
COPY . ./
RUN dotnet publish src/MyWebApi/MyWebApi.csproj -c Release -o /app/publish --no-restore

# Stage 2: Runtime
FROM mcr.microsoft.com/dotnet/aspnet:8.0 AS runtime
WORKDIR /app

# Create non-root user (optional but recommended)
RUN adduser --disabled-password --gecos '' appuser && chown -R appuser /app
USER appuser

# Copy published app
COPY --from=build /app/publish .

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
    CMD curl -f http://localhost:80/health || exit 1

# Expose port
EXPOSE 80

# Set entrypoint
ENTRYPOINT ["dotnet", "MyWebApi.dll"]
```

## Step 3: .dockerignore File

```dockerignore
# Git
.git
.gitignore
.gitattributes

# Documentation
README.md
*.md

# Docker
Dockerfile*
docker-compose*

# .NET
bin/
obj/
out/
.vs/
.vscode/

# OS
.DS_Store
Thumbs.db

# Logs
*.log
logs/

# Runtime
*.user
*.cache
```

## Step 4: Application Configuration

### appsettings.json
```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=localhost;Database=MyWebApiDb;Trusted_Connection=true;TrustServerCertificate=true;"
  },
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning"
    }
  },
  "AllowedHosts": "*"
}
```

### appsettings.Production.json
```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=sqlserver;Database=MyWebApiDb;User Id=sa;Password=${SA_PASSWORD};TrustServerCertificate=true;"
  },
  "Logging": {
    "LogLevel": {
      "Default": "Warning",
      "Microsoft.AspNetCore": "Warning"
    }
  }
}
```

## Step 5: Docker Compose Configuration

### docker-compose.yml (Base Configuration)
```yaml
version: '3.8'

services:
  mywebapi:
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - "8080:80"
    depends_on:
      sqlserver:
        condition: service_healthy
    environment:
      - ASPNETCORE_ENVIRONMENT=Production
      - SA_PASSWORD=${SA_PASSWORD}
    networks:
      - app-network
    restart: unless-stopped

  sqlserver:
    image: mcr.microsoft.com/mssql/server:2022-latest
    environment:
      - ACCEPT_EULA=Y
      - SA_PASSWORD=${SA_PASSWORD}
      - MSSQL_PID=Express
    ports:
      - "1433:1433"
    volumes:
      - sqlserver_data:/var/opt/mssql
      - ./scripts:/docker-entrypoint-initdb.d
    networks:
      - app-network
    restart: unless-stopped
    healthcheck:
      test: ["CMD-SHELL", "/opt/mssql-tools/bin/sqlcmd -S localhost -U sa -P ${SA_PASSWORD} -Q 'SELECT 1'"]
      interval: 30s
      timeout: 10s
      retries: 5
      start_period: 30s

volumes:
  sqlserver_data:
    driver: local

networks:
  app-network:
    driver: bridge
```

### docker-compose.override.yml (Development Overrides)
```yaml
version: '3.8'

services:
  mywebapi:
    environment:
      - ASPNETCORE_ENVIRONMENT=Development
    volumes:
      - ./src:/src
      - ~/.nuget/packages:/root/.nuget/packages:ro
    ports:
      - "5000:80"

  sqlserver:
    ports:
      - "1434:1433"
    environment:
      - SA_PASSWORD=DevPassword123!
```

## Step 6: Environment Variables

### .env file
```env
# Database Configuration
SA_PASSWORD=YourStrongPassword123!

# Application Configuration
ASPNETCORE_ENVIRONMENT=Production

# Compose Project Name
COMPOSE_PROJECT_NAME=mywebapi
```

## Step 7: Database Initialization Scripts

### scripts/01-create-database.sql
```sql
USE master;
GO

IF NOT EXISTS (SELECT name FROM sys.databases WHERE name = 'MyWebApiDb')
BEGIN
    CREATE DATABASE MyWebApiDb;
END
GO

USE MyWebApiDb;
GO

-- Create your tables here
IF NOT EXISTS (SELECT * FROM sysobjects WHERE name='Users' AND xtype='U')
BEGIN
    CREATE TABLE Users (
        Id INT IDENTITY(1,1) PRIMARY KEY,
        Name NVARCHAR(100) NOT NULL,
        Email NVARCHAR(255) NOT NULL UNIQUE,
        CreatedAt DATETIME2 DEFAULT GETUTCDATE()
    );
END
GO
```

## Step 8: Workflow Commands

### Development Workflow

1. **Start Development Environment**
   ```bash
   # Create .env file with development settings
   echo "SA_PASSWORD=DevPassword123!" > .env
   
   # Start services
   docker-compose up -d
   
   # View logs
   docker-compose logs -f mywebapi
   ```

2. **Database Operations**
   ```bash
   # Connect to SQL Server
   docker-compose exec sqlserver /opt/mssql-tools/bin/sqlcmd -S localhost -U sa -P DevPassword123!
   
   # Backup database
   docker-compose exec sqlserver /opt/mssql-tools/bin/sqlcmd -S localhost -U sa -P DevPassword123! -Q "BACKUP DATABASE MyWebApiDb TO DISK = '/var/opt/mssql/backup/MyWebApiDb.bak'"
   ```

3. **Rebuild Application**
   ```bash
   # Rebuild only the API
   docker-compose build mywebapi
   docker-compose up -d mywebapi
   ```

### Production Deployment

1. **Production Build and Deploy**
   ```bash
   # Set production environment
   echo "SA_PASSWORD=YourProductionPassword123!" > .env
   echo "ASPNETCORE_ENVIRONMENT=Production" >> .env
   
   # Build and start
   docker-compose -f docker-compose.yml up -d --build
   ```

2. **Health Checks**
   ```bash
   # Check service status
   docker-compose ps
   
   # Check health
   docker-compose exec mywebapi curl http://localhost:80/health
   ```

## Step 9: Additional Configurations

### Adding Redis Cache
```yaml
# Add to docker-compose.yml services section
redis:
  image: redis:7-alpine
  ports:
    - "6379:6379"
  volumes:
    - redis_data:/data
  networks:
    - app-network
  restart: unless-stopped

# Add to volumes section
volumes:
  redis_data:
    driver: local
```

### Adding Nginx Reverse Proxy
```yaml
nginx:
  image: nginx:alpine
  ports:
    - "80:80"
    - "443:443"
  volumes:
    - ./nginx.conf:/etc/nginx/nginx.conf:ro
    - ./certs:/etc/nginx/certs:ro
  depends_on:
    - mywebapi
  networks:
    - app-network
  restart: unless-stopped
```

## Step 10: Monitoring and Logging

### docker-compose.logging.yml
```yaml
version: '3.8'

services:
  mywebapi:
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"

  sqlserver:
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"
```

## Best Practices Summary

1. **Multi-stage builds** for smaller production images
2. **Health checks** for reliable service discovery
3. **Environment-specific configurations** using override files
4. **Volume persistence** for database data
5. **Network isolation** using custom networks
6. **Proper secrets management** using environment files
7. **Database initialization** scripts for setup automation
8. **Logging configuration** for debugging and monitoring
9. **Resource limits** and restart policies for production
10. **Security considerations** with non-root users and proper passwords

## Quick Start Commands

```bash
# Clone/setup project
git clone <your-repo>
cd my-web-api

# Create environment file
echo "SA_PASSWORD=DevPassword123!" > .env

# Start everything
docker-compose up -d

# Check status
docker-compose ps

# View logs
docker-compose logs -f

# Stop everything
docker-compose down

# Clean up (removes volumes)
docker-compose down -v
```
