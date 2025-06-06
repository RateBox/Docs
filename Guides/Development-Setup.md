# RateBox Development Setup Guide

## 🎯 Overview

This guide helps you set up a complete RateBox development environment on Windows 11 with PowerShell.

## 🛠️ Prerequisites

### Required Software

```powershell
# Check PowerShell version (should be 7+)
$PSVersionTable.PSVersion

# Required tools
winget install Git.Git
winget install Docker.DockerDesktop
winget install Microsoft.VisualStudioCode
winget install OpenJS.NodeJS
```

### Development Tools

```powershell
# Install additional development tools
npm install -g pnpm
npm install -g @turborepo/cli
npm install -g typescript
npm install -g ts-node
```

## 📁 Project Structure

```
D:\Projects\JOY\
├── RateBox-Docs\          # 📚 Documentation repository
├── Rate-New\              # 🏗️ Core platform (Strapi + Next.js)
├── Rate-Importer\         # 🕷️ Data crawler module
└── Rate-Extension\        # 🌐 Browser extension (future)
```

## 🚀 Quick Setup

### 1. Clone Repositories

```powershell
# Navigate to projects directory
cd D:\Projects\JOY\

# Clone documentation
git clone https://github.com/RateBox/Docs.git RateBox-Docs

# Clone core platform
git clone https://github.com/RateBox/Rate-New.git Rate-New

# Clone crawler module
git clone https://github.com/RateBox/Rate-Importer.git Rate-Importer
```

### 2. Environment Setup

```powershell
# Copy environment templates
Copy-Item "Rate-New\.env.example" "Rate-New\.env"
Copy-Item "Rate-Importer\.env.example" "Rate-Importer\.env"

# Edit environment files
code Rate-New\.env
code Rate-Importer\.env
```

### 3. Database Setup

```powershell
# Start PostgreSQL with Docker
docker run -d `
  --name ratebox-db `
  --network ratebox `
  -e POSTGRES_DB=ratebox `
  -e POSTGRES_USER=ratebox `
  -e POSTGRES_PASSWORD=password `
  -p 5432:5432 `
  postgres:14

# Verify database connection
docker exec -it ratebox-db psql -U ratebox -d ratebox -c "\dt"
```

### 4. Install Dependencies

```powershell
# Rate-New dependencies
cd Rate-New
pnpm install

# Rate-Importer dependencies
cd ..\Rate-Importer
npm install
```

## 🔧 Module-Specific Setup

### Rate-New (Core Platform)

```powershell
cd D:\Projects\JOY\Rate-New

# Install dependencies
pnpm install

# Setup database
pnpm run db:migrate
pnpm run db:seed

# Start development server
pnpm run dev
```

**Access Points:**
- **Strapi Admin**: http://localhost:1337/admin
- **Next.js Frontend**: http://localhost:3000
- **API Endpoint**: http://localhost:1337/api

### Rate-Importer (Crawler)

```powershell
cd D:\Projects\JOY\Rate-Importer

# Install dependencies
npm install

# Start FlareSolverr (required for crawler)
docker run -d `
  --name flaresolverr `
  --network ratebox `
  -p 8191:8191 `
  ghcr.io/flaresolverr/flaresolverr:latest

# Test crawler
npx ts-node src/test-crawler.ts 0901234567

# Start crawler service
npm run start
```

**Environment Variables:**
```bash
FLARESOLVERR_URL=http://localhost:8191
CHECKSCAM_BASE_URL=https://checkscam.vn
API_BASE_URL=http://localhost:1337/api
```

## 🐳 Docker Development

### Complete Stack

```yaml
# docker-compose.dev.yml
version: '3.8'

services:
  db:
    image: postgres:14
    environment:
      POSTGRES_DB: ratebox
      POSTGRES_USER: ratebox
      POSTGRES_PASSWORD: password
    ports:
      - "5432:5432"
    networks:
      - ratebox

  flaresolverr:
    image: ghcr.io/flaresolverr/flaresolverr:latest
    ports:
      - "8191:8191"
    networks:
      - ratebox

  rate-new:
    build: ./Rate-New
    ports:
      - "1337:1337"
      - "3000:3000"
    environment:
      - DATABASE_URL=postgresql://ratebox:password@db:5432/ratebox
    depends_on:
      - db
    networks:
      - ratebox

  rate-importer:
    build: ./Rate-Importer
    environment:
      - FLARESOLVERR_URL=http://flaresolverr:8191
      - API_BASE_URL=http://rate-new:1337/api
    depends_on:
      - rate-new
      - flaresolverr
    networks:
      - ratebox

networks:
  ratebox:
    driver: bridge
