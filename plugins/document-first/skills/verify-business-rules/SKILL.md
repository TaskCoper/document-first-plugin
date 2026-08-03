---
name: verify-business-rules
description: Xác minh implementation, pull request hoặc test suite theo Business Rule và contract đã được phê duyệt trong Document First. Dùng khi người dùng yêu cầu kiểm tra tính đúng nghiệp vụ, acceptance criteria, validation, trạng thái, lỗi, authorization hoặc regression của một feature trước khi merge/release.
---

# Xác minh Business Rule

## Lấy nguồn chuẩn

1. Xác định `projectId`; gọi `document_first_list_projects` nếu cần.
2. Gọi `document_first_prepare_story_context` khi có Story. Nếu không có Story, dùng
   `document_first_search` để tìm Business Rule/TDD/test rồi gọi `document_first_fetch` với
   `contentHash` cho từng tài liệu cần kiểm.
3. Chỉ chấp nhận nội dung Approved và ghi lại `documentKey`, `approvalId`, `contentHash`.
4. Dừng với `Blocked` nếu rule bắt buộc chưa được phê duyệt, không có quyền hoặc reference quan
   trọng chưa resolve. Không suy đoán nội dung bí mật nằm ngoài phản hồi MCP.
5. Không yêu cầu Release, GitHub commit hoặc liên kết GitHub repository.

## Kiểm chứng

- Chuyển từng rule thành một mệnh đề có thể kiểm: điều kiện, hành động/kết quả và ngoại lệ.
- Tìm đường code thực thi rule, bao gồm validation, persistence, transaction, authorization,
  background processing và error response.
- Đối chiếu test hiện có với nhánh thành công, biên và lỗi; chạy test thực tế khi môi trường cho phép.
- Phân biệt rõ code không tuân thủ, test thiếu và bằng chứng không đủ.

## Báo cáo

Lập ma trận ngắn với các cột: rule/content hash, code path, test evidence và kết luận
`Pass`/`Fail`/`Blocked`. Với `Fail`, mô tả hành vi quan sát được và hành vi tài liệu yêu cầu.
Không tự sửa code trừ khi người dùng đồng thời yêu cầu triển khai bản sửa.
