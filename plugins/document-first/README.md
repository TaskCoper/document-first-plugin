# Document First plugin

Plugin Codex chứa cấu hình kết nối MCP và ba bundled skills mô tả workflow công khai. Mã nguồn
nghiệp vụ, authorization, business rule thực tế và cách tổng hợp context nằm trong backend
Document First, không được đóng gói vào plugin gửi cho khách hàng.

## Bundled skills

- `implement-story`: bắt buộc chuẩn bị và đọc approved-only context trước khi sửa code.
- `review-impact`: phân tích tài liệu và code path có thể bị ảnh hưởng.
- `verify-business-rules`: đối chiếu implementation/test với rule đã phê duyệt.

Người cài plugin có thể đọc toàn bộ `SKILL.md`; vì vậy các file này không chứa proprietary logic.

## Xác thực

Plugin dùng OAuth 2.1 Authorization Code + PKCE. Sau khi MCP production được deploy, đăng nhập
bằng tài khoản Document First khi Codex yêu cầu hoặc chủ động chạy:

```bash
codex mcp login document-first
```

Không cần copy access token hoặc cấu hình secret tại máy khách. Access token được ràng buộc với
MCP resource; refresh token được xoay vòng và có thể thu hồi khi disconnect.

## Tool

- `document_first_list_projects`: liệt kê project người dùng được đọc.
- `document_first_search`: tìm kiếm nhưng chỉ trả nội dung hiện tại đã được phê duyệt.
- `document_first_fetch`: đọc Markdown Approved và có thể ghim `contentHash`.
- `document_first_prepare_story_context`: tự search, tìm related/reference rồi đọc các
  tài liệu Approved trước khi agent implement story.

Không cần Release, GitHub commit hoặc liên kết GitHub repository. Nếu nội dung thay đổi sau khi
duyệt, approval tự reset và tài liệu biến mất khỏi MCP cho đến khi được phê duyệt lại.

Backend production phải triển khai endpoint `https://document-api.vnzdna.com/mcp` trước khi
phát hành plugin. Chạy validator từ skill `plugin-creator` trước mỗi lần đóng gói.

## Phân phối public

Plugin được submit trực tiếp qua OpenAI Plugin Submission Portal dưới loại **With MCP**. Không
publish package này lên npm; `package.json` giữ `private: true` để ngăn publish nhầm và chỉ phục vụ
kiểm tra nội dung bundle bằng `npm pack --dry-run`.

Khi submit, dùng production MCP URL `https://document-api.vnzdna.com/mcp`, chạy **Scan Tools** và
upload ba bundled skills. Technical app ID trong `.app.json` chỉ phục vụ kết nối local đã đăng ký;
không dùng nó thay cho MCP URL trong public submission.

Người cài vẫn đọc được manifest, MCP URL và bundled skills. Cách bảo vệ sở hữu trí tuệ là giữ
thuật toán search, authorization, business rule và prompt nội bộ trên backend; không dùng đóng gói
JavaScript hoặc npm như một cơ chế che giấu nội dung phía client.