```

### Start Development Stack

```powershell
# Start complete development environment
docker-compose -f docker-compose.dev.yml up -d

# View logs
docker-compose -f docker-compose.dev.yml logs -f

# Stop environment
docker-compose -f docker-compose.dev.yml down
```

## 🧪 Testing Setup

### Unit Tests

```powershell
# Rate-New tests
cd Rate-New
pnpm run test

# Rate-Importer tests
cd ..\Rate-Importer
npm run test
```

### Integration Tests

```powershell
# Start test database
docker run -d `
  --name ratebox-test-db `
  -e POSTGRES_DB=ratebox_test `
  -e POSTGRES_USER=ratebox `
  -e POSTGRES_PASSWORD=password `
  -p 5433:5432 `
  postgres:14

# Run integration tests
npm run test:integration
```

### E2E Tests

```powershell
# Install Playwright
npx playwright install

# Run E2E tests
npm run test:e2e
```

## 🔍 Development Tools

### VS Code Extensions

```json
// .vscode/extensions.json
{
  "recommendations": [
    "ms-vscode.vscode-typescript-next",
    "bradlc.vscode-tailwindcss",
    "ms-vscode.vscode-json",
    "esbenp.prettier-vscode",
    "ms-vscode.vscode-eslint",
    "ms-python.python",
    "ms-vscode.powershell"
  ]
}
```

### VS Code Settings

```json
// .vscode/settings.json
{
  "typescript.preferences.importModuleSpecifier": "relative",
  "editor.formatOnSave": true,
  "editor.codeActionsOnSave": {
    "source.fixAll.eslint": true
  },
  "files.associations": {
    "*.env.*": "dotenv"
  }
}
```

## 📊 Monitoring & Debugging

### Health Checks

```powershell
# Check service health
curl http://localhost:1337/api/health
curl http://localhost:8191/v1

# Database connection test
docker exec -it ratebox-db psql -U ratebox -d ratebox -c "SELECT version();"
```

### Log Monitoring

```powershell
# View application logs
docker-compose logs -f rate-new
docker-compose logs -f rate-importer

# Database logs
docker logs ratebox-db

# FlareSolverr logs
docker logs flaresolverr
```

### Performance Monitoring

```powershell
# Monitor resource usage
docker stats

# Check port usage
netstat -an | findstr :1337
netstat -an | findstr :3000
netstat -an | findstr :8191
```

## 🔧 Common Development Tasks

### Database Management

```powershell
# Reset database
docker exec -it ratebox-db psql -U ratebox -d ratebox -c "DROP SCHEMA public CASCADE; CREATE SCHEMA public;"

# Backup database
docker exec -t ratebox-db pg_dump -U ratebox ratebox > backup.sql

# Restore database
docker exec -i ratebox-db psql -U ratebox ratebox < backup.sql
```

### Code Generation

```powershell
# Generate Strapi types
cd Rate-New
pnpm run strapi:generate-types

# Generate API client
pnpm run generate:api-client
```

### Package Management

```powershell
# Update dependencies
cd Rate-New
pnpm update

cd ..\Rate-Importer
npm update

# Check for security vulnerabilities
npm audit
pnpm audit
```

## 🚨 Troubleshooting

### Common Issues

**Port Conflicts:**
```powershell
# Find processes using ports
netstat -ano | findstr :1337
netstat -ano | findstr :3000

# Kill process by PID
taskkill /PID <PID> /F
```

**Docker Issues:**
```powershell
# Reset Docker
docker system prune -a

# Restart Docker Desktop
Restart-Service docker
```

**Database Connection:**
```powershell
# Test database connection
docker exec -it ratebox-db psql -U ratebox -d ratebox -c "SELECT 1;"

# Check database logs
docker logs ratebox-db
```

### Development Workflow

1. **Start Services**: Database, FlareSolverr
2. **Start Rate-New**: Strapi backend + Next.js frontend
3. **Start Rate-Importer**: Crawler service
4. **Test Integration**: Verify data flow
5. **Monitor Logs**: Check for errors

## 📚 Additional Resources

- **[Platform Overview](../platform/overview.md)** - System architecture
- **[Integration Guide](../platform/integration-guide.md)** - Module integration
- **[MVP Architecture](../platform/mvp-architecture.md)** - Development roadmap
- **[Rate-New Docs](../modules/rate-new/README.md)** - Core platform docs
- **[Rate-Importer Docs](../modules/rate-importer/README.md)** - Crawler docs

---

**Environment**: Windows 11 + PowerShell 7  
**Stack**: Node.js, TypeScript, Docker, PostgreSQL  
**Architecture**: MVP-first modular platform
