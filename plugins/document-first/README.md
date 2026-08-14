# Hướng dẫn sử dụng Document First plugin

Document First giúp Codex và Claude lấy đúng User Story, Acceptance Criteria, Business Rule,
technical design và API contract đã được phê duyệt trước khi agent thay đổi code. Plugin chỉ có
quyền đọc và chỉ nhìn thấy các dự án mà tài khoản hiện tại được cấp quyền.

## 1. Chuẩn bị dữ liệu trên Document First

Trước khi dùng plugin, bảo đảm rằng:

1. Bạn đã có tài khoản tại <https://document-first.vnzdna.com>.
2. Tài khoản đã được thêm vào dự án cần làm việc.
3. User Story và các tài liệu liên quan đã được Approve.
4. Prompt gửi cho agent có ít nhất project hoặc story key đủ rõ, ví dụ `PAYMENT` và
   `STORY-042`.

Các tool phục vụ triển khai chỉ trả tài liệu Approved. Tool `document_first_fetch_draft` có thể
đọc Draft/InReview trong dự án mà tài khoản là thành viên; tài liệu thuộc dự án không được cấp
quyền sẽ không được MCP trả về.

## 2. Cài plugin từ GitHub

### Codex

Thêm repository làm marketplace:

```bash
codex plugin marketplace add TaskCoper/document-first-plugin --ref main
```

Cài Document First:

```bash
codex plugin add document-first@document-first-local
```

Kiểm tra plugin đã được cài và bật:

```bash
codex plugin list
```

Bạn cũng có thể chạy `codex`, nhập `/plugins`, chọn marketplace **Document First Local** rồi cài
**Document First**. Sau khi cài, hãy bắt đầu một phiên Codex mới; phiên đang mở trước đó không tự
nạp các skill và tool mới.

### Claude Code

Thêm repository làm marketplace:

```bash
claude plugin marketplace add TaskCoper/document-first-plugin@main
```

Cài Document First:

```bash
claude plugin install document-first@document-first-local
```

Kiểm tra plugin:

```bash
claude plugin list
```

Trong giai đoạn phát triển local, có thể validate và load trực tiếp:

```bash
claude plugin validate plugins/document-first
claude --plugin-dir ./plugins/document-first
```

## 3. Đăng nhập OAuth

Khi agent yêu cầu kết nối, đăng nhập bằng tài khoản Document First và chọn **Cho phép**.

Với Codex, có thể chủ động bắt đầu đăng nhập bằng CLI:

```bash
codex mcp login document-first \
  --scopes projects:read,documents:read,context:prepare,documents:read.draft,offline_access
```

Kiểm tra MCP trong phiên Codex bằng `/mcp`, hoặc từ terminal:

```bash
codex mcp list
```

Với Claude Code, chạy `/mcp`, chọn MCP server `document-first` do plugin cung cấp và hoàn tất luồng
đăng nhập trong browser. Claude Code lưu và refresh OAuth credential tự động.

Không nhập access token, refresh token hoặc password vào repository, prompt hay file cấu hình.

## 4. Sử dụng trong dự án code

Mở Codex hoặc Claude Code tại repository cần triển khai và mô tả kết quả mong muốn. Bạn có thể gọi
workflow bằng ngôn ngữ tự nhiên; agent sẽ chọn bundled skill và MCP tool phù hợp. Trên Claude Code,
có thể gọi trực tiếp `/document-first:implement-story`, `/document-first:review-impact` hoặc
`/document-first:verify-business-rules`.

### Triển khai một User Story

```text
Hãy triển khai STORY-042 trong project PAYMENT. Trước khi sửa code, bắt buộc lấy và đọc toàn bộ
approved context liên quan từ Document First. Sau khi hoàn tất, chạy các test phù hợp và đối chiếu
kết quả với Acceptance Criteria và Business Rule đã đọc.
```

Workflow dự kiến:

1. Xác định project và story.
2. Gọi `document_first_prepare_story_context`.
3. Đọc approved-only context và các tài liệu liên quan.
4. Phân tích phạm vi thay đổi.
5. Sửa code và chạy kiểm thử.
6. Báo cáo tài liệu/rule nào đã được dùng và những điểm chưa đủ context.

### Phân tích tác động trước khi sửa code

```text
Phân tích tác động của STORY-042 trong project PAYMENT. Liệt kê module, API, database, test và
Business Rule có thể bị ảnh hưởng. Chưa thay đổi source code.
```

### Xác minh Business Rule

```text
Kiểm tra implementation hiện tại của STORY-042 trong project PAYMENT theo các Business Rule và
Acceptance Criteria đã Approved. Nêu rõ rule nào pass, fail hoặc chưa có đủ bằng chứng.
```

### Tìm hoặc đọc tài liệu

```text
Tìm các tài liệu Approved trong project PAYMENT liên quan đến refund và tóm tắt các rule về thời
hạn hoàn tiền. Ghi rõ document key của từng nguồn.
```

### Review User Story trước khi phê duyệt

```text
Review STORY-042 trong project PAYMENT. Đọc nội dung hiện tại dù Story đang Draft hoặc InReview,
chỉ ra Flow/Acceptance Criteria còn thiếu hoặc mơ hồ. Không implement source code.
```

Workflow review ưu tiên `document_first_fetch_draft`; nếu Story đã Approved thì dùng
`document_first_get_story` hoặc `document_first_fetch`. Chỉ workflow implement mới bắt buộc Story
đã Approved.

## 5. Bundled skills

- `implement-story`: chuẩn bị và đọc approved-only context trước khi sửa code.
- `review-impact`: đọc nội dung hiện tại khi review tài liệu; chỉ bắt buộc Approved khi phân tích
  để triển khai, rồi xác định rule, contract, module và test có thể bị ảnh hưởng.
- `verify-business-rules`: đối chiếu implementation hoặc test với nội dung đã phê duyệt.

Các skill mô tả workflow công khai. Business rule thực tế và prompt nội bộ không nằm trong bundle
mà được backend trả về theo quyền của từng người dùng.

## 6. MCP tools

- `document_first_list_projects`: liệt kê dự án tài khoản hiện tại được phép đọc.
- `document_first_search`: tìm kiếm trong nội dung Approved của dự án được phép truy cập.
- `document_first_fetch`: đọc Markdown Approved và trả về thông tin định danh nội dung.
- `document_first_prepare_story_context`: tìm story, resolve reference và tổng hợp approved-only
  context cho một yêu cầu triển khai.
- `document_first_get_story`: đọc một User Story Approved dưới dạng field có cấu trúc (metadata,
  flow, acceptance criteria, reference) thay vì Markdown thô.
- `document_first_list_rules`: duyệt danh sách Business Rule Approved kèm phân trang, hoặc đọc
  Markdown đầy đủ của các rule chỉ định qua `documentKeys`.

Tất cả tool hiện tại đều read-only. Plugin không sửa tài liệu, approve nội dung, commit code hoặc
push lên GitHub.

### Tool dành cho reviewer

Hai tool dưới đây đọc nội dung **chưa được phê duyệt** nên nằm sau một scope OAuth riêng,
`documents:read.draft`. Scope được cấu hình mặc định nhưng vẫn hiện riêng trên màn hình consent:

- `document_first_review_queue`: liệt kê tài liệu đang chờ duyệt mà chính bạn được giao làm
  reviewer hoặc approver. Chỉ trả metadata.
- `document_first_fetch_draft`: đọc Markdown bản nháp của tài liệu Draft/InReview. Mọi thành viên
  của dự án đều có quyền đọc; người ngoài dự án không được truy cập.

Phản hồi của `document_first_fetch_draft` **không có `approvalId`** và luôn kèm `unstable: true`:
nội dung có thể đổi bất cứ lúc nào nên không được trích dẫn như nguồn đã phê duyệt, cũng không
được dùng làm căn cứ khi implement.

