---
name: draft-tdd
description: Soạn bản nháp Technical Design Document theo cấu trúc Document First, gồm Context & Goals, Architecture, ba loại diagram, Data Model, Internal API, External API, References và Change Log, bám theo User Story và Business Rule hiện tại. Dùng khi người dùng cần thiết kế kỹ thuật cho một Story, một tích hợp bên thứ ba hoặc một thay đổi kiến trúc trước khi viết code.
---

# Soạn Technical Design Document

## Lấy context trong project

1. Xác định `projectId`; gọi `document_first_list_projects` nếu cần.
2. Có mã tài liệu thì dùng `document_first_fetch`; dùng `document_first_get_story` để đọc Story có cấu trúc, gồm flow, AC và reference. Dùng `document_first_search` khi cần tìm tài liệu liên quan.
3. Duyệt Business Rule bằng `document_first_list_rules` với `limit`/`offset`, rồi đọc chi tiết bằng `documentKeys` (tối đa 20 mã/lần). Khi kiểm tra xung đột hoặc traceability, duyệt đủ các trang; search không chứng minh được danh sách đầy đủ.
4. Cần thêm TDD, reference và tài liệu liên quan cho Story thì dùng `document_first_prepare_story_context` (contract `2026-09-05`, `evidence.readDocuments`). Context có giới hạn; đọc bổ sung tài liệu cần thiết.
5. Mọi thành viên project được đọc tài liệu ở Draft, InReview, Approved và đã lưu trữ. Không yêu cầu phê duyệt hoặc Release để dùng nội dung làm căn cứ. Ghi lại trạng thái thực tế, không mô tả tài liệu chưa duyệt là đã duyệt.
6. Truy vết bằng `documentKey#contentHash`; `approvalId` chỉ ghi khi có. `missingKeys`/`unresolvedReferences` là phần chưa lấy được, không tự suy đoán nội dung. `resolved` chỉ phản ánh liên kết đã tìm thấy tài liệu đích.

Đọc flow/AC của Story, rule chi phối và các TDD liên quan để giữ nhất quán contract, mã lỗi và data model.

## Đối chiếu repository

- Đọc hướng dẫn của repository đang mở để nắm kiến trúc, convention đặt tên, tầng service và
  cách khai báo migration hiện có.
- Thiết kế phải bám hiện trạng code: nêu rõ module nào sửa, module nào thêm mới.
- Không sửa source code trong skill này; đây là bước thiết kế.

## Cấu trúc bắt buộc

Giữ nguyên tên và thứ tự heading:

```markdown
# TDD-<mã>

## Document Info

- **Feature**:
- **Author**:
- **Reviewer**:
- **Status**: Draft
- **Version**: v1.0
- **Updated At**: <YYYY-MM-DD>

## Context & Goals

### Problem

### Goals

### Non-goals

## Architecture

## Sequence Diagram

## Activity Diagram

## State Diagram

## Data Model

## Internal API

### Endpoints

- **<METHOD>** `<path>` — <mô tả ngắn>

### Examples

#### <METHOD> <path>

### Error Codes

- **<CODE>** (<http status>): <khi nào xảy ra>

## External API

### Endpoints

### Fields

### Error Handling

### Quirks

## References

### User Stories

### Business Rules

### Use Cases

### Others

## Change Log

#### v1.0 (<YYYY-MM-DD>)

- **Change**: Tạo tài liệu
- **Author**:
```

Bỏ trống `## External API` nếu tính năng không gọi hệ thống bên thứ ba, nhưng vẫn giữ heading.

## Quy tắc nội dung

- `Problem` mô tả vấn đề nghiệp vụ đang gặp, không mô tả giải pháp.
- `Non-goals` phải khớp `outOfScope` của Story tương ứng.
- Ba diagram phục vụ ba mục đích khác nhau, không lặp lại nhau: sequence cho "ai gọi ai, message
  gì", activity cho "các bước và nhánh rẽ", state cho "vòng đời trạng thái". Chi tiết cách vẽ nằm
  ở các skill `draw-sequence-diagram`, `draw-activity-diagram`, `draw-state-diagram`.
- `Error Codes` phải là registry đầy đủ: mỗi mã kèm HTTP status và điều kiện phát sinh; mọi phần
  tử `flows.exception` của Story phải ánh xạ được tới một mã.
- `Data Model` nêu bảng, cột, kiểu, ràng buộc, index và khoá idempotency nếu có.
- `References` chỉ chứa `documentKey` có thật lấy từ kết quả MCP.
- `Quirks` ghi các hành vi bất thường của hệ thống ngoài, kèm cách phòng vệ.

## Bàn giao

Trả bản nháp Markdown trong một code block, kèm:

- ánh xạ từ Acceptance Criteria và Business Rule sang thành phần thiết kế chịu trách nhiệm;
- `documentKey#contentHash` và `approvalId` nếu có, của nguồn đã dùng;
- quyết định kỹ thuật cần review cùng phương án thay thế đã cân nhắc;
- khoảng trống do reference chưa resolve hoặc nội dung tài liệu chưa đầy đủ.

## Khi được yêu cầu lưu vào Document First

Nếu người dùng yêu cầu tạo hoặc cập nhật tài liệu trong project, thực hiện ghi qua MCP theo
[manage-documents](../manage-documents/SKILL.md), rồi trả mã tài liệu và kết quả đã lưu.
Chỉ soạn để xem hoặc dry-run thì dùng định dạng bàn giao ở trên.
