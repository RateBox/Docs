# Rate-Importer Architecture

## System Overview

Rate-Importer is a Node.js-based web scraping system designed to extract and process scammer data from CheckScam.vn and other sources. The system uses FlareSolverr for Cloudflare bypass and provides a unified CLI interface for data extraction and management.

## Architecture Diagram

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   CheckScam.vn  │    │  Other Sources  │    │  Future APIs    │
│   (Cloudflare)  │    │   (Direct)      │    │   (GraphQL)     │
└─────────┬───────┘    └─────────┬───────┘    └─────────┬───────┘
          │                      │                      │
          └──────────────────────┼──────────────────────┘
                                 │
                    ┌─────────────▼───────────────┐
                    │      FlareSolverr Proxy     │
                    │    (Cloudflare Bypass)      │
                    └─────────────┬───────────────┘
                                  │
                    ┌─────────────▼───────────────┐
                    │      Rate-Importer Core     │
                    │                             │
                    │  ┌─────────────────────────┐│
                    │  │     CLI Interface       ││
                    │  │   (crawl.js)           ││
                    │  └─────────────────────────┘│
                    │                             │
                    │  ┌─────────────────────────┐│
                    │  │   Link Extractor        ││
                    │  │ (crawl-with-flaresolverr)││
                    │  └─────────────────────────┘│
                    │                             │
                    │  ┌─────────────────────────┐│
                    │  │   Data Crawler          ││
                    │  │ (CheckScamCrawlerV2)    ││
                    │  └─────────────────────────┘│
                    │                             │
                    │  ┌─────────────────────────┐│
                    │  │   Data Processor        ││
                    │  │  (Validation & Clean)   ││
                    │  └─────────────────────────┘│
                    └─────────────┬───────────────┘
                                  │
                    ┌─────────────▼───────────────┐
                    │       Data Storage          │
                    │                             │
                    │  ┌─────────────────────────┐│
                    │  │     JSON Files          ││
                    │  │   (Results/)            ││
                    │  └─────────────────────────┘│
                    │                             │
                    │  ┌─────────────────────────┐│
                    │  │   Future: Database      ││
                    │  │  (PostgreSQL/Redis)     ││
                    │  └─────────────────────────┘│
                    └─────────────────────────────┘
```

## Core Components

### 1. CLI Interface (`crawl.js`)
**Purpose**: Unified command-line interface for all crawler operations

**Features**:
- Command routing (links, crawl, cleanup)
- Argument parsing and validation
- Progress reporting and logging
- Error handling and recovery

**Usage**:
```bash
node crawl.js links          # Extract scammer links
node crawl.js crawl --max 10 # Crawl scammer data
node crawl.js cleanup        # Clean obsolete files
```

### 2. Link Extractor (`Scripts/crawl-with-flaresolverr.js`)
**Purpose**: Extract scammer profile links from CheckScam.vn

**Process**:
1. Fetch HTML via FlareSolverr proxy
2. Parse HTML for scammer links
3. Filter and validate URLs
4. Save links to JSON file

**Output**: `Results/scam_links_YYYY-MM-DDTHH-mm-ss-sssZ.json`

### 3. Data Crawler (`Scripts/CheckScamCrawlerV2.js`)
**Purpose**: Extract detailed scammer information from individual pages

**Features**:
- Phone number extraction
- Bank account parsing
- Scam amount detection
- Content analysis
- Quality scoring

**Process**:
1. Load scammer links from JSON
2. Fetch individual pages via FlareSolverr
3. Parse and extract structured data
4. Validate and score data quality
5. Save results with metadata

### 4. FlareSolverr Proxy
**Purpose**: Bypass Cloudflare protection on target websites

**Configuration**:
- **Port**: 8191
- **Network**: Rate (Docker network)
- **Health Check**: HTTP endpoint monitoring
- **Timeout**: Configurable (default 60s)

**API**:
```json
POST /v1
{
  "cmd": "request.get",
  "url": "https://checkscam.vn/...",
  "maxTimeout": 60000
}
```

## Data Flow

### 1. Link Extraction Flow
```
CheckScam.vn → FlareSolverr → Link Extractor → JSON File
```

1. **Request**: CLI triggers link extraction
2. **Fetch**: FlareSolverr fetches HTML from CheckScam.vn
3. **Parse**: Extract scammer links using regex patterns
4. **Filter**: Remove navigation and system links
5. **Save**: Store links in timestamped JSON file

### 2. Data Crawling Flow
```
JSON Links → FlareSolverr → Data Crawler → Structured Data → JSON File
```

1. **Load**: Read latest scammer links file
2. **Iterate**: Process each link with delay
3. **Fetch**: Get individual page via FlareSolverr
4. **Extract**: Parse structured data (phone, bank, amount, etc.)
5. **Validate**: Check data quality and completeness
6. **Save**: Store results with metadata

### 3. Error Handling Flow
```
Error → Retry Logic → Fallback → Logging → Continue/Abort
```

1. **Detection**: Catch network, parsing, or validation errors
2. **Retry**: Attempt request up to max retries
3. **Fallback**: Use alternative extraction methods
4. **Logging**: Record error details for debugging
5. **Decision**: Continue with next item or abort batch

## Configuration Management

### Environment Variables
```bash
# Core settings
NODE_ENV=development
LOG_LEVEL=info

