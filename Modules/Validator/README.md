# MVP Validator Service - Kiến trúc & Kế hoạch triển khai (06/2025)

## 🎉 TRẠNG THÁI: REDIS STREAM INTEGRATION HOÀN THÀNH

**Cập nhật:** 2025-06-08 00:27:00  
**Phiên bản:** 1.0.0  
**Tình trạng:** Production Ready

### ✅ Đã Hoàn Thành Redis Stream Integration

- **Enhanced Redis Stream Worker** - Đang chạy và xử lý real-time
- **Database Schema Migration** - Đã áp dụng thành công
- **Real-time Monitoring Dashboard** - Đang hoạt động
- **Performance:** 217+ messages đã được xử lý, 0 pending messages
- **Consumer Groups:** validator_workers với fault tolerance

---

## Mục tiêu
- Ra mắt nhanh, ưu tiên throughput & p95 latency cho admin UI.
- Production-ready: dependency lock, healthcheck, monitoring.
- Dễ mở rộng: khi scale vượt ngưỡng sẽ chuyển sang CDC/Kafka/Splink.

---

## Kiến trúc MVP
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
## 7. linker_settings.json (ví dụ)
```json
{
  "link_type": "dedupe_only",
  "blocking_rules_to_generate_predictions": [
    "l.phone = r.phone",
    "l.bank_account = r.bank_account",
    "substr(l.title,1,8) = substr(r.title,1,8)"
  ],
  "comparison_columns": [
    { "col_name": "title", "num_levels": 3 },
    { "col_name": "name",  "num_levels": 3, "term_frequency_adjustments": true }
  ]
}
```

## 8. splink_api.py (FastAPI wrapper)
- Nhận POST `/resolve` với body: id, title, phone, bank_account, name
- Gọi Splink, trả về cluster_id nếu match.

**Ví dụ request:**
```bash
curl -X POST http://localhost:8090/resolve \
     -H "Content-Type: application/json" \
     -d '{"id":1,"title":"Shop ABC","phone":"0909123456","bank_account":"0123456789"}'
```
**Ví dụ output:**
```json
{
  "cluster_id": 123
}
```

## 9. dedup_pipeline.py (gọi từ importer)
- `_hash_match`: check trùng md5(url)
- `_trgm_candidates`: lấy các bản ghi gần giống (similarity > 0.8)
- `check_duplicate`: nếu hash/trgm match thì trả về luôn, nếu không thì dùng Splink để scoring nâng cao.

## 10. Tích hợp vào importer
- Gọi hàm `RateDedupPipeline.check_duplicate(record)` để kiểm tra trùng lặp trước khi insert vào DB.
- Nếu cần, có thể gọi REST API `/resolve` từ bất kỳ service nào.

## 11. Triển khai
- Build + run: `docker compose up -d --build`
- Test REST: như ví dụ trên.

## 12. Lộ trình công việc
| Tuần                | Việc cần làm                                                                 |
| ------------------- | ---------------------------------------------------------------------------- |
| **1 – POC**         | Load 1k cặp gán nhãn ⇒ chạy `linker.predict` ⇒ đo precision/recall.         |
| **2 – Integration** | Gọi `RateDedupPipeline.check_duplicate()` trong importer & Chrome extension. |
| **3 – Prod**        | Deploy trên Swarm/K8s, thêm Grafana dashboard (P95 latency, % qua Splink).   |

## 13. Khi nào nâng cấp?
- Khi > 50 triệu bản ghi hoặc cần SLA < 100 ms P95, cân nhắc dùng DuckDB + Splink hoặc Spark cluster.

-----------------------------
## Troubleshooting
- Nếu không tạo được extension, kiểm tra lại image Postgres (bản >= 15, bạn đang dùng **17** là chuẩn).
- Nếu API `/resolve` trả về null, kiểm tra lại linker_settings.json và dữ liệu input.
- Nếu pipeline importer bị chậm, kiểm tra lại các bước hash/trgm trước khi gọi Splink.
-----------------------------
Chỉ cần copy cấu hình, chạy docker compose là có pipeline dedup mạnh, nhẹ, rẻ, tự host, dễ tích hợp cho mọi importer/crawler.
-----------------------------
