# Data Validation Pipeline for Rate-Importer

## 🏗️ Layered Validation Architecture

Áp dụng 5 lớp kiểm tra dữ liệu để tối ưu chất lượng, hiệu năng và chi phí cho crawler:

```
Raw Data → Pre-Processing → Basic Validation → Business Rules → AI/ML Verification → Final Enrichment
```

---

## 1️⃣ Pre-Processing (Lọc sơ bộ)
- Làm sạch dữ liệu: loại bỏ HTML, chuẩn hóa whitespace, decode URL
- Kiểm tra trường bắt buộc (link, domain, nội dung)
- Loại bỏ bản ghi noise, trùng lặp, hoặc thiếu thông tin

**Ví dụ:**
```js
function preProcess(record) {
  // Clean HTML, trim, decode, check required fields
  // Remove duplicates
  ...
}
```

---

## 2️⃣ Basic Validation (Rule-based)
- **Checklist các bước kiểm tra cơ bản:**
  - [ ] Link phải hợp lệ (bắt đầu bằng http/https, không chứa ký tự lạ)
  - [ ] Domain phải thuộc whitelist (nếu có) và không thuộc blacklist
  - [ ] Nội dung không được rỗng, không chứa script/html nguy hiểm
  - [ ] Độ dài link và domain trong giới hạn cho phép
  - [ ] Không chứa ký tự unicode bất thường (emoji, ký tự control...)
  - [ ] Gắn flag lỗi rõ ràng nếu không hợp lệ (ví dụ: `invalid_link`, `blacklisted_domain`, `empty_content`)
  - [ ] Đánh điểm cơ bản (score) cho record: hợp lệ = 100, nghi ngờ = 50, lỗi = 0 (thang điểm 0-100)

**Schema record:**
- **name:** Tên nghi phạm/scammer (nickname, tên đăng bài, tên giả)
- **bank_account_name:** Tên chủ tài khoản ngân hàng (nếu extract được, thường là tên thật)
- **phone:** Số điện thoại (theo định dạng Việt Nam: 0/ +84 và 8-10 số)
- **bank:** Thông tin ngân hàng (tên ngân hàng + số tài khoản, nhận diện qua danh sách ngân hàng phổ biến)
- **scam_amount:** Số tiền liên quan đến vụ scam (dạng số, có thể kèm đơn vị VNĐ, triệu, nghìn, v.v.)
- **content:** Nội dung chính của bài viết (lọc từ các thẻ HTML như entry-content, post-content, article)
- **evidence_images:** Danh sách URL hình ảnh bằng chứng (ảnh giao dịch, chat, chuyển khoản, ...)
- **url:** Đường dẫn gốc của trang bị crawl
- **timestamp:** Thời điểm crawl dữ liệu
- **success:** Trạng thái crawl thành công (true/false)
- **error:** (chỉ có khi lỗi) – thông báo lỗi nếu parse thất bại

**Ví dụ:**
```js
function applyBasicValidation(record, whitelistDomains, blacklistDomains) {
  // Kiểm tra link hợp lệ
  const urlRegex = /^(https?:\/\/)[\w.-]+(\.[\w\.-]+)+[\w\-\._~:/?#[\]@!$&'()*+,;=.]+$/;
  if (!record.link || !urlRegex.test(record.link)) {
    record.flag = 'invalid_link';
    record.score = 0;
    return record;
  }

  // Kiểm tra domain
  const domain = (new URL(record.link)).hostname.replace(/^www\./, '');
  if (blacklistDomains.includes(domain)) {
    record.flag = 'blacklisted_domain';
    record.score = 0;
    return record;
  }
  if (whitelistDomains.length && !whitelistDomains.includes(domain)) {
    record.flag = 'not_in_whitelist';
    record.score = 0.5;
  }

  // Kiểm tra nội dung
  if (!record.content || record.content.trim().length < 10) {
    record.flag = 'empty_content';
    record.score = 0.5;
    return record;
  }
  // Không chứa script/html nguy hiểm
  if (/<script|<iframe|onerror=|onload=/.test(record.content)) {
    record.flag = 'unsafe_content';
    record.score = 0;
    return record;
  }

  // Nếu qua hết các bước trên
  record.flag = 'valid';
  record.score = 1;
  return record;
}
```
- **Tip:**
  - Có thể mở rộng thêm các rule khác tùy theo đặc thù nguồn crawl.
  - Nên log lại các record bị flag để dễ debug và audit.

---

## 3️⃣ Business Rules (Domain Logic)
- Áp dụng rule nghiệp vụ: phát hiện scam keyword, kiểm tra số tiền nghi ngờ, mapping với blacklist/whitelist
- Điều chỉnh score, gắn flag đặc biệt (scam, nghi ngờ, hợp lệ)

**Ví dụ:**
```js
function applyBusinessRules(record) {
  // Trừ/cộng điểm theo rule nghiệp vụ, flag scam
  ...
}
```

---

## 4️⃣ AI/ML Verification (Chỉ khi cần)
- Gọi LLM/AI khi record mơ hồ, score trung bình, hoặc nội dung phức tạp
- Trả về đánh giá AI: confidence, category, severity

**Ví dụ:**
```js
async function aiVerification(record) {
  // Gọi API AI/ML phân loại scam
  ...
}
```

---

## 5️⃣ Final Enrichment (Bổ sung dữ liệu)
- Chuẩn hóa output cuối cùng, bổ sung metadata (timestamp, source, crawlId)
- Mapping dữ liệu sang schema chuẩn (chuẩn bị push lên Strapi/API)

**Ví dụ:**
```js
function enrichRecord(record) {
  // Add metadata, chuẩn hóa schema
  ...
}
```

---

## 🔗 Liên kết & Tham khảo
- [Validation pipeline của Rate-Extension](../../Rate-Extension/Validation.md)
- [MVP Architecture](../../Platform/MVP-Architecture.md)
- [Roadmap Rate-Importer](./Roadmap.md)

---

## 📌 Ghi chú
- Mỗi lớp validation nên tách thành function riêng, dễ test/unit test.
- Có thể mở rộng thêm rule hoặc tích hợp AI khi cần.
- Tất cả lỗi/flag nên log lại để debug và audit.

---

> **Cập nhật:** File này sẽ được update liên tục khi pipeline validation thay đổi hoặc bổ sung tính năng mới.
