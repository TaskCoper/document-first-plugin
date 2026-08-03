# Kế hoạch xây dựng Document First plugin

## 1. Mục tiêu

Xây dựng một plugin Document First có thể cài đặt trên Codex và ChatGPT, cho phép agent
đọc User Story, Business Rule, TDD, API contract và tài liệu liên quan trước khi triển khai
code.

Các yêu cầu chính:

- Khách hàng không nhận source code backend hoặc logic nghiệp vụ nội bộ.
- Plugin phía khách hàng chứa metadata, branding, cấu hình kết nối và bundled skills công khai.
- Bundled skills chỉ mô tả workflow; không chứa business rule hoặc proprietary logic.
- Toàn bộ tìm kiếm, phân quyền và tổng hợp context chạy trên hạ tầng Document First.
- Nội dung hiện tại đã được phê duyệt đúng content hash là nguồn chuẩn duy nhất cho agent.
- Phiên bản đầu chỉ cung cấp thao tác đọc.
- Submit trực tiếp lên OpenAI Plugin Submission Portal; không phân phối qua private npm.

## 2. Kiến trúc mục tiêu

```text
Codex / ChatGPT
      │
      ▼
Document First plugin
(manifest, bundled skills, branding, MCP connection)
      │
      ▼ OAuth 2.1
Remote MCP server
      │
      ├── Authentication
      ├── Authorization
      ├── Approved-document search
      ├── Related-document workflow
      ├── Business rules
      └── Audit logs
      │
      ▼
Document First database
```

Plugin được phân phối có thể công khai vì không chứa proprietary logic. MCP server và
backend tiếp tục nằm trong repository private của công ty.

## 3. Phase 1 — Chốt phạm vi plugin

**Trạng thái: Hoàn tất.**

### Công việc

- Chọn kiến trúc bundled skills kết hợp remote MCP server.
- Chỉ đưa workflow công khai vào `SKILL.md`; giữ business rule và logic nội bộ trên MCP server.
- Chỉ cung cấp tool read-only trong phiên bản đầu.
- Xác định reviewer account và project fixture an toàn cho OpenAI review.
- Chốt dữ liệu nào được phép trả cho agent.
- Chốt workflow chính là `prepare_story_context`.
- Chốt quy tắc nội dung Approved là nguồn chuẩn duy nhất; không yêu cầu Release/GitHub.

### Definition of Done

- Có danh sách use case và tool được phê duyệt.
- Có data-classification cho draft, release và dữ liệu nhạy cảm.
- Có quyết định rõ phạm vi public release và dữ liệu reviewer được phép đọc.
- Có quyết định bundled skills kết hợp remote MCP server.

## 4. Phase 2 — Hoàn thiện MCP server

**Trạng thái: Hoàn tất phần code và regression; MCP Inspector production được kiểm ở Phase 5.**

### Phần đã có

- Streamable HTTP stateless tại `/mcp`.
- `document_first_list_projects`.
- `document_first_search`.
- `document_first_fetch`.
- `document_first_prepare_story_context`.
- Kiểm tra project membership.
- Chỉ trả relational document hiện tại có approval hợp lệ cho đúng content hash.

### Đã hoàn tất

- Tạo approved-only search trên `documents.search_vector` và chunk hiện tại có
  `document_version_id IS NULL`.
- Áp dụng gate `Approved` + current approval + submitted/draft/approval hash khớp nhau; render và
  tính lại SHA-256 trước khi trả nội dung.
- Khai báo output schema, contract `2026-08-03` và MCP annotations read-only/non-destructive.
- Chuẩn hóa lỗi MCP:
  - unauthenticated;
  - forbidden;
  - project not found;
  - approved document not found;
  - validation failed;
  - rate limited.
- Thêm timeout, cancellation, giới hạn response và rate-limit policy `heavy`.
- Ghi `MCP_AUDIT` theo user, project, tool, document IDs, latency và response size.
- Test Draft/InReview leak, Approved chưa từng release, content-hash pin, approval reset,
  tenant isolation và revoked access; log không chứa nội dung/token.

### Definition of Done

