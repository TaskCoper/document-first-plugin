# Hướng dẫn sử dụng Document First plugin

Document First giúp Codex và Claude đọc User Story, AC, Business Rule, TDD và test trong project
mà tài khoản hiện tại là thành viên. Tất cả trạng thái đều đọc được: Draft, InReview, Approved
và đã lưu trữ. Plugin hỗ trợ đọc, tạo, sửa và xoá cả 5 loại tài liệu. Quyền ghi cần scope `documents:write` và vai trò Editor trở lên; phê duyệt vẫn thực hiện trong ứng dụng.

## 1. Chuẩn bị

Có tài khoản Document First và membership của project cần làm việc. Cung cấp project/story key
đủ rõ để agent tìm đúng dữ liệu. Không cần phê duyệt, Release hoặc liên kết GitHub repository.

Plugin 0.7.0 dùng contract MCP `2026-09-05`. Khi nâng cấp cần deploy backend hỗ trợ contract mới,
cập nhật plugin và mở phiên mới.

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
  --scopes projects:read,documents:read,documents:write,context:prepare,documents:read.draft,offline_access
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

### Ví dụ sử dụng

```text
Triển khai STORY-042 trong project PAYMENT theo nội dung hiện tại và các rule liên quan.
Review STORY-042 trong project PAYMENT, chỉ ra Flow/AC còn thiếu.
Kiểm tra implementation của STORY-042 theo Business Rule trong project.
```

Agent lấy context bằng `document_first_prepare_story_context` khi triển khai Story, đọc trực tiếp
bằng `document_first_fetch`/`document_first_get_story` khi đã biết key và dùng search để tìm nguồn
liên quan. Mọi trạng thái đều hợp lệ để đọc; tài liệu thiếu nội dung cần được làm rõ theo yêu cầu.

## 5. Bundled skills

14 skill hỗ trợ soạn Story/BR/TDD/AC, vẽ diagram, thiết kế test, implement, review và kiểm tra
traceability. Skill đọc nội dung project ở mọi trạng thái. Khi được yêu cầu lưu, skill dùng MCP để tạo/cập nhật trực tiếp; `manage-documents` xử lý thao tác ghi và xoá. Khi chỉ yêu cầu soạn để xem, trả Markdown.

## 6. MCP tools và bằng chứng

| Tool — tiền tố `document_first_` | Hành vi |
|---|---|
| `list_projects` | Project của người gọi |
| `search` | Tìm tài liệu ở mọi trạng thái |
| `fetch` | Markdown hiện tại, có thể ghim contentHash |
| `get_document` | Đọc cấu trúc đầy đủ các section để sửa mà giữ metadata/ID |
| `get_story` | Story dạng field, flow, AC và reference |
| `list_rules` | Duyệt rule có phân trang hoặc đọc theo key |
| `prepare_story_context` | Tổng hợp Story, reference, search và semantic-related |
| `review_queue` | Metadata InReview được giao review/approve |
| `fetch_draft` | Tool tương thích đọc mọi trạng thái; giữ draftContentHash/unstable |
| `create_document` | Tạo một trong 5 loại, sinh mã và lưu nội dung ban đầu |
| `update_document` | Thay các section được gửi, kiểm hash và lưu nguyên tử |
| `delete_document` | Xoá mềm tài liệu đã được người dùng chỉ định |

`documents:read` cho phép đọc mọi trạng thái trong project và cả tool tương thích. Scope cũ
`documents:read.draft` vẫn được hỗ trợ cho hai tool review. OAuth scope không thay thế membership.

Kết quả có `approvalState`, `isArchived` và hash tính từ nội dung được trả về; `approvalId` và
`approvedAt` chỉ có khi tồn tại phê duyệt hợp lệ. Truy vết bằng `documentKey#contentHash`.
Context có `evidence.searched` và `evidence.readDocuments`, tối đa 20 tài liệu; đọc bổ sung khi cần.
`missingKeys`/`unresolvedReferences` không đồng nghĩa với chưa phê duyệt.

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
  --scopes projects:read,documents:read,documents:write,context:prepare,documents:read.draft,offline_access
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

Kiểm tra membership, project/story key và thử truy vấn cụ thể hơn. Draft/InReview vẫn tìm được.
Tài liệu đã xoá không được trả về; dùng fetch khi biết chính xác key.

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

### Quyền ghi tài liệu từ MCP

Ba tool `document_first_create_document`, `document_first_update_document` và
`document_first_delete_document` cần `documents:write` và vai trò Editor/Admin/Owner.
Sau khi deploy backend mới, kết nối OAuth lại để cấp scope ghi; token chỉ đọc cũ không tự có quyền ghi.
Sửa cần đọc `get_document` để giữ đủ metadata/ID; sửa/xoá cần `expectedContentHash` hiện tại. Section không gửi được giữ nguyên; section được gửi
thay toàn bộ nội dung, mảng rỗng xoá section. Không tự ghi lại khi báo xung đột hoặc mất phản hồi:
đọc và đối chiếu trạng thái trước. Thay đổi nội dung Approved/InReview đưa tài liệu về Draft.
