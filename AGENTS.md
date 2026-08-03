# Repository Guidelines

## Phạm vi

Repository này chỉ chứa thin plugin Document First. Giữ toàn bộ search, authorization, dữ liệu,
business rule và prompt nội bộ trên remote MCP/backend. Mọi file trong package phía client phải
được xem là người cài có thể đọc.

## Cấu trúc và validation

- Plugin: `plugins/document-first/`.
- Marketplace local/team: `.agents/plugins/marketplace.json`.
- Kế hoạch và runbook: `documentation/`.
- Khi sửa plugin, chạy plugin validator và validator cho từng bundled skill.
- Khi đổi manifest hoặc skill trong lúc phát triển local, cập nhật cachebuster bằng helper của
  `plugin-creator`, reinstall từ marketplace `document-first-local`, rồi thử trong thread mới.
- Dùng `npm pack --dry-run --json` để xác nhận package chỉ chứa file được phép phân phối.

## Bảo mật và phát hành

- Không commit secret, token OAuth/npm, demo password hoặc cấu hình production nhạy cảm.
- Không thêm proprietary logic vào `SKILL.md`; skill chỉ mô tả workflow công khai.
- Không publish registry, push remote hoặc submit public plugin nếu người dùng chưa yêu cầu rõ.
- Giữ `package.json` ở trạng thái `private: true` cho đến khi registry/version/quyền publish đã
  được duyệt.
