---
name: design-unit-tests
description: Thiết kế đặc tả unit test ở mức hàm, service và validator theo nhánh điều kiện, giá trị biên và ngoại lệ của Business Rule hiện tại trong Document First, có truy vết ngược về documentKey. Dùng khi người dùng cần bổ sung test cho một rule hoặc module cụ thể, hoặc phát hiện coverage thiếu nhánh; không dùng cho test end-to-end theo Story.
---

# Thiết kế Unit Test

## Lấy context trong project

1. Xác định `projectId`; gọi `document_first_list_projects` nếu cần.
2. Có mã tài liệu thì dùng `document_first_fetch`; dùng `document_first_get_story` để đọc Story có cấu trúc, gồm flow, AC và reference. Dùng `document_first_search` khi cần tìm tài liệu liên quan.
3. Duyệt Business Rule bằng `document_first_list_rules` với `limit`/`offset`, rồi đọc chi tiết bằng `documentKeys` (tối đa 20 mã/lần). Khi kiểm tra xung đột hoặc traceability, duyệt đủ các trang; search không chứng minh được danh sách đầy đủ.
4. Cần thêm TDD, reference và tài liệu liên quan cho Story thì dùng `document_first_prepare_story_context` (contract `2026-09-05`, `evidence.readDocuments`). Context có giới hạn; đọc bổ sung tài liệu cần thiết.
5. Mọi thành viên project được đọc tài liệu ở Draft, InReview, Approved và đã lưu trữ. Không yêu cầu phê duyệt hoặc Release để dùng nội dung làm căn cứ. Ghi lại trạng thái thực tế, không mô tả tài liệu chưa duyệt là đã duyệt.
6. Truy vết bằng `documentKey#contentHash`; `approvalId` chỉ ghi khi có. `missingKeys`/`unresolvedReferences` là phần chưa lấy được, không tự suy đoán nội dung. `resolved` chỉ phản ánh liên kết đã tìm thấy tài liệu đích.

Lấy điều kiện, kết quả và ngoại lệ từ rule; đọc TDD để xác định hàm/service/validator và mã lỗi cần kiểm.

## Xác định đơn vị cần test

- Đọc hướng dẫn của repository đang mở để nắm test framework, cách đặt tên và cách mock đang dùng.
- Tìm hàm, service, validator hoặc mapper thực thi rule; unit test bám vào đơn vị đó, không đi qua
  HTTP, database thật hay hệ thống ngoài.
- Ghi lại phụ thuộc cần thay thế bằng test double và lý do.

## Sinh ca kiểm thử

Với mỗi rule hoặc mệnh đề logic, duyệt đủ bốn nhóm, không dừng ở nhánh thành công:

1. **Nhánh đúng** — mỗi tổ hợp điều kiện trong `When` cho ra kết quả `Then`.
2. **Biên** — giá trị ngay tại ngưỡng, ngay dưới và ngay trên ngưỡng; chuỗi rỗng, `null`, tập
   rỗng, số 0, số âm, tràn kiểu, ngày ranh giới múi giờ.
3. **Ngoại lệ** — mọi trường hợp liệt kê trong `Except`, mỗi trường hợp một ca riêng.
4. **Lỗi** — đầu vào không hợp lệ, phụ thuộc ném exception, timeout; kiểm tra đúng loại lỗi và
   đúng mã lỗi trong registry của TDD.

Bổ sung ca cho làm tròn, thứ tự áp dụng khi nhiều rule cùng tác động, và tính idempotent nếu đơn
vị đó xử lý sự kiện lặp.

## Định dạng đặc tả

Mỗi ca trình bày gồm: mã ca, đơn vị được test, đầu vào cụ thể, kỳ vọng cụ thể, test double cần
thiết và `documentKey#contentHash` làm căn cứ. Dùng giá trị thật, không dùng "dữ liệu hợp lệ".

Nhóm các ca theo đơn vị được test, và trong mỗi nhóm theo thứ tự nhánh đúng → biên → ngoại lệ →
lỗi.

## Kiểm tra trước khi trả

- Mỗi mệnh đề trong `When`, `Then`, `Except` của rule đều có ít nhất một ca.
- Không có hai ca kiểm cùng một điều kiện với dữ liệu chỉ khác nhau về hình thức.
- Không có ca nào kiểm chi tiết hiện thực nội bộ thay vì hành vi quan sát được.
- Kỳ vọng lấy từ tài liệu hiện tại, không lấy từ hành vi hiện tại của code.

## Bàn giao

Trả đặc tả test kèm:

- ma trận phủ: rule hoặc mệnh đề → mã ca tương ứng;
- nhánh chưa phủ được và lý do;
- test đã tồn tại trong repository có thể tái sử dụng hoặc đang mâu thuẫn với rule hiện tại;
- `documentKey#contentHash` và `approvalId` nếu có, của nguồn đã dùng.

Chỉ viết code test khi người dùng yêu cầu rõ; mặc định skill này dừng ở đặc tả.

## Khi được yêu cầu lưu vào Document First

Nếu người dùng yêu cầu tạo hoặc cập nhật tài liệu trong project, thực hiện ghi qua MCP theo
[manage-documents](../manage-documents/SKILL.md), rồi trả mã tài liệu và kết quả đã lưu.
Chỉ soạn để xem hoặc dry-run thì dùng định dạng bàn giao ở trên.
