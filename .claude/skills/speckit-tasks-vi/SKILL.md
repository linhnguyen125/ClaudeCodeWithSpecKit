---
name: "speckit-tasks"
description: "Tạo file tasks.md có thể thực thi, được sắp xếp theo thứ tự phụ thuộc cho tính năng dựa trên các artifact thiết kế sẵn có."
argument-hint: "Ràng buộc tùy chọn cho việc tạo task"
compatibility: "Yêu cầu cấu trúc project spec-kit với thư mục .specify/"
metadata:
  author: "github-spec-kit"
  source: "templates/commands/tasks.md"
user-invocable: true
disable-model-invocation: true
---

## Đầu vào từ người dùng

```text
$ARGUMENTS
```

Bạn **BẮT BUỘC** phải xem xét đầu vào từ người dùng trước khi tiếp tục (nếu không rỗng).

## Kiểm tra trước khi thực thi

**Kiểm tra extension hooks (trước khi tạo tasks)**:
- Kiểm tra xem `.specify/extensions.yml` có tồn tại trong thư mục gốc của project hay không.
- Nếu tồn tại, đọc và tìm các mục trong key `hooks.before_tasks`
- Nếu YAML không thể parse hoặc không hợp lệ, bỏ qua kiểm tra hook một cách âm thầm và tiếp tục bình thường
- Lọc ra các hook có `enabled` được đặt rõ ràng là `false`. Các hook không có trường `enabled` được coi là enabled theo mặc định.
- Với mỗi hook còn lại, **KHÔNG** cố gắng diễn giải hoặc đánh giá biểu thức `condition` của hook:
  - Nếu hook không có trường `condition`, hoặc nó là null/rỗng, coi hook là có thể thực thi
  - Nếu hook định nghĩa `condition` không rỗng, bỏ qua hook và để việc đánh giá condition cho HookExecutor implementation
- Với mỗi hook có thể thực thi, xuất ra thông tin sau dựa trên flag `optional` của nó:
  - **Hook tùy chọn** (`optional: true`):
    ```
    ## Extension Hooks

    **Optional Pre-Hook**: {extension}
    Command: `/{command}`
    Description: {description}

    Prompt: {prompt}
    To execute: `/{command}`
    ```
  - **Hook bắt buộc** (`optional: false`):
    ```
    ## Extension Hooks

    **Automatic Pre-Hook**: {extension}
    Executing: `/{command}`
    EXECUTE_COMMAND: {command}

    Chờ kết quả của hook command trước khi tiến hành Outline.
    ```
- Nếu không có hook nào được đăng ký hoặc `.specify/extensions.yml` không tồn tại, bỏ qua một cách âm thầm

## Outline

1. **Setup**: Chạy `.specify/scripts/powershell/check-prerequisites.ps1 -Json` từ repo root và parse FEATURE_DIR và AVAILABLE_DOCS list. Tất cả đường dẫn phải là tuyệt đối. Với single quotes trong args như "I'm Groot", sử dụng cú pháp escape: ví dụ 'I'\''m Groot' (hoặc dùng double-quote nếu có thể: "I'm Groot").

2. **Load design documents**: Đọc từ FEATURE_DIR:
   - **Bắt buộc**: plan.md (tech stack, libraries, structure), spec.md (user stories với độ ưu tiên)
   - **Tùy chọn**: data-model.md (entities), contracts/ (interface contracts), research.md (quyết định), quickstart.md (test scenarios)
   - Lưu ý: Không phải mọi project đều có đủ các document. Tạo tasks dựa trên những gì có sẵn.

3. **Analyze scope và spawn team**:
   - Analyze plan.md → xác định FE/BE scope (cả 2, chỉ FE, hoặc chỉ BE)
   - Nếu có cả FE + BE: TeamCreate → spawn fe-tasks + be-tasks song song
   - Nếu chỉ FE hoặc BE: Spawn chỉ agent cần thiết
   - Xem Team Orchestration section bên dưới cho chi tiết workflow

4. **Subagent tasks (parallel)**:
   - **fe-tasks**: Extract FE requirements → generate FE tasks
   - **be-tasks**: Extract BE requirements → generate BE tasks
   - Sync via SendMessage khi cần cross-agent info

5. **Merge (lead only)**: tasks-lead merge FE + BE sections → final tasks.md

6. **Report**: Xuất đường dẫn đến tasks.md đã tạo và tóm tắt:
   - Tổng số task
   - Số task cho mỗi user story
   - Các cơ hội parallel đã xác định
   - Tiêu chí test độc lập cho mỗi story
   - Suggested MVP scope (thường chỉ User Story 1)
   - Format validation: Xác nhận TẤT CẢ tasks đều tuân theo checklist format (checkbox, ID, labels, file paths)

7. **Shutdown**: Send shutdown_request → TeamDelete

