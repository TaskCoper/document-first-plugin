---
name: audit-document-traceability
description: Rà soát liên kết giữa User Story, TDD, Business Rule và test trong Document First để tìm reference gãy, tài liệu mồ côi, Acceptance Criteria chưa có test và Business Rule chưa được TDD nào hiện thực hóa. Dùng khi cần kiểm tra độ đầy đủ tài liệu của một project hoặc một Story trước sprint review, release hoặc bàn giao; không dùng để soạn tài liệu mới.
---

# Kiểm tra traceability tài liệu

## Lấy context trong project

1. Xác định `projectId`; gọi `document_first_list_projects` nếu cần.
2. Có mã tài liệu thì dùng `document_first_fetch`; dùng `document_first_get_story` để đọc Story có cấu trúc, gồm flow, AC và reference. Dùng `document_first_search` khi cần tìm tài liệu liên quan.
3. Duyệt Business Rule bằng `document_first_list_rules` với `limit`/`offset`, rồi đọc chi tiết bằng `documentKeys` (tối đa 20 mã/lần). Khi kiểm tra xung đột hoặc traceability, duyệt đủ các trang; search không chứng minh được danh sách đầy đủ.
4. Cần thêm TDD, reference và tài liệu liên quan cho Story thì dùng `document_first_prepare_story_context` (contract `2026-09-05`, `evidence.readDocuments`). Context có giới hạn; đọc bổ sung tài liệu cần thiết.
5. Mọi thành viên project được đọc tài liệu ở Draft, InReview, Approved và đã lưu trữ. Không yêu cầu phê duyệt hoặc Release để dùng nội dung làm căn cứ. Ghi lại trạng thái thực tế, không mô tả tài liệu chưa duyệt là đã duyệt.
6. Truy vết bằng `documentKey#contentHash`; `approvalId` chỉ ghi khi có. `missingKeys`/`unresolvedReferences` là phần chưa lấy được, không tự suy đoán nội dung. `resolved` chỉ phản ánh liên kết đã tìm thấy tài liệu đích.

Phạm vi audit gồm tài liệu ở mọi trạng thái. Search chỉ trả kết quả xếp hạng; ghi rõ phần đã đọc và không kết luận không có TDD/test nào nếu chưa có danh sách đầy đủ.

## Các chiều cần đối chiếu

Kiểm cả hai hướng của mỗi liên kết, vì thiếu sót thường nằm ở chiều ngược lại:

- **Story → TDD** — mọi `documentKey` trong `References / TDDs` tồn tại và đọc được trong project.
- **TDD → Story** — mọi TDD có ít nhất một Story trong `References / User Stories`; TDD không có
  Story nào là tài liệu mồ côi.
- **Story → Business Rule** — mọi `documentKey` trong `References / Rules` tồn tại và đọc được trong project.
- **Business Rule → nơi sử dụng** — mỗi rule được ít nhất một Story hoặc TDD tham chiếu; rule
  không ai tham chiếu là rule mồ côi hoặc đã lỗi thời.
- **Acceptance Criteria → test** — mỗi `AC-xxx` có test document hoặc test trong repository nhận
  trách nhiệm.
- **Exception Flow → Error Codes** — mỗi nhánh `EXC-xx` ánh xạ được tới một mã trong registry
  `Error Codes` của TDD, và ngược lại mỗi mã lỗi có nguồn gốc từ một nhánh hoặc rule.
- **State Diagram → Business Rule** — mỗi transition có guard truy về được một rule hiện tại.
- **Rule ↔ Rule** — phát hiện rule mâu thuẫn, chồng lấn phạm vi hoặc phụ thuộc thứ tự nhưng không
  ghi rõ trong `Notes`.

Khi repository đang mở có liên quan, đối chiếu thêm test hiện có với `AC-xxx` để phân biệt "chưa
có tài liệu test" và "đã có test trong code nhưng chưa ghi vào tài liệu".

## Phân loại phát hiện

Mỗi phát hiện gán một loại và một mức độ:

- `BrokenReference` — trỏ tới `documentKey` không tồn tại.
- `UnresolvedReference` — chưa resolve được tài liệu đích; cần đọc theo key để xác minh.
- `Orphan` — tài liệu không được ai tham chiếu.
- `MissingCoverage` — Acceptance Criteria, nhánh Flow hoặc rule chưa có test hoặc chưa có thiết kế.
- `Inconsistent` — hai tài liệu hiện tại nói khác nhau về cùng một hành vi.
- `Blocked` — không đủ quyền hoặc tool lỗi nên không kết luận được.

Mức độ: `High` khi làm sai hành vi hoặc chặn release, `Medium` khi gây rủi ro hiểu nhầm,
`Low` khi chỉ thiếu chỉnh chu.

## Bàn giao

Trả báo cáo sắp xếp theo mức độ giảm dần, mỗi mục gồm:

- loại và mức độ;
- `documentKey#contentHash` cùng `approvalId` nếu có của các tài liệu liên quan;
- mô tả cụ thể liên kết nào thiếu hoặc mâu thuẫn ở đâu;
- hành động đề xuất và ai nên xử lý.

Kết thúc bằng bảng tổng hợp số lượng theo loại và danh sách tài liệu không quét được. Không sửa
tài liệu hay source code trong skill này.
