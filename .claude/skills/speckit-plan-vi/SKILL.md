---
name: "speckit-plan"
description: "Thực thi implementation planning workflow dựa trên plan template để generate ra các design artifacts."
argument-hint: "Hướng dẫn tùy chọn cho giai đoạn lập kế hoạch"
compatibility: "Yêu cầu cấu trúc project spec-kit với thư mục .specify/"
metadata:
  author: "github-spec-kit"
  source: "templates/commands/plan.md"
user-invocable: true
disable-model-invocation: true
---

## Đầu vào từ người dùng

```text
$ARGUMENTS
```

Bạn **PHẢI** xem xét đầu vào từ người dùng trước khi tiếp tục (nếu không rỗng).

## Kiểm tra trước khi thực thi

**Kiểm tra extension hooks (trước khi lập kế hoạch)**:

- Kiểm tra xem `.specify/extensions.yml` có tồn tại trong thư mục gốc của project hay không.
- Nếu tồn tại, đọc file và tìm các mục dưới key `hooks.before_plan`
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

1. **Setup**: Chạy `.specify/scripts/powershell/setup-plan.ps1 -Json` từ thư mục gốc của repo và parse JSON để lấy FEATURE_SPEC, IMPL_PLAN, SPECS_DIR, BRANCH. Với single quotes trong args như "I'm Groot", sử dụng cú pháp escape: ví dụ 'I'\''m Groot' (hoặc dùng double-quote nếu có thể: "I'm Groot").

2. **Load context**: Đọc FEATURE_SPEC và `.specify/memory/constitution.md`. Load template IMPL_PLAN (đã được copy sẵn).

3. **Execute plan workflow**: Tuân theo cấu trúc trong template IMPL_PLAN để:
   - Điền Technical Context (đánh dấu các phần chưa biết là "NEEDS CLARIFICATION")
   - Điền Constitution Check section từ constitution
   - Đánh giá các gates (ERROR nếu các vi phạm không được giải thích)
   - Phase 0: Tạo research.md (giải quyết tất cả NEEDS CLARIFICATION)
   - Phase 1: Tạo data-model.md, contracts/, quickstart.md
   - Phase 2: Cập nhật agent context bằng cách chạy agent script
   - Đánh giá lại Constitution Check sau khi thiết kế

4. **Stop and report**: Command kết thúc sau Phase 2 planning. Báo cáo branch, đường dẫn IMPL_PLAN, và các artifacts đã được tạo.

5. **Kiểm tra extension hooks**: Sau khi báo cáo, kiểm tra xem `.specify/extensions.yml` có tồn tại trong thư mục gốc của project hay không.
   - Nếu tồn tại, đọc file và tìm các mục dưới key `hooks.after_plan`
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

## Team Orchestration

Skill này hỗ trợ **multi-agent planning** cho FlowPilot monorepo với 2 tech stack khác nhau:
- **FlowPilotUI**: Electron + Vue 3 + TypeScript
- **FlowPilotBE**: Python + FastAPI

### Team Structure

```
plan-lead (main session - skill executor)
├── fe-planner (subagent) ← chạy song song
└── be-planner (subagent) ← chạy song song
```

### Khi nào spawn team?

| Scenario | Behavior |
|----------|----------|
| Feature spec có cả FE + BE | Spawn cả 2 subagent, chạy song song |
| Feature spec chỉ có FE | Spawn chỉ fe-planner, bypass BE |
| Feature spec chỉ có BE | Spawn chỉ be-planner, bypass FE |
| FlowPilotBE chưa có code | BE agent vẫn plan dựa trên spec, không research existing code |

### Workflow

```
1. Setup → Load Constitution & Template (giữ nguyên)
2. Analyze FEATURE_SPEC → xác định FE/BE scope
3. TeamCreate("flowpilot-planning", "Multi-agent planning cho FlowPilot monorepo")
4. Create tasks:
   - Task "research": unblocked
   - Task "fe-plan": blocked by research
   - Task "be-plan": blocked by research
   - Task "synthesis": blocked by fe-plan AND be-plan
5. Spawn fe-planner + be-planner (nếu cần)
6. fe-planner + be-planner:
   - Phase 0 (Research): Đọc FEATURE_SPEC, viết vào research.md
   - Phase 1 (Design): Tạo data-model-{fe,be}.md, contracts/
   - Phase 2 (Plan): Viết plan-{fe,be}.md
   - Sync via SendMessage khi cần cross-agent info
7. plan-lead: Tổng hợp → viết IMPL_PLAN.md
8. Report
9. Send shutdown_request tới các subagent
10. TeamDelete
```

