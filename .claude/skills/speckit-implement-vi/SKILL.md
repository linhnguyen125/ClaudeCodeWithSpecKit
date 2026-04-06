---
name: "speckit-implement"
description: "Thực thi implementation plan bằng cách process và hoàn thành toàn bộ tasks được định nghĩa trong tasks.md"
argument-hint: "Hướng dẫn implementation tùy chọn hoặc task filter"
compatibility: "Yêu cầu cấu trúc project spec-kit với thư mục .specify/"
metadata:
  author: "github-spec-kit"
  source: "templates/commands/implement.md"
user-invocable: true
disable-model-invocation: true
---

## Đầu vào từ người dùng

```text
$ARGUMENTS
```

Bạn **BẮT BUỘC** phải xem xét đầu vào từ người dùng trước khi tiếp tục (nếu không rỗng).

## Kiểm tra trước khi thực thi

**Kiểm tra extension hooks (trước khi implement)**:

- Kiểm tra xem `.specify/extensions.yml` có tồn tại trong thư mục gốc của project hay không.
- Nếu tồn tại, đọc và tìm các mục trong key `hooks.before_implement`
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

1. Chạy `.specify/scripts/powershell/check-prerequisites.ps1 -Json -RequireTasks -IncludeTasks` từ repo root và parse FEATURE_DIR và AVAILABLE_DOCS list. Tất cả đường dẫn phải là tuyệt đối. Với single quotes trong args như "I'm Groot", sử dụng cú pháp escape: ví dụ 'I'\''m Groot' (hoặc dùng double-quote nếu có thể: "I'm Groot").

2. **Kiểm tra trạng thái checklists** (nếu FEATURE_DIR/checklists/ tồn tại):
   - Scan tất cả các checklist files trong thư mục checklists/
   - Với mỗi checklist, đếm:
     - Tổng số items: Tất cả các dòng matching `- [ ]` hoặc `- [X]` hoặc `- [x]`
     - Số items đã hoàn thành: Các dòng matching `- [X]` hoặc `- [x]`
     - Số items chưa hoàn thành: Các dòng matching `- [ ]`
   - Tạo bảng trạng thái:

     ```text
     | Checklist | Total | Completed | Incomplete | Status |
     |-----------|-------|-----------|------------|--------|
     | ux.md     | 12    | 12        | 0          | ✓ PASS |
     | test.md   | 8     | 5         | 3          | ✗ FAIL |
     | security.md | 6   | 6         | 0          | ✓ PASS |
     ```

   - Tính trạng thái tổng thể:
     - **PASS**: Tất cả checklists có 0 incomplete items
     - **FAIL**: Một hoặc nhiều checklists có incomplete items

   - **Nếu bất kỳ checklist nào chưa hoàn thành**:
     - Hiển thị bảng với số incomplete item
     - **DỪNG** và hỏi: "Một số checklist chưa hoàn thành. Bạn có muốn tiếp tục implementation không? (yes/no)"
     - Đợi phản hồi từ người dùng trước khi tiếp tục
     - Nếu người dùng nói "no" hoặc "wait" hoặc "stop", dừng thực thi
     - Nếu người dùng nói "yes" hoặc "proceed" hoặc "continue", tiến hành bước 3

   - **Nếu tất cả checklists đã hoàn thành**:
     - Hiển thị bảng cho thấy tất cả checklists đã pass
     - Tự động tiến hành bước 3

3. Load và phân tích implementation context:
   - **BẮT BUỘC**: Đọc tasks.md cho danh sách task đầy đủ và execution plan
   - **BẮT BUỘC**: Đọc plan.md cho tech stack, architecture và file structure
   - **NẾU TỒN TẠI**: Đọc data-model.md cho entities và relationships
   - **NẾU TỒN TẠI**: Đọc contracts/ cho API specifications và test requirements
   - **NẾU TỒN TẠI**: Đọc research.md cho technical decisions và constraints
   - **NẾU TỒN TẠI**: Đọc quickstart.md cho integration scenarios

