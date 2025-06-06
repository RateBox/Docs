# Data Validation Pipeline for Rate-Extension

## 🏗️ Layered Validation Architecture

Áp dụng 5 lớp kiểm tra dữ liệu để tối ưu chất lượng, hiệu năng và chi phí:

```
Raw Data → Pre-Processing → Basic Validation → Business Rules → AI/ML Verification → Final Enrichment
```

---

## 1️⃣ Pre-Processing (Lọc sơ bộ)
- Làm sạch dữ liệu: loại bỏ HTML, chuẩn hóa whitespace
- Kiểm tra trường bắt buộc (phone/account/content)
- Loại bỏ bản ghi noise quá ngắn hoặc không có thông tin
### 3.1. Các trường dữ liệu extract từ trang scam
- **id**: Định danh duy nhất (timestamp + index)
- **title**: Tiêu đề bài viết hoặc dòng đầu tiên nội dung
- **owner**: Chủ sở hữu (ưu tiên phone/account đầu tiên)
- **phone**: Số điện thoại (E.164, đã chuẩn hóa)
- **account**: Số tài khoản ngân hàng (đã chuẩn hóa)
- **bank**: Tên ngân hàng
- **amount**: Số tiền phát hiện
- **content**: Nội dung bài viết (tối đa 500 ký tự)
- **url**: URL trang hiện tại
- **extractedAt**: Thời điểm extract
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
