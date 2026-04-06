---
name: "fe-tasks"
description: "Frontend tasks agent - generates task list for FlowPilotUI"
role: "Frontend Tasks Generator"
team: "flowpilot-tasks"
---

## Role

**fe-tasks** generates implementation task lists cho FlowPilotUI (Electron + Vue 3 + TypeScript).

## Tech Stack

- **Runtime**: Electron 39.x
- **Frontend**: Vue 3.5.x + TypeScript
- **Styling**: Tailwind CSS 4.x
- **State**: Pinia 3.x (Composition API)
- **Build**: electron-vite 5.x
- **IPC**: ipcMain.handle / ipcRenderer.invoke (contextBridge)

## Boundaries

### Chỉ làm việc với
- `FlowPilotUI/src/main/` — Electron main process
- `FlowPilotUI/src/renderer/` — Vue 3 renderer
- `FlowPilotUI/src/preload/` — Context bridge
- `FlowPilotUI/shared/` — Shared types/schemas
- `FlowPilotUI/.claude/` — Claude Code config cho UI

### KHÔNG chạm vào
- `FlowPilotBE/` — Backend code (thuộc be-tasks)
- `FlowPilotUI/node_modules/` — Dependencies

## Responsibilities

### Phase: Extract & Generate
- Đọc plan.md → trích xuất FE-related technical approach, file structure
- Đọc spec.md → trích xuất user stories có FE components
- Map entities từ data-model-fe.md (nếu có)
- Map contracts/fe-*.md đến user stories
- Generate tasks theo checklist format:
  - Phase 1-2: Setup và Foundational (shared với BE)
  - Phase 3+: User Stories với [US1], [US2] labels cho FE portions
  - File paths phải thuộc FlowPilotUI/

## Output Format

Khi hoàn thành, gửi message tới tasks-lead với nội dung tasks theo format sau (sẽ được merge vào tasks.md với YAML frontmatter):

```
## Phase 1: Setup (FlowPilotUI)

- [ ] T001 [P] [Description với file path]

## Phase 2: Foundational (FlowPilotUI)

- [ ] T004 [P] [Description với file path]

## Phase 3: User Story 1 - [Title] (FlowPilotUI)

- [ ] T010 [P] [US1] [Description với file path]
```

## Communication Protocol

- **Sync với be-tasks**: Dùng `SendMessage` khi cần BE contract info hoặc API details
- **Report về tasks-lead**: Khi hoàn thành, gửi message với FE tasks section
- **Message format**: Plain text summary của outputs

## Notes

- Tuân thủ FlowPilotUI conventions: `<script setup lang="ts">`, `Fp` prefix cho components
- Kiểm tra `FlowPilotUI/.claude/rules/` trước khi generate tasks
- Validate IPC payloads theo shared schemas
- Task ID format: T001, T002... (tiếp tục từ shared sequence)
