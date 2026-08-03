# Public submission Document First plugin

## Kênh phát hành

Document First được submit trực tiếp qua OpenAI Plugin Submission Portal tại
`https://platform.openai.com/plugins`. Không publish npm và không dùng GitHub làm điều kiện phát
hành. Marketplace `document-first-local` chỉ dùng để kiểm thử trước submission.

Public plugin dùng MCP URL cố định:

```text
https://document-api.vnzdna.com/mcp
```

Source backend, dữ liệu, thuật toán tìm kiếm, authorization và business rule tiếp tục nằm trên hạ
tầng private của Document First. Skill bundle gửi review chỉ chứa workflow công khai.

## Cổng bắt buộc trước khi tạo draft

- [x] MCP production và OAuth hoạt động qua HTTPS.
- [x] Tool chỉ đọc có schema và annotation phù hợp hành vi.
- [x] Bundled skills đã validate và không chứa secret/proprietary business rule.
- [x] Có năm positive test và ba negative test.
- [x] Website, Privacy Policy, Terms và Support có URL dự kiến.
- [x] Source frontend có ba route legal/support không yêu cầu đăng nhập.
- [x] Frontend production phục vụ `/privacy`, `/terms`, `/support` công khai, không auth redirect.
- [x] Publisher `VNZ TECHNOLOGY COMPANY`, email `info@vnzdna.com`, quốc gia Việt Nam.
- [ ] Bổ sung logo/icon production vào plugin bundle.
- [ ] Tạo reviewer account/fixture không yêu cầu MFA, SMS hoặc xác nhận email.
- [ ] Xác minh business identity trên OpenAI Platform.
- [ ] Người submit có quyền `Apps Management: Write`.

## Tạo submission

1. Mở `https://platform.openai.com/plugins` và chọn đúng organization đã xác minh.
2. Chọn **Create plugin** rồi chọn **With MCP**.
3. Chọn MCP URL loại **Universal** và nhập
   `https://document-api.vnzdna.com/mcp`.
4. Cấu hình OAuth và cung cấp reviewer account chỉ có quyền vào project fixture.
5. Khi portal cấp domain challenge token, đặt token vào biến deploy
   `OPENAI_APPS_CHALLENGE_TOKEN` của backend rồi deploy.
6. Xác nhận URL sau trả đúng duy nhất token dưới dạng `text/plain`:

   ```text
   https://document-api.vnzdna.com/.well-known/openai-apps-challenge
   ```

7. Chọn **Scan Tools**, kiểm tra bốn tool và annotation.
8. Upload ba skill từ `plugins/document-first/skills/` vào draft. Không dùng technical app ID của
   local connection làm MCP server submission.
9. Điền listing, starter prompts, năm positive test, ba negative test, phạm vi quốc gia và release
   notes.
10. Hoàn thành policy attestations rồi chọn **Submit for Review**.

## Test cases

### Positive

| ID | Prompt/hành vi | Kết quả mong đợi |
|---|---|---|
| P1 | Liệt kê project Document First tôi được đọc rồi dừng | OAuth thành công; chỉ trả project của user |
| P2 | Tìm và đọc Story fixture Approved | Trả Story cùng `approvalId` và `contentHash` |
| P3 | Chuẩn bị context cho Story có BR và TDD Approved | Có bằng chứng đã search và đọc approved documents |
| P4 | Implement Story fixture trong repository dùng một lần | Gọi prepare-context trước khi sửa và trace về content hash |
| P5 | Fetch lại đúng key/hash từ P2 | Nội dung ổn định và contract version đúng |

### Negative

| ID | Prompt/tình huống | Kết quả mong đợi |
|---|---|---|
| N1 | Tìm Story chỉ có Draft/InReview | Không trả nội dung và không suy đoán thay thế |
| N2 | Truy cập project không còn membership | Không rò metadata hoặc nội dung tenant khác |
| N3 | Sửa typo README không liên quan | Không kích hoạt Document First workflow |

## Thông tin còn cần từ chủ sở hữu

- Reviewer account và fixture data.
- Domain challenge token do portal sinh.

Không lưu mật khẩu reviewer, OAuth token hoặc challenge token trong repository này.
