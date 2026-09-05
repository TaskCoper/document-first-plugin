---
name: write-acceptance-criteria
description: Viết hoặc rà soát Acceptance Criteria dạng Given/When/Then cho một User Story đã có trong Document First, bám theo Main Flow, Alternative Flow, Exception Flow và Business Rule hiện tại. Dùng khi người dùng nói AC còn thiếu, mơ hồ, không kiểm thử được, hoặc cần bổ sung tiêu chí trước khi implement; không dùng để soạn mới toàn bộ Story.
---

# Viết Acceptance Criteria

## Lấy context trong project

1. Xác định `projectId`; gọi `document_first_list_projects` nếu cần.
2. Có mã tài liệu thì dùng `document_first_fetch`; dùng `document_first_get_story` để đọc Story có cấu trúc, gồm flow, AC và reference. Dùng `document_first_search` khi cần tìm tài liệu liên quan.
3. Duyệt Business Rule bằng `document_first_list_rules` với `limit`/`offset`, rồi đọc chi tiết bằng `documentKeys` (tối đa 20 mã/lần). Khi kiểm tra xung đột hoặc traceability, duyệt đủ các trang; search không chứng minh được danh sách đầy đủ.
4. Cần thêm TDD, reference và tài liệu liên quan cho Story thì dùng `document_first_prepare_story_context` (contract `2026-09-05`, `evidence.readDocuments`). Context có giới hạn; đọc bổ sung tài liệu cần thiết.
5. Mọi thành viên project được đọc tài liệu ở Draft, InReview, Approved và đã lưu trữ. Không yêu cầu phê duyệt hoặc Release để dùng nội dung làm căn cứ. Ghi lại trạng thái thực tế, không mô tả tài liệu chưa duyệt là đã duyệt.
6. Truy vết bằng `documentKey#contentHash`; `approvalId` chỉ ghi khi có. `missingKeys`/`unresolvedReferences` là phần chưa lấy được, không tự suy đoán nội dung. `resolved` chỉ phản ánh liên kết đã tìm thấy tài liệu đích.

Đọc AC hiện có và từng nhánh flow bằng `get_story` trước khi bổ sung; đối chiếu rule chi phối để tránh AC trùng hoặc mâu thuẫn.

## Suy ra tiêu chí

Duyệt có hệ thống, không viết theo cảm tính:

1. Mỗi phần tử `flows.main` tạo ra kết quả quan sát được → ít nhất một AC.
2. Mỗi phần tử `flows.alternative` → một AC cho đường rẽ đó, tham chiếu bằng `code` của nhánh.
3. Mỗi phần tử `flows.exception` → một AC cho thông báo lỗi, mã lỗi và trạng thái sau lỗi.
4. Mỗi Business Rule đọc được ở bước trên → một AC cho điều kiện `When`/`Then` và một AC cho từng
   trường hợp trong `Except`.
5. Mỗi phần tử `nonFunctional` có ngưỡng đo được → một AC.
6. Mỗi ràng buộc về authorization, idempotency hoặc trạng thái dữ liệu → một AC riêng.

Không tạo AC cho phần nằm trong `outOfScope`.

Reference có `resolved: false` là liên kết chưa resolve được tài liệu đích: ghi nhận thành khoảng trống, không
tự suy ra nội dung rule rồi viết AC dựa trên đó.

## Định dạng

Giữ đúng cấu trúc mà Document First parse:

```markdown
#### AC-001

- **Given**: <trạng thái ban đầu cụ thể>
- **When**: <một hành động duy nhất>
- **Then**: <kết quả kiểm chứng được>
- **And**: <ràng buộc bổ sung, lặp lại khi cần>
```

- Đánh số liên tục `AC-001`, `AC-002`; khi bổ sung vào Story đã có, lấy `code` lớn nhất trong
  `acceptanceCriteria` rồi tiếp tục từ đó, không đánh lại từ đầu.
- `When` chỉ chứa một hành động. Nhiều hành động thì tách thành nhiều AC.
- `Then` phải nêu giá trị, trạng thái, mã lỗi hoặc ngưỡng cụ thể — không dùng "đúng như mong đợi",
  "hoạt động tốt", "hiển thị phù hợp".
- Dùng `And` cho ràng buộc phụ thuộc cùng một hành động, không dùng để gộp kịch bản khác.

## Kiểm tra chất lượng

Trước khi trả kết quả, tự soát từng AC:

- có thể viết thành một test case cụ thể không;
- có mâu thuẫn với AC khác hoặc với Business Rule hiện tại không;
- có bước nào của Flow chưa được AC nào phủ không;
- có AC nào mô tả cách hiện thực hóa thay vì hành vi quan sát được không.

## Bàn giao

Trả các AC trong một code block Markdown sẵn sàng dán vào Story, kèm:

- ma trận phủ ngắn: bước Flow hoặc Business Rule → AC tương ứng;
- danh sách khoảng trống chưa phủ được và lý do;
- `documentKey#contentHash` cùng `approvalId` nếu có với tài liệu hiện tại; hoặc `draftContentHash` và
  `unstable=true` với Story Draft/InReview.

Không sửa source code trong skill này.

## Khi được yêu cầu lưu vào Document First

Nếu người dùng yêu cầu tạo hoặc cập nhật tài liệu trong project, thực hiện ghi qua MCP theo
[manage-documents](../manage-documents/SKILL.md), rồi trả mã tài liệu và kết quả đã lưu.
Chỉ soạn để xem hoặc dry-run thì dùng định dạng bàn giao ở trên.
