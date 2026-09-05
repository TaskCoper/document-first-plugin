---
name: design-system-tests
description: Thiết kế system test và end-to-end test đi hết một User Story hiện tại trong Document First, phủ Main Flow, Alternative Flow, Exception Flow và ràng buộc Non-Functional, mỗi ca truy vết về Acceptance Criteria. Dùng khi cần bộ test nghiệm thu cho một feature trước khi release; không dùng cho test ở mức hàm hay validator.
---

# Thiết kế System Test

## Lấy context trong project

1. Xác định `projectId`; gọi `document_first_list_projects` nếu cần.
2. Có mã tài liệu thì dùng `document_first_fetch`; dùng `document_first_get_story` để đọc Story có cấu trúc, gồm flow, AC và reference. Dùng `document_first_search` khi cần tìm tài liệu liên quan.
3. Duyệt Business Rule bằng `document_first_list_rules` với `limit`/`offset`, rồi đọc chi tiết bằng `documentKeys` (tối đa 20 mã/lần). Khi kiểm tra xung đột hoặc traceability, duyệt đủ các trang; search không chứng minh được danh sách đầy đủ.
4. Cần thêm TDD, reference và tài liệu liên quan cho Story thì dùng `document_first_prepare_story_context` (contract `2026-09-05`, `evidence.readDocuments`). Context có giới hạn; đọc bổ sung tài liệu cần thiết.
5. Mọi thành viên project được đọc tài liệu ở Draft, InReview, Approved và đã lưu trữ. Không yêu cầu phê duyệt hoặc Release để dùng nội dung làm căn cứ. Ghi lại trạng thái thực tế, không mô tả tài liệu chưa duyệt là đã duyệt.
6. Truy vết bằng `documentKey#contentHash`; `approvalId` chỉ ghi khi có. `missingKeys`/`unresolvedReferences` là phần chưa lấy được, không tự suy đoán nội dung. `resolved` chỉ phản ánh liên kết đã tìm thấy tài liệu đích.

Dùng `flows.main`, `flows.alternative`, `flows.exception`, `acceptanceCriteria` và `nonFunctional` của Story để thiết kế hành trình nghiệm thu; đối chiếu contract trong TDD.

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
- Kỳ vọng lấy từ tài liệu hiện tại, không lấy từ hành vi hiện tại của hệ thống.
- Các ca độc lập nhau về dữ liệu, chạy được theo bất kỳ thứ tự nào.

## Bàn giao

Trả đặc tả test kèm:

- ma trận phủ: `code` của acceptance criteria và của nhánh flow → mã ca tương ứng;
- phần chưa phủ được và lý do;
- test đã tồn tại trong repository có thể tái sử dụng;
- `documentKey#contentHash` và `approvalId` nếu có, của nguồn đã dùng.

Chỉ viết code test khi người dùng yêu cầu rõ; mặc định skill này dừng ở đặc tả.

## Khi được yêu cầu lưu vào Document First

Nếu người dùng yêu cầu tạo hoặc cập nhật tài liệu trong project, thực hiện ghi qua MCP theo
[manage-documents](../manage-documents/SKILL.md), rồi trả mã tài liệu và kết quả đã lưu.
Chỉ soạn để xem hoặc dry-run thì dùng định dạng bàn giao ở trên.
