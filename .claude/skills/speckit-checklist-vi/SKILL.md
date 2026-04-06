---
name: "speckit-checklist"
description: "Tạo checklist tùy chỉnh cho tính năng hiện tại dựa trên yêu cầu người dùng."
argument-hint: "Domain hoặc vùng tập trung cho checklist"
compatibility: "Yêu cầu cấu trúc dự án spec-kit với thư mục .specify/"
metadata:
  author: "github-spec-kit"
  source: "templates/commands/checklist.md"
user-invocable: true
disable-model-invocation: true
---

## Mục đích Checklist: "Unit Tests cho Requirements viết bằng tiếng Anh"

**KHÁI NIỆM QUAN TRỌNG**: Checklist là **UNIT TESTS CHO VIỆC VIẾT REQUIREMENTS** - chúng xác thực chất lượng, sự rõ ràng và tính đầy đủ của requirements trong một domain cho trước.

**KHÔNG phải để verification/testing**:

- ❌ KHÔNG phải "Xác nhận nút bấm hoạt động đúng"
- ❌ KHÔNG phải "Kiểm tra xử lý lỗi hoạt động"
- ❌ KHÔNG phải "Xác nhận API trả về 200"
- ❌ KHÔNG phải kiểm tra code/implementation có khớp spec hay không

**CHO việc xác thực chất lượng requirements**:

- ✅ "Có định nghĩa yêu cầu về visual hierarchy cho tất cả các loại card không?" (tính đầy đủ)
- ✅ "'Hiển thị nổi bật' đã được định lượng bằng kích thước/vị trí cụ thể chưa?" (sự rõ ràng)
- ✅ "Các yêu cầu về hover state có nhất quán trên tất cả các interactive elements không?" (tính nhất quán)
- ✅ "Có định nghĩa yêu cầu về accessibility cho keyboard navigation không?" (coverage)
- ✅ "Spec có định nghĩa điều gì xảy ra khi logo image fail để load không?" (edge cases)

**Ẩn dụ**: Nếu spec của bạn là code được viết bằng tiếng Anh, checklist chính là test suite của nó. Bạn đang kiểm tra xem requirements có được viết tốt, đầy đủ, không mơ hồ và sẵn sàng triển khai hay không — KHÔNG phải kiểm tra implementation có hoạt động hay không.

## Đầu vào từ người dùng

```text
$ARGUMENTS
```

Bạn **PHẢI** xem xét đầu vào từ người dùng trước khi tiến hành (nếu không rỗng).

## Các bước thực thi

1. **Setup**: Chạy `.specify/scripts/powershell/check-prerequisites.ps1 -Json` từ thư mục gốc repo và parse JSON cho FEATURE_DIR và danh sách AVAILABLE_DOCS.
   - Tất cả đường dẫn file phải là tuyệt đối.
   - Đối với dấu nháy đơn trong args như "I'm Groot", sử dụng cú pháp escape: ví dụ 'I'\''m Groot' (hoặc dùng dấu nháy kép nếu có thể: "I'm Groot").

