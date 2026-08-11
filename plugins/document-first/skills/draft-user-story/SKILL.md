---
name: draft-user-story
description: Soạn bản nháp User Story theo đúng cấu trúc Markdown của Document First, gồm Metadata, Conditions, Flow, Acceptance Criteria, References, Non-Functional và Out of Scope. Dùng khi người dùng mô tả một yêu cầu, tính năng hoặc nhu cầu nghiệp vụ mới và cần chuyển thành User Story để đưa vào Document First; không dùng khi Story đã tồn tại và chỉ cần triển khai.
---

# Soạn User Story

## Thu thập context

1. Xác định `projectId`; gọi `document_first_list_projects` nếu người dùng chưa cung cấp.
2. Gọi `document_first_search` với từ khóa của yêu cầu để tìm Story và TDD đã phê duyệt có liên
   quan. Đọc Story nghi trùng bằng `document_first_get_story` để so cấu trúc `flows` và
   `acceptanceCriteria` thay vì so tiêu đề.
3. Nếu đã có Story trùng phạm vi, dừng lại và báo `documentKey` đó thay vì soạn bản trùng lặp.
4. Duyệt Business Rule của project bằng `document_first_list_rules` để mục `References / Rules`
   chỉ chứa `documentKey` có thật; đọc chi tiết rule cần bám bằng `documentKeys`.
5. Chỉ dùng nội dung Approved làm căn cứ. Không suy đoán Business Rule chưa được phê duyệt.
6. Không yêu cầu Release, GitHub commit hoặc liên kết GitHub repository.

## Làm rõ trước khi viết

Hỏi người dùng khi thiếu, không tự bịa:

- vai trò người dùng, mục tiêu và lý do nghiệp vụ;
- điều kiện tiên quyết và sự kiện kích hoạt;
- các nhánh rẽ và tình huống lỗi đã biết;
- Business Rule hoặc TDD phải tham chiếu;
- phạm vi bị loại trừ.

Nếu người dùng yêu cầu bản nháp nhanh, viết trước rồi đánh dấu `TODO(cần xác nhận)` tại từng chỗ
còn thiếu, không im lặng điền giá trị mặc định.

## Cấu trúc bắt buộc

Markdown là contract được parse theo đúng tên heading và tiền tố bullet. Giữ nguyên thứ tự và
chính tả của heading:

```markdown
# STORY-<mã>

## Metadata

- **Story**: Là một <vai trò>, tôi muốn <mục tiêu> để <giá trị>.
- **Context**: <bối cảnh nghiệp vụ>
- **Sprint**:
- **Priority**: Must | Should | Could | Won't
- **Status**: Draft
- **Creator**:
- **Assignee**:
  - Frontend:
  - Backend:

## Conditions

### Preconditions

- <điều kiện phải đúng trước khi luồng bắt đầu>

### Trigger

<một câu mô tả sự kiện kích hoạt>

## Flow

### Main Flow

1. <bước>

### Alternative Flow

#### ALT-01

1. <bước>

### Exception Flow

#### EXC-01

1. <bước>

## Acceptance Criteria

#### AC-001

- **Given**: <trạng thái ban đầu>
- **When**: <hành động>
- **Then**: <kết quả kiểm chứng được>
- **And**: <ràng buộc bổ sung>

## References

### TDDs

- <TDD-xxx>

### Rules

- <BR-xx>

### Dependencies

## Non-Functional

- <hiệu năng, bảo mật, accessibility, tương thích>

## Out of Scope

- <phần bị loại trừ, kèm Story tiếp nhận nếu có>
```

## Quy tắc nội dung

- Main Flow đánh số liên tục, mỗi bước một hành động của một chủ thể rõ ràng.
- Mỗi nhánh `ALT-xx` và `EXC-xx` phải nói rõ quay lại bước nào của Main Flow hoặc kết thúc ra sao.
- Acceptance Criteria phải kiểm chứng được: có ngưỡng, mã lỗi hoặc trạng thái cụ thể, không dùng
  "nhanh", "thân thiện", "hợp lý".
- Mỗi mục trong `Rules` và `TDDs` phải là `documentKey` có thật, lấy từ kết quả MCP.
- Đặt `Status: Draft`; skill này không phê duyệt tài liệu.

## Bàn giao

Trả bản nháp Markdown hoàn chỉnh trong một code block để người dùng dán vào Document First, kèm:

- danh sách `documentKey#contentHash` và `approvalId` đã dùng làm căn cứ;
- các mục `TODO(cần xác nhận)` còn lại;
- giả định không xuất phát từ tài liệu đã phê duyệt.
