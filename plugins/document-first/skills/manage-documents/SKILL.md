---
name: manage-documents
description: Tạo, sửa hoặc xoá User Story, TDD, Business Rule, Unit Test và System Test trong project Document First qua MCP khi người dùng yêu cầu lưu thay đổi. Không áp dụng cho yêu cầu chỉ đọc hoặc review.
---

# Quản lý tài liệu trong project

Xác định project bằng `document_first_list_projects`. Nếu chưa rõ tài liệu đích, dùng
`document_first_search` để tìm; chỉ hỏi khi không thể xác định project hoặc tài liệu từ yêu cầu.
Tool ghi cần OAuth `documents:write` và vai trò Editor/Admin/Owner. Nếu thiếu quyền, báo rõ
lỗi backend; không coi file Markdown cục bộ là tài liệu đã được lưu lên hệ thống.

## Tạo

Gọi `document_first_create_document` với `projectId`, `document` và `changes` nếu đã có nội dung.
`document.docType` là `UserStory`, `Tdd`, `BusinessRule`, `UnitTest` hoặc `SystemTest`;
`document.title` là tiêu đề. Backend sinh mã; `keyPrefix` tuỳ chọn cho các nhóm test hoặc prefix riêng.
Tài liệu mới là Draft. Metadata loại-specific: Story có sprint/priority; BR có category/effectiveDate.

Ví dụ BR:

```json
{
  "projectId": "<UUID>",
  "document": {"docType": "BusinessRule", "title": "Hiệu lực OTP"},
  "changes": {"detail": {"ruleName": "Hiệu lực OTP", "statement": "OTP hết hiệu lực sau 5 phút."}}
}
```

## Sửa

Đọc cấu trúc mới nhất bằng `document_first_get_document`, lấy `changes` và `contentHash`, rồi gọi
`document_first_update_document` với `projectId`, `documentKey`, `expectedContentHash`, `changes`.

- Chỉ gửi section cần thay đổi. Section không gửi hoặc null được giữ nguyên.
- Section được gửi thay **toàn bộ** nội dung. Với `metadata`/`detail`, gửi đủ giá trị muốn giữ;
  field thiếu trong section được gửi bị xoá về null. Không suy đoán nội dung bị thiếu khi đọc.
- Các mảng `flows`, `acceptanceCriteria`, `diagrams`, `assignees`, `links`, `tagIds` thay trọn mảng;
  `[]` xoá hết phần tử. Khi thêm một diagram/AC, phải giữ các phần tử cũ chưa được yêu cầu xoá.
- `listSections` thay từng nhóm `itemType`; chỉ các nhóm được gửi bị thay. `api` thay cùng lúc
  endpoints và errorCodes. Dùng tên enum đúng như schema tool công bố.
- Cả lời gọi được lưu nguyên tử. Sửa nội dung InReview/Approved đưa tài liệu về Draft.
  Archive chặn sửa; không tự bỏ archive hoặc thay đổi quy trình phê duyệt.

`detail` theo loại: Story cần storyStatement; TDD cần featureName; BR cần ruleName/statement.
UnitTest cần module/unitUnderTest/unitTestType/testSuite/testPriority/expectedOutput/rationale;
SystemTest cần storyKey/systemTestType/testSuite/testPriority/expectedResult/rationale.
Không tự đặt trạng thái phê duyệt, version, hash, ID bảng con hoặc lịch sử.

## Xoá

Chỉ xoá tài liệu người dùng đã chỉ định rõ. Đọc lại tài liệu rồi gọi
`document_first_delete_document` với `projectId`, `documentKey` và `expectedContentHash`.
Đây là xoá mềm toàn bộ tài liệu. Xoá một section là thao tác update, không dùng delete_document.

## Kết quả và lỗi

Trả mã tài liệu, loại thao tác và kết quả backend. Sau tạo/sửa, dùng fetch để kiểm tra nội dung đã lưu.
`DOCUMENT_CONTENT_CHANGED`/`CONCURRENCY_CONFLICT`: đọc lại và đối chiếu trước khi xây thay đổi mới.
Không tự gọi lại thao tác ghi khi timeout/mất phản hồi; tìm hoặc fetch trước vì lần trước có thể
đã commit. Không báo thành công nếu tool trả lỗi. Đối với chỉ soạn để xem hoặc dry-run, trả nội dung
đề xuất và không gọi tool ghi.
