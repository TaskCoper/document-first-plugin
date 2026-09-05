---
name: draft-user-story
description: Soạn bản nháp User Story theo đúng cấu trúc Markdown của Document First, gồm Metadata, Conditions, Flow, Acceptance Criteria, References, Non-Functional và Out of Scope. Dùng khi người dùng mô tả một yêu cầu, tính năng hoặc nhu cầu nghiệp vụ mới và cần chuyển thành User Story để đưa vào Document First; không dùng khi Story đã tồn tại và chỉ cần triển khai.
---

# Soạn User Story

## Lấy context trong project

1. Xác định `projectId`; gọi `document_first_list_projects` nếu cần.
2. Có mã tài liệu thì dùng `document_first_fetch`; dùng `document_first_get_story` để đọc Story có cấu trúc, gồm flow, AC và reference. Dùng `document_first_search` khi cần tìm tài liệu liên quan.
3. Duyệt Business Rule bằng `document_first_list_rules` với `limit`/`offset`, rồi đọc chi tiết bằng `documentKeys` (tối đa 20 mã/lần). Khi kiểm tra xung đột hoặc traceability, duyệt đủ các trang; search không chứng minh được danh sách đầy đủ.
4. Cần thêm TDD, reference và tài liệu liên quan cho Story thì dùng `document_first_prepare_story_context` (contract `2026-09-05`, `evidence.readDocuments`). Context có giới hạn; đọc bổ sung tài liệu cần thiết.
5. Mọi thành viên project được đọc tài liệu ở Draft, InReview, Approved và đã lưu trữ. Không yêu cầu phê duyệt hoặc Release để dùng nội dung làm căn cứ. Ghi lại trạng thái thực tế, không mô tả tài liệu chưa duyệt là đã duyệt.
6. Truy vết bằng `documentKey#contentHash`; `approvalId` chỉ ghi khi có. `missingKeys`/`unresolvedReferences` là phần chưa lấy được, không tự suy đoán nội dung. `resolved` chỉ phản ánh liên kết đã tìm thấy tài liệu đích.

Trước khi soạn Story mới, search nội dung liên quan và đọc Story nghi trùng bằng `get_story` để so flow/AC. Nếu đã có Story trùng phạm vi, báo mã đó trước khi tạo thêm tài liệu.

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

- danh sách `documentKey#contentHash` và `approvalId` nếu có, của nguồn đã dùng làm căn cứ;
- các mục `TODO(cần xác nhận)` còn lại;
- giả định không xuất phát từ tài liệu hiện tại.

## Khi được yêu cầu lưu vào Document First

Nếu người dùng yêu cầu tạo hoặc cập nhật tài liệu trong project, thực hiện ghi qua MCP theo
[manage-documents](../manage-documents/SKILL.md), rồi trả mã tài liệu và kết quả đã lưu.
Chỉ soạn để xem hoặc dry-run thì dùng định dạng bàn giao ở trên.
