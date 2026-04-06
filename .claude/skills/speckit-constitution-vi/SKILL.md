---
name: "speckit-constitution"
description: "Tạo hoặc cập nhật project constitution dựa trên các principles được cung cấp, đảm bảo tính đồng bộ trên tất cả các dependent templates."
argument-hint: "Các principles hoặc giá trị cho project constitution"
compatibility: "Yêu cầu cấu trúc project spec-kit với thư mục .specify/"
metadata:
  author: "github-spec-kit"
  source: "templates/commands/constitution.md"
user-invocable: true
disable-model-invocation: true
---

## Đầu vào từ người dùng

```text
$ARGUMENTS
```

Bạn **PHẢI** xem xét đầu vào từ người dùng trước khi tiếp tục (nếu không rỗng).

## Outline

Bạn đang cập nhật project constitution tại `.specify/memory/constitution.md`. File này là một TEMPLATE chứa các placeholder tokens dạng ngoặc vuông (ví dụ: `[PROJECT_NAME]`, `[PRINCIPLE_1_NAME]`). Nhiệm vụ của bạn là (a) thu thập/đưa ra các giá trị cụ thể, (b) điền template một cách chính xác, và (c) propagate các amendments qua các artifacts phụ thuộc.

**Lưu ý**: Nếu `.specify/memory/constitution.md` chưa tồn tại, nó phải được khởi tạo từ `.specify/templates/constitution-template.md` trong quá trình setup project. Nếu thiếu, copy template trước.

Tuân theo execution flow này:

1. Load constitution hiện tại tại `.specify/memory/constitution.md`.
   - Xác định mọi placeholder token dạng `[ALL_CAPS_IDENTIFIER]`.
     **QUAN TRỌNG**: User có thể cần ít hoặc nhiều principles hơn các cái trong template. Nếu có số được chỉ định, tôn trọng nó - tuân theo template chung. Bạn sẽ cập nhật doc tương ứng.

2. Thu thập/đưa ra giá trị cho các placeholders:
   - Nếu user input (cuộc trò chuyện) cung cấp giá trị, sử dụng nó.
   - Ngược lại, suy ra từ existing repo context (README, docs, các phiên bản constitution trước nếu được nhúng).
   - Cho các governance dates: `RATIFICATION_DATE` là ngày adopt gốc (nếu không biết thì hỏi hoặc đánh dấu TODO), `LAST_AMENDED_DATE` là hôm nay nếu có thay đổi, ngược lại giữ nguyên giá trị trước.
   - `CONSTITUTION_VERSION` phải tăng theo semantic versioning rules:
     - MAJOR: Các governance/principle removals hoặc redefinitions không tương thích backward.
     - MINOR: Principle/section mới được thêm hoặc mở rộng đáng kể.
     - PATCH: Clarifications, wording, typo fixes, các refinements không mang tính semantic.
   - Nếu loại version bump mơ hồ, đề xuất reasoning trước khi finalizing.

3. Soạn nội dung constitution đã cập nhật:
   - Thay thế mọi placeholder bằng text cụ thể (không còn bracketed tokens ngoại trừ các template slots được giữ lại có chủ đích mà project đã chọn chưa define - justify rõ ràng bất kỳ placeholder nào được giữ lại).
   - Giữ nguyên heading hierarchy và các comments có thể được remove một khi đã được replace trừ khi chúng vẫn bổ sung clarifying guidance.
   - Đảm bảo mỗi Principle section: dòng tên ngắn gọn, paragraph (hoặc bullet list) capture các rules không thể thương lượng, rationale rõ ràng nếu không hiển nhiên.
   - Đảm bảo Governance section liệt kê amendment procedure, versioning policy, và các compliance review expectations.

4. Consistency propagation checklist (chuyển đổi checklist trước thành các validations chủ động):
   - Đọc `.specify/templates/plan-template.md` và đảm bảo bất kỳ "Constitution Check" hoặc rules nào align với các principles đã cập nhật.
   - Đọc `.specify/templates/spec-template.md` cho scope/requirements alignment — cập nhật nếu constitution thêm/remove mandatory sections hoặc constraints.
   - Đọc `.specify/templates/tasks-template.md` và đảm bảo task categorization phản ánh các task types mới hoặc bị remove do principle-driven (ví dụ: observability, versioning, testing discipline).
   - Đọc mỗi command file trong `.specify/templates/commands/*.md` (bao gồm file này) để verify không còn outdated references (agent-specific names như CLAUDE) khi cần generic guidance.
   - Đọc bất kỳ runtime guidance docs nào (ví dụ: `README.md`, `docs/quickstart.md`, hoặc agent-specific guidance files nếu có). Cập nhật references đến các principles đã thay đổi.

5. Tạo Sync Impact Report (prepend như một HTML comment ở đầu file constitution sau khi cập nhật):
   - Thay đổi version: cũ → mới
   - Danh sách các principles đã được sửa đổi (tiêu đề cũ → tiêu đề mới nếu được rename)
   - Các sections đã thêm
   - Các sections đã xóa
   - Các templates cần cập nhật (✅ updated / ⚠ pending) với file paths
   - Các TODOs cần theo dõi nếu có placeholders cố tình được defer.

6. Validation trước khi final output:
   - Không còn bracket tokens không được giải thích.
   - Dòng version khớp với report.
   - Dates theo định dạng ISO YYYY-MM-DD.
   - Principles phải mang tính declarative, testable, và không có ngôn ngữ mơ hồ ("should" → thay bằng MUST/SHOULD rationale ở những chỗ phù hợp).

7. Viết constitution đã hoàn thành vào `.specify/memory/constitution.md` (ghi đè).

8. Xuất summary cuối cùng cho user với:
   - Version mới và bump rationale.
   - Bất kỳ files nào được flag cho manual follow-up.
   - Suggested commit message (ví dụ: `docs: amend constitution to vX.Y.Z (principle additions + governance update)`).

Formatting & Style Requirements:

- Sử dụng Markdown headings chính xác như trong template (không demote/promote levels).
- Wrap các dòng rationale dài để giữ readability (<100 chars lý tưởng) nhưng không hard enforce với các breaks không tự nhiên.
- Giữ một blank line duy nhất giữa các sections.
- Tránh trailing whitespace.

Nếu user cung cấp partial updates (ví dụ: chỉ một principle revision), vẫn thực hiện validation và version decision steps.

Nếu thiếu critical info (ví dụ: ratification date thực sự không biết), chèn `TODO(<FIELD_NAME>): explanation` và include trong Sync Impact Report dưới deferred items.

Không tạo template mới; luôn thao tác trên file `.specify/memory/constitution.md` hiện có.
