---
name: audit-document-traceability
description: Rà soát liên kết giữa User Story, TDD, Business Rule và test trong Document First để tìm reference gãy, tài liệu mồ côi, Acceptance Criteria chưa có test và Business Rule chưa được TDD nào hiện thực hóa. Dùng khi cần kiểm tra độ đầy đủ tài liệu của một project hoặc một Story trước sprint review, release hoặc bàn giao; không dùng để soạn tài liệu mới.
---

# Kiểm tra traceability tài liệu

## Thu thập bằng chứng

1. Xác định `projectId`; gọi `document_first_list_projects` nếu người dùng chưa cung cấp.
2. Lấy **toàn bộ** Business Rule đã phê duyệt bằng `document_first_list_rules` ở chế độ duyệt,
   lặp `offset` cho tới khi đủ `total`. Đây là bước không thể thay bằng `document_first_search`:
   search xếp hạng theo truy vấn nên không bao giờ chứng minh được "không có rule nào tham chiếu
   tới X". Không có danh sách đầy đủ thì không phát hiện được rule mồ côi.
3. Nếu phạm vi là một Story, gọi `document_first_get_story` để lấy `references.tdds`,
   `references.rules` và cờ `resolved` của từng reference — `resolved: false` chính là
   `BrokenReference` hoặc `UnapprovedReference`, không cần suy đoán.
4. Đọc nội dung rule cần soi kỹ bằng `document_first_list_rules` với `documentKeys`; dùng
   `document_first_search` và `document_first_fetch` cho TDD và tài liệu test.
5. Chỉ tính nội dung Approved là đã có. Tài liệu ở Draft hoặc InReview được ghi nhận là khoảng
   trống, không tính là đã phủ.
6. Ghi lại toàn bộ `unresolvedReferences` và `missingKeys` trả về từ MCP; đây là bằng chứng trực
   tiếp của reference gãy.
7. Nêu rõ phạm vi đã quét được và phần nào không đọc được do thiếu quyền; không suy rộng kết luận
   ra ngoài phạm vi đó.
8. Không yêu cầu Release, GitHub commit hoặc liên kết GitHub repository.

## Các chiều cần đối chiếu

Kiểm cả hai hướng của mỗi liên kết, vì thiếu sót thường nằm ở chiều ngược lại:

- **Story → TDD** — mọi `documentKey` trong `References / TDDs` tồn tại và đã phê duyệt.
- **TDD → Story** — mọi TDD có ít nhất một Story trong `References / User Stories`; TDD không có
  Story nào là tài liệu mồ côi.
- **Story → Business Rule** — mọi `documentKey` trong `References / Rules` tồn tại và đã phê duyệt.
- **Business Rule → nơi sử dụng** — mỗi rule được ít nhất một Story hoặc TDD tham chiếu; rule
  không ai tham chiếu là rule mồ côi hoặc đã lỗi thời.
- **Acceptance Criteria → test** — mỗi `AC-xxx` có test document hoặc test trong repository nhận
  trách nhiệm.
- **Exception Flow → Error Codes** — mỗi nhánh `EXC-xx` ánh xạ được tới một mã trong registry
  `Error Codes` của TDD, và ngược lại mỗi mã lỗi có nguồn gốc từ một nhánh hoặc rule.
- **State Diagram → Business Rule** — mỗi transition có guard truy về được một rule đã duyệt.
- **Rule ↔ Rule** — phát hiện rule mâu thuẫn, chồng lấn phạm vi hoặc phụ thuộc thứ tự nhưng không
  ghi rõ trong `Notes`.

Khi repository đang mở có liên quan, đối chiếu thêm test hiện có với `AC-xxx` để phân biệt "chưa
có tài liệu test" và "đã có test trong code nhưng chưa ghi vào tài liệu".

## Phân loại phát hiện

Mỗi phát hiện gán một loại và một mức độ:

- `BrokenReference` — trỏ tới `documentKey` không tồn tại.
- `UnapprovedReference` — tài liệu đích tồn tại nhưng chưa được phê duyệt.
- `Orphan` — tài liệu không được ai tham chiếu.
- `MissingCoverage` — Acceptance Criteria, nhánh Flow hoặc rule chưa có test hoặc chưa có thiết kế.
- `Inconsistent` — hai tài liệu đã duyệt nói khác nhau về cùng một hành vi.
- `Blocked` — không đủ quyền hoặc tool lỗi nên không kết luận được.

Mức độ: `High` khi làm sai hành vi hoặc chặn release, `Medium` khi gây rủi ro hiểu nhầm,
`Low` khi chỉ thiếu chỉnh chu.

## Bàn giao

Trả báo cáo sắp xếp theo mức độ giảm dần, mỗi mục gồm:

- loại và mức độ;
- `documentKey#contentHash` cùng `approvalId` của các tài liệu liên quan;
- mô tả cụ thể liên kết nào thiếu hoặc mâu thuẫn ở đâu;
- hành động đề xuất và ai nên xử lý.

Kết thúc bằng bảng tổng hợp số lượng theo loại và danh sách tài liệu không quét được. Không sửa
tài liệu hay source code trong skill này.
