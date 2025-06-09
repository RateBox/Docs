# Rate Validator Module Documentation

## Overview

`Rate-Validator` là module chịu trách nhiệm validate, enrich, deduplicate và chuẩn hóa dữ liệu scammer trước khi ghi vào hệ thống Rate Platform (Strapi CMS, PostgreSQL).

- **Core logic**: Được đóng gói thành package `@ratebox/core-validator` (TypeScript, dùng chung cho Importer/Validator/Extension).
- **Schema chuẩn**: Định nghĩa bằng Zod (`scamReportSchema`).
- **Tích hợp**: Import trực tiếp vào Importer, Validator, Extension.

---

## 1. Cấu trúc thư mục

```
D:/Projects/JOY/Rate/Modules/Packages/Validator
├── package.json
├── tsconfig.json
├── README.md
├── src/
│   ├── index.ts
│   ├── index.test.ts
│   └── scamReportSchema.ts
```

---

## 2. Sử dụng core-validator trong các module khác

```ts
import { validateScamReport } from '@ratebox/core-validator';

const result = validateScamReport(data);
if (!result.valid) {
  console.error('Invalid:', result.error);
} else {
  console.log('Valid data:', result.data);
}
```

---

## 3. Schema chuẩn cho dữ liệu scammer

```ts
import { scamReportSchema } from '@ratebox/core-validator';

const result = scamReportSchema.safeParse(data);
if (!result.success) {
  // Xử lý lỗi validate
  console.log(result.error.issues);
} else {
  // Dữ liệu hợp lệ
  console.log(result.data);
}
```

### Các trường bắt buộc:
- `phone`: string, 8-15 ký tự
- `name`: string, >=2 ký tự
- `bank`: string, >=2 ký tự
- `amount`: number, >= 1,000
- `content`: string, >=5 ký tự

### Các trường mở rộng (optional):
- `accountNumber`, `category`, `region`, `trustScore`, `createdAt`, `updatedAt`

---

## 4. Quy trình validate & enrich
1. **Validate schema** (Zod)
2. **Enrich**: Bổ sung trường thiếu, chuẩn hóa dữ liệu
3. **Deduplicate**: Phát hiện trùng lặp (theo phone/account)
4. **Scoring**: Tính trustScore, phân loại

---

## 5. Test & mở rộng
- Viết test case trong `index.test.ts`
- Mở rộng schema hoặc enrichment logic trong `scamReportSchema.ts` và `index.ts`

---

## 6. Liên kết tài liệu
- [Platform/Overview.md](../Platform/Overview.md)
- [Platform/Integration-Guide.md](../Platform/Integration-Guide.md)
- [Modules/Rate-Importer](./Rate-Importer)

---

**Mọi thay đổi về schema nên cập nhật lại tài liệu này và README của core-validator!**