# FlareSolverr configuration
FLARESOLVERR_URL=http://localhost:8191/v1
FLARESOLVERR_TIMEOUT=60000

# Crawler settings
CHECKSCAM_BASE_URL=https://checkscam.vn
CRAWLER_DELAY_MS=5000
CRAWLER_MAX_RETRIES=3
CRAWLER_TIMEOUT_MS=30000

# Output settings
RESULTS_DIR=Results
LOGS_DIR=Logs
```

### Docker Configuration
```yaml
# docker-compose.yml
services:
  flaresolverr:
    image: flaresolverr/flaresolverr:latest
    environment:
      - LOG_LEVEL=info
      - LOG_HTML=false
    ports:
      - "8191:8191"
    networks:
      - Rate

networks:
  Rate:
    driver: bridge
```

## Data Models

### Scammer Link Schema
```typescript
interface ScammerLink {
  name: string;        // Identifier (phone/account)
  url: string;         // Full URL to scammer page
}
```

### Scammer Data Schema
```typescript
interface ScammerData {
  id: string;          // Unique identifier
  title: string;       // Page title
  phone: string;       // Phone number
  bank: string;        // Bank name
  scam: string;        // Scam type/amount
  content: string;     // Detailed description
  url: string;         // Source URL
  quality: 'high' | 'medium' | 'low';  // Data quality score
  timestamp: string;   // ISO timestamp
}
```

### Batch Result Schema
```typescript
interface CrawlResult {
  metadata: {
    startTime: string;
    endTime: string;
    totalProcessed: number;
    successful: number;
    failed: number;
    qualityDistribution: {
      high: number;
      medium: number;
      low: number;
    };
  };
  data: ScammerData[];
}
```

## Performance Characteristics

### Throughput
- **Link extraction**: 1-5 links/second
- **Data crawling**: 0.1-0.2 pages/second (with delays)
- **Batch processing**: 10-50 records/minute

### Resource Usage
- **Memory**: 100-500MB (depending on batch size)
- **CPU**: Low (I/O bound operations)
- **Network**: 1-10 MB/hour (depending on activity)
- **Storage**: 1-10 MB/day (JSON files)

### Scalability Limits
- **Single instance**: ~1000 records/hour
- **Rate limiting**: Constrained by target website
- **FlareSolverr**: Single proxy instance bottleneck
- **Memory**: Limited by Node.js heap size

## Security Considerations

### Network Security
- FlareSolverr runs in isolated Docker network
- No direct exposure of internal services
- Configurable timeouts prevent hanging connections

### Data Security
- No sensitive credentials stored in code
- Environment-based configuration
- Local file storage (no external transmission)

### Privacy
- Only public scammer data collected
- No personal information beyond reported scams
- Compliance with robots.txt and rate limiting

## Future Architecture

### Planned Enhancements

#### 1. Database Integration
```
JSON Files → PostgreSQL/MongoDB
- Structured storage
- Query capabilities
- Data relationships
- Backup/recovery
```

#### 2. API Layer
```
REST/GraphQL APIs
- External integrations
- Real-time access
- Authentication
- Rate limiting
```

#### 3. Message Queue
```
Redis/RabbitMQ
- Async processing
- Job scheduling
- Retry mechanisms
- Horizontal scaling
```

#### 4. Monitoring
```
Prometheus/Grafana
- Performance metrics
- Health monitoring
- Alerting
- Dashboards
```

## Deployment Patterns

### Development
```
Local Node.js + Docker Compose
- Fast iteration
- Easy debugging
- Local file storage
```

### Staging
```
Docker Containers + Shared Storage
- Production-like environment
- Integration testing
- Performance validation
```

### Production
```
Kubernetes + Persistent Volumes
- High availability
- Auto-scaling
- Load balancing
- Monitoring integration
```