- Draft, InReview và tài liệu archived không ảnh hưởng nội dung hoặc thứ hạng kết quả MCP.
- MCP Inspector có thể list và gọi tất cả tool.
- Tool schema có version và có regression test.
- Authorization được kiểm chứng giữa nhiều tenant/project.

## 5. Phase 3 — Xây dựng bundled skills

**Trạng thái: Hoàn tất source package và validation; end-to-end install được kiểm ở Phase 9.**

### Cấu trúc mục tiêu

```text
plugins/document-first/
└── skills/
    ├── implement-story/
    │   └── SKILL.md
    ├── review-impact/
    │   └── SKILL.md
    └── verify-business-rules/
        └── SKILL.md
```

### Skill dự kiến

- `implement-story`: gọi `document_first_prepare_story_context` trước khi sửa code và đối
  chiếu implementation với Acceptance Criteria đã phê duyệt.
- `review-impact`: tìm tài liệu, API contract, test và module có thể bị ảnh hưởng.
- `verify-business-rules`: kiểm tra implementation theo Business Rule đã phê duyệt.

### Nguyên tắc bảo mật

- Người cài plugin có thể đọc toàn bộ `SKILL.md`.
- Skill chỉ chứa workflow công khai, điều kiện kích hoạt và tiêu chí hoàn thành.
- Không đưa business rule thực tế, prompt nội bộ, thuật toán search hoặc logic phân quyền vào skill.
- Mọi dữ liệu nghiệp vụ và quyết định nhạy cảm tiếp tục được trả từ MCP server theo quyền user.
- Không đóng gói script chứa secret hoặc gọi API bằng credential tĩnh.

### Definition of Done

- Mỗi skill có trigger cụ thể và không kích hoạt cho yêu cầu không liên quan.
- `implement-story` gọi đúng composite MCP tool trước khi đề xuất sửa code.
- Skill dừng với lỗi rõ ràng khi Story chưa được phê duyệt hoặc không có quyền đọc.
- Skill references resolve sau khi cài plugin.
- Skill validator pass và không chứa proprietary logic.

### Kết quả

- Đã tạo `implement-story`, `review-impact`, `verify-business-rules` kèm `agents/openai.yaml`.
- Cả ba khai báo dependency tới MCP `document-first` và đều pass `quick_validate.py`.
- Manifest plugin hiện ở `0.4.0+codex.20260803150313`, trỏ `skills` tới `./skills/` và pass
  plugin validator.

## 6. Phase 4 — OAuth 2.1

**Trạng thái: Hoàn tất source code và test tự động; login production được kiểm ở Phase 5–6.**

### Mục tiêu

Thay cơ chế `DOCUMENT_FIRST_ACCESS_TOKEN` thủ công bằng luồng đăng nhập và cấp quyền phù hợp
cho khách hàng.

### Đã hoàn tất

- Tích hợp OpenIddict làm authorization server và lưu application/authorization/token/scope trong
  PostgreSQL.
- Cung cấp authorization, token, revocation và Dynamic Client Registration endpoint.
- Bắt buộc Authorization Code + PKCE `S256`; chỉ nhận public client, không phát client secret.
- Giữ nguyên `resource` qua authorize/token và kiểm audience chính xác tại `/mcp`.
- Hỗ trợ refresh-token rotation; cho phép hai MCP client refresh đồng thời trong cửa sổ 5 giây,
  sau đó từ chối replay; token revocation có hiệu lực tức thời.
- Cung cấp protected-resource metadata và OAuth/OIDC discovery.
- Giữ `iss` trong authorization response nhưng tạm công bố RFC 9207 là không hỗ trợ để tương thích
  lỗi callback của Codex CLI 0.146.0; xoá workaround khi upstream sửa.
- Định nghĩa và kiểm tra scope theo từng tool:
  - `projects:read`;
  - `documents:read`;
  - `context:prepare`.
- Giữ tenant isolation bằng user subject và kiểm project membership trực tiếp ở mỗi tool call.
- Disconnect dùng revocation endpoint; xóa thành viên project có hiệu lực ngay cả với session MCP
  đã initialize.
