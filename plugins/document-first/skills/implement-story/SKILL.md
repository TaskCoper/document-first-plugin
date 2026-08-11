---
name: implement-story
description: Triển khai một User Story bằng context đã được phê duyệt trong Document First. Dùng khi người dùng yêu cầu implement, sửa hoặc hoàn thiện một Story/feature có story key, project Document First, Acceptance Criteria hoặc Business Rule liên quan; bắt buộc lấy approved-only context trước khi thay đổi source code.
---

# Triển khai User Story

## Cổng context bắt buộc

1. Xác định `projectId` và `storyKey`. Nếu chưa biết project, gọi
   `document_first_list_projects`; không đoán UUID.
2. Trước khi sửa source code, gọi `document_first_prepare_story_context` với Story và mô tả
   phạm vi người dùng yêu cầu.
3. Xác nhận phản hồi có `evidence.searched = true`,
   `evidence.readApprovedDocuments = true` và ít nhất tài liệu Story đã được phê duyệt.
4. Dừng và báo rõ lý do nếu không có quyền, Story chưa được phê duyệt, tool lỗi hoặc contract
   version không được hỗ trợ. Không thay thế bằng Draft, InReview, kiến thức nhớ lại hoặc business
   rule tự suy đoán.

## Đọc context

- Đọc toàn bộ Markdown trong context package, ưu tiên theo `reason`: `user_story`,
  `referenced_by_story`, `search_match`, rồi `semantic_related`.
- Gọi `document_first_fetch` với `documentKey` và `contentHash` đã nhận khi cần đọc lại một tài
  liệu hoặc khi context package không chứa đủ nội dung để quyết định.
- Cần soát lại Acceptance Criteria hoặc nhánh flow theo từng mục thì gọi `document_first_get_story`:
  tool trả `acceptanceCriteria` và `flows` đã tách sẵn, chính xác hơn tự đọc lại Markdown.
- Cần đọc nhiều Business Rule cùng lúc thì gọi `document_first_list_rules` với `documentKeys` thay
  vì fetch từng rule.
- Ghi nhận `documentKey`, `approvalId`, `contentHash`, Acceptance Criteria, Business Rule,
  API contract và test document liên quan.
- Xem `unresolvedReferences` là khoảng trống cần nêu ra; không tự tạo nội dung cho reference đó.
- Không yêu cầu tài liệu phải được Release, commit lên GitHub hoặc ánh xạ với GitHub repository.
  Repository đang mở là workspace người dùng đã chọn để triển khai.

## Triển khai

1. Đọc hướng dẫn của repository đang mở và xác định module hiện thực hóa yêu cầu đã duyệt.
2. Lập ánh xạ ngắn từ từng yêu cầu đã phê duyệt sang file/module/test chịu trách nhiệm.
3. Thực hiện thay đổi nhỏ nhất đáp ứng Story, giữ kiến trúc và convention của repository.
4. Thêm hoặc cập nhật test cho Acceptance Criteria, nhánh lỗi và Business Rule bị tác động.
5. Chạy kiểm thử phù hợp; không tuyên bố hoàn tất khi còn rule bắt buộc chưa được chứng minh.
6. Nếu người dùng yêu cầu `dry-run`, chỉ đọc context và source; không tạo/sửa/xoá file hoặc chạy
   lệnh làm đổi trạng thái.

## Bàn giao

Nêu các file đã đổi, test đã chạy và traceability theo `documentKey#contentHash` kèm `approvalId`.
Tách rõ:

- yêu cầu đã đáp ứng;
- yêu cầu chưa thể xác minh;
- reference chưa resolve hoặc tài liệu chưa được phê duyệt;
- giả định kỹ thuật không xuất phát từ Document First.