2. **Làm rõ ý định (động)**: Tạo tối đa BA câu hỏi làm rõ ngữ cảnh ban đầu (không có catalog pre-baked). Chúng PHẢI:
   - Được tạo từ cách diễn đạt của người dùng + các tín hiệu trích xuất từ spec/plan/tasks
   - Chỉ hỏi về thông tin thay đổi đáng kể nội dung checklist
   - Bỏ qua từng câu riêng lẻ nếu đã rõ ràng trong `$ARGUMENTS`
   - Ưu tiên độ chính xác hơn độ rộng

   Thuật toán tạo câu hỏi:
   1. Trích xuất tín hiệu: từ khóa feature domain (ví dụ: auth, latency, UX, API), chỉ báo rủi ro ("critical", "must", "compliance"), gợi ý stakeholder ("QA", "review", "security team"), và deliverables rõ ràng ("a11y", "rollback", "contracts").
   2. Gom nhóm tín hiệu thành các vùng tập trung ứng viên (tối đa 4) xếp hạng theo mức độ liên quan.
   3. Xác định audience và thời điểm probable (author, reviewer, QA, release) nếu không rõ ràng.
   4. Phát hiện các chiều còn thiếu: độ rộng scope, độ sâu/nghiêm ngặt, nhấn mạnh rủi ro, ranh giới loại trừ, tiêu chí chấp nhận đo lường được.
   5. Soạn câu hỏi chọn từ các nguyên mẫu sau:
      - Tinh chỉnh scope (ví dụ: "Có nên bao gồm các touchpoint integration với X và Y hay chỉ giới hạn ở correctness của local module?")
      - Ưu tiên rủi ro (ví dụ: "Các vùng rủi ro tiềm năng nào nên nhận các kiểm tra bắt buộc?")
      - Hiệu chỉnh độ sâu (ví dụ: "Đây là pre-commit sanity list nhẹ hay formal release gate?")
      - Xác định audience (ví dụ: "Checklist này sẽ chỉ được dùng bởi author hay bởi peers khi PR review?")
      - Loại trừ ranh giới (ví dụ: "Có nên loại trừ rõ ràng các mục performance tuning vòng này không?")
      - Khoảng trống scenario class (ví dụ: "Không phát hiện recovery flows — các đường dẫn rollback / partial failure có trong scope không?")

   Quy tắc định dạng câu hỏi:
   - Nếu trình bày các lựa chọn, tạo bảng nhỏ gọn với các cột: Lựa chọn | Ứng viên | Tại sao quan trọng
   - Giới hạn tối đa A–E lựa chọn; bỏ bảng nếu câu trả lời dạng tự do rõ ràng hơn
   - Không bao giờ yêu cầu người dùng lặp lại điều họ đã nói
   - Tránh các danh mục suy đoán (không tưởng tượng). Nếu không chắc chắn, hỏi rõ ràng: "Xác nhận liệu X có thuộc scope không."

   Mặc định khi tương tác không khả thi:
   - Độ sâu: Standard
   - Audience: Reviewer (PR) nếu liên quan code; Author nếu không
   - Tập trung: 2 vùng relevance cao nhất

   Xuất các câu hỏi (đánh nhãn Q1/Q2/Q3). Sau câu trả lời: nếu ≥2 scenario class (Alternate / Exception / Recovery / Non-Functional domain) vẫn chưa rõ, bạn CÓ THỂ hỏi tối đa THÊM HAI câu follow-up có mục tiêu (Q4/Q5) với mỗi câu có một dòng justification (ví dụ: "Rủi ro recovery path chưa được giải quyết"). Không vượt quá năm câu hỏi tổng cộng. Bỏ qua escalation nếu người dùng từ chối rõ ràng.

3. **Hiểu yêu cầu người dùng**: Kết hợp `$ARGUMENTS` + các câu trả lời làm rõ:
   - Suy ra checklist theme (ví dụ: security, review, deploy, ux)
   - Hợp nhất các mục must-have rõ ràng được người dùng đề cập
   - Ánh xạ các lựa chọn tập trung tới cấu trúc category
   - Suy ra ngữ cảnh còn thiếu từ spec/plan/tasks (KHÔNG tưởng tượng)

4. **Tải ngữ cảnh tính năng**: Đọc từ FEATURE_DIR:
   - spec.md: Yêu cầu tính năng và scope
   - plan.md (nếu có): Chi tiết kỹ thuật, dependencies
   - tasks.md (nếu có): Các task triển khai

   **Chiến lược tải ngữ cảnh**:
   - Chỉ tải các phần cần thiết liên quan đến các vùng tập trung đang hoạt động (tránh dump toàn bộ file)
   - Ưu tiên tóm tắt các phần dài thành các bullet scenario/requirement ngắn gọn
   - Sử dụng hiển thị tăng dần: chỉ truy xuất tiếp nếu phát hiện khoảng trống
   - Nếu docs nguồn lớn, tạo các mục tóm tắt tạm thời thay vì nhúng văn bản thô

