# Rate Platform - MVP Architecture

## 🎯 MVP Goals
- **Validate current crawler data** with shared validation logic
- **Simple API** for data access and validation
- **Basic web interface** for viewing/managing data
- **Foundation** for future scaling

## 🏗️ MVP Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                           DATA SOURCES                               │
├─────────────────┬─────────────────┬─────────────────┬──────────────┤
│  Current        │ Browser         │   Manual        │   File       │
│  Crawler        │ Extension       │   Entry         │   Import     │
│  (CheckScam)    │ (Future)        │   (Future)      │   (Future)   │
└────────┬────────┴────────┬────────┴────────┬────────┴──────┬───────┘
         │                 │                 │                │
         ▼                 ▼                 ▼                ▼
┌─────────────────────────────────────────────────────────────────────┐
│                      VALIDATION CORE                                 │
├─────────────────────────────────────────────────────────────────────┤
│  Schema Validation → Business Rules → Data Enrichment → Dedup Check │
└─────────────────────────────────────────────────────────────────────┘
                                   │
                                   ▼
┌─────────────────────────────────────────────────────────────────────┐
│                         STORAGE                                       │
├──────────────────┬──────────────────┬───────────────────────────────┤
│   JSON Files     │   SQLite DB      │      Local Cache              │
│   (Results)      │   (Structured)   │    (In-Memory)                │
└──────────────────┴──────────────────┴───────────────────────────────┘
                                   │
                                   ▼
┌─────────────────────────────────────────────────────────────────────┐
│                          API LAYER                                   │
├──────────────────┬──────────────────┬───────────────────────────────┤
│   Express API    │  File API        │      Health Check             │
│  (CRUD/Validate) │  (Export)        │    (Status/Metrics)           │
└──────────────────┴──────────────────┴───────────────────────────────┘
                                   │
                                   ▼
┌─────────────────────────────────────────────────────────────────────┐
│                         WEB UI                                        │
├──────────────────┬──────────────────┬───────────────────────────────┤
│   Next.js App    │  Data Viewer     │      Admin Panel              │
│  (Dashboard)     │  (Reports)       │    (Validation Queue)         │
└──────────────────┴──────────────────┴───────────────────────────────┘
```

## 📁 MVP Project Structure

```
rate-platform-mvp/
├── packages/
│   └── core-validator/          # Shared validation logic
│       ├── src/
│       │   ├── validators/
│       │   │   ├── schema.js
│       │   │   ├── business.js
│       │   │   └── enrichment.js
│       │   ├── utils/
│       │   │   ├── phone.js
│       │   │   ├── bank.js
│       │   │   └── duplicate.js
│       │   └── index.js
│       ├── tests/
│       └── package.json
│
├── apps/
│   ├── api/                     # Express API server
│   │   ├── src/
│   │   │   ├── routes/
│   │   │   │   ├── reports.js
│   │   │   │   ├── validate.js
│   │   │   │   └── health.js
│   │   │   ├── middleware/
│   │   │   │   ├── cors.js
│   │   │   │   └── error.js
│   │   │   ├── db/
│   │   │   │   ├── sqlite.js
│   │   │   │   └── migrations/
│   │   │   └── server.js
│   │   └── package.json
│   │
│   ├── web/                     # Next.js frontend
│   │   ├── pages/
│   │   │   ├── index.js         # Dashboard
│   │   │   ├── reports/
│   │   │   │   ├── index.js     # Reports list
│   │   │   │   └── [id].js      # Report detail
│   │   │   └── api/
│   │   │       └── proxy.js     # API proxy
│   │   ├── components/
│   │   │   ├── Dashboard.js
│   │   │   ├── ReportTable.js
│   │   │   └── ValidationStatus.js
│   │   └── package.json
│   │
│   └── crawler/                 # Enhanced current crawler
│       ├── src/
│       │   ├── crawlers/
│       │   │   └── checkscam.js
│       │   ├── adapters/
│       │   │   └── checkscam-adapter.js
│       │   └── cli.js
│       └── package.json
│
├── docker-compose.yml           # Development setup
├── package.json                 # Monorepo root
└── turbo.json                   # Build orchestration
```

## 🔧 Core Components

### 1. Core Validator (Shared Package)
```javascript
// packages/core-validator/src/index.js
export class CoreValidator {
  constructor(config = {}) {
    this.config = {
      strictMode: false,
      enableEnrichment: true,
      duplicateThreshold: 0.8,
      ...config
    };
  }

  async validate(data) {
    const result = {
      isValid: true,
      errors: [],
      warnings: [],
      enriched: { ...data },
      metadata: {
        validated_at: new Date(),
        validator_version: '1.0.0'
      }
    };

    // Step 1: Schema validation
    const schemaResult = this.validateSchema(data);
    if (!schemaResult.isValid) {
      result.isValid = false;
      result.errors.push(...schemaResult.errors);
    }

    // Step 2: Business rules
    const businessResult = this.validateBusiness(data);
    if (!businessResult.isValid) {
      result.warnings.push(...businessResult.warnings);
    }

    // Step 3: Data enrichment
    if (this.config.enableEnrichment) {
      result.enriched = await this.enrichData(result.enriched);
    }

    return result;
  }
}
```

### 2. Simple API Server
```javascript
// apps/api/src/server.js
import express from 'express';
import { CoreValidator } from '@rate-platform/core-validator';
import { SQLiteDB } from './db/sqlite.js';

