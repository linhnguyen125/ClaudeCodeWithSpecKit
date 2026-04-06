---
name: "speckit-analyze"
description: "Thực hiện phân tích tính nhất quán và chất lượng (non-destructive) trên các artifacts (spec.md, plan.md, tasks.md) sau khi khởi tạo task."
argument-hint: "Các vùng tập trung phân tích tùy chọn"
compatibility: "Yêu cầu cấu trúc dự án spec-kit với thư mục .specify/"
metadata:
  author: "github-spec-kit"
  source: "templates/commands/analyze.md"
user-invocable: true
disable-model-invocation: true
---

## Đầu vào từ người dùng

```text
$ARGUMENTS
```

Bạn **PHẢI** xem xét đầu vào từ người dùng trước khi tiến hành (nếu không rỗng).

## Mục tiêu

Xác định các điểm mâu thuẫn (inconsistencies), trùng lặp (duplications), mơ hồ (ambiguities) và các yêu cầu chưa được định nghĩa rõ (underspecified) trên ba artifacts cốt lõi (`spec.md`, `plan.md`, `tasks.md`) trước khi implement. Lệnh này CHỈ được gọi lếch sau khi `/speckit.tasks` đã generate thành công `tasks.md`.

## Các ràng buộc vận hành

**STRICT READ-ONLY**: Tuyệt đối không chỉnh sửa bất kỳ file nào. Output ra một structured analysis report. Cung cấp remediation plan (tùy chọn) - user phải explicitly approve trước khi trigger các mandate chỉnh sửa tiếp theo.

**Thẩm quyền của Constitution**: Project constitution (`.specify/memory/constitution.md`) là **non-negotiable** trong scope phân tích này. Bất kỳ conflict nào với constitution đều tự động bị đánh dấu mức độ CRITICAL, yêu cầu điều chỉnh lại spec, plan hoặc tasks — nghiêm cấm việc làm loãng (diluding), diễn giải sai (misinterpreting) hoặc phớt lờ các nguyên tắc. Nếu nguyên tắc (principle) gốc cần thay đổi, tiến hành qua constitution update workflow bên ngoài `/speckit.analyze`.

## Các bước thực thi

### 1. Khởi tạo ngữ cảnh phân tích

Chạy `.specify/scripts/powershell/check-prerequisites.ps1 -Json -RequireTasks -IncludeTasks` một lần từ thư mục gốc repo và parse JSON để lấy FEATURE_DIR và AVAILABLE_DOCS. Tính đường dẫn tuyệt đối:

- SPEC = FEATURE_DIR/spec.md
- PLAN = FEATURE_DIR/plan.md
- TASKS = FEATURE_DIR/tasks.md

Dừng lại với thông báo lỗi nếu bất kỳ file bắt buộc nào bị thiếu (hướng dẫn người dùng chạy lệnh điều kiện tiên quyết còn thiếu).
Đối với dấu nháy đơn trong args như "I'm Groot", sử dụng cú pháp escape: ví dụ 'I'\''m Groot' (hoặc dùng dấu nháy kép nếu có thể: "I'm Groot").

### 2. Tải các Artifact (Hiển thị theo từng bước)

Chỉ tải ngữ cảnh cần thiết tối thiểu từ mỗi artifact:

**Từ spec.md:**

- Tổng quan / Bối cảnh (Overview/Context)
- Yêu cầu chức năng (Functional Requirements)
- Tiêu chí thành công (Success Criteria — kết quả đo lường được — ví dụ: hiệu năng, bảo mật, khả dụng, thành công của người dùng, tác động kinh doanh)
- User Stories
- Các trường hợp biên (Edge Cases) (nếu có)

**Từ plan.md:**

- Các lựa chọn kiến trúc/stack
- Tham chiếu Data Model
- Các Giai đoạn (Phases)
- Các ràng buộc kỹ thuật

**Từ tasks.md:**

