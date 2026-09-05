# Document First plugin

Document First cung cấp cho Codex và Claude nội dung hiện tại của mọi tài liệu trong project mà
tài khoản là thành viên. Draft, InReview, Approved và tài liệu đã lưu trữ đều có thể dùng để đọc,
review và triển khai. Plugin phân phối metadata, cấu hình MCP và workflow; backend giữ dữ liệu,
phân quyền và logic tìm kiếm. Plugin 0.7.0 dùng contract MCP `2026-09-05`.

## Cài cho Codex từ GitHub

Yêu cầu:

- Codex CLI phiên bản có hỗ trợ plugin.
- Tài khoản Document First được cấp quyền vào ít nhất một dự án.
- Tài khoản được phép đọc tài liệu trong project; không yêu cầu phê duyệt.

Thêm GitHub repository làm marketplace rồi cài plugin:

```bash
codex plugin marketplace add TaskCoper/document-first-plugin --ref main
codex plugin add document-first@document-first-local
```

Kiểm tra kết quả:

```bash
codex plugin marketplace list
codex plugin list
```

Sau khi cài, đóng phiên Codex hiện tại và mở một phiên mới để Codex nạp bundled skills và MCP
server. Hướng dẫn sử dụng đầy đủ nằm tại
[`plugins/document-first/README.md`](plugins/document-first/README.md).

## Cài cho Claude Code từ GitHub

Yêu cầu Claude Code có hỗ trợ plugin. Thêm repository làm marketplace rồi cài plugin:

```bash
claude plugin marketplace add TaskCoper/document-first-plugin@main
claude plugin install document-first@document-first-local
```

Trong Claude Code, chạy `/mcp`, chọn MCP server của plugin và hoàn tất đăng nhập OAuth. Có thể
kiểm thử source local trước khi cài marketplace:

```bash
claude plugin validate plugins/document-first
claude --plugin-dir ./plugins/document-first
```

Claude Code tự nạp 14 skill dưới namespace `document-first`. Ví dụ:

```text
/document-first:implement-story STORY-042 trong project PAYMENT
/document-first:draft-tdd cho STORY-042
/document-first:audit-document-traceability project PAYMENT
```

Không cần gõ lệnh cũng được: mỗi skill có `description` mô tả rõ khi nào dùng, nên chỉ cần nói
yêu cầu bằng ngôn ngữ tự nhiên là agent tự nạp skill phù hợp.

Trên Claude.ai, Claude Desktop hoặc mobile, có thể dùng riêng phần connector bằng cách thêm custom
connector với URL `https://document-api.vnzdna.com/mcp`. Full plugin gồm bundled skills được dùng
trên Claude Code và Cowork.

## Cấu trúc repository

```text
.agents/plugins/marketplace.json          Marketplace dùng cho GitHub/local
.claude-plugin/marketplace.json           Marketplace dành cho Claude
plugins/document-first/                   Plugin bundle
  .codex-plugin/plugin.json               Manifest Codex
  .claude-plugin/plugin.json              Manifest Claude
  .mcp.json                               Kết nối MCP dành cho Codex
                                          (Claude khai inline trong plugin.json)
  .app.json                               Technical app mapping
  skills/                                 14 bundled skills công khai
  README.md                               Hướng dẫn dành cho người cài
documentation/plugin-plan.md              Kế hoạch phát hành
documentation/plugin-public-submission.md Runbook OpenAI submission
```

## Phát triển và kiểm thử local

Từ máy đã clone repository, thêm marketplace bằng đường dẫn tuyệt đối tới repository root:

```bash
codex plugin marketplace add /absolute/path/to/document-first-plugin
codex plugin add document-first@document-first-local
```

Khi manifest hoặc skill thay đổi, cập nhật cachebuster bằng helper của `plugin-creator`, refresh
marketplace, cài lại plugin rồi kiểm thử trong một phiên Codex mới:

```bash
python3 /path/to/plugin-creator/scripts/update_plugin_cachebuster.py \
  plugins/document-first
codex plugin marketplace upgrade document-first-local
codex plugin add document-first@document-first-local
```

Validate plugin và nội dung package trước khi commit:

```bash
python3 /path/to/plugin-creator/scripts/validate_plugin.py plugins/document-first
claude plugin validate plugins/document-first

for skill in plugins/document-first/skills/*; do
  python3 /path/to/skill-creator/scripts/quick_validate.py "$skill"
done

npm pack ./plugins/document-first --dry-run --json \
  --cache /tmp/document-first-npm-cache
```

`npm pack --dry-run` chỉ kiểm tra file bundle. Plugin không được phát hành qua npm và
`package.json` phải tiếp tục giữ `private: true`.

## Nguyên tắc bảo mật

Mọi file trong plugin đều có thể được người cài đọc. Không đưa vào repository:

- source backend hoặc business rule độc quyền;
- prompt nội bộ hoặc thuật toán tìm kiếm;
- OAuth token, demo password, registry credential hoặc production secret;
- dữ liệu tài liệu của khách hàng.

Plugin đọc mọi tài liệu chưa xoá trong những dự án mà người dùng là thành viên. Access token và
refresh token được xử lý qua OAuth; người dùng không phải sao chép token vào source code.

## Phát hành public

Plugin Codex/ChatGPT được submit trực tiếp qua OpenAI Plugin Submission Portal dưới loại **With
MCP**. Bản Claude được phân phối qua GitHub marketplace và dùng cùng remote MCP, OAuth và bundled
skills. Không publish package lên npm. Xem [kế hoạch plugin](documentation/plugin-plan.md) và
[runbook public submission](documentation/plugin-public-submission.md) trước khi tạo phiên bản mới.

### Quyền ghi tài liệu từ MCP

Ba tool `document_first_create_document`, `document_first_update_document` và
`document_first_delete_document` cần `documents:write` và vai trò Editor/Admin/Owner.
Sau khi deploy backend mới, kết nối OAuth lại để cấp scope ghi; token chỉ đọc cũ không tự có quyền ghi.
Sửa/xoá cần `expectedContentHash` từ `fetch`. Section không gửi được giữ nguyên; section được gửi
thay toàn bộ nội dung, mảng rỗng xoá section. Không tự ghi lại khi báo xung đột hoặc mất phản hồi:
đọc và đối chiếu trạng thái trước. Thay đổi nội dung Approved/InReview đưa tài liệu về Draft.
