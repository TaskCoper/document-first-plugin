---
name: write-acceptance-criteria
description: Viết hoặc rà soát Acceptance Criteria dạng Given/When/Then cho một User Story đã có trong Document First, bám theo Main Flow, Alternative Flow, Exception Flow và Business Rule đã phê duyệt. Dùng khi người dùng nói AC còn thiếu, mơ hồ, không kiểm thử được, hoặc cần bổ sung tiêu chí trước khi implement; không dùng để soạn mới toàn bộ Story.
---

# Viết Acceptance Criteria

## Lấy nguồn chuẩn

1. Xác định `projectId` và `storyKey`; gọi `document_first_list_projects` nếu cần.
2. Gọi `document_first_fetch_draft` trước để đọc nội dung hiện tại khi Story còn Draft/InReview.
   Nếu tool trả `NOT_FOUND` vì Story đã Approved, gọi `document_first_get_story`; tool này trả sẵn
   `flows.main`, `flows.alternative`, `flows.exception`, `acceptanceCriteria` và
   `references.rules` dưới dạng field có cấu trúc.
3. Với Draft/InReview, đọc các mục Flow, Acceptance Criteria và References từ Markdown được trả
   về; luôn ghi rõ nội dung `unstable=true` và có thể thay đổi trước khi phê duyệt.
4. Đọc Business Rule chi phối bằng `document_first_list_rules` với `documentKeys` lấy từ
   `references.rules`; một lần gọi cho tất cả rule thay vì fetch từng cái.
5. Nếu chưa biết `storyKey`, dùng `document_first_review_queue` để tìm Story được giao review;
   `document_first_search` chỉ dùng để tìm nội dung Approved.
6. Review không yêu cầu Story phải Approved. Dừng với `Blocked` chỉ khi thiếu quyền hoặc không tìm
   thấy Story; không dùng Draft/InReview làm căn cứ triển khai source code.
7. Không yêu cầu Release, GitHub commit hoặc liên kết GitHub repository.

Chỉ dùng `document_first_prepare_story_context` khi cần thêm cả TDD và tài liệu semantic-related;
để viết AC thì `document_first_get_story` gọn và chính xác hơn.

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

Reference có `resolved: false` là tài liệu đích chưa tồn tại: ghi nhận thành khoảng trống, không
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
- có mâu thuẫn với AC khác hoặc với Business Rule đã duyệt không;
- có bước nào của Flow chưa được AC nào phủ không;
- có AC nào mô tả cách hiện thực hóa thay vì hành vi quan sát được không.

## Bàn giao

Trả các AC trong một code block Markdown sẵn sàng dán vào Story, kèm:

- ma trận phủ ngắn: bước Flow hoặc Business Rule → AC tương ứng;
- danh sách khoảng trống chưa phủ được và lý do;
- `documentKey#contentHash` cùng `approvalId` với tài liệu Approved; hoặc `draftContentHash` và
  `unstable=true` với Story Draft/InReview.

Không sửa source code trong skill này.
