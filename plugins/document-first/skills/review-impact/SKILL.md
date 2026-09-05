---
name: review-impact
description: Phân tích tác động của Story, tài liệu hoặc thay đổi code bằng nội dung hiện tại trong project Document First, xác định rule, contract, module và test liên quan.
---

# Phân tích tác động

## Thu thập bằng chứng

1. Xác định project bằng `document_first_list_projects` nếu cần.
2. Có `documentKey` thì đọc trực tiếp bằng `document_first_fetch`; với Story cần field và AC dùng `document_first_get_story`. Không cần thử tool khác theo trạng thái phê duyệt.
3. Cần context rộng của Story thì dùng `document_first_prepare_story_context`; với mô tả thay đổi dùng `document_first_search`, rồi fetch kết quả kèm `contentHash`.
4. Khi đụng nghiệp vụ, duyệt Business Rule bằng `document_first_list_rules` với phân trang, rồi đọc rule liên quan bằng `documentKeys`. Search không đủ để chứng minh không rule nào bị ảnh hưởng.
5. Đọc tài liệu ở mọi trạng thái trong project; ghi nhận `approvalState`, `isArchived`, `contentHash`, các reference thiếu và `approvalId` nếu có. Không yêu cầu Approved hoặc Release.

## Đối chiếu repository

- Tìm controller, service, entity/migration, worker, client và test thực thi contract.
- Theo dõi caller, consumer, dữ liệu lưu, quyền truy cập, lỗi, background job và compatibility.
- Không sửa code khi người dùng chỉ yêu cầu review/report.

## Kết quả

Ưu tiên theo rủi ro; mỗi mục có bằng chứng `documentKey#contentHash`, code path, tác động, test cần kiểm và mức chắc chắn `Confirmed`, `Likely` hoặc `Blocked` khi thiếu bằng chứng/quyền.

Không xem semantic-related là quan hệ chắc chắn nếu nội dung không xác nhận. Nêu rõ suy luận từ source code và phạm vi chưa đọc được.
