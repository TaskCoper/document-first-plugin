---
name: draft-tdd
description: Soạn bản nháp Technical Design Document theo cấu trúc Document First, gồm Context & Goals, Architecture, ba loại diagram, Data Model, Internal API, External API, References và Change Log, bám theo User Story và Business Rule đã phê duyệt. Dùng khi người dùng cần thiết kế kỹ thuật cho một Story, một tích hợp bên thứ ba hoặc một thay đổi kiến trúc trước khi viết code.
---

# Soạn Technical Design Document

## Cổng context bắt buộc

1. Xác định `projectId`; gọi `document_first_list_projects` nếu người dùng chưa cung cấp.
2. Nếu TDD phục vụ một Story, gọi `document_first_get_story`. Tool trả sẵn `acceptanceCriteria`,
   ba nhóm `flows`, `nonFunctional`, `outOfScope` và `references` — đây đúng là bộ đầu vào để
   dựng `Context & Goals`, ba diagram và registry `Error Codes`.
3. Đọc Business Rule chi phối bằng `document_first_list_rules` với `documentKeys` lấy từ
   `references.rules`. Cần TDD đã có để bám kiến trúc hiện tại thì dùng `document_first_search`
   rồi `document_first_fetch` với `contentHash`.
4. Dùng `document_first_prepare_story_context` thay cho bước 2 khi cần cả tài liệu
   semantic-related, ví dụ khi thiết kế đụng nhiều module chưa rõ ranh giới.
5. Chỉ thiết kế dựa trên nội dung Approved. Dừng với `Blocked` nếu Story hoặc rule bắt buộc chưa
   được phê duyệt.
6. Ghi nhận mọi `unresolvedReferences`, `missingKeys` và reference có `resolved: false` là khoảng
   trống thiết kế, không tự lấp bằng suy đoán.
7. Không yêu cầu Release, GitHub commit hoặc liên kết GitHub repository.

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
- `documentKey#contentHash` và `approvalId` đã dùng;
- quyết định kỹ thuật cần review cùng phương án thay thế đã cân nhắc;
- khoảng trống do reference chưa resolve hoặc tài liệu chưa phê duyệt.