5. **Tạo checklist** - Tạo "Unit Tests cho Requirements":
   - Tạo thư mục `FEATURE_DIR/checklists/` nếu chưa tồn tại
   - Tạo tên file checklist duy nhất:
     - Dùng tên ngắn, mô tả dựa trên domain (ví dụ: `ux.md`, `api.md`, `security.md`)
     - Định dạng: `[domain].md`
   - Hành vi xử lý file:
     - Nếu file KHÔNG tồn tại: Tạo file mới và đánh số các mục bắt đầu từ CHK001
     - Nếu file tồn tại: Nối các mục mới vào file hiện có, tiếp tục từ CHK ID cuối cùng (ví dụ: nếu mục cuối là CHK015, bắt đầu các mục mới tại CHK016)
   - Không bao giờ xóa hoặc thay thế nội dung checklist hiện có - luôn bảo tồn và nối

   **NGUYÊN TẮC CỐT LÕI - Kiểm tra Requirements, không phải Implementation**:
   Mỗi checklist item PHẢI đánh giá CHÍNH REQUIREMENTS về:
   - **Tính đầy đủ (Completeness)**: Tất cả requirements cần thiết có mặt không?
   - **Sự rõ ràng (Clarity)**: Requirements có không mơ hồ và cụ thể không?
   - **Tính nhất quán (Consistency)**: Requirements có align với nhau không?
   - **Tính đo lường được (Measurability)**: Requirements có thể được xác minh khách quan không?
   - **Coverage**: Tất cả các scenario/edge cases có được address không?

   **Cấu trúc Category** - Gom nhóm các mục theo các chiều chất lượng requirement:
   - **Tính đầy đủ của Requirement** (Tất cả requirements cần thiết có được ghi lại không?)
   - **Sự rõ ràng của Requirement** (Requirements có cụ thể và không mơ hồ không?)
   - **Tính nhất quán của Requirement** (Requirements có align mà không xung đột không?)
   - **Chất lượng Acceptance Criteria** (Tiêu chí thành công có đo lường được không?)
   - **Coverage Scenario** (Tất cả các flows/cases có được address không?)
   - **Coverage Edge Case** (Các điều kiện biên có được định nghĩa không?)
   - **Non-Functional Requirements** (Performance, Security, Accessibility, v.v. - có được chỉ định không?)
   - **Dependencies & Assumptions** (Có được ghi lại và xác thực không?)
   - **Ambiguities & Conflicts** (Cần làm rõ điều gì?)

   **CÁCH VIẾT CÁC MỤC CHECKLIST - "Unit Tests cho tiếng Anh"**:

   ❌ **SAI** (Testing implementation):
   - "Xác nhận landing page hiển thị 3 episode cards"
   - "Kiểm tra hover states hoạt động trên desktop"
   - "Xác nhận logo click navigate về home"

   ✅ **ĐÚNG** (Testing chất lượng requirements):
   - "Có chỉ định số lượng và layout chính xác của featured episodes không?" [Completeness]
   - "'Hiển thị nổi bật' đã được định lượng bằng kích thước/vị trí cụ thể chưa?" [Clarity]
   - "Các yêu cầu hover state có nhất quán trên tất cả interactive elements không?" [Consistency]
   - "Có định nghĩa yêu cầu keyboard navigation cho tất cả interactive UI không?" [Coverage]
   - "Có chỉ định fallback behavior khi logo image fail load không?" [Edge Cases]
   - "Có định nghĩa loading states cho asynchronous episode data không?" [Completeness]
   - "Spec có định nghĩa visual hierarchy cho các UI elements cạnh tranh nhau không?" [Clarity]

   **CẤU TRÚC MỤC**:
   Mỗi mục nên theo pattern này:
   - Định dạng câu hỏi hỏi về chất lượng requirement
   - Tập trung vào những gì ĐƯỢC VIẾT (hoặc không viết) trong spec/plan
   - Bao gồm chiều chất lượng trong ngoặc vuông [Completeness/Clarity/Consistency/etc.]
   - Tham chiếu section spec `[Spec §X.Y]` khi kiểm tra requirements hiện có
   - Dùng marker `[Gap]` khi kiểm tra requirements còn thiếu

   **VÍ DỤ THEO CHIỀU CHẤT LƯỢNG**:

   Completeness:
   - "Có định nghĩa yêu cầu xử lý lỗi cho tất cả API failure modes không? [Gap]"
   - "Có chỉ định yêu cầu accessibility cho tất cả interactive elements không? [Completeness]"
   - "Có định nghĩa yêu cầu mobile breakpoint cho responsive layouts không? [Gap]"

   Clarity:
   - "'Fast loading' đã được định lượng bằng ngưỡng thời gian cụ thể chưa? [Clarity, Spec §NFR-2]"
   - "Tiêu chí lựa chọn 'related episodes' có được định nghĩa rõ ràng không? [Clarity, Spec §FR-5]"
   - "'Nổi bật' có được định nghĩa bằng các thuộc tính visual có thể đo lường không? [Ambiguity, Spec §FR-4]"

   Consistency:
   - "Các yêu cầu navigation có align trên tất cả các trang không? [Consistency, Spec §FR-10]"
   - "Các yêu cầu card component có nhất quán giữa landing và detail pages không? [Consistency]"

   Coverage:
   - "Có định nghĩa requirements cho zero-state scenarios (không có episodes) không? [Coverage, Edge Case]"
   - "Các kịch bản tương tác người dùng đồng thời có được address không? [Coverage, Gap]"
   - "Có chỉ định requirements cho partial data loading failures không? [Coverage, Exception Flow]"

   Measurability:
   - "Các yêu cầu visual hierarchy có thể đo lường/kiểm thử được không? [Acceptance Criteria, Spec §FR-1]"
   - "'Balanced visual weight' có thể được xác minh khách quan không? [Measurability, Spec §FR-2]"

   **Phân loại Scenario & Coverage** (Tập trung vào Chất lượng Requirements):
   - Kiểm tra requirements có tồn tại cho: Primary, Alternate, Exception/Error, Recovery, Non-Functional scenarios
   - Với mỗi scenario class, hỏi: "Requirements [scenario type] có đầy đủ, rõ ràng và nhất quán không?"
   - Nếu scenario class thiếu: "Requirements [scenario type] có được loại trừ có chủ đích hay thiếu? [Gap]"
   - Bao gồm resilience/rollback khi state mutation xảy ra: "Có định nghĩa requirements rollback cho migration failures không? [Gap]"

   **Yêu cầu Traceability**:
   - TỐI THIỂU: ≥80% các mục PHẢI bao gồm ít nhất một tham chiếu traceability
   - Mỗi mục nên tham chiếu: section spec `[Spec §X.Y]`, hoặc dùng các markers: `[Gap]`, `[Ambiguity]`, `[Conflict]`, `[Assumption]`
   - Nếu không có hệ thống ID: "Có thiết lập scheme ID cho requirement & acceptance criteria chưa? [Traceability]"

   **Phát hiện & Giải quyết Issues** (Các vấn đề Chất lượng Requirements):
   Hỏi các câu hỏi về chính requirements:
   - Ambiguities: "Thuật ngữ 'fast' đã được định lượng bằng metrics cụ thể chưa? [Ambiguity, Spec §NFR-1]"
   - Conflicts: "Các yêu cầu navigation có xung đột giữa §FR-10 và §FR-10a không? [Conflict]"
   - Assumptions: "Giả định 'podcast API luôn available' đã được xác thực chưa? [Assumption]"
   - Dependencies: "Có ghi lại requirements cho external podcast API không? [Dependency, Gap]"
   - Thiếu định nghĩa: "'Visual hierarchy' có được định nghĩa bằng tiêu chí đo lường được không? [Gap]"

   **Hợp nhất nội dung**:
   - Soft cap: Nếu các mục ứng viên thô > 40, ưu tiên theo risk/impact
   - Hợp nhất các gần-trùng lặp kiểm tra cùng khía cạnh requirement
   - Nếu >5 edge cases tác động thấp, tạo một mục: "Các edge cases X, Y, Z có được address trong requirements không? [Coverage]"

   **🚫 TUYỆT ĐỐI CẤM** - Đây làm cho nó trở thành implementation test, không phải requirements test:
   - ❌ Bất kỳ mục nào bắt đầu bằng "Verify", "Test", "Confirm", "Check" + implementation behavior
   - ❌ Tham chiếu đến code execution, user actions, system behavior
   - ❌ "Hiển thị đúng", "hoạt động đúng", "chức năng như mong đợi"
   - ❌ "Click", "navigate", "render", "load", "execute"
   - ❌ Test cases, test plans, QA procedures
   - ❌ Implementation details (frameworks, APIs, algorithms)

   **✅ CÁC PATTERN BẮT BUỘC** - Đây kiểm tra chất lượng requirements:
   - ✅ "Có định nghĩa/chỉ định/ghi lại [requirement type] cho [scenario] không?"
   - ✅ "[Thuật ngữ mơ hồ] đã được định lượng/làm rõ bằng tiêu chí cụ thể chưa?"
   - ✅ "Requirements có nhất quán giữa [section A] và [section B] không?"
   - ✅ "[Requirement] có thể được đo lường/xác minh khách quan không?"
   - ✅ "[Edge cases/scenarios] có được address trong requirements không?"
   - ✅ "Spec có định nghĩa [khía cạnh còn thiếu] không?"

