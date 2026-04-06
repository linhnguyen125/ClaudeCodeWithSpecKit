---
name: "speckit-specify"
description: "Tạo hoặc cập nhật feature specification từ mô tả feature bằng ngôn ngữ tự nhiên."
argument-hint: "Mô tả feature bạn muốn specify"
compatibility: "Yêu cầu cấu trúc project spec-kit với thư mục .specify/"
metadata:
  author: "github-spec-kit"
  source: "templates/commands/specify.md"
user-invocable: true
disable-model-invocation: true
---


## Đầu vào từ người dùng

```text
$ARGUMENTS
```

Bạn **PHẢI** xem xét đầu vào từ người dùng trước khi tiếp tục (nếu không rỗng).

## Kiểm tra trước khi thực thi

**Kiểm tra extension hooks (trước khi specification)**:
- Kiểm tra xem `.specify/extensions.yml` có tồn tại trong thư mục gốc của project hay không.
- Nếu tồn tại, đọc file và tìm các mục dưới key `hooks.before_specify`
- Nếu YAML không thể parse hoặc không hợp lệ, bỏ qua việc kiểm tra hook một cách im lặng và tiếp tục bình thường
- Lọc ra các hooks có `enabled` được đặt rõ ràng là `false`. Các hooks không có trường `enabled` được coi là enabled theo mặc định.
- Với mỗi hook còn lại, **KHÔNG** cố gắng diễn giải hoặc đánh giá các biểu thức `condition` của hook:
  - Nếu hook không có trường `condition`, hoặc nó là null/rỗng, coi hook đó là có thể thực thi
  - Nếu hook định nghĩa một `condition` không rỗng, bỏ qua hook và để việc đánh giá condition cho implementation của HookExecutor
- Với mỗi hook có thể thực thi, xuất output sau đây dựa trên flag `optional` của nó:
  - **Optional hook** (`optional: true`):
    ```
    ## Extension Hooks

    **Optional Pre-Hook**: {extension}
    Command: `/{command}`
    Description: {description}

    Prompt: {prompt}
    To execute: `/{command}`
    ```
  - **Mandatory hook** (`optional: false`):
    ```
    ## Extension Hooks

    **Automatic Pre-Hook**: {extension}
    Executing: `/{command}`
    EXECUTE_COMMAND: {command}

    Đợi kết quả của hook command trước khi tiến hành Outline.
    ```
- Nếu không có hooks nào được đăng ký hoặc `.specify/extensions.yml` không tồn tại, bỏ qua một cách im lặng

## Outline

Text mà người dùng gõ sau `/speckit.specify` trong message kích hoạt **chính là** feature description. Giả định bạn luôn có nó trong cuộc trò chuyện này ngay cả khi `$ARGUMENTS` xuất hiện nguyên dạng bên dưới. Không hỏi người dùng nhắc lại trừ khi họ cung cấp một command rỗng.

Với feature description đó, thực hiện:

1. **Tạo một short name ngắn gọn** (2-4 từ) cho branch:
   - Phân tích feature description và trích xuất các keywords có ý nghĩa nhất
   - Tạo short name 2-4 từ nắm bắt bản chất của feature
   - Ưu tiên dùng format action-noun khi có thể (ví dụ: "add-user-auth", "fix-payment-bug")
   - Giữ nguyên các thuật ngữ kỹ thuật và acronyms (OAuth2, API, JWT, v.v.)
   - Giữ ngắn gọn nhưng đủ mô tả để hiểu feature ngay nhìn
   - Ví dụ:
     - "I want to add user authentication" → "user-auth"
     - "Implement OAuth2 integration for the API" → "oauth2-api-integration"
     - "Create a dashboard for analytics" → "analytics-dashboard"
     - "Fix payment processing timeout bug" → "fix-payment-timeout"

