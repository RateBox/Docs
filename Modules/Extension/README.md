# Rate-Extension Documentation

## 🧩 Overview

**Rate-Extension** là browser extension thuộc hệ sinh thái RateBox, phục vụ cho việc thu thập, xác thực và xuất dữ liệu scam/phản hồi người dùng từ các nền tảng như checkscam.vn. Extension này hỗ trợ passive crawling, export dữ liệu, popup UI, và tích hợp validation pipeline.

---

## 📦 Module Features
- Passive data crawling trên các trang scam (checkscam.vn, v.v.)
- Popup UI: preview, export, debug extraction
- Tích hợp validation pipeline (adapter + core-validator)
- Export JSON tự động vào thư mục Downloads
- Hỗ trợ AJAX, SPA, popup Discord detection
- Logging/debug trực tiếp trong background/service worker

---

## 📚 Documentation Structure

- `README.md`: Giới thiệu tổng quan module
- `api.md`: Mô tả API giữa các thành phần (content, background, popup)
- `development.md`: Hướng dẫn phát triển, debug, test extension
- `validation.md`: Tích hợp pipeline validation, AI/LLM support
- `changelog.md`: Lịch sử cập nhật

---

## 🚀 Quick Start

1. **Cài đặt extension unpacked:**
   - Vào Chrome Extensions → Load unpacked → trỏ tới thư mục build của extension
2. **Bật debug logs:**
   - Mở background/service worker console để xem log khi crawl/export
3. **Test passive crawling:**
   - Lướt các trang scam, mở popup để xem data preview
4. **Export dữ liệu:**
   - Nhấn Export, kiểm tra file trong Downloads/rate-crawler

---

## 🛠️ Development & Contribution
- Xem `development.md` để biết quy trình build, test, debug
- Đóng góp: PRs, docs update, issue report đều welcome!

---

## 📄 Related Docs
- [RateBox Platform Overview](../../platform/overview.md)
- [Validation Pipeline](../../platform/integration-guide.md)
- [Rate-Importer Crawler](../rate-importer/README.md)

---

**Last Updated:** 2025-06-06
**Maintainer:** RateBox Team
