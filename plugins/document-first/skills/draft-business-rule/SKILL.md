---
name: draft-business-rule
description: Soạn bản nháp Business Rule theo cấu trúc Rule Info, Statement, When, Then, Except, Notes của Document First, tách rõ điều kiện, hành động và ngoại lệ để kiểm thử được. Dùng khi người dùng mô tả một chính sách, quy định tính toán, điều kiện hợp lệ hoặc ràng buộc nghiệp vụ cần ghi lại thành BR; không dùng để kiểm tra code có tuân thủ rule hay không.
---

# Soạn Business Rule

## Lấy context trong project

1. Xác định `projectId`; gọi `document_first_list_projects` nếu cần.
2. Có mã tài liệu thì dùng `document_first_fetch`; dùng `document_first_get_story` để đọc Story có cấu trúc, gồm flow, AC và reference. Dùng `document_first_search` khi cần tìm tài liệu liên quan.
3. Duyệt Business Rule bằng `document_first_list_rules` với `limit`/`offset`, rồi đọc chi tiết bằng `documentKeys` (tối đa 20 mã/lần). Khi kiểm tra xung đột hoặc traceability, duyệt đủ các trang; search không chứng minh được danh sách đầy đủ.
4. Cần thêm TDD, reference và tài liệu liên quan cho Story thì dùng `document_first_prepare_story_context` (contract `2026-09-05`, `evidence.readDocuments`). Context có giới hạn; đọc bổ sung tài liệu cần thiết.
5. Mọi thành viên project được đọc tài liệu ở Draft, InReview, Approved và đã lưu trữ. Không yêu cầu phê duyệt hoặc Release để dùng nội dung làm căn cứ. Ghi lại trạng thái thực tế, không mô tả tài liệu chưa duyệt là đã duyệt.
6. Truy vết bằng `documentKey#contentHash`; `approvalId` chỉ ghi khi có. `missingKeys`/`unresolvedReferences` là phần chưa lấy được, không tự suy đoán nội dung. `resolved` chỉ phản ánh liên kết đã tìm thấy tài liệu đích.

Trước khi soạn rule mới, duyệt đủ rule trong project để tìm chồng lấn. Nếu có mâu thuẫn, nêu mã rule và làm rõ hướng thay thế hoặc thu hẹp phạm vi.

## Tách mệnh đề

Một Business Rule tốt phải quy về đúng một mệnh đề kiểm chứng được:

- **When** — điều kiện kích hoạt, viết bằng thuộc tính dữ liệu cụ thể, nối bằng `VÀ` / `HOẶC`.
- **Then** — hành động hoặc kết quả bắt buộc, gồm cả cách hiển thị và dữ liệu ghi xuống nếu có.
- **Except** — trường hợp rule không áp dụng, liệt kê đầy đủ, không để ngầm hiểu.

Nếu người dùng mô tả nhiều điều kiện độc lập cho ra kết quả khác nhau, tách thành nhiều BR riêng
thay vì nhồi vào một rule có `if/else` lồng nhau.

## Cấu trúc bắt buộc

Giữ nguyên tên và thứ tự heading:

```markdown
# BR-<mã>

## Rule Info

- **Name**: <tên ngắn gọn>
- **Category**: <nhóm nghiệp vụ>
- **Status**: Draft
- **Version**: v1.0
- **Effective Date**: <YYYY-MM-DD>
- **Owner**: <người chịu trách nhiệm nghiệp vụ>
- **Source**: <nguồn chính sách>

## Statement

<một câu duy nhất phát biểu trọn vẹn rule>

## When

<điều kiện kích hoạt>

## Then

<hành động hoặc kết quả bắt buộc>

## Except

<các trường hợp không áp dụng>

## Notes

<thứ tự áp dụng, làm tròn, tham chiếu rule liên quan>
```

## Quy tắc nội dung

- `Statement` phải đứng độc lập đọc hiểu được, không phụ thuộc phần khác.
- Con số phải kèm đơn vị và cơ sở tính: "8% trên giá hàng, trước phí ship" thay vì "cộng 8%".
- Khi rule phụ thuộc thứ tự với rule khác, ghi rõ trong `Notes` kèm `documentKey` của rule đó.
- Ngày tháng dùng `YYYY-MM-DD`; hỏi người dùng nếu chưa có `Effective Date`, không lấy ngày hôm nay.
- Rule mới có trạng thái Draft. Khi lưu qua MCP, để backend quản lý trạng thái và version;
  ghi lý do thay đổi trong `Notes`, không tự tăng version hoặc giả lập phê duyệt.

## Bàn giao

Trả bản nháp Markdown trong một code block, kèm:

- `documentKey#contentHash` và `approvalId` nếu có của các rule đã tham chiếu;
- danh sách rule hiện tại có thể xung đột;
- các mục `TODO(cần xác nhận)` còn thiếu, đặc biệt là `Owner`, `Source` và `Effective Date`.

## Khi được yêu cầu lưu vào Document First

Nếu người dùng yêu cầu tạo hoặc cập nhật tài liệu trong project, thực hiện ghi qua MCP theo
[manage-documents](../manage-documents/SKILL.md), rồi trả mã tài liệu và kết quả đã lưu.
Chỉ soạn để xem hoặc dry-run thì dùng định dạng bàn giao ở trên.