### Sync Protocol

Khi fe-planner hoặc be-planner cần info từ agent kia:

```
be-planner cần FE contract schema
→ SendMessage(to: "fe-planner", message: "Need FE IPC contract schema for {topic}")
→ fe-planner responds via SendMessage
→ be-planner tiếp tục planning

fe-planner cần BE API spec
→ SendMessage(to: "be-planner", message: "Need BE API spec for {topic}")
→ be-planner responds via SendMessage
→ fe-planner tiếp tục planning
```

### Output: IMPL_PLAN.md

Plan-lead tổng hợp tất cả outputs thành 1 file `IMPL_PLAN.md` theo format của `.specify/templates/plan-template.md`:

```markdown
# Implementation Plan: {feature_name}

**Branch**: `{branch_name}` | **Date**: {DATE} | **Spec**: [link]
**Input**: Feature specification from `/specs/{feature_name}/spec.md`

## Summary

[Extract from feature spec: primary requirement + technical approach from research]

## Technical Context

[Tech stack details from plan-{fe,be}.md]

## Constitution Check

[Gates from constitution]

## FE Implementation ({FlowPilotUI})
[data-model-fe.md content]
### IPC Contracts
[contracts/fe-* content]
### Plan
[plan-fe.md content]

## BE Implementation ({FlowPilotBE})
[data-model-be.md content]
### API Contracts
[contracts/be-* content]
### Plan
[plan-be.md content]

## Shared
[research.md - cross-cutting concerns]
```

## Các Phase (Multi-Agent)

### Phase 0: Outline & Research (Parallel)

**fe-planner** và **be-planner** chạy song song:

1. **Trích xuất các phần chưa biết từ Technical Context**:
   - **fe-planner**: FE-related NEEDS CLARIFICATION → research FE topics
   - **be-planner**: BE-related NEEDS CLARIFICATION → research BE topics

2. **Research agents dispatch**:
   ```text
   fe-planner: Research {unknown FE topics} for {feature context}
   be-planner: Research {unknown BE topics} for {feature context}
   ```

3. **Tổng hợp kết quả** trong `research.md`:
   - `## FE Research` — fe-planner writes
   - `## BE Research` — be-planner writes
   - `## Shared Decisions` — cross-cutting concerns

**Output**: research.md với tất cả NEEDS CLARIFICATION đã được giải quyết

### Phase 1: Design & Contracts (Parallel)

**Điều kiện tiên quyết:** `research.md` hoàn thành

**fe-planner**:
1. Trích xuất FE entities → `data-model-fe.md`
2. Định nghĩa IPC contracts → `contracts/fe-*.md`

**be-planner**:
1. Trích xuất BE entities → `data-model-be.md`
2. Định nghĩa API contracts → `contracts/be-*.md`

**Cross-agent sync** (nếu cần):
- be-planner gửi `SendMessage` tới fe-planner để request FE contract schema
- fe-planner gửi `SendMessage` tới be-planner để request BE API spec

**Output**: data-model-fe.md, data-model-be.md, contracts/fe-*.md, contracts/be-*.md

### Phase 2: Implementation Planning (Parallel)

**fe-planner**: Viết `plan-fe.md`
**be-planner**: Viết `plan-be.md`

**Điều kiện tiên quyết:** Phase 1 outputs hoàn thành

**Output**: plan-fe.md, plan-be.md

### Phase 3: Synthesis (Lead only)

**Điều kiện tiên quyết:** Phase 2 outputs hoàn thành

1. **Cập nhật agent context**:
   - Chạy `.specify/scripts/powershell/update-agent-context.ps1 -AgentType claude`

2. **Tổng hợp** tất cả outputs vào `IMPL_PLAN.md`:
   - FE section: data-model-fe.md, contracts/fe-*.md, plan-fe.md
   - BE section: data-model-be.md, contracts/be-*.md, plan-be.md
   - Shared: research.md cross-cutting decisions

**Output**: IMPL_PLAN.md

## Key rules

- Sử dụng absolute paths
- ERROR khi gate failures hoặc các clarifications chưa được giải quyết
