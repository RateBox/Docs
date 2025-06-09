# CheckScam.vn Crawler Pipeline Documentation

## Overview
Pipeline tự động crawl dữ liệu scammer từ checkscam.vn sử dụng FlareSolverr để bypass anti-bot/Cloudflare.

## Architecture
```
CheckScam.vn → FlareSolverr → Extract Links → Crawl Each Profile → Save JSON/Redis
```

## Components

### 1. crawl-with-flaresolverr.js
**Mục đích:** Lấy danh sách link scammer từ trang chủ checkscam.vn

**Cách chạy:**
```bash
node Scripts/crawl-with-flaresolverr.js
```

**Output:** 
- File JSON chứa danh sách links: `Results/scam_links_YYYY-MM-DDTHH-mm-ss-SSSZ.json`
- Format:
```json
{
  "timestamp": "2025-06-08T17:46:31.841Z",
  "source": "https://checkscam.vn/",
  "total_links": 16,
  "links": [
    {
      "name": "nguyen-minh-hieu-28",
      "url": "https://checkscam.vn/nguyen-minh-hieu-28/"
    }
  ]
}
```

**Giới hạn:**
- Chỉ lấy được links render sẵn trong HTML (không AJAX)
- Thường lấy được 15-20 links đầu tiên

### 2. CheckScamCrawlerV2.js
**Mục đích:** Crawl chi tiết từng profile scammer từ danh sách links

**Cách chạy:**
```bash
# Crawl với giới hạn số lượng
node Scripts/CheckScamCrawlerV2.js --file Results/scam_links_latest.json --max 10 --no-redis

# Crawl toàn bộ
node Scripts/CheckScamCrawlerV2.js --file Results/scam_links_latest.json --no-redis

# Crawl và gửi lên Redis
node Scripts/CheckScamCrawlerV2.js --file Results/scam_links_latest.json
```

**Parameters:**
- `--file`: Path đến file JSON chứa danh sách links
- `--max`: Số lượng links tối đa cần crawl (mặc định: không giới hạn)
- `--delay`: Delay giữa mỗi request (mặc định: 5000ms)
- `--no-redis`: Không gửi lên Redis stream
- `--redis-host`: Redis host (mặc định: localhost)
- `--redis-port`: Redis port (mặc định: 6379)

**Output:**
- File JSON: `Results/checkscam_crawl_YYYY-MM-DDTHH-mm-ss-SSSZ.json`
- Format:
```json
{
  "timestamp": "2025-06-08T18:00:55.873Z",
  "source": "CheckScamCrawlerV2",
  "total": 5,
  "successful": 5,
  "failed": 0,
  "scammers": [
    {
      "url": "https://checkscam.vn/nguyen-minh-hieu-28/",
      "success": true,
      "data": {
        "name": "NGUYEN MINH H",
        "phone": "0462800001",
        "bank": "Mb Bank",
        "account": "",
        "scamAmount": "tiền",
        "content": "Nội dung lừa đảo...",
        "images": ["url1", "url2"],
        "timestamp": "2025-06-08T18:00:40.123Z"
      },
      "quality": "high",
      "issues": []
    }
  ],
  "summary": {
    "high": 3,
    "medium": 2,
    "low": 0
  }
}
```

## Full Pipeline Script

### Automatic Pipeline Runner
```bash
# Tạo script chạy pipeline tự động
cat > Scripts/run-checkscam-pipeline.js << 'EOF'
import { execSync } from 'child_process';
import fs from 'fs';
import path from 'path';

console.log('🚀 Starting CheckScam Full Pipeline...\n');

// Step 1: Extract links
console.log('📋 Step 1: Extracting scammer links...');
execSync('node Scripts/crawl-with-flaresolverr.js', { stdio: 'inherit' });

// Find latest links file
const resultsDir = path.join(process.cwd(), 'Results');
const files = fs.readdirSync(resultsDir)
  .filter(f => f.startsWith('scam_links_') && f.endsWith('.json'))
  .sort()
  .reverse();

if (files.length === 0) {
  console.error('❌ No links file found!');
  process.exit(1);
}

const latestFile = path.join(resultsDir, files[0]);
console.log(`\n✅ Found links file: ${files[0]}`);

// Step 2: Crawl all profiles
console.log('\n📍 Step 2: Crawling all profiles...');
const crawlCmd = `node Scripts/CheckScamCrawlerV2.js --file "${latestFile}" --delay 3000 --no-redis`;
console.log(`Running: ${crawlCmd}\n`);

execSync(crawlCmd, { stdio: 'inherit' });

console.log('\n✅ Pipeline completed!');
EOF
```

## Important Notes

### Performance
- Mỗi profile mất ~3-10 giây để crawl (tùy FlareSolverr response time)
- Delay 3-5 giây giữa requests để tránh bị block
- Crawl 100 profiles có thể mất 10-20 phút

### Error Handling
- Tự động retry 3 lần nếu FlareSolverr fail
- Skip profile nếu fail sau 3 lần retry
- Log chi tiết errors trong console và file output

### Command Execution Best Practices
⚠️ **QUAN TRỌNG:** Với các lệnh chạy lâu (crawl nhiều URLs):
1. Sử dụng **non-blocking mode** (`Blocking: false`)
2. Check status thường xuyên bằng `command_status` tool
3. Expect delays: 5-10 giây/link
4. Total time: số link × (crawl time + delay)

### Redis Integration
Khi bật Redis (`--no-redis` bị bỏ):
- Stream name: `validation_requests`
- Message format: Chuẩn theo Rate-Validator
- Automatic retry on connection failure

## Troubleshooting

### FlareSolverr Errors
- Đảm bảo FlareSolverr đang chạy: `http://localhost:8191`
- Check Docker: `docker ps | grep flaresolverr`
- Restart nếu cần: `docker restart flaresolverr`

### Timeout Issues
- Tăng `--delay` nếu bị rate limit
- Giảm `--max` để test với số lượng nhỏ trước

### Memory/Performance
- Với 1000+ links, chia nhỏ thành batches
- Monitor RAM usage của FlareSolverr container