- Task IDs
- Mô tả
- Phân nhóm theo giai đoạn
- Các marker song song [P]
- Các đường dẫn file được tham chiếu

**Từ constitution:**

- Tải `.specify/memory/constitution.md` để xác thực nguyên tắc

### 3. Xây dựng mô hình ngữ nghĩa (Semantic Models)

Tạo biểu diễn nội bộ (không bao gồm artifact thô trong đầu ra):

- **Danh mục yêu cầu (Requirements inventory)**: Với mỗi Functional Requirement (FR-###) và Success Criterion (SC-###), thiết lập một stable key. Ưu tiên sử dụng identifier FR-/SC- rõ ràng làm khóa chính, có thể gen slug mệnh lệnh để tăng readability (ví dụ: "User can upload file" → `user-can-upload-file`). Chỉ chọn lọc các Success Criteria mà có thể translate thành tasks (ví dụ: hạ tầng load-testing, công cụ security audit), lược bỏ các KPI sau launch (ví dụ: "Giảm ticket hỗ trợ 50%").
- **Danh mục user story/action**: Các hành động user định cừ (distinct) đi kèm acceptance criteria.
- **Bản đồ phạm vi task (Task coverage mapping)**: Ánh xạ 1 task với 1 or n yêu cầu/story (suy luận qua keyword / reference explicit như ID).
- **Tập quy tắc Hiến pháp (Constitution rule set)**: Extrapolate title nguyên tắc và các strict normative constraints (MUST/SHOULD).

### 4. Các bước phát hiện (Token-Efficient Analysis)

Tập trung vào các phát hiện có tín hiệu cao. Giới hạn ở 50 phát hiện; tổng hợp phần còn lại trong tóm tắt overflow.

#### A. Phát hiện trùng lặp (Duplication Detection)

- Xác định các yêu cầu gần giống nhau
- Đánh dấu cách diễn đạt chất lượng thấp hơn để hợp nhất

#### B. Phát hiện sự mơ hồ (Ambiguity Detection)

- Đánh dấu các tính từ mơ hồ (nhanh, có thể mở rộng, bảo mật, trực quan, mạnh mẽ) thiếu tiêu chí đo lường được
- Đánh dấu các placeholder chưa được giải quyết (TODO, TKTK, ???, `<placeholder>`, v.v.)

#### C. Chưa chỉ định đủ (Underspecification)

- Yêu cầu có động từ nhưng thiếu đối tượng hoặc kết quả đo lường được
- User story thiếu alignment với tiêu chí chấp nhận
- Task tham chiếu file hoặc component không được định nghĩa trong spec/plan

#### D. Căn chỉnh Hiến pháp (Constitution Alignment)

- Bất kỳ yêu cầu hoặc phần tử plan nào xung đột với nguyên tắc MUST
- Thiếu các section hoặc cổng chất lượng bắt buộc từ constitution

#### E. Khoảng trống phạm vi (Coverage Gaps)

- Yêu cầu không có task nào liên kết
- Task không có yêu cầu/story được ánh xạ
- Success Criteria cần công việc xây dựng (hiệu năng, bảo mật, khả dụng) không được phản ánh trong tasks

#### F. Không nhất quán (Inconsistency)

- Trôi dạt thuật ngữ (cùng khái niệm được đặt tên khác nhau giữa các file)
- Data entity được tham chiếu trong plan nhưng không có trong spec (hoặc ngược lại)
- Thứ tự task mâu thuẫn (ví dụ: integration task trước foundational setup task mà không có ghi chú phụ thuộc)
- Yêu cầu xung đột (ví dụ: một cái yêu cầu Next.js trong khi cái kia chỉ định Vue)

### 5. Gán mức độ nghiêm trọng (Severity Assignment)

Sử dụng heuristic sau để ưu tiên các phát hiện:

- **CRITICAL**: Vi phạm MUST trong Hiến pháp, thiếu artifact spec cốt lõi, hoặc yêu cầu có coverage bằng 0 chặn chức năng cơ bản
- **HIGH**: Yêu cầu trùng lặp hoặc xung đột, thuộc tính bảo mật/hiệu năng mơ hồ, tiêu chí chấp nhận không thể kiểm thử
- **MEDIUM**: Trôi dạt thuật ngữ, thiếu coverage task phi chức năng, trường hợp biên chưa chỉ định rõ
- **LOW**: Cải thiện về style/cách diễn đạt, dư thừa nhỏ không ảnh hưởng thứ tự thực thi

### 6. Tạo báo cáo phân tích ngắn gọn

Xuất báo cáo Markdown (không ghi file) với cấu trúc sau:

## Báo cáo phân tích Specification

| ID  | Danh mục  | Mức độ | Vị trí           | Tóm tắt                  | Khuyến nghị                                       |
| --- | --------- | ------ | ---------------- | ------------------------ | ------------------------------------------------- |
| A1  | Trùng lặp | HIGH   | spec.md:L120-134 | Hai yêu cầu tương tự ... | Hợp nhất cách diễn đạt; giữ phiên bản rõ ràng hơn |

(Thêm một hàng cho mỗi phát hiện; tạo ID ổn định có tiền tố là chữ cái đầu của danh mục.)

**Bảng tóm tắt Coverage:**

| Requirement Key | Có Task? | Task IDs | Ghi chú |
| --------------- | -------- | -------- | ------- |

**Các vấn đề Căn chỉnh Hiến pháp:** (nếu có)

**Các Task chưa được ánh xạ:** (nếu có)

**Các chỉ số:**

- Tổng số Requirements
- Tổng số Tasks
- Coverage % (requirements có >=1 task)
- Số lượng Ambiguity
- Số lượng Duplication
- Số lượng Critical Issues

### 7. Đề xuất hành động tiếp theo

Ở cuối báo cáo, xuất khối Next Actions ngắn gọn:

- Nếu có vấn đề CRITICAL: Khuyến nghị giải quyết trước `/speckit.implement`
- Nếu chỉ có LOW/MEDIUM: Người dùng có thể tiến hành, nhưng đưa ra gợi ý cải thiện
- Cung cấp các đề xuất lệnh rõ ràng: ví dụ: "Chạy /speckit.specify với refinement", "Chạy /speckit.plan để điều chỉnh kiến trúc", "Chỉnh sửa thủ công tasks.md để thêm coverage cho 'performance-metrics'"

### 8. Đề xuất phương án khắc phục

Hỏi người dùng: "Bạn có muốn tôi đề xuất các chỉnh sửa khắc phục cụ thể cho N vấn đề hàng đầu không?" (KHÔNG tự động áp dụng.)

## Nguyên tắc vận hành

### Hiệu quả ngữ cảnh

- **Token tối thiểu, tín hiệu cao**: Tập trung vào các phát hiện có thể hành động, không phải tài liệu hóa toàn diện
- **Hiển thị theo từng bước**: Tải artifact tăng dần; không dump toàn bộ nội dung vào phân tích
- **Đầu ra hiệu quả về token**: Giới hạn bảng phát hiện ở 50 hàng; tóm tắt phần tràn
- **Kết quả xác định được**: Chạy lại mà không thay đổi nên tạo ra ID và số lượng nhất quán

### Hướng dẫn phân tích

- **KHÔNG BAO GIỜ sửa file** (đây là phân tích chỉ đọc)
- **KHÔNG BAO GIỜ tưởng tượng các phần bị thiếu** (nếu không có, báo cáo chính xác)
- **Ưu tiên vi phạm Hiến pháp** (đây luôn là CRITICAL)
- **Dùng ví dụ thay vì quy tắc exhaustive** (trích dẫn các trường hợp cụ thể, không phải mẫu chung)
- **Báo cáo 0 vấn đề một cách duyên dáng** (xuất báo cáo thành công kèm theo số liệu coverage)

## Ngữ cảnh

$ARGUMENTS
