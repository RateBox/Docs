# Rate Crawler Module

## 📋 Overview

The Rate Crawler module is responsible for extracting scam reports and fraud data from various sources, with primary focus on CheckScam.vn integration.

## 🏗️ Architecture

### Components
- **CheckScam Crawler**: Main crawler for checkscam.vn
- **FlareSolverr Integration**: Cloudflare bypass proxy
- **Core Validator**: Data validation and enrichment
- **Result Storage**: JSON and database persistence

### Data Flow
```
CheckScam.vn → FlareSolverr → Crawler → Validator → Storage
```

## 🚀 Quick Start

### Prerequisites
```bash
# Docker & Docker Compose
docker --version
docker compose version

# Node.js (Latest)
node --version
npm --version
```

### Setup
```bash
# 1. Start FlareSolverr
cd Rate-Importer
docker compose up -d

# 2. Install dependencies
npm install

# 3. Configure environment
cp .env.example .env
# Edit .env with your settings
```

### Usage
```bash
# Extract scammer links
node crawl.js links

# Crawl detailed data
node crawl.js crawl

# Clean up project
node crawl.js cleanup
```

## 📊 Current Status

- ✅ **Working**: CheckScam.vn crawler with FlareSolverr
- ✅ **Integrated**: Docker Compose setup
- ✅ **Documented**: Complete API and deployment docs
- 🔄 **In Progress**: MVP platform integration

## 🔗 Related Documentation

- [Crawler API Reference](./crawler-API.md)
- [Deployment Guide](./crawler-Deployment.md)
- [Architecture Details](./crawler-Architecture.md)
- [MVP Integration Plan](../Rate-Extension/mvp-Architecture.md)

## 📁 Module Location

```
Rate-Importer/                      # Standalone crawler project
├── Scripts/                        # Crawler implementation
├── Results/                        # Crawled data output
├── Logs/                           # Crawler logs
├── docker-compose.yml              # FlareSolverr setup
└── crawl.js                        # CLI interface
```

## 🎯 Integration Roadmap

### Phase 1: MVP Integration (Week 1-8)
- [ ] Extract validation logic to shared package
- [ ] Create crawler adapter for MVP platform
- [ ] Integrate with central API server
- [ ] Add web dashboard for crawler management

### Phase 2: Advanced Features (Month 3-4)
- [ ] Multi-source support (other fraud sites)
- [ ] Real-time crawling with queues
- [ ] Advanced validation with ML
- [ ] Monitoring and alerting

## 🔧 Configuration

### Environment Variables
```bash
# FlareSolverr
FLARESOLVERR_URL=http://localhost:8191/v1
FLARESOLVERR_TIMEOUT=60000

# CheckScam
CHECKSCAM_BASE_URL=https://checkscam.vn
CRAWLER_DELAY_MS=2000
CRAWLER_MAX_RETRIES=3
```

### Docker Network
```yaml
# Uses 'Rate' network for integration
networks:
  Rate:
    driver: bridge
```

## 📈 Performance Metrics

- **Success Rate**: ~85% with FlareSolverr
- **Speed**: ~100 records/hour
- **Accuracy**: 95%+ data validation
- **Reliability**: Auto-retry with exponential backoff

## 🐛 Troubleshooting

### Common Issues
1. **FlareSolverr not responding**: Check Docker container status
2. **Cloudflare blocking**: Verify proxy configuration
3. **Data validation errors**: Check schema compatibility
4. **Network timeouts**: Adjust timeout settings

### Debug Commands
```bash
# Check FlareSolverr health
curl http://localhost:8191/v1

# Test crawler with debug
DEBUG=true node crawl.js links

# View recent logs
tail -f Logs/crawler.log
```

---

*This module is part of the Rate Platform ecosystem. For system-wide documentation, see [Rate Platform Overview](../README.md).*
