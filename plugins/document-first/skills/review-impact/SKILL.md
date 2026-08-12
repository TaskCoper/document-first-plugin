---
name: review-impact
description: Phân tích tác động của một Story, tài liệu hoặc thay đổi code trong Document First. Khi review tài liệu thì đọc Draft/InReview hiện tại nếu được phép; khi phân tích để triển khai thì chỉ dùng nội dung đã phê duyệt. Dùng trước khi sửa kiến trúc/API/database, khi review pull request, điều tra regression hoặc cần xác định Business Rule, TDD, test và module liên quan có thể bị ảnh hưởng.
---

# Phân tích tác động

## Thu thập bằng chứng

1. Xác định project bằng `document_first_list_projects` nếu người dùng chưa cung cấp `projectId`.
2. Nếu người dùng yêu cầu review một `storyKey` hoặc `documentKey` cụ thể, gọi
   `document_first_fetch_draft` trước để đọc Draft/InReview hiện tại. Nếu tool trả `NOT_FOUND` vì
   tài liệu đã Approved, dùng `document_first_get_story` hoặc `document_first_fetch`.
3. Nếu người dùng yêu cầu phân tích để implement một User Story, gọi
   `document_first_prepare_story_context` và bắt buộc Story đã Approved. Nếu chỉ có mô tả hoặc mã
   tài liệu khác, gọi `document_first_search`, sau đó `document_first_fetch` với `contentHash` cho
   các kết quả liên quan.
4. Khi thay đổi đụng tới quy tắc nghiệp vụ, duyệt toàn bộ Business Rule bằng
   `document_first_list_rules` rồi đọc chi tiết bằng `documentKeys`. Search chỉ trả thứ khớp truy
   vấn, nên không đủ để kết luận "không rule nào bị ảnh hưởng".
5. Cần đối chiếu từng Acceptance Criteria hoặc nhánh flow của Story Approved thì dùng
   `document_first_get_story` để lấy bản đã tách cấu trúc. Với Draft/InReview, đọc Markdown từ
   `document_first_fetch_draft` và ghi rõ `unstable=true`.
6. Chỉ áp dụng cổng Approved khi mục đích là implement. Khi mục đích là review tài liệu, được dùng
   Draft/InReview nếu tool cho phép; dừng và báo `Blocked` nếu người dùng không có quyền đọc.
7. Ghi nhận `approvalId`, content hash, loại quan hệ và mọi `unresolvedReferences` hoặc
   `missingKeys`.
8. Không yêu cầu Release, GitHub commit hoặc liên kết GitHub repository.

## Đối chiếu repository

- Tìm controller/route, service, entity/migration, worker, client và test đang hiện thực hóa các
  contract đã đọc trong workspace người dùng đang mở.
- Theo dõi cả tác động trực tiếp và gián tiếp: caller, consumer, persisted state, authorization,
  error contract, background job và compatibility.
- Không sửa code khi người dùng chỉ yêu cầu review hoặc report.

## Kết quả review

Trả báo cáo ưu tiên theo mức độ rủi ro, mỗi mục gồm:

- bằng chứng `documentKey#contentHash` và `approvalId` với Approved; hoặc `draftContentHash` cùng
  cảnh báo `unstable=true` với Draft/InReview;
- code path hoặc file chịu tác động;
- hành vi có thể thay đổi;
- test cần chạy hoặc cần bổ sung;
- trạng thái `Confirmed`, `Likely` hoặc `Blocked`.

Không biến kết quả semantic-related thành quan hệ chắc chắn nếu tài liệu không nói vậy. Nêu rõ
phần nào là suy luận từ source code.