8. **Check cho extension hooks**: Sau khi tasks.md được tạo, kiểm tra xem `.specify/extensions.yml` có tồn tại trong thư mục gốc của project hay không.
   - Nếu tồn tại, đọc và tìm các mục trong key `hooks.after_tasks`
   - Nếu YAML không thể parse hoặc không hợp lệ, bỏ qua kiểm tra hook một cách âm thầm và tiếp tục bình thường
   - Lọc ra các hook có `enabled` được đặt rõ ràng là `false`. Các hook không có trường `enabled` được coi là enabled theo mặc định.
   - Với mỗi hook còn lại, **KHÔNG** cố gắng diễn giải hoặc đánh giá biểu thức `condition` của hook:
     - Nếu hook không có trường `condition`, hoặc nó là null/rỗng, coi hook là có thể thực thi
     - Nếu hook định nghĩa `condition` không rỗng, bỏ qua hook và để việc đánh giá condition cho HookExecutor implementation
   - Với mỗi hook có thể thực thi, xuất ra thông tin sau dựa trên flag `optional` của nó:
     - **Hook tùy chọn** (`optional: true`):
       ```
       ## Extension Hooks

       **Optional Hook**: {extension}
       Command: `/{command}`
       Description: {description}

       Prompt: {prompt}
       To execute: `/{command}`
       ```
     - **Hook bắt buộc** (`optional: false`):
       ```
       ## Extension Hooks

       **Automatic Hook**: {extension}
       Executing: `/{command}`
       EXECUTE_COMMAND: {command}
       ```
   - Nếu không có hook nào được đăng ký hoặc `.specify/extensions.yml` không tồn tại, bỏ qua một cách âm thầm

## Team Orchestration

Skill này hỗ trợ **multi-agent task generation** cho FlowPilot monorepo với 2 tech stack khác nhau:
- **FlowPilotUI**: Electron + Vue 3 + TypeScript
- **FlowPilotBE**: Python + FastAPI

### Team Structure

```
tasks-lead (main session - skill executor)
├── fe-tasks (subagent) ← chạy song song
└── be-tasks (subagent) ← chạy song song
```

### Khi nào spawn team?

| Scenario | Behavior |
|----------|----------|
| plan.md có cả FE + BE | Spawn cả 2 subagent, chạy song song |
| plan.md chỉ có FE | Spawn chỉ fe-tasks, bypass BE |
| plan.md chỉ có BE | Spawn chỉ be-tasks, bypass FE |
| FlowPilotBE chưa có code | BE agent vẫn generate tasks dựa trên plan, không research existing code |

### Workflow

```
1. Setup → Run check-prerequisites.ps1 → parse FEATURE_DIR
2. Load design documents: plan.md, spec.md, data-model.md, contracts/
3. Analyze plan.md → xác định FE/BE scope
4. TeamCreate("flowpilot-tasks", "Multi-agent task generation cho FlowPilot")
5. Create tasks:
   - "fe-tasks": unblocked (generate FE section)
   - "be-tasks": unblocked (generate BE section)
   - "merge": blocked by fe-tasks AND be-tasks
6. Spawn fe-tasks + be-tasks (nếu cần)
7. fe-tasks + be-tasks (parallel):
   - Extract FE/BE requirements từ plan.md
   - Map user stories từ spec.md theo thứ tự ưu tiên
   - Generate tasks theo checklist format
8. tasks-lead: Merge → viết final tasks.md
9. Report
10. Send shutdown_request → TeamDelete
```

### Sync Protocol

Khi fe-tasks hoặc be-tasks cần info từ agent kia:

```
fe-tasks cần BE contract info
→ SendMessage(to: "be-tasks", message: "Need BE contract info for {topic}")
→ be-tasks responds via SendMessage
→ fe-tasks tiếp tục generate

be-tasks cần FE contract info
→ SendMessage(to: "fe-tasks", message: "Need FE contract info for {topic}")
→ fe-tasks responds via SendMessage
→ be-tasks tiếp tục generate
```

### Output: tasks.md

tasks-lead merge outputs thành 1 file `tasks.md` theo format của `.specify/templates/tasks-template.md`:

```yaml
---
description: "Task list template for feature implementation"
---

# Tasks: {FEATURE NAME}

**Input**: Design documents from `/specs/{feature-name}/`
**Prerequisites**: plan.md (required), spec.md (required for user stories), research.md, data-model.md, contracts/

[... rest of tasks content ...]
```

File phải có:
- YAML frontmatter với `description` field ở đầu file
- Phase 1: Setup (FE + BE shared infrastructure)
- Phase 2: Foundational (FE + BE shared blocking tasks)
- Phase 3+: User Stories (FE tasks + BE tasks merged by story)
- Final Phase: Polish

## Các Phase (Multi-Agent)

### Phase 1: Extract & Map (Parallel)

**fe-tasks** task: Trích xuất FE requirements từ plan.md
**be-tasks** task: Trích xuất BE requirements từ plan.md