2. **Tạo feature branch** bằng cách chạy script với `--short-name` (và `--json`). Ở sequential mode, KHÔNG truyền `--number` — script tự động detect số tiếp theo. Ở timestamp mode, script tự động tạo prefix `YYYYMMDD-HHMMSS`:

   **Branch numbering mode**: Trước khi chạy script, kiểm tra `.specify/init-options.json` và đọc giá trị `branch_numbering`.
   - Nếu là `"timestamp"`, thêm `--timestamp` (Bash) hoặc `-Timestamp` (PowerShell) vào lời gọi script
   - Nếu là `"sequential"` hoặc không có, không thêm flag nào (behavior mặc định)

   - Ví dụ Bash: `.specify/scripts/powershell/create-new-feature.ps1 "$ARGUMENTS" --json --short-name "user-auth" "Add user authentication"`
   - Bash (timestamp): `.specify/scripts/powershell/create-new-feature.ps1 "$ARGUMENTS" --json --timestamp --short-name "user-auth" "Add user authentication"`
   - Ví dụ PowerShell: `.specify/scripts/powershell/create-new-feature.ps1 "$ARGUMENTS" -Json -ShortName "user-auth" "Add user authentication"`
   - PowerShell (timestamp): `.specify/scripts/powershell/create-new-feature.ps1 "$ARGUMENTS" -Json -Timestamp -ShortName "user-auth" "Add user authentication"`

   **QUAN TRỌNG**:
   - KHÔNG truyền `--number` — script tự xác định số tiếp theo đúng
   - Luôn bao gồm JSON flag (`--json` cho Bash, `-Json` cho PowerShell) để output có thể được parse đáng tin cậy
   - Bạn chỉ được chạy script này một lần duy nhất cho mỗi feature
   - JSON được cung cấp trong terminal dưới dạng output — luôn tham chiếu đến nó để lấy nội dung thực sự bạn đang tìm
   - JSON output sẽ chứa BRANCH_NAME và đường dẫn SPEC_FILE
   - Với single quotes trong args như "I'm Groot", sử dụng cú pháp escape: ví dụ 'I'\''m Groot' (hoặc dùng double-quote nếu có thể: "I'm Groot")

3. Load `.specify/templates/spec-template.md` để hiểu các sections bắt buộc.

4. Tuân theo execution flow này:

    1. Parse user description từ Input
       Nếu rỗng: ERROR "No feature description provided"
    2. Trích xuất key concepts từ description
       Xác định: actors, actions, data, constraints
    3. Với các khía cạnh chưa rõ:
       - Đưa ra các guesses có căn cứ dựa trên context và industry standards
       - Chỉ đánh dấu với [NEEDS CLARIFICATION: specific question] nếu:
         - Lựa chọn ảnh hưởng đáng kể đến feature scope hoặc user experience
         - Có nhiều cách diễn giải hợp lý với các implications khác nhau
         - Không có default hợp lý nào tồn tại
       - **GIỚI HẠN: Tối đa 3 markers [NEEDS CLARIFICATION]**
       - Ưu tiên clarifications theo impact: scope > security/privacy > user experience > technical details
    4. Điền User Scenarios & Testing section
       Nếu không có clear user flow: ERROR "Cannot determine user scenarios"
    5. Tạo Functional Requirements
       Mỗi requirement phải có thể test được
       Sử dụng các defaults hợp lý cho các chi tiết chưa được chỉ định (document assumptions trong Assumptions section)
    6. Định nghĩa Success Criteria
       Tạo các outcomes đo lường được, không phụ thuộc công nghệ
       Bao gồm cả metrics định lượng (time, performance, volume) và các đo lường định tính (user satisfaction, task completion)
       Mỗi criterion phải có thể verify mà không cần implementation details
    7. Xác định Key Entities (nếu có data liên quan)
    8. Return: SUCCESS (spec sẵn sàng cho planning)

5. Viết specification vào SPEC_FILE sử dụng template structure, thay thế các placeholders bằng các chi tiết cụ thể từ feature description (arguments) trong khi giữ nguyên thứ tự và headings của các sections.

