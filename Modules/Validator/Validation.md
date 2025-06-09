# MVP Validator Service - Kiến trúc & Kế hoạch triển khai (06/2025)

## 🎯 Mục tiêu
- Ra mắt nhanh, ưu tiên throughput & p95 latency cho admin UI.
- Production-ready: dependency lock, healthcheck, monitoring.
- Dễ mở rộng: khi scale vượt ngưỡng sẽ chuyển sang CDC/Kafka/Splink.

---

## 🏗️ Kiến trúc MVP
- **Strapi 5.15 (Node 22)**: afterCreate hook → XADD vào Redis Stream.
- **Redis 7.2 Stream**: batch lưu raw_items, worker đọc batch 500.
- **Python Worker 3.12**: validate đơn giản (unique index, trường thiếu, trạng thái).
- **PostgreSQL 17.4**: bảng raw_items (uuid_v7 PK), raw_item_errors (partition theo thời gian).

### Flow tổng quát
1. Strapi afterCreate hook → XADD vào Redis stream `raw_items` (maxlen ~1M).
2. Python worker đọc batch 500 từ Redis, validate, ghi verdict/error_code vào Postgres.
3. Postgres lưu bảng raw_items (uuid_v7 PK) & raw_item_errors (partition theo occurred_at).

### Validation logic MVP
- Check trùng phone/bank_account bằng unique index tạm thời.
- Thiếu trường bắt buộc: error_code=100.
- validation_state = 'DONE'.

### Gate chuyển phase
- Khi queue backlog > 5 phút hoặc tổng bản ghi > 3 triệu, chuyển sang CDC/Kafka/Splink/partition.

### Monitoring
- Prometheus, Grafana, alert p95, queue length, memory.

---

## 📋 Kế hoạch triển khai 2 tuần

| Ngày/tuần       | Việc cần làm                                                                                                                     | Lệnh/Ghi chú then chốt                                                                               |
| --------------- | -------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------- |
| **Ngày 1**      | **Chuyển sang yarn** & lock                                                                                                      | `corepack enable && yarn import && yarn install --frozen-lockfile --prefer-offline --ignore-scripts` |
| **Ngày 1**      | **Nâng Strapi → 5.15**                                                                                                           | `yarn up @strapi/*@latest` → `yarn build`                                                     |
| **Ngày 2**      | **Tạo UUID v7**                                                                                                                  | `CREATE EXTENSION pg_uuidv7; ALTER TABLE raw_items ALTER id SET DEFAULT uuid_generate_v7();`         |
| **Ngày 2**      | **Tách bảng lỗi**                                                                                                                | Tạo `raw_item_errors` + index `(item_id, occurred_at DESC)`                                          |
| **Ngày 3**      | **Viết worker.py** (aioredis + psycopg3)                                                                                         | Bắt gói batch 500 → validate → UPSERT verdict/error_code                                            |
| **Ngày 3**      | **Dockerfile** Python 3.12-slim + HEALTHCHECK shell (`redis-cli ping && psql -c "select 1"`)                                     |                                                                                                      |
| **Ngày 4–5**    | **Sinh 1 M bản ghi giả (10 KB/row, 5 % trùng)** → Benchmark<br>• p95 Admin < 800 ms<br>• Worker ≥ 1 000 row/s                    | Dùng SQL `generate_series()`, run k6                                                                 |
| **Tuần 2**      | **Monitoring cơ bản**<br>• @strapi/metrics<br>• postgres_exporter, redis_exporter<br>• alert p95 Admin, queue length, mem>85 % | Prometheus 2.51, Grafana 11                                                                          |
| **Cuối Tuần 2** | **Go/No-Go** dựa trên gate: queue backlog > 5 phút? tổng bản ghi > 3 M?                                                          | Nếu vượt ngưỡng ⇒ lập kế hoạch Phase 1 (Debezium + Kafka, partition).                                |

---

## 🚦 Gate chuyển **Phase 1**

| Tín hiệu                                               | Hành động tiếp theo                                                                      |
| ------------------------------------------------------ | ---------------------------------------------------------------------------------------- |
| Backlog Redis > 5 phút **hoặc** tổng bản ghi > 3 triệu | Triển khai Debezium 3.1 + Kafka 4.0, pg_partman, (tùy) Hasura.                          |
| Số lượng validate cần logic ML                         | Khi đó mới tích hợp Splink (hoặc thư viện ML khác) trong worker hoặc tách micro-service. |

---

