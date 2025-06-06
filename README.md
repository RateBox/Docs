# RateBox Documentation

## 📋 Overview

Central documentation repository for the RateBox platform ecosystem, including all modules, architecture, and development guides.

## 🏗️ Platform Architecture

RateBox is a comprehensive fraud detection and review platform consisting of multiple interconnected modules:

### Core Platform
- **Rate-New**: Main platform (Strapi + Next.js)
- **Rate-Importer**: Data crawler module
- **Rate-Extension**: Browser extension for data collection

### Architecture Approach
- **MVP First**: Start simple, scale gradually
- **Modular Design**: Independent modules with clear interfaces
- **Latest Tech**: Bleeding-edge packages and frameworks
- **Monorepo**: Turborepo for build orchestration

## 📁 Documentation Structure

```
RateBox-Docs/
├── README.md                       # This file
├── Platform/                      # Platform-wide documentation
│   ├── Overview.md                 # System overview
│   ├── MVP-Architecture.md         # MVP architecture plan
│   ├── Architecture-Comparison.md  # Full vs MVP analysis
│   └── Integration-Guide.md        # Module integration
├── Modules/                        # Module-specific docs
│   ├── rate-new/                   # Main platform docs
│   │   ├── README.md               # Platform overview
│   │   ├── API.md                  # API documentation
│   │   ├── Deployment-Guide.md     # Deployment guides
│   │   └── Development-Guide.md    # Development guides
│   ├── rate-importer/              # Crawler module docs
│   │   ├── README.md               # Crawler overview
│   │   ├── API.md                  # CLI and API reference
│   │   ├── Architecture.md         # Technical details
│   │   ├── Deployment.md           # Setup and deployment
│   │   └── Roadmap.md              # Development roadmap
│   └── rate-extension/             # Browser extension docs
│       ├── README.md               # Extension overview
│       ├── API.md                  # Extension API
│       └── Development-Setup.md    # Development guide
├── Guides/                         # Development guides
│   ├── Quick-Start.md              # Quick start guide
│   ├── Development-Workflow.md     # Development process
│   ├── Deployment-Guide.md         # Production deployment
│   └── Troubleshooting.md          # Common issues
└── Resources/                      # Additional resources
    ├── Glossary.md                 # Terms and definitions
    ├── Changelog.md                # Platform changelog
    ├── Contributing.md             # Contribution guidelines
    ├── Adding-New-Locale.md        # Adding new locale
    ├── Backup-Disaster-Recovery.md # Backup and disaster recovery
    ├── Dynamic-Content-Architecture.md # Dynamic content architecture
    ├── Implementation-Plan-Dynamic-Zone-Native.md # Implementation plan
    └── Strapi-I18N-Single-Types.md # Strapi I18N single types
```

## 🚀 Quick Start

### Prerequisites
- Node.js 18+
- Docker & Docker Compose
- PowerShell 7 (Windows 11)
- Git

### Platform Setup
```bash
# 1. Clone main platform
git clone <rate-new-repo> Rate-New
cd Rate-New
yarn install

# 2. Clone crawler module
git clone <rate-importer-repo> Rate-Importer

# 3. Clone documentation
git clone https://github.com/RateBox/Docs RateBox-Docs

# 4. Start development
yarn dev
```

## 📚 Documentation Index

### 🏗️ Platform Architecture
- [System Overview](./Platform/Overview.md)
- [MVP Architecture](./Platform/MVP-Architecture.md)
- [Architecture Comparison](./Platform/Architecture-Comparison.md)
- [Module Integration](./Platform/Integration-Guide.md)

### 📦 Module Documentation
- [Rate-New Platform](./Modules/Rate-New/README.md) - Main Strapi + Next.js platform
- [Rate-Importer Crawler](./Modules/Rate-Importer/README.md) - Data crawling module
- [Rate-Extension](./Modules/Rate-Extension/README.md) - Browser extension

### 🛠️ Development Guides
- [Quick Start](./Guides/Quick-Start.md) - Setup and first steps
- [Development Workflow](./Guides/Development-Workflow.md) - Development process
- [Deployment Guide](./Guides/Deployment-Guide.md) - Production deployment
- [Troubleshooting](./Guides/Troubleshooting.md) - Common issues and solutions

### 📖 Resources
- [Glossary](./Resources/Glossary.md) - Terms and definitions
- [Changelog](./Resources/Changelog.md) - Platform updates
- [Contributing](./Resources/Contributing.md) - How to contribute
- [Adding New Locale](./Resources/Adding-New-Locale.md) - Adding new locale
- [Backup and Disaster Recovery](./Resources/Backup-Disaster-Recovery.md) - Backup and disaster recovery
- [Dynamic Content Architecture](./Resources/Dynamic-Content-Architecture.md) - Dynamic content architecture
- [Implementation Plan](./Resources/Implementation-Plan-Dynamic-Zone-Native.md) - Implementation plan
- [Strapi I18N Single Types](./Resources/Strapi-I18N-Single-Types.md) - Strapi I18N single types

## 🎯 Current Status

### ✅ Completed
- Rate-Importer crawler with FlareSolverr integration
- MVP architecture planning and documentation
- Basic Rate-New platform structure

### 🔄 In Progress
- MVP implementation (Week 1-8)
- Core validator package development
- API server with SQLite

### 📋 Planned
- Web dashboard development
- Multi-source crawler support
- Advanced validation with ML
- Production deployment

## 🔗 Repository Links

- **Documentation**: https://github.com/RateBox/Docs
- **Main Platform**: [Rate-New Repository]
- **Crawler Module**: [Rate-Importer Repository]
- **Browser Extension**: [Rate-Extension Repository]

## 🤝 Contributing

1. **Documentation Updates**: Submit PRs to this repository
2. **Module Changes**: Update relevant module docs
3. **Architecture Changes**: Update platform docs
4. **New Features**: Add documentation before implementation

### Documentation Guidelines
- Use clear, concise language
- Include code examples where relevant
- Update cross-references when moving content
- Follow the established structure

## 📞 Support

For questions or issues:
1. Check the [Troubleshooting Guide](./Guides/troubleshooting.md)
2. Search existing documentation
3. Create an issue in the relevant repository
4. Contact the development team

---

**Last Updated**: 2025-06-06  
**Version**: 1.0  
**Architecture**: MVP-First Modular Platform