6. **Specification Quality Validation**: Sau khi viết spec ban đầu, validate nó đối với các tiêu chí chất lượng:

   a. **Tạo Spec Quality Checklist**: Tạo file checklist tại `FEATURE_DIR/checklists/requirements.md` sử dụng cấu trúc checklist template với các validation items:

      ```markdown
      # Specification Quality Checklist: [FEATURE NAME]

      **Purpose**: Validate specification completeness and quality before proceeding to planning
      **Created**: [DATE]
      **Feature**: [Link to spec.md]

      ## Content Quality

      - [ ] No implementation details (languages, frameworks, APIs)
      - [ ] Focused on user value and business needs
      - [ ] Written for non-technical stakeholders
      - [ ] All mandatory sections completed

      ## Requirement Completeness

      - [ ] No [NEEDS CLARIFICATION] markers remain
      - [ ] Requirements are testable and unambiguous
      - [ ] Success criteria are measurable
      - [ ] Success criteria are technology-agnostic (no implementation details)
      - [ ] All acceptance scenarios are defined
      - [ ] Edge cases are identified
      - [ ] Scope is clearly bounded
      - [ ] Dependencies and assumptions identified

      ## Feature Readiness

      - [ ] All functional requirements have clear acceptance criteria
      - [ ] User scenarios cover primary flows
      - [ ] Feature meets measurable outcomes defined in Success Criteria
      - [ ] No implementation details leak into specification

      ## Notes

      - Items marked incomplete require spec updates before `/speckit.clarify` or `/speckit.plan`
      ```

   b. **Run Validation Check**: Review spec đối với từng checklist item:
      - Với mỗi item, xác định nó pass hay fail
      - Document các issues cụ thể tìm được (quote các spec sections liên quan)

   c. **Handle Validation Results**:

      - **Nếu tất cả items pass**: Đánh dấu checklist complete và tiến hành bước 7

      - **Nếu items fail (không tính [NEEDS CLARIFICATION])**:
        1. Liệt kê các failing items và các issues cụ thể
        2. Cập nhật spec để giải quyết từng issue
        3. Chạy lại validation cho đến khi tất cả items pass (tối đa 3 iterations)
        4. Nếu vẫn fail sau 3 iterations, document các issues còn lại trong checklist notes và warn user

      - **Nếu [NEEDS CLARIFICATION] markers còn tồn tại**:
        1. Trích xuất tất cả [NEEDS CLARIFICATION: ...] markers từ spec
        2. **KIỂM TRA GIỚI HẠN**: Nếu có hơn 3 markers, chỉ giữ lại 3 cái quan trọng nhất (theo scope/security/UX impact) và đưa ra informed guesses cho các cái khác
        3. Với mỗi clarification cần thiết (tối đa 3), present các options cho user theo format này:

           ```markdown
           ## Question [N]: [Topic]

           **Context**: [Quote relevant spec section]

           **What we need to know**: [Specific question from NEEDS CLARIFICATION marker]

           **Suggested Answers**:

           | Option | Answer | Implications |
           |--------|--------|--------------|
           | A      | [First suggested answer] | [What this means for the feature] |
           | B      | [Second suggested answer] | [What this means for the feature] |
           | C      | [Third suggested answer] | [What this means for the feature] |
           | Custom | Provide your own answer | [Explain how to provide custom input] |

           **Your choice**: _[Wait for user response]_
           ```

        4. **QUAN TRỌNG - Table Formatting**: Đảm bảo markdown tables được format đúng:
           - Sử dụng spacing nhất quán với các pipes được căn chỉnh
           - Mỗi cell nên có spaces xung quanh nội dung: `| Content |` không phải `|Content|`
           - Header separator phải có ít nhất 3 dấu gạch ngang: `|--------|`
           - Test xem table có render đúng trong markdown preview
        5. Đánh số questions tuần tự (Q1, Q2, Q3 - tối đa 3)
        6. Present tất cả questions cùng nhau trước khi đợi responses
        7. Đợi user trả lời với lựa chọn của họ cho tất cả questions (ví dụ: "Q1: A, Q2: Custom - [details], Q3: B")
        8. Cập nhật spec bằng cách thay thế mỗi [NEEDS CLARIFICATION] marker bằng câu trả lời đã chọn hoặc được cung cấp bởi user
        9. Chạy lại validation sau khi tất cả clarifications được giải quyết

   d. **Update Checklist**: Sau mỗi validation iteration, cập nhật checklist file với pass/fail status hiện tại

7. Báo cáo completion với branch name, spec file path, checklist results, và readiness cho phase tiếp theo (`/speckit.clarify` hoặc `/speckit.plan`).