## ✅ Tóm tắt nhanh
- Stack: Strapi 5.15 (Node 22) → Redis 7.2 stream → Python worker 3.12 → PostgreSQL 17.4 (uuid_v7, bảng raw_item_errors).
- 2 tuần:
  1) yarn lock, nâng Strapi, bật pg_uuidv7.
  2) Viết worker.py (aioredis + psycopg3), Dockerfile với HEALTHCHECK.
  3) Sinh 1M dữ liệu (10 KB/row, 5 % duplicate), benchmark p95 Admin < 0.8 s, worker ≥ 1 k row/s.
  4) Triển khai Prometheus + basic alert (p95, queue len, memory).
- Gate: backlog > 5 phút **hoặc** > 3 triệu record ⇒ Phase 1 (Debezium/Kafka/partition & có thể thêm Splink).
- **source**: Nguồn (passive-extension...)
- **score**: Điểm chất lượng sau validate
- **flags**: Danh sách cờ lỗi/ghi chú
- **Nickname**: Nickname scammer (nếu có, optional, enrich từ source)
- **SourceUpvoteCount**: Số tán thành từ nguồn (optional)
- **SourceDownvoteCount**: Số không tán thành từ nguồn (optional)
- **SourceReporterReputation**: Điểm uy tín người tố cáo từ nguồn (optional)
- **SourceReporterReportCount**: Lượt tố cáo của người này từ nguồn (optional)
- **SourceReporterJoinedAt**: Ngày tham gia hệ thống từ nguồn (optional)
- **ReporterPhone**: Số điện thoại người báo cáo (thay cho ReporterZalo)
- (có thể mở rộng thêm)

**Ví dụ:**
```js
function preProcess(record) {
  // Clean HTML, trim, check required fields
  ...
}
```

---

## 2️⃣ Basic Validation (Rule-based)
- Kiểm tra định dạng số điện thoại, tài khoản
- Gắn cờ lỗi nếu không hợp lệ
- Đánh điểm cơ bản (score)
### 2.2. Chuẩn hóa số điện thoại, tài khoản ngân hàng
- Số điện thoại được chuẩn hóa về **dạng quốc tế E.164** (ví dụ: +84987654321)
- Chỉ lưu duy nhất 1 trường `phone` (E.164), không tách country/national trong schema mặc định
- Lý do: Theo best practice quốc tế (Google, Twilio, Facebook...), dễ mở rộng đa quốc gia, tránh lỗi mapping về sau, phù hợp mọi hệ thống xác thực/SMS/API
- Nếu cần tách country/national có thể mapping động ở backend hoặc khi xuất dữ liệu
- Tài khoản ngân hàng vẫn chuẩn hóa lấy số đầu tiên, loại ký tự lạ

**Ví dụ:**
```js
function applyBasicValidation(record) {
  // Regex kiểm tra phone/account, flag lỗi
  ...
}
```

---

## 3️⃣ Business Rules (Domain Logic)
- Áp dụng rule nghiệp vụ: số tiền nghi ngờ, từ khóa scam, ngân hàng hợp lệ
- Điều chỉnh score, gắn flag đặc biệt

**Ví dụ:**
```js
function applyBusinessRules(record) {
  // Trừ/cộng điểm theo rule nghiệp vụ
  ...
}
```

---

## 4️⃣ AI/ML Verification (Chỉ khi cần)
- Gọi LLM/AI khi record mơ hồ, score trung bình, hoặc nội dung phức tạp
- Trả về đánh giá AI: confidence, category, severity

**Ví dụ:**
```js
async function verifyWithAI(record) {
  // Chỉ gọi AI khi thực sự cần
  ...
}
```

---

## 5️⃣ Final Enrichment & Export
- Thêm metadata: processedAt, version, status cuối cùng
- Chuẩn hóa output cho export/API

**Ví dụ:**
```js
function enrichAndExport(record) {
  // Thêm metadata, xác định trạng thái cuối
  ...
}
```

---

## 🚦 Sample Pipeline Flow
```js
async function processRecord(rawRecord) {
  const pipeline = [
    preProcess,
    applyBasicValidation,
    applyBusinessRules,
    verifyWithAI,      // Bật/tắt tùy nhu cầu
    enrichAndExport
  ];
  return pipeline.reduce(
    async (recordPromise, step) => step(await recordPromise),
    Promise.resolve(rawRecord)
  );
}
```

---

## 🎯 Best Practices
- Ưu tiên rule-based, chỉ dùng AI khi thực sự cần
- Tách biệt code từng layer để dễ debug, mở rộng
- Log lại flag và score từng bước cho traceability
- Có thể batch xử lý để tối ưu hiệu năng

---

**Last Updated:** 2025-06-06
**Author:** RateBox Team