6. **Tham chiếu cấu trúc**: Tạo checklist theo template chuẩn trong `.specify/templates/checklist-template.md` cho tiêu đề, phần meta, các heading category và định dạng ID. Nếu template không khả dụng, dùng: tiêu đề H1, các dòng meta purpose/created, các phần `##` category chứa các dòng `- [ ] CHK### <requirement item>` với ID tăng dần toàn cục bắt đầu từ CHK001.

7. **Báo cáo**: Xuất đường dẫn đầy đủ tới file checklist, số lượng item, và tóm tắt liệu run tạo file mới hay nối vào file hiện có. Tóm tắt:
   - Các vùng tập trung được chọn
   - Mức độ sâu
   - Actor/thời điểm
   - Bất kỳ mục must-have nào được người dùng chỉ định rõ đã được tích hợp

**Quan trọng**: Mỗi lần gọi lệnh `/speckit.checklist` sử dụng tên file checklist ngắn, mô tả và tạo file mới hoặc nối vào file hiện có. Điều này cho phép:

- Nhiều checklist các loại khác nhau (ví dụ: `ux.md`, `test.md`, `security.md`)
- Tên file đơn giản, dễ nhớ thể hiện mục đích checklist
- Dễ nhận diện và điều hướng trong thư mục `checklists/`

