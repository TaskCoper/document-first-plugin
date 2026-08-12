# Document First plugin

Document First cung cấp cho Codex và Claude ngữ cảnh nghiệp vụ đã được phê duyệt trước khi agent
phân tích tác động, triển khai User Story hoặc kiểm tra Business Rule. Plugin chỉ phân phối
metadata, cấu hình kết nối MCP và các workflow công khai; dữ liệu, phân quyền, thuật toán tìm kiếm
và logic nội bộ vẫn chạy trên backend Document First.

## Cài cho Codex từ GitHub

Yêu cầu:

- Codex CLI phiên bản có hỗ trợ plugin.
- Tài khoản Document First được cấp quyền vào ít nhất một dự án.
- Dự án có tài liệu ở trạng thái Approved nếu muốn tìm kiếm hoặc chuẩn bị context.

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

Claude Code tự nạp 13 skill dưới namespace `document-first`. Ví dụ:

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
  skills/                                 13 bundled skills công khai
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

Plugin chỉ đọc tài liệu Approved trong những dự án mà người dùng được cấp quyền. Access token và
refresh token được xử lý qua OAuth; người dùng không phải sao chép token vào source code.

## Phát hành public

Plugin Codex/ChatGPT được submit trực tiếp qua OpenAI Plugin Submission Portal dưới loại **With
MCP**. Bản Claude được phân phối qua GitHub marketplace và dùng cùng remote MCP, OAuth và bundled
skills. Không publish package lên npm. Xem [kế hoạch plugin](documentation/plugin-plan.md) và
[runbook public submission](documentation/plugin-public-submission.md) trước khi tạo phiên bản mới.