const app = express();
const validator = new CoreValidator();
const db = new SQLiteDB();

// Validate single report
app.post('/api/validate', async (req, res) => {
  try {
    const result = await validator.validate(req.body);
    
    if (result.isValid) {
      // Save to database
      const saved = await db.saveReport(result.enriched);
      res.json({ ...result, id: saved.id });
    } else {
      res.status(400).json(result);
    }
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

// Get reports
app.get('/api/reports', async (req, res) => {
  const reports = await db.getReports(req.query);
  res.json(reports);
});
```

### 3. Simple Web Dashboard
```javascript
// apps/web/pages/index.js
import { useState, useEffect } from 'react';
import { ReportTable } from '../components/ReportTable';

export default function Dashboard() {
  const [reports, setReports] = useState([]);
  const [stats, setStats] = useState({});

  useEffect(() => {
    fetch('/api/reports')
      .then(res => res.json())
      .then(setReports);
      
    fetch('/api/stats')
      .then(res => res.json())
      .then(setStats);
  }, []);

  return (
    <div className="container mx-auto p-4">
      <h1 className="text-2xl font-bold mb-4">Rate Platform Dashboard</h1>
      
      <div className="grid grid-cols-4 gap-4 mb-6">
        <div className="bg-blue-100 p-4 rounded">
          <h3>Total Reports</h3>
          <p className="text-2xl">{stats.total || 0}</p>
        </div>
        <div className="bg-green-100 p-4 rounded">
          <h3>Valid</h3>
          <p className="text-2xl">{stats.valid || 0}</p>
        </div>
        <div className="bg-yellow-100 p-4 rounded">
          <h3>Warnings</h3>
          <p className="text-2xl">{stats.warnings || 0}</p>
        </div>
        <div className="bg-red-100 p-4 rounded">
          <h3>Errors</h3>
          <p className="text-2xl">{stats.errors || 0}</p>
        </div>
      </div>

      <ReportTable reports={reports} />
    </div>
  );
}
```

## 🚀 Implementation Roadmap

### **Milestone 1: Foundation (Week 1-2)**
- ✅ Setup monorepo với Turborepo
- ✅ Create core-validator package
- ✅ Migrate current crawler validation logic
- ✅ Basic SQLite database setup
- ✅ Simple Express API

**Deliverable**: Working validation API

### **Milestone 2: Data Integration (Week 3-4)**
- ✅ Enhance current crawler với core-validator
- ✅ Create CheckScam adapter
- ✅ Database migrations và seeding
- ✅ API endpoints cho CRUD operations
- ✅ Basic error handling

**Deliverable**: Crawler saving validated data to API

### **Milestone 3: Web Interface (Week 5-6)**
- ✅ Next.js setup với Tailwind CSS
- ✅ Dashboard với basic stats
- ✅ Reports listing và detail pages
- ✅ Validation status indicators
- ✅ Export functionality

**Deliverable**: Working web dashboard

### **Milestone 4: Production Ready (Week 7-8)**
- ✅ Docker setup cho development
- ✅ Environment configuration
- ✅ Basic monitoring và health checks
- ✅ Documentation và deployment guide
- ✅ Testing setup

**Deliverable**: Production-ready MVP

## 📊 Technology Stack

### **Backend**
- **Validation**: Custom JavaScript/TypeScript
- **API**: Express.js với latest packages
- **Database**: SQLite (development) → PostgreSQL (production)
- **Cache**: In-memory (development) → Redis (production)

### **Frontend**
- **Framework**: Next.js 14+ (App Router)
- **Styling**: Tailwind CSS latest
- **Components**: Headless UI
- **Charts**: Chart.js hoặc Recharts

### **DevOps**
- **Monorepo**: Turborepo latest
- **Containerization**: Docker & Docker Compose
- **Process Manager**: PM2
- **Monitoring**: Simple logging → Sentry (later)

## 🎯 Success Metrics

### **MVP Success Criteria**
- ✅ Current crawler data validated và stored
- ✅ Web interface showing reports
- ✅ API responding < 200ms
- ✅ 95%+ validation accuracy
- ✅ Easy to deploy và maintain

### **Future Growth Path**
- **Scale**: Add Redis, PostgreSQL, queue processing
- **Sources**: Browser extension, manual entry, file import
- **Features**: Real-time validation, advanced analytics
- **Infrastructure**: Kubernetes, monitoring, CI/CD

## 💡 Key Benefits

1. **Quick to Build**: 6-8 weeks to MVP
2. **Low Complexity**: Simple tech stack, easy debugging
3. **Scalable Foundation**: Can grow to full architecture
4. **Immediate Value**: Validates current data better
5. **Learning Platform**: Test concepts before scaling

---

**This MVP gives us a solid foundation while keeping complexity manageable!** 🚀