Để tránh lộn xộn, dùng các loại mô tả và dọn dẹp các checklist lỗi thời khi xong.

## Ví dụ các loại Checklist & Mục mẫu

**Chất lượng Requirements UX:** `ux.md`

Các mục mẫu (kiểm tra requirements, KHÔNG phải implementation):

- "Có định nghĩa yêu cầu visual hierarchy bằng tiêu chí đo lường được không? [Clarity, Spec §FR-1]"
- "Số lượng và vị trí của UI elements có được chỉ định rõ ràng không? [Completeness, Spec §FR-1]"
- "Các yêu cầu interaction state (hover, focus, active) có được định nghĩa nhất quán không? [Consistency]"
- "Có chỉ định yêu cầu accessibility cho tất cả interactive elements không? [Coverage, Gap]"
- "Có định nghĩa fallback behavior khi images fail load không? [Edge Case, Gap]"
- "'Hiển thị nổi bật' có thể được đo lường khách quan không? [Measurability, Spec §FR-4]"

**Chất lượng Requirements API:** `api.md`

Các mục mẫu:

- "Có chỉ định định dạng error response cho tất cả failure scenarios không? [Completeness]"
- "Yêu cầu rate limiting đã được định lượng bằng ngưỡng cụ thể chưa? [Clarity]"
- "Yêu cầu authentication có nhất quán trên tất cả endpoints không? [Consistency]"
- "Có định nghĩa yêu cầu retry/timeout cho external dependencies không? [Coverage, Gap]"
- "Chiến lược versioning có được ghi lại trong requirements không? [Gap]"

