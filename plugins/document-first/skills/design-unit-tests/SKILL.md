---
name: design-unit-tests
description: Thiết kế đặc tả unit test ở mức hàm, service và validator theo nhánh điều kiện, giá trị biên và ngoại lệ của Business Rule đã phê duyệt trong Document First, có truy vết ngược về documentKey. Dùng khi người dùng cần bổ sung test cho một rule hoặc module cụ thể, hoặc phát hiện coverage thiếu nhánh; không dùng cho test end-to-end theo Story.
---

# Thiết kế Unit Test

## Lấy nguồn chuẩn

1. Xác định `projectId`; gọi `document_first_list_projects` nếu cần.
2. Khi đã biết rule cần phủ, đọc trọn bằng `document_first_list_rules` với `documentKeys` — một
   lần gọi cho nhiều rule, thay vì `document_first_fetch` từng cái.
3. Khi chưa biết rule nào chi phối module đang test, duyệt danh sách bằng
   `document_first_list_rules` rồi chọn theo `ruleName`, `category` và `summary`.
4. Khi bám theo một Story, `document_first_get_story` cho `references.rules` và
   `acceptanceCriteria`; `document_first_prepare_story_context` chỉ cần khi muốn cả TDD.
5. Khoá trong `missingKeys` là rule chưa phê duyệt hoặc không tồn tại — ghi `Blocked`, không tự
   suy ra kỳ vọng.
6. Chỉ dùng nội dung Approved làm nguồn kỳ vọng. Dừng với `Blocked` nếu rule bắt buộc chưa được
   phê duyệt hoặc thiếu quyền đọc.
7. Không yêu cầu Release, GitHub commit hoặc liên kết GitHub repository.

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
- Kỳ vọng lấy từ tài liệu đã duyệt, không lấy từ hành vi hiện tại của code.

## Bàn giao

Trả đặc tả test kèm:

- ma trận phủ: rule hoặc mệnh đề → mã ca tương ứng;
- nhánh chưa phủ được và lý do;
- test đã tồn tại trong repository có thể tái sử dụng hoặc đang mâu thuẫn với rule đã duyệt;
- `documentKey#contentHash` và `approvalId` đã dùng.

Chỉ viết code test khi người dùng yêu cầu rõ; mặc định skill này dừng ở đặc tả.