8. **Kiểm tra extension hooks**: Sau khi báo cáo completion, kiểm tra xem `.specify/extensions.yml` có tồn tại trong thư mục gốc của project hay không.
   - Nếu tồn tại, đọc file và tìm các mục dưới key `hooks.after_specify`
   - Nếu YAML không thể parse hoặc không hợp lệ, bỏ qua việc kiểm tra hook một cách im lặng và tiếp tục bình thường
   - Lọc ra các hooks có `enabled` được đặt rõ ràng là `false`. Các hooks không có trường `enabled` được coi là enabled theo mặc định.
   - Với mỗi hook còn lại, **KHÔNG** cố gắng diễn giải hoặc đánh giá các biểu thức `condition` của hook:
     - Nếu hook không có trường `condition`, hoặc nó là null/rỗng, coi hook đó là có thể thực thi
     - Nếu hook định nghĩa một `condition` không rỗng, bỏ qua hook và để việc đánh giá condition cho implementation của HookExecutor
   - Với mỗi hook có thể thực thi, xuất output sau đây dựa trên flag `optional` của nó:
     - **Optional hook** (`optional: true`):
       ```
       ## Extension Hooks

       **Optional Hook**: {extension}
       Command: `/{command}`
       Description: {description}

       Prompt: {prompt}
       To execute: `/{command}`
       ```
     - **Mandatory hook** (`optional: false`):
       ```
       ## Extension Hooks

       **Automatic Hook**: {extension}
       Executing: `/{command}`
       EXECUTE_COMMAND: {command}
       ```
   - Nếu không có hooks nào được đăng ký hoặc `.specify/extensions.yml` không tồn tại, bỏ qua một cách im lặng

**NOTE:** Script tạo và checkout new branch và khởi tạo spec file trước khi viết.

## Quick Guidelines

- Tập trung vào **WHAT** người dùng cần và **WHY**.
- Tránh HOW để implement (không có tech stack, APIs, code structure).
- Viết cho business stakeholders, không phải developers.
- KHÔNG tạo bất kỳ checklists nào được nhúng trong spec. Đó sẽ là một command riêng.

### Section Requirements

- **Mandatory sections**: Phải được hoàn thành cho mỗi feature
- **Optional sections**: Chỉ include khi liên quan đến feature
- Khi một section không áp dụng, xóa nó hoàn toàn (không để là "N/A")

### For AI Generation

Khi tạo spec này từ user prompt:

1. **Make informed guesses**: Sử dụng context, industry standards và common patterns để lấp đầy gaps
2. **Document assumptions**: Ghi lại các defaults hợp lý trong Assumptions section
3. **Limit clarifications**: Tối đa 3 markers [NEEDS CLARIFICATION] - chỉ dùng cho các quyết định quan trọng:
   - Ảnh hưởng đáng kể đến feature scope hoặc user experience
   - Có nhiều cách diễn giải hợp lý với các implications khác nhau
   - Thiếu bất kỳ default hợp lý nào
4. **Prioritize clarifications**: scope > security/privacy > user experience > technical details
5. **Think like a tester**: Mọi requirement mơ hồ nên fail item "testable and unambiguous" trong checklist
6. **Các vùng phổ biến cần clarification** (chỉ nếu không có default hợp lý):
   - Feature scope và boundaries (include/exclude các use cases cụ thể)
   - User types và permissions (nếu có nhiều cách diễn giải xung đột)
   - Security/compliance requirements (khi có ý nghĩa pháp lý/tài chính)

**Examples of reasonable defaults** (không hỏi về những cái này):

- Data retention: Industry-standard practices cho domain đó
- Performance targets: Standard web/mobile app expectations trừ khi được chỉ định
- Error handling: User-friendly messages với các fallbacks phù hợp
- Authentication method: Standard session-based hoặc OAuth2 cho web apps
- Integration patterns: Sử dụng patterns phù hợp với project (REST/GraphQL cho web services, function calls cho libraries, CLI args cho tools, v.v.)

### Success Criteria Guidelines

Success criteria phải:

1. **Measurable**: Bao gồm các metrics cụ thể (time, percentage, count, rate)
2. **Technology-agnostic**: Không đề cập đến frameworks, languages, databases, hoặc tools
3. **User-focused**: Mô tả outcomes từ góc độ user/business, không phải system internals
4. **Verifiable**: Có thể được test/validate mà không cần biết implementation details

**Good examples**:

- "Users can complete checkout in under 3 minutes"
- "System supports 10,000 concurrent users"
- "95% of searches return results in under 1 second"
- "Task completion rate improves by 40%"

**Bad examples** (implementation-focused):

- "API response time is under 200ms" (quá technical, dùng "Users see results instantly")
- "Database can handle 1000 TPS" (implementation detail, dùng user-facing metric)
- "React components render efficiently" (framework-specific)
- "Redis cache hit rate above 80%" (technology-specific)