Both agents:
- Đọc plan.md → trích xuất technical approach, file structure
- Đọc spec.md → trích xuất user stories theo thứ tự ưu tiên
- Map contracts đến user stories

### Phase 2: Generate (Parallel, after Phase 1)

**fe-tasks**: Generate tasks cho FE portion
**be-tasks**: Generate tasks cho BE portion

**Output**: FE tasks section + BE tasks section (theo checklist format)

### Phase 3: Merge (Lead only)

**Điều kiện tiên quyết**: Phase 2 outputs hoàn thành

1. **Merge** FE + BE sections vào tasks.md:
   - Phase 1 & 2: Combined setup + foundational
   - Phase 3+: User Stories với cả FE và BE tasks

2. **Validate** task completeness:
   - Mỗi user story có đủ FE và BE tasks cần thiết
   - Checklist format chính xác

**Output**: tasks.md

Context cho task generation: $ARGUMENTS

tasks.md phải có thể thực thi ngay lập tức - mỗi task phải đủ cụ thể để LLM có thể hoàn thành mà không cần thêm context.

## Task Generation Rules

**QUAN TRỌNG**: Tasks phải được tổ chức theo user story để cho phép implementation và testing độc lập.

**Tests là TÙY CHỌN**: Chỉ tạo test tasks nếu được yêu cầu rõ ràng trong feature specification hoặc nếu người dùng yêu cầu approach TDD.

### Checklist Format (BẮT BUỘC)

Mọi task phải TUYỆT ĐỐI tuân theo format này:

```text
- [ ] [TaskID] [P?] [Story?] Description with file path
```

**Các thành phần của Format**:

1. **Checkbox**: LUÔN bắt đầu với `- [ ]` (markdown checkbox)
2. **Task ID**: Số tuần tự (T001, T002, T003...) theo thứ tự thực thi
3. **Marker [P]**: Bao gồm CHỈ KHI task có thể chạy song song (file khác nhau, không phụ thuộc vào các task chưa hoàn thành)
4. **Label [Story]**: BẮT BUỘC cho các task ở user story phase
   - Format: [US1], [US2], [US3], v.v. (map đến user stories từ spec.md)
   - Setup phase: KHÔNG có story label
   - Foundational phase: KHÔNG có story label
   - User Story phases: PHẢI có story label
   - Polish phase: KHÔNG có story label
5. **Description**: Hành động rõ ràng với đường dẫn file chính xác

**Ví dụ**:

- ✅ ĐÚNG: `- [ ] T001 Create project structure per implementation plan`
- ✅ ĐÚNG: `- [ ] T005 [P] Implement authentication middleware in src/middleware/auth.py`
- ✅ ĐÚNG: `- [ ] T012 [P] [US1] Create User model in src/models/user.py`
- ✅ ĐÚNG: `- [ ] T014 [US1] Implement UserService in src/services/user_service.py`
- ❌ SAI: `- [ ] Create User model` (thiếu ID và Story label)
- ❌ SAI: `T001 [US1] Create model` (thiếu checkbox)
- ❌ SAI: `- [ ] [US1] Create User model` (thiếu Task ID)
- ❌ SAI: `- [ ] T001 [US1] Create model` (thiếu file path)

### Tổ chức Task

1. **Từ User Stories (spec.md)** - TỔ CHỨC CHÍNH:
   - Mỗi user story (P1, P2, P3...) được một phase riêng
   - Map tất cả các component liên quan đến story của chúng:
     - Models cần cho story đó
     - Services cần cho story đó
     - Interfaces/UI cần cho story đó
     - Nếu tests được yêu cầu: Tests cụ thể cho story đó
   - Đánh dấu story dependencies (hầu hết các story nên độc lập)

2. **Từ Contracts**:
   - Map mỗi interface contract → đến user story mà nó phục vụ
   - Nếu tests được yêu cầu: Mỗi interface contract → task contract test [P] trước khi implementation trong phase của story đó

3. **Từ Data Model**:
   - Map mỗi entity đến user story(ies) cần nó
   - Nếu entity phục vụ nhiều stories: Đặt trong story sớm nhất hoặc Setup phase
   - Relationships → service layer tasks trong phase story phù hợp

4. **Từ Setup/Infrastructure**:
   - Shared infrastructure → Setup phase (Phase 1)
   - Foundational/blocking tasks → Foundational phase (Phase 2)
   - Story-specific setup → trong phase của story đó

### Cấu trúc Phase

- **Phase 1**: Setup (khởi tạo project)
- **Phase 2**: Foundational (điều kiện tiên quyết blocking - PHẢI hoàn thành trước các user stories)
- **Phase 3+**: User Stories theo thứ tự ưu tiên (P1, P2, P3...)
  - Trong mỗi story: Tests (nếu được yêu cầu) → Models → Services → Endpoints → Integration
  - Mỗi phase nên là một increment hoàn chỉnh, có thể test độc lập
- **Final Phase**: Polish & Cross-Cutting Concerns