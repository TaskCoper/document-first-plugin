---
name: draft-business-rule
description: Soạn bản nháp Business Rule theo cấu trúc Rule Info, Statement, When, Then, Except, Notes của Document First, tách rõ điều kiện, hành động và ngoại lệ để kiểm thử được. Dùng khi người dùng mô tả một chính sách, quy định tính toán, điều kiện hợp lệ hoặc ràng buộc nghiệp vụ cần ghi lại thành BR; không dùng để kiểm tra code có tuân thủ rule hay không.
---

# Soạn Business Rule

## Thu thập context

1. Xác định `projectId`; gọi `document_first_list_projects` nếu người dùng chưa cung cấp.
2. Duyệt **toàn bộ** Business Rule của project bằng `document_first_list_rules`, lặp `offset`
   cho tới khi đủ `total`. Không thay bằng `document_first_search`: rule chồng lấn thường dùng từ
   ngữ khác hẳn nên tìm theo từ khoá sẽ bỏ sót, mà bỏ sót ở đây nghĩa là ban hành một rule mâu
   thuẫn với rule đang có hiệu lực.
3. Đọc chi tiết các rule nghi chồng lấn bằng `document_first_list_rules` với `documentKeys`, dựa
   trên `ruleName`, `category` và `summary` ở bước duyệt để chọn.
4. Nếu rule mới mâu thuẫn hoặc chồng lấn với rule đã duyệt, nêu rõ `documentKey` bị ảnh hưởng và
   hỏi người dùng muốn thay thế, tạo version mới hay thu hẹp phạm vi.
5. Chỉ dùng nội dung Approved làm căn cứ. Không suy đoán chính sách chưa có tài liệu.
6. Không yêu cầu Release, GitHub commit hoặc liên kết GitHub repository.

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
- Đặt `Status: Draft` và `Version: v1.0` cho rule mới; khi sửa rule đã duyệt thì tăng version và
  ghi lý do trong `Notes`.

## Bàn giao

Trả bản nháp Markdown trong một code block, kèm:

- `documentKey#contentHash` và `approvalId` của các rule đã tham chiếu;
- danh sách rule đã duyệt có thể xung đột;
- các mục `TODO(cần xác nhận)` còn thiếu, đặc biệt là `Owner`, `Source` và `Effective Date`.
