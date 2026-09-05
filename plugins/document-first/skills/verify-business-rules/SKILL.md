---
name: verify-business-rules
description: Xác minh implementation, pull request hoặc test suite theo Business Rule, Acceptance Criteria và contract hiện tại trong project Document First.
---

# Xác minh Business Rule

## Lấy nguồn

1. Xác định project; gọi `document_first_list_projects` nếu cần.
2. Có Story thì dùng `document_first_get_story` lấy AC, flow và reference; đọc rule qua `document_first_list_rules` với `documentKeys`.
3. Chưa có Story thì duyệt rule bằng phân trang, chọn rule thuộc phạm vi và đọc chi tiết; dùng search/fetch cho TDD và test.
4. Chấp nhận tài liệu ở mọi trạng thái trong project. Trạng thái chưa Approved không chặn kiểm chứng; không yêu cầu Release hoặc GitHub repository.
5. `missingKeys` là khoá không tìm thấy hoặc sai loại. Ghi rõ phần thiếu quyền, tài liệu thiếu hoặc mâu thuẫn thay vì suy đoán nội dung.

## Kiểm chứng và báo cáo

- Chuyển rule thành điều kiện, kết quả và ngoại lệ có thể kiểm.
- Đối chiếu validation, persistence, transaction, authorization và các nhánh thực thi liên quan; chạy test phù hợp.
- Lập ma trận rule/content hash, code path, test evidence và `Pass`/`Fail`/`Blocked` khi thiếu bằng chứng.
- Truy vết `documentKey#contentHash` cùng trạng thái; `approvalId` là metadata tuỳ chọn.
- Không tự sửa code nếu người dùng chỉ yêu cầu kiểm tra.
