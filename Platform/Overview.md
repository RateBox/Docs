# RateBox Platform Overview

## 🎯 Mission

RateBox is a comprehensive fraud detection and review platform designed to protect users from scams and fraudulent activities across multiple channels and sources.

## 🏗️ System Architecture

### Platform Components

```
RateBox Platform
├── Rate-New (Core Platform)
│   ├── Strapi CMS (Backend API)
│   ├── Next.js Frontend (Web Dashboard)
│   └── PostgreSQL Database
├── Rate-Importer (Data Crawler)
│   ├── CheckScam.vn Crawler
│   ├── FlareSolverr Integration
│   └── Data Validation Pipeline
└── Rate-Extension (Browser Extension)
    ├── Real-time Fraud Detection
    ├── User Reporting Interface
    └── Data Collection
```

### Data Flow

```
External Sources → Rate-Importer → Validation → Rate-New → Rate-Extension
     ↓                ↓              ↓           ↓            ↓
CheckScam.vn     FlareSolverr   Core Validator  Strapi API   Browser
User Reports     Proxy Rotation  Data Enrichment PostgreSQL  Real-time
API Imports      Captcha Solver  Duplicate Check Dashboard   Detection
```

## 🎯 MVP Architecture Strategy

### Phase 1: Foundation (Week 1-8)
- **Core Validator Package**: Shared validation logic
- **Simple API Server**: Express.js + SQLite
- **Enhanced Crawler**: Integration with validation
- **Web Dashboard**: Next.js 14+ interface

### Benefits of MVP Approach
- ✅ **Quick Time to Value**: 6-8 weeks vs 6+ months
- ✅ **Risk Reduction**: Incremental development
- ✅ **Latest Technology**: Bleeding-edge packages
- ✅ **Natural Migration**: Path to full architecture

## 🔧 Technology Stack

### Backend
- **API Framework**: Express.js → Strapi CMS
- **Database**: SQLite → PostgreSQL
- **Validation**: Custom TypeScript package
- **Queue Processing**: Bull Queue (future)

### Frontend
- **Framework**: Next.js 14+ App Router
- **Styling**: Tailwind CSS
- **State Management**: Zustand/React Query
- **UI Components**: Shadcn/ui

### DevOps & Infrastructure
- **Monorepo**: Turborepo (latest)
- **Containerization**: Docker Compose
- **Process Management**: PM2
- **Deployment**: Docker + CI/CD

### Data Sources
- **Primary**: CheckScam.vn crawler
- **Secondary**: User reports, API imports
- **Browser Extension**: Real-time data collection
- **Manual Input**: Admin interface

## 📊 Current Implementation Status

### ✅ Completed
- **Rate-Importer**: Functional crawler with FlareSolverr
- **MVP Planning**: Architecture and roadmap defined
- **Documentation**: Comprehensive docs structure
- **Rate-New**: Basic platform structure

### 🔄 In Progress
- **Core Validator**: Shared validation package
- **API Server**: Express.js with SQLite
- **Integration**: Crawler → Platform data flow

### 📋 Planned
- **Web Dashboard**: Next.js interface
- **User Management**: Authentication and profiles
- **Advanced Features**: ML validation, multi-source
- **Production**: Scaling and monitoring

## 🎯 Key Features

### Data Collection
- **Automated Crawling**: CheckScam.vn integration
- **Real-time Reports**: Browser extension
- **Manual Entry**: Admin interface
- **API Integration**: External data sources

### Validation & Processing
- **Schema Validation**: Data structure verification
- **Business Rules**: Fraud detection logic
- **Duplicate Detection**: Prevent data redundancy
- **Data Enrichment**: Additional context and metadata

### User Interface
- **Web Dashboard**: Comprehensive admin interface
- **Public API**: Developer access
- **Browser Extension**: Real-time protection
- **Mobile App**: Future mobile access

### Security & Reliability
- **Data Validation**: Multi-layer verification
- **Rate Limiting**: API protection
- **Monitoring**: Health checks and alerts
- **Backup**: Data protection and recovery

## 🔗 Module Integration

### Rate-Importer → Rate-New
```typescript
// Data flow from crawler to platform
Crawler.extract() → Validator.validate() → API.store() → Dashboard.display()
```

### Rate-Extension → Rate-New
```typescript
// Real-time data from extension
Extension.report() → API.receive() → Validator.process() → Database.store()
```

### Cross-Module Communication
- **Shared Packages**: Common validation logic
- **API Contracts**: Standardized data formats
- **Event System**: Real-time updates
- **Configuration**: Centralized settings

## 📈 Scalability Plan

### Phase 1: MVP (1-2 months)
- Single server deployment
- SQLite database
- Basic validation
- Core features

### Phase 2: Growth (3-6 months)
- PostgreSQL migration
- Queue-based processing
- Advanced validation
- Multi-source support

### Phase 3: Scale (6-12 months)
- Distributed architecture
- Microservices
- ML-powered validation
- Global deployment

## 🎯 Success Metrics

### Technical Metrics
- **Data Quality**: 95%+ validation accuracy
- **Performance**: <200ms API response time
- **Reliability**: 99.9% uptime
- **Scalability**: Handle 1M+ records

### Business Metrics
- **User Protection**: Fraud prevention rate
- **Data Coverage**: Source diversity
- **User Adoption**: Platform usage
- **Community Growth**: User contributions

## 🤝 Development Workflow

### 1. Module Development
- Independent module development
- Shared package updates
- API contract maintenance
- Documentation updates

### 2. Integration Testing
- Cross-module compatibility
- Data flow validation
- Performance testing
- Security verification

### 3. Deployment
- Staged rollouts
- Monitoring and alerts
- Rollback procedures
- Performance optimization

---

**Architecture**: MVP-First Modular Platform  
**Timeline**: 6-8 weeks MVP, 6-12 months full scale  
**Technology**: Latest packages, bleeding-edge features
