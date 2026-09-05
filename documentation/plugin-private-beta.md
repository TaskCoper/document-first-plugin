# Private beta Document First plugin

> Cập nhật 2026-09-05: chính sách Approved-only/chỉ đọc bên dưới là lịch sử. Plugin 0.7.0
> đọc mọi trạng thái và hỗ trợ tạo/sửa/xoá qua scope `documents:write` + Editor; xem README hiện hành.

> Luồng này đã được bỏ qua theo quyết định phát hành trực tiếp lên OpenAI Plugins Directory.
> Tài liệu chỉ được giữ làm phương án rollback; quy trình hiện hành nằm tại
> [`plugin-public-submission.md`](./plugin-public-submission.md).

## Mục tiêu và phạm vi

Private beta xác minh plugin với 3–5 người dùng được chọn trước khi submit public. Phiên bản beta
chỉ đọc nội dung hiện tại đã được phê duyệt đúng content hash; không ghi document, không publish
GitHub và không chứa source backend.

Mỗi người thử nghiệm phải có tài khoản Document First, là thành viên đúng project và đăng nhập qua
OAuth. Không chia sẻ access token, refresh token, npm token hoặc tài khoản dùng chung.

## Kênh phân phối

Hướng mặc định khi không dùng GitHub là một private npm registry của công ty:

1. Đội vận hành tạo registry/scope riêng và cấp read-only package token cho nhóm beta.
2. Cấu hình npm trên máy người dùng; credential chỉ nằm trong npm configuration hoặc secret store.
3. Bỏ `private: true` khỏi `plugins/document-first/package.json` trong release branch đã duyệt,
   publish package `@document-first/codex-plugin` với một version cố định, rồi bật lại protection
   cho nhánh phát triển nếu cần.
4. Marketplace riêng dùng source sau; thay URL registry và version bằng giá trị đã chốt:

```json
{
  "name": "document-first-beta",
  "interface": {
    "displayName": "Document First Beta"
  },
  "plugins": [
    {
      "name": "document-first",
      "source": {
        "source": "npm",
        "package": "@document-first/codex-plugin",
        "version": "0.4.0-beta.1",
        "registry": "https://registry.company.example"
      },
      "policy": {
        "installation": "AVAILABLE",
        "authentication": "ON_INSTALL"
      },
      "category": "Developer Tools"
    }
  ]
}
```

Không đưa credential vào trường `registry`: đây phải là HTTPS URL không chứa username, password,
query hoặc fragment. Máy khách cần npm CLI và tự lấy quyền registry từ cấu hình npm.

Package npm không che giấu nội dung phía client. Người cài đọc được `.codex-plugin`, `.mcp.json`,
`.app.json` và `SKILL.md`; vì vậy package chỉ chứa workflow công khai. Backend, dữ liệu, thuật toán
tìm kiếm, authorization và business rule vẫn ở hạ tầng Document First.

## Dữ liệu thử nghiệm an toàn

- Tạo một project beta riêng, chỉ có tài liệu tổng hợp phục vụ kiểm thử.
- Approve Story, Business Rule và TDD bằng đúng flow production; không sao chép tài liệu khách hàng.
- Dùng repository fixture bỏ đi được nếu test có sửa code. Story phải mô tả đúng repository fixture
  đang mở; Approved không đồng nghĩa với liên quan tới mọi workspace.
- Với project production không liên quan, chỉ chạy smoke test đọc như `list_projects`; không yêu cầu
  agent implement vào repository hiện tại.
- Gỡ thành viên và archive/xoá dữ liệu fixture sau khi beta kết thúc theo retention policy.

## Bộ test beta

### Năm ca dương

| ID | Prompt/hành vi | Kết quả mong đợi |
|---|---|---|
| P1 | Liệt kê project Document First tôi được đọc rồi dừng | OAuth thành công; chỉ trả project của user; không sửa trạng thái |
| P2 | Tìm và đọc Story beta Approved chưa từng Release | Search/fetch trả Story cùng `approvalId`, `contentHash`; không cần GitHub |
| P3 | Chuẩn bị context cho Story beta có BR và TDD Approved | `evidence.searched` và `evidence.readApprovedDocuments` là `true`; related context đúng project |
| P4 | Implement Story beta trong disposable repository đã khớp fixture | Agent gọi prepare-context trước khi sửa; thay đổi/test trace được về document hash |
| P5 | Fetch lại đúng `documentKey` và `contentHash` đã nhận | Nội dung ổn định; contract version đúng; audit không chứa Markdown/token |

### Ba ca âm

| ID | Prompt/tình huống | Kết quả mong đợi |
|---|---|---|
| N1 | Tìm Story chỉ ở Draft hoặc InReview | Không trả nội dung; agent dừng, không thay bằng kiến thức suy đoán |
| N2 | Truy cập project mà user không còn là thành viên | Forbidden/not found theo contract; không rò metadata tenant khác |
| N3 | Sửa typo README không liên quan Document First | Không tự kích hoạt workflow Story hoặc gọi MCP không cần thiết |

Test thêm cho vận hành: access token hết hạn được refresh; token bị revoke yêu cầu đăng nhập lại;
rate limit trả lỗi có thể retry; semantic search hỏng vẫn không làm rò Draft/InReview.

## Chỉ số và feedback

Theo dõi theo user/project/tool nhưng không log query, Markdown hoặc token:

- tỷ lệ login thành công và lỗi OAuth;
- số lần gọi từng tool, p50/p95 latency và error rate;
- tỷ lệ Story tìm được, số unresolved reference và số lần agent bị `Blocked`;
- tỷ lệ prompt đúng chọn `prepare_story_context`;
- feedback về context thiếu, sai liên quan hoặc quá dài.

Mỗi feedback cần ghi plugin version, contract version, project fixture, document key/content hash và
correlation ID; không chép toàn bộ tài liệu vào ticket.

## Rollout và rollback

1. Bắt đầu với một người nội bộ và project fixture.
2. Khi P1–P5 và N1–N3 pass, mở cho tối đa 3–5 người dùng.
3. Chỉ tăng phạm vi nếu 24 giờ không có lỗi authorization, tenant isolation hoặc data leakage.
4. Khi có sự cố nghiêm trọng, đặt marketplace `policy.installation` thành `NOT_AVAILABLE`, thu hồi
   OAuth grant/token liên quan và unpublish/yank beta version khỏi registry nếu registry hỗ trợ.
5. Khắc phục backend trước nếu lỗi nằm ở search/authorization; phát hành package version mới nếu
   manifest hoặc bundled skill thay đổi. Không ghi đè một version đã phân phối.

## Cổng sẵn sàng

- [x] MCP production, OAuth và approved-only contract hoạt động.
- [x] Plugin local cài được và luồng positive approved fixture đã pass.
- [x] Package chỉ chứa thin client workflow, không chứa secret hoặc backend source.
- [x] Có 5 ca dương, 3 ca âm và rollback plan.
- [ ] Chốt private registry URL và quyền publish/read.
- [ ] Chốt danh sách 3–5 beta user và project fixture.
- [ ] Publish immutable beta version rồi kiểm tra cài mới trên máy sạch.
- [ ] Thiết lập dashboard/alert và kênh nhận feedback.
- [ ] Trước public submission: deploy trang support, Privacy Policy và Terms thật; hoàn thiện logo.