- Đổi `.mcp.json` sang OAuth native; xoá hoàn toàn `DOCUMENT_FIRST_ACCESS_TOKEN` khỏi plugin.
- Thêm migration `AddMcpOAuth`, 10 test OAuth end-to-end và chạy sạch toàn bộ regression
  (202 pass, 5 skip).

### Definition of Done

- Người dùng cài plugin và đăng nhập bằng tài khoản Document First.
- Không cần copy access token thủ công.
- Refresh, revoke và logout hoạt động.
- Token của tenant này không truy cập được tenant khác.

Ba mục đầu đã được chứng minh trong host test; thao tác đăng nhập từ Codex/ChatGPT qua endpoint
production được chốt sau deploy và scan connection ở Phase 5–6.

## 7. Phase 5 — Deploy MCP production

**Trạng thái: Hoàn tất endpoint, discovery và OAuth production; dashboard/alert nâng cao còn theo
dõi trong private beta.**

### Endpoint mục tiêu

```text
https://document-api.vnzdna.com/mcp
```

### Công việc

- Deploy code MCP lên production.
- Bảo đảm HTTPS và Streamable HTTP hoạt động qua proxy/tunnel.
- Thêm health check riêng cho MCP.
- Cấu hình rate limiting.
- Cấu hình centralized logging và error monitoring.
- Thu thập metrics:
  - tool-call count;
  - latency percentile;
  - error rate;
  - authorization failures;
  - số tài liệu được đọc;
  - response size.
- Kiểm tra horizontal scaling cho stateless transport.

### Definition of Done

- Endpoint production không còn trả `404`.
- MCP Inspector kết nối được từ internet.
- OAuth discovery hoạt động.
- Có dashboard và alert cho lỗi MCP.

## 8. Phase 6 — Đăng ký MCP connection

**Trạng thái: Hoàn tất.**

### Công việc

1. Bật Developer Mode trong ChatGPT.
2. Tạo MCP connection tới endpoint production.
3. Hoàn thành authentication.
4. Scan và review danh sách tool.
5. Lấy technical ID dạng `plugin_asdk_app_...`.
6. Tạo `.app.json` ánh xạ plugin tới registered MCP connection.
7. Kiểm tra `.codex-plugin/plugin.json` trỏ đúng tới `.app.json`.

### Definition of Done

- Connection hiển thị đúng các tool đã công bố.
- Login hoạt động từ developer mode.
- Tool có thể gọi trực tiếp qua connection đã đăng ký.

### Kết quả

- ChatGPT Developer Mode đã kết nối và hoàn tất OAuth với MCP production.
- Registered connection có technical ID `plugin_asdk_app_6a705c3a25bc819189e69c7cce9d9823`.
- Plugin ánh xạ connection qua `.app.json` bằng app ID `asdk_app_6a705c3a25bc819189e69c7cce9d9823`;
  key `document-first-chatgpt` được tách khỏi MCP key `document-first` để hai capability không đè
  nhau trong Codex runtime.

## 9. Phase 7 — Hoàn thiện package plugin

**Trạng thái: Đang hoàn thiện cho public submission — đã có manifest `0.4.0`, website production,
OAuth MCP config, registered app và bundled skills. Legal metadata, public access tới legal pages
và assets còn phải chốt.**

### Cấu trúc mục tiêu

```text
plugins/document-first/
├── .codex-plugin/
│   └── plugin.json
├── .app.json
├── .mcp.json
├── skills/
│   ├── implement-story/
│   │   └── SKILL.md
│   ├── review-impact/
│   │   └── SKILL.md
│   └── verify-business-rules/
│       └── SKILL.md
├── assets/
│   ├── icon.png
│   ├── logo.png
│   ├── logo-dark.png
│   └── screenshots/
└── README.md
```

### Metadata cần bổ sung

- Publisher name và URL. ✅
- Publisher email. ✅
- Homepage. ✅
- Repository của thin plugin.
- License.
- Website URL. ✅
- Privacy Policy URL. ✅ (còn phải mở truy cập public)
- Terms of Service URL. ✅ (còn phải mở truy cập public)
- Brand color. ✅
- Composer icon. ✅
- Logo. ✅
- Screenshots.
- Starter prompts phản ánh đúng workflow thực tế.
- Manifest trỏ `skills` tới `./skills/`.

