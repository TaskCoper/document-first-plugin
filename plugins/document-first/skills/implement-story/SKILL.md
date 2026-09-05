---
name: implement-story
description: Triển khai hoặc hoàn thiện User Story bằng tài liệu hiện tại trong project Document First, gồm yêu cầu, Business Rule, TDD và test liên quan.
---

# Triển khai User Story

## Lấy context

1. Xác định `projectId` và `storyKey`; gọi `document_first_list_projects` nếu chưa biết project, không đoán UUID.
2. Gọi `document_first_prepare_story_context` trước khi sửa code. Contract hiện tại là `2026-09-05`; phản hồi có `evidence.searched`, `evidence.readDocuments` và Story trong `documents`.
3. Đọc Markdown theo thứ tự `user_story`, `referenced_by_story`, `search_match`, `semantic_related`. Dùng `document_first_fetch` ghim `contentHash` khi cần đọc lại; dùng `document_first_get_story` để đối chiếu flow và AC, `document_first_list_rules` với `documentKeys` để đọc nhiều rule.
4. Mọi thành viên project được đọc và dùng tài liệu Draft, InReview, Approved hoặc đã lưu trữ. Không yêu cầu phê duyệt, Release hay GitHub repository. `approvalState` và `isArchived` chỉ cung cấp ngữ cảnh về tài liệu.
5. Nếu thiếu quyền, Story không tồn tại hoặc tool lỗi, báo rõ phần context chưa lấy được. Không bịa nội dung cho `unresolvedReferences` hoặc `missingKeys`.

## Triển khai

- Đọc hướng dẫn repository và ánh xạ yêu cầu sang module, API, dữ liệu và test liên quan.
- Thực hiện thay đổi theo phạm vi người dùng yêu cầu và convention hiện có; kiểm thử các AC, nhánh lỗi và rule bị tác động.
- Khi tài liệu còn thiếu hoặc mâu thuẫn, nêu rõ điểm cần làm rõ; trạng thái chưa Approved tự nó không phải lý do chặn triển khai.
- Với `dry-run`, chỉ đọc và báo cáo, không sửa file hoặc trạng thái.

## Bàn giao

Nêu file đã đổi, kiểm thử, yêu cầu đã đáp ứng và phần chưa đủ bằng chứng. Truy vết bằng `documentKey#contentHash` và trạng thái thực tế; bổ sung `approvalId` khi phản hồi có giá trị này, không bắt buộc nó.
