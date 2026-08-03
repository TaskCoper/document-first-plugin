# Document First plugin

Repository phân phối thin plugin Document First cho Codex và ChatGPT. Package chỉ chứa metadata,
cấu hình kết nối tới remote MCP và bundled skills công khai; search, authorization, business rule,
dữ liệu và prompt nội bộ tiếp tục chạy trên backend Document First.

## Cấu trúc

```text
.agents/plugins/marketplace.json      Marketplace local để kiểm thử
plugins/document-first/              Plugin bundle gửi OpenAI review
documentation/plugin-plan.md         Kế hoạch phát hành
documentation/plugin-public-submission.md Runbook public submission
```

## Validate

```bash
python3 /path/to/plugin-creator/scripts/validate_plugin.py plugins/document-first
npm pack ./plugins/document-first --dry-run --json \
  --cache /tmp/document-first-npm-cache
```

Ba `SKILL.md` phía client luôn có thể được người cài đọc. Không thêm source backend, secret,
business rule thực tế, prompt nội bộ hoặc credential registry vào repository này.

`npm pack --dry-run` chỉ dùng để kiểm tra file bundle; plugin không được phát hành qua npm.

Xem [kế hoạch plugin](documentation/plugin-plan.md) và
[runbook public submission](documentation/plugin-public-submission.md) trước khi gửi OpenAI review.