### Không được đóng gói

- Source code backend.
- Business-rule implementation.
- Internal prompts.
- Proprietary workflow hoặc business rule trong `SKILL.md`.
- Database schema nội bộ.
- Secret, access token hoặc API key.
- File cấu hình môi trường production.

### Definition of Done

- Plugin validator pass.
- Không có placeholder hoặc đường dẫn hỏng.
- Package không chứa proprietary logic hay secret.
- Metadata đủ cho Plugins Directory.

## 10. Phase 8 — Tạo marketplace thử nghiệm

**Trạng thái: Hoàn tất cho local development — marketplace đã tạo/đăng ký, plugin `0.4.0` đã cài,
enable, reinstall và xác thực trong thread mới. Marketplace local không phải kênh public release.**

### File mục tiêu

```text
.agents/plugins/marketplace.json
```

### Marketplace entry dự kiến

```json
{
  "name": "document-first-local",
  "interface": {
    "displayName": "Document First Local"
  },
  "plugins": [
    {
      "name": "document-first",
      "source": {
        "source": "local",
        "path": "./plugins/document-first"
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

### Công việc

- Add marketplace vào Codex khi dùng marketplace ngoài vị trí mặc định.
- Refresh hoặc restart Codex/ChatGPT desktop app.
- Cài plugin từ Plugins Directory.
- Kiểm tra enable, disable, uninstall và reinstall.
- Kiểm tra authentication sau khi cài.

### Definition of Done

- Plugin xuất hiện trong Plugins Directory.
- Có thể cài đặt mà không phải chỉnh MCP config thủ công.
- Plugin sử dụng đúng version/package đã chọn.

## 11. Phase 9 — End-to-end evaluation

**Trạng thái: Hoàn tất production safety gate cho contract approved-only `2026-08-03`. Health,
readiness, OAuth discovery, login PKCE, plugin loading, `list_projects` và negative search trên cả
4 project đều pass; regression local đạt 205 pass/5 skip. Luồng GitHub repository mismatch đã bị
loại khỏi bundled skills theo quyết định sản phẩm mới. Evidence lần trước nằm tại
`artifacts/e2e/20260803T1029Z/`; kết quả mới nhất được giữ trong backend repository tại
`results/api-integration-tester.json`.**

### Chính sách cách ly

- Chỉ dùng tài liệu Approved của project người dùng có quyền truy cập.
- Không dùng dữ liệu production không liên quan để sửa repository hiện tại; E2E positive dùng một
  Story Approved tổng hợp có yêu cầu an toàn, nhỏ và có thể xác minh độc lập.
- Không cần project/repository mapping hoặc GitHub commit. Workspace đang mở do người dùng chọn là
  đích triển khai.
- `dry-run` vẫn không được sửa file hay chạy lệnh làm đổi trạng thái.

### Smoke test an toàn

```text
Dry-run only. Không sửa file và không chạy lệnh làm đổi trạng thái. Hãy liệt kê các project
Document First tôi có quyền truy cập, sau đó dừng lại.
```

```text
Dry-run only. Hãy tìm và đọc một Story đã được phê duyệt, báo `approvalId`, `contentHash` và các
tài liệu liên quan; không sửa code.
```

### Prompt tích cực

```text
Implement STORY-001.
```

```text
Thêm luồng đăng nhập OTP theo yêu cầu sản phẩm.
```

### Prompt tiêu cực

```text
Sửa typo trong README.
```

### Tình huống cần kiểm tra

- Story hợp lệ và đã Approved nhưng chưa từng release.
- Story Draft hoặc InReview.
- Business Rule chưa Approved.
- User không thuộc project.
- User vừa bị thu hồi quyền.
- Token hết hạn hoặc bị revoke.
- Semantic search không hoạt động.
- Related document bị dangling.
- Response vượt giới hạn.
- Tenant isolation.
- Prompt không liên quan không được kích hoạt tool không cần thiết.

### Definition of Done

- Prompt liên quan chọn đúng tool.
- Context chứa đủ Story, Business Rule, TDD và test liên quan.
- Không có Draft/InReview/Archived data trong output.
- Unsupported prompt không kích hoạt plugin sai.
- Kết quả đủ để agent implement đúng acceptance criteria.

## 12. Phase 10 — Hard gate tùy chọn

**Trạng thái: Chưa bắt đầu.**

MCP không thể tuyệt đối bắt Codex gọi tool trước khi chỉnh sửa source code cục bộ. Nếu cần
enforcement thực sự mà vẫn giữ logic bí mật, sử dụng server-issued receipt và CI gate.

### Thiết kế đề xuất

1. `prepare_story_context` trả một `context_receipt` đã ký.
2. Backend lưu receipt theo user, tenant, story, approval ID và content hash.
3. Pull request hoặc CI gọi Document First API để xác nhận receipt.
4. CI kiểm tra Story và Business Rule Approved bắt buộc đã được đọc.
5. Receipt hết hạn hoặc content hash không còn Approved làm CI fail.

### Definition of Done

- Không có receipt hợp lệ thì release gate thất bại.
- Receipt không thể được dùng lại cho tenant/story khác.
- Logic xác nhận nằm trên server, không nằm trong plugin phía khách hàng.

## 13. Phase 11 — Phân phối

**Trạng thái: Đang thực hiện — bỏ qua private npm và chuyển thẳng sang public submission theo
[`plugin-public-submission.md`](./plugin-public-submission.md).**

### Public release trực tiếp

- Hoàn thiện legal pages, support, publisher metadata và assets.
- Tạo reviewer account cùng project fixture không chứa dữ liệu khách hàng.
- Tạo draft loại **With MCP** bằng production MCP URL loại **Universal**.
- Deploy domain challenge token do portal cấp và chạy **Scan Tools**.
- Upload ba bundled skills, starter prompts, năm positive test và ba negative test.
- Chuẩn bị version `1.0.0`, submit review và phát hành sau khi được duyệt.

## 14. Tách repository

```text
document-first-be                 # Private
├── MCP server
├── OAuth
├── business logic
└── database

