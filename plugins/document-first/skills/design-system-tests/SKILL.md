---
name: design-system-tests
description: Thiết kế system test và end-to-end test đi hết một User Story đã phê duyệt trong Document First, phủ Main Flow, Alternative Flow, Exception Flow và ràng buộc Non-Functional, mỗi ca truy vết về Acceptance Criteria. Dùng khi cần bộ test nghiệm thu cho một feature trước khi release; không dùng cho test ở mức hàm hay validator.
---

# Thiết kế System Test

## Cổng context bắt buộc

1. Xác định `projectId` và `storyKey`; gọi `document_first_list_projects` nếu cần.
2. Gọi `document_first_get_story` để lấy `acceptanceCriteria` đã tách Given/When/Then/And, ba
   nhóm `flows`, `nonFunctional` và `outOfScope` — đây chính là danh sách cần phủ, không phải tự
   parse từ Markdown.
3. Đọc Business Rule bằng `document_first_list_rules` với `documentKeys` từ `references.rules`.
4. Lấy `Error Codes` và `State Diagram` của TDD qua `document_first_search` rồi
   `document_first_fetch` với `contentHash`. Dùng `document_first_prepare_story_context` khi muốn
   gom cả cụm tài liệu liên quan trong một lần.
5. Chỉ dùng nội dung Approved làm nguồn kỳ vọng. Dừng với `Blocked` nếu Story chưa được phê duyệt
   hoặc thiếu quyền đọc.
6. Không yêu cầu Release, GitHub commit hoặc liên kết GitHub repository.

## Phạm vi

System test chạy qua ranh giới thật của hệ thống: HTTP endpoint, database, hàng đợi, job nền và
stub cho hệ thống bên thứ ba. Test ở mức hàm hoặc validator thuộc `design-unit-tests`.

Xác định trước khi thiết kế:

- điểm vào của luồng và dữ liệu khởi tạo cần có;
- hệ thống ngoài nào phải stub, và stub phải mô phỏng hành vi nào trong mục `Quirks` của TDD;
- trạng thái dữ liệu cần kiểm sau khi luồng kết thúc.

## Sinh ca kiểm thử

1. **Happy path** — đi hết `flows.main`, kiểm cả phản hồi API lẫn trạng thái dữ liệu cuối cùng.
2. **Alternative Flow** — mỗi phần tử `flows.alternative` một ca, kiểm đúng điểm rẽ và điểm quay lại.
3. **Exception Flow** — mỗi phần tử `flows.exception` một ca, kiểm HTTP status, mã lỗi trong
   registry của TDD, thông báo cho người dùng và trạng thái dữ liệu sau lỗi.
4. **Acceptance Criteria** — mỗi phần tử `acceptanceCriteria` ánh xạ tới ít nhất một ca; dùng
   thẳng `given`, `when`, `then`, `and` làm điều kiện, thao tác và kỳ vọng.
5. **Non-Functional** — mỗi phần tử `nonFunctional` có ngưỡng đo được thành một ca riêng: thời
   gian phản hồi, authorization, audit log, tương thích trình duyệt, accessibility.
6. **Chuyển trạng thái** — kiểm các transition hợp lệ và chặn transition không hợp lệ theo State
   Diagram của TDD.

Bổ sung ca cho tính idempotent và chạy lặp nếu luồng có webhook, retry hoặc job nền.

## Định dạng đặc tả

Mỗi ca gồm: mã ca, `code` của acceptance criteria hoặc nhánh flow tương ứng, tiền điều kiện dữ liệu, các bước thao tác,
kỳ vọng quan sát được, dữ liệu cần dọn sau khi chạy và `documentKey#contentHash` làm căn cứ.

Nêu rõ ca nào tự động hóa được và ca nào phải kiểm thủ công, kèm lý do.

## Kiểm tra trước khi trả

- Mọi phần tử `acceptanceCriteria` đều được phủ.
- Mọi phần tử `flows.alternative` và `flows.exception` đều có ca tương ứng.
- Không có ca nào kiểm phần nằm trong `outOfScope`.
- Kỳ vọng lấy từ tài liệu đã duyệt, không lấy từ hành vi hiện tại của hệ thống.
- Các ca độc lập nhau về dữ liệu, chạy được theo bất kỳ thứ tự nào.

## Bàn giao

Trả đặc tả test kèm:

- ma trận phủ: `code` của acceptance criteria và của nhánh flow → mã ca tương ứng;
- phần chưa phủ được và lý do;
- test đã tồn tại trong repository có thể tái sử dụng;
- `documentKey#contentHash` và `approvalId` đã dùng.

Chỉ viết code test khi người dùng yêu cầu rõ; mặc định skill này dừng ở đặc tả.