4. **Xác minh Project Setup**:
   - **BẮT BUỘC**: Tạo/verify ignore files dựa trên project setup thực tế:

   **Detection & Creation Logic**:
   - Kiểm tra xem lệnh sau có thành công không để xác định repository có phải là git repo không (tạo/verify .gitignore nếu có):

     ```sh
     git rev-parse --git-dir 2>/dev/null
     ```

   - Kiểm tra Dockerfile\* có tồn tại hoặc Docker trong plan.md → tạo/verify .dockerignore
   - Kiểm tra .eslintrc\* có tồn tại → tạo/verify .eslintignore
   - Kiểm tra eslint.config.\* có tồn tại → đảm bảo config's `ignores` entries cover required patterns
   - Kiểm tra .prettierrc\* có tồn tại → tạo/verify .prettierignore
   - Kiểm tra .npmrc hoặc package.json có tồn tại → tạo/verify .npmignore (nếu publishing)
   - Kiểm tra terraform files (\*.tf) có tồn tại → tạo/verify .terraformignore
   - Kiểm tra .helmignore có cần thiết không (helm charts present) → tạo/verify .helmignore

   **Nếu ignore file đã tồn tại**: Verify nó chứa essential patterns, chỉ append các missing critical patterns
   **Nếu ignore file bị thiếu**: Tạo với full pattern set cho công nghệ được detect

   **Common Patterns by Technology** (từ plan.md tech stack):
   - **Node.js/JavaScript/TypeScript**: `node_modules/`, `dist/`, `build/`, `*.log`, `.env*`
   - **Python**: `__pycache__/`, `*.pyc`, `.venv/`, `venv/`, `dist/`, `*.egg-info/`
   - **Java**: `target/`, `*.class`, `*.jar`, `.gradle/`, `build/`
   - **C#/.NET**: `bin/`, `obj/`, `*.user`, `*.suo`, `packages/`
   - **Go**: `*.exe`, `*.test`, `vendor/`, `*.out`
   - **Ruby**: `.bundle/`, `log/`, `tmp/`, `*.gem`, `vendor/bundle/`
   - **PHP**: `vendor/`, `*.log`, `*.cache`, `*.env`
   - **Rust**: `target/`, `debug/`, `release/`, `*.rs.bk`, `*.rlib`, `*.prof*`, `.idea/`, `*.log`, `.env*`
   - **Kotlin**: `build/`, `out/`, `.gradle/`, `.idea/`, `*.class`, `*.jar`, `*.iml`, `*.log`, `.env*`
   - **C++**: `build/`, `bin/`, `obj/`, `out/`, `*.o`, `*.so`, `*.a`, `*.exe`, `*.dll`, `.idea/`, `*.log`, `.env*`
   - **C**: `build/`, `bin/`, `obj/`, `out/`, `*.o`, `*.a`, `*.so`, `*.exe`, `*.dll`, `autom4te.cache/`, `config.status`, `config.log`, `.idea/`, `*.log`, `.env*`
   - **Swift**: `.build/`, `DerivedData/`, `*.swiftpm/`, `Packages/`
   - **R**: `.Rproj.user/`, `.Rhistory`, `.RData`, `.Ruserdata`, `*.Rproj`, `packrat/`, `renv/`
   - **Universal**: `.DS_Store`, `Thumbs.db`, `*.tmp`, `*.swp`, `.vscode/`, `.idea/`

   **Tool-Specific Patterns**:
   - **Docker**: `node_modules/`, `.git/`, `Dockerfile*`, `.dockerignore`, `*.log*`, `.env*`, `coverage/`
   - **ESLint**: `node_modules/`, `dist/`, `build/`, `coverage/`, `*.min.js`
   - **Prettier**: `node_modules/`, `dist/`, `build/`, `coverage/`, `package-lock.json`, `yarn.lock`, `pnpm-lock.yaml`
   - **Terraform**: `.terraform/`, `*.tfstate*`, `*.tfvars`, `.terraform.lock.hcl`
   - **Kubernetes/k8s**: `*.secret.yaml`, `secrets/`, `.kube/`, `kubeconfig*`, `*.key`, `*.crt`

5. Parse cấu trúc tasks.md và trích xuất:
   - **Task phases**: Setup, Tests, Core, Integration, Polish
   - **Task dependencies**: Quy tắc thực thi tuần tự vs song song
   - **Task details**: ID, description, file paths, parallel markers [P]
   - **Execution flow**: Thứ tự và yêu cầu về dependencies

6. Thực thi implementation theo task plan:
   - **Thực thi theo từng phase**: Hoàn thành mỗi phase trước khi chuyển sang phase tiếp theo
   - **Tôn trọng dependencies**: Chạy các task tuần tự theo thứ tự, các task song song [P] có thể chạy cùng nhau
   - **Tuân theo approach TDD**: Thực thi các test tasks trước các implementation tasks tương ứng
   - **Phối hợp dựa trên file**: Các tasks ảnh hưởng đến cùng file phải chạy tuần tự
   - **Validation checkpoints**: Xác minh mỗi phase hoàn thành trước khi tiếp tục

7. Các quy tắc thực thi implementation:
   - **Setup trước**: Khởi tạo project structure, dependencies, configuration
   - **Tests trước code**: Nếu cần viết tests cho contracts, entities, và integration scenarios
   - **Core development**: Implement models, services, CLI commands, endpoints
   - **Integration work**: Database connections, middleware, logging, external services
   - **Polish and validation**: Unit tests, performance optimization, documentation

8. Theo dõi tiến độ và xử lý lỗi:
   - Báo cáo tiến độ sau mỗi task hoàn thành
   - Dừng thực thi nếu bất kỳ task không song song nào thất bại
   - Với các task song song [P], tiếp tục với các task thành công, báo cáo các task thất bại
   - Cung cấp thông báo lỗi rõ ràng với context để debug
   - Đề xuất các bước tiếp theo nếu implementation không thể tiếp tục
   - **QUAN TRỌNG** Với các task đã hoàn thành, đảm bảo đánh dấu task đó là [X] trong file tasks.

9. Xác minh hoàn thành:
   - Verify tất cả required tasks đã hoàn thành
   - Kiểm tra các features đã implement khớp với specification gốc
   - Validate rằng tests pass và coverage đáp ứng requirements
   - Xác nhận implementation tuân theo technical plan
   - Báo cáo trạng thái cuối cùng với tóm tắt công việc đã hoàn thành

Lưu ý: Lệnh này giả định rằng đã có task breakdown đầy đủ trong tasks.md. Nếu tasks không đầy đủ hoặc bị thiếu, đề xuất chạy `/speckit.tasks` trước để regenerate task list.

10. **Kiểm tra extension hooks**: Sau khi xác minh hoàn thành, kiểm tra xem `.specify/extensions.yml` có tồn tại trong thư mục gốc của project hay không.
    - Nếu tồn tại, đọc và tìm các mục trong key `hooks.after_implement`
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