**Chất lượng Requirements Hiệu năng:** `performance.md`

Các mục mẫu:

- "Yêu cầu hiệu năng đã được định lượng bằng metrics cụ thể chưa? [Clarity]"
- "Có định nghĩa performance targets cho tất cả critical user journeys không? [Coverage]"
- "Yêu cầu hiệu năng dưới các điều kiện load khác nhau có được chỉ định không? [Completeness]"
- "Yêu cầu hiệu năng có thể được đo lường khách quan không? [Measurability]"
- "Có định nghĩa yêu cầu degradation cho high-load scenarios không? [Edge Case, Gap]"

**Chất lượng Requirements Bảo mật:** `security.md`

Các mục mẫu:

- "Có chỉ định yêu cầu authentication cho tất cả protected resources không? [Coverage]"
- "Có định nghĩa yêu cầu bảo vệ dữ liệu cho thông tin nhạy cảm không? [Completeness]"
- "Threat model có được ghi lại và requirements có align với nó không? [Traceability]"
- "Yêu cầu bảo mật có nhất quán với các nghĩa vụ tuân thủ không? [Consistency]"
- "Có định nghĩa yêu cầu response cho security failure/breach không? [Gap, Exception Flow]"

## Các Anti-Examples: KHÔNG NÊN làm gì

**❌ SAI - Đây kiểm tra implementation, không phải requirements:**

```markdown
- [ ] CHK001 - Xác nhận landing page hiển thị 3 episode cards [Spec §FR-001]
- [ ] CHK002 - Kiểm tra hover states hoạt động đúng trên desktop [Spec §FR-003]
- [ ] CHK003 - Xác nhận logo click navigate về home page [Spec §FR-010]
- [ ] CHK004 - Kiểm tra related episodes section hiển thị 3-5 items [Spec §FR-005]
```

**✅ ĐÚNG - Đây kiểm tra chất lượng requirements:**

```markdown
- [ ] CHK001 - Có chỉ định rõ ràng số lượng và layout của featured episodes không? [Completeness, Spec §FR-001]
- [ ] CHK002 - Các yêu cầu hover state có được định nghĩa nhất quán cho tất cả interactive elements không? [Consistency, Spec §FR-003]
- [ ] CHK003 - Các yêu cầu navigation có rõ ràng cho tất cả clickable brand elements không? [Clarity, Spec §FR-010]
- [ ] CHK004 - Tiêu chí lựa chọn related episodes có được ghi lại không? [Gap, Spec §FR-005]
- [ ] CHK005 - Có định nghĩa yêu cầu loading state cho asynchronous episode data không? [Gap]
- [ ] CHK006 - Các yêu cầu 'visual hierarchy' có thể được đo lường khách quan không? [Measurability, Spec §FR-001]
```

**Sự khác biệt chính:**

- Sai: Kiểm tra hệ thống có hoạt động đúng không
- Đúng: Kiểm tra requirements có được viết đúng không
- Sai: Xác nhận hành vi
- Đúng: Xác thực chất lượng requirement
- Sai: "Nó có làm X không?"
- Đúng: "X có được chỉ định rõ ràng không?"