Scope này tách riêng có chủ đích: consent luôn cho biết connector đang xin quyền đọc nội dung chưa
phê duyệt. Quyền OAuth không thay thế phân quyền project — chỉ thành viên của project mới đọc được
Draft/InReview trong project đó. Token cũ chưa có scope này không tự nhiên được nới quyền; cần
clear authentication và kết nối lại một lần.

## 7. Cập nhật plugin

Với Codex, refresh snapshot từ GitHub rồi cài lại plugin:

```bash
codex plugin marketplace upgrade document-first-local
codex plugin add document-first@document-first-local
```

Sau đó đóng phiên Codex cũ và mở phiên mới.

Với Claude Code:

```bash
claude plugin marketplace update document-first-local
claude plugin update document-first@document-first-local
```

Chạy `/reload-plugins` hoặc bắt đầu một phiên Claude Code mới sau khi cập nhật.

## 8. Xử lý sự cố

### `invalid_grant: The specified token is invalid`

Refresh token cũ đã hết hiệu lực, bị thu hồi hoặc không còn giải mã được. Với Codex, đóng mọi phiên
đang chạy rồi đăng nhập lại:

```bash
codex mcp logout document-first
codex mcp login document-first \
  --scopes projects:read,documents:read,context:prepare,documents:read.draft,offline_access
```

Hoàn tất màn hình **Cho phép**, rồi mở một phiên Codex mới. Nếu lỗi tiếp tục xuất hiện ngay sau
khi đăng nhập thành công, liên hệ quản trị viên Document First; không xoá thủ công file credential
và không gửi token trong ticket hỗ trợ.

Với Claude Code, mở `/mcp`, chọn server `plugin:document-first:document-first`, dùng **Clear
authentication** rồi kết nối lại.

### MCP startup incomplete hoặc không thấy tool

Với Codex, chạy lần lượt:

```bash
codex plugin list
codex mcp list
```

Kiểm tra plugin đang enabled, xác thực OAuth lại nếu được yêu cầu, sau đó khởi động phiên Codex
mới. Nếu server không phản hồi, kiểm tra kết nối tới `https://document-api.vnzdna.com/mcp`.

Với Claude Code, chạy `claude plugin list`, sau đó mở `/mcp`. Server của plugin phải xuất hiện dưới
tên `plugin:document-first:document-first`; trạng thái **Needs authentication** nghĩa là cần hoàn
tất OAuth, còn **Failed to connect** cần kiểm tra DNS, HTTPS hoặc endpoint production.

### Không thấy project

Tài khoản OAuth chưa được cấp quyền vào project hoặc quyền vừa bị thu hồi. Đăng nhập Document First
để kiểm tra membership; nếu cần, yêu cầu project owner cấp quyền đọc rồi thử lại.

### Tìm kiếm không có kết quả

Chỉ nội dung Approved mới được trả về. Kiểm tra trạng thái approval, project/story key và thử câu
truy vấn cụ thể hơn. Không dùng tài liệu của project khác để thay thế khi thiếu context.

### Gỡ cài đặt Codex

```bash
codex mcp logout document-first
codex plugin remove document-first@document-first-local
```

Chỉ xoá marketplace nếu không còn plugin nào khác sử dụng nó:

```bash
codex plugin marketplace remove document-first-local
```

### Gỡ cài đặt Claude Code

```bash
claude plugin uninstall document-first@document-first-local
claude plugin marketplace remove document-first-local
```

## Hỗ trợ và chính sách

- Website: <https://document-first.vnzdna.com>
- Hỗ trợ: <https://document-first.vnzdna.com/support>
- Chính sách riêng tư: <https://document-first.vnzdna.com/privacy>
- Điều khoản sử dụng: <https://document-first.vnzdna.com/terms>
- Email: <info@vnzdna.com>