document-first-plugin             # Repo thin plugin riêng, có thể private/public
├── .agents/plugins/marketplace.json
├── plugins/document-first/
│   ├── plugin manifest
│   ├── bundled skills công khai
│   ├── MCP/app connection
│   └── README/package metadata
└── documentation/
    ├── plugin-plan.md
    └── plugin-public-submission.md
```

## 15. Thứ tự triển khai đề xuất

1. Xây dựng approved-only search và content-hash gate. ✅
2. Ổn định tool schema và MCP annotations.
3. Xây dựng và validate bundled skills.
4. Hoàn thiện OAuth 2.1. ✅
5. Deploy production MCP. ✅
6. Đăng ký MCP connection. ✅
7. Hoàn thiện manifest, legal metadata và assets. (manifest/app ✅; legal/assets còn lại)
8. Tạo marketplace thử nghiệm. ✅
9. Deploy contract `2026-08-03`, cài lại và test plugin end-to-end. ✅
10. Deploy legal/support pages. ✅
11. Tạo public submission, hoàn thiện reviewer fixture, xác minh domain và Scan Tools. ← bước hiện tại
12. Submit review; thêm context receipt/CI gate sau nếu cần hard enforcement.

## 16. Ước lượng

Public submission preparation dự kiến cần khoảng 3–7 ngày làm việc, chưa bao gồm:

- thời gian chuẩn bị privacy policy và terms;
- thiết kế branding/assets;
- thời gian review khi submit public plugin;
- thay đổi hạ tầng identity nếu hệ thống hiện tại chưa hỗ trợ OAuth 2.1.

## 17. Tài liệu tham khảo

- [Plugin architecture](https://developers.openai.com/plugins/concepts/plugins)
- [Package your plugin](https://developers.openai.com/plugins/build/plugins)
- [Build an MCP server](https://developers.openai.com/plugins/build/mcp-server)
- [Connect and test your plugin](https://developers.openai.com/plugins/deploy/connect-chatgpt)
- [Submit plugins](https://developers.openai.com/plugins/deploy/submission)
