# RateBox Quick Start Guide

## 🚀 Get Running in 15 Minutes

This guide gets you up and running with RateBox platform quickly for development and testing.

## 📋 Prerequisites

```powershell
# Check requirements
node --version    # Should be 18+
docker --version  # Should be 20+
git --version     # Any recent version
```

## ⚡ Quick Setup

### 1. Clone Projects

```powershell
# Create projects directory
mkdir D:\Projects\JOY
cd D:\Projects\JOY

# Clone repositories
git clone https://github.com/RateBox/Docs.git RateBox-Docs
git clone https://github.com/RateBox/Rate-New.git Rate-New
git clone https://github.com/RateBox/Rate-Importer.git Rate-Importer
```

### 2. Start Core Services

```powershell
# Create Docker network
docker network create ratebox

# Start PostgreSQL
docker run -d --name ratebox-db --network ratebox `
  -e POSTGRES_DB=ratebox -e POSTGRES_USER=ratebox -e POSTGRES_PASSWORD=password `
  -p 5432:5432 postgres:14

# Start FlareSolverr
docker run -d --name flaresolverr --network ratebox `
  -p 8191:8191 ghcr.io/flaresolverr/flaresolverr:latest
```

### 3. Setup Rate-New Platform

```powershell
cd Rate-New

# Copy environment
Copy-Item .env.example .env

# Install dependencies
pnpm install

# Setup database
pnpm run db:migrate

# Start development server
pnpm run dev
```

**Access**: http://localhost:1337/admin (Strapi) + http://localhost:3000 (Next.js)

### 4. Setup Rate-Importer

```powershell
cd ..\Rate-Importer

# Copy environment
Copy-Item .env.example .env

# Edit environment file
notepad .env
```

**Required Environment Variables:**
```bash
FLARESOLVERR_URL=http://localhost:8191
CHECKSCAM_BASE_URL=https://checkscam.vn
API_BASE_URL=http://localhost:1337/api
```

```powershell
# Install dependencies
npm install

# Test crawler
npx ts-node src/test-crawler.ts 0901234567
```

## 🧪 Test Integration

### 1. Verify Services

```powershell
# Check database
docker exec -it ratebox-db psql -U ratebox -d ratebox -c "SELECT version();"

# Check FlareSolverr
curl http://localhost:8191/v1

# Check Rate-New API
curl http://localhost:1337/api/health
```

### 2. Test Data Flow

```powershell
# Run crawler test
cd Rate-Importer
npm run test:integration

# Check data in Rate-New
cd ..\Rate-New
pnpm run db:query "SELECT COUNT(*) FROM scam_reports;"
```

## 📊 Verify Setup

### Health Checks

```powershell
# Service status
docker ps

# Port usage
netstat -an | findstr ":1337\|:3000\|:8191\|:5432"

# Application logs
docker logs ratebox-db
docker logs flaresolverr
```

### Access Points

- **Strapi Admin**: http://localhost:1337/admin
- **Next.js Frontend**: http://localhost:3000
- **FlareSolverr**: http://localhost:8191
- **Database**: localhost:5432

## 🎯 Next Steps

1. **Read Documentation**: Browse `RateBox-Docs/` for detailed guides
2. **Explore Modules**: Check individual module documentation
3. **Development**: Follow [Development Setup](development-setup.md) for full environment
4. **Contribute**: See [Contributing Guide](../resources/contributing.md)

## 🚨 Common Issues

**Port Conflicts:**
```powershell
# Kill processes using ports
Get-Process -Id (Get-NetTCPConnection -LocalPort 1337).OwningProcess | Stop-Process
```

**Docker Issues:**
```powershell
# Restart services
docker restart ratebox-db flaresolverr
```

**Database Connection:**
```powershell
# Reset database
docker exec -it ratebox-db psql -U ratebox -d ratebox -c "DROP SCHEMA public CASCADE; CREATE SCHEMA public;"
```

---

**Next**: [Development Setup](development-setup.md) for complete environment
