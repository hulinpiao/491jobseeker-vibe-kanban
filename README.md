# 491JobSeeker - Vibe Kanban 配置与工作流程

这是 491JobSeeker 项目的 Vibe Kanban 任务管理配置和 **AI 协作核心工作流程**。

## 🎯 项目信息

- **项目名称**: 491JobSeeker
- **项目ID**: `b643874c-ca76-4705-864f-84a26a0740e9`
- **访问地址**: http://127.0.0.59375
- **管理范围**: 全部 5 个子模块（frontend, backend, test, brightdata, vibe-kanban）

---

# 🚀 Vibe Coding 核心工作流程

**这是所有 AI 协作开发必须遵守的核心流程！**

## 阶段 1：企划与架构设计（Planning）

**目标：** 生成 PRD、Solution Architecture、Task Plan

**使用 Skills：**
- `brainstorming` - 分析功能需求
- `writing-plans` - 创建执行计划

**输出文档（保存在 `modules/vibe-kanban/`）：**
1. **PRD** - 产品需求文档
2. **solution-architecture.md** - 技术架构设计（符合 v5 架构）
3. **task-plan.md** - 任务计划

**Task Plan 要求：**
- 创建最少的 Epic
- 每个 Epic 下创建 Tasks
- 每个 Task 需要：
  - 🏷️ **Tags** - 标识 target repo（frontend/backend/test）和 priority
  - 📄 **详细操作文档** - 包括 task plan
  - 📦 **所需 Skills** - 比如 test agent 可能需要 E2E test skill

## 阶段 2：Ticket 创建（Vibe Kanban）

**自动生成逻辑：**
根据 Epic 的 Tags 自动创建 Ticket

### Epic → Ticket 映射规则

| Epic Tags | Ticket 分配给 Agent |
|-----------|-------------------|
| `backend`, `P0` | Backend Agent（整个 Epic） |
| `frontend`, `P0` | Frontend Agent（整个 Epic） |
| `test-backend`, `P1` | Test-Backend Agent |
| `test-frontend`, `P1` | Test-Frontend Agent |
| `infrastructure`, `P1` | Team Lead（协调） |

### Ticket 结构模板

```markdown
## Ticket: [Epic Name]

### 描述
简要说明这个 Epic 的功能

### 成功指标
- [ ] 指标 1
- [ ] 指标 2

### 包含的 Tasks
- Task 1 (path/to/task1.md)
- Task 2 (path/to/task2.md)

### 所需 Skills
- Frontend: React/Next.js
- Test: E2E testing

### 执行顺序
1. Task 1 → Task 2 → Task 3
```

## 阶段 3：Agent Team 配置（CC 自动配置）

**Team Lead 根据 Ticket 的 Tags 自动创建和配置 Agents**

### Agent 配置规则

| Agent Type | 触发条件 | 工作目录 | 职责 |
|------------|----------|----------|------|
| **Team Lead** | 始终运行 | 主仓库 | Orchestrator、项目管理、调度 agents |
| **Frontend Agent** | Ticket tag: `frontend` | `modules/frontend` | UI 开发、组件开发、样式、路由 |
| **Backend Agent** | Ticket tag: `backend` | `modules/backend` | API 开发、数据库、业务逻辑 |
| **Test-Frontend Agent** | Ticket tag: `test-frontend` | `modules/test` | 前端功能测试、E2E 测试 |
| **Test-Backend Agent** | Ticket tag: `test-backend` | `modules/test` | 后端 API 测试、集成测试 |
| **Review Agent** | Ticket tag: `code-review` | 对应的 repo | 代码审查 |

### Agent 创建流程（由 Team Lead 执行）

1. 分析 Ticket 的 Tags
2. 搜索并安装所需的 Skills
3. 创建对应的 Agent
4. 分配 Ticket 给 Agent
5. 监控执行进度

**重要：不在 prompt 里 hardcode agent 技能！所有技能配置由 CC 根据企划文档自动生成。**

## 阶段 4：开发执行（Development）

### Vibe Kanban 工作流

```
┌─────────┐
│   TODO   │ ← 企划组在这里开启 Ticket
└────┬────┘
     │ Team Lead 移动到 In Process 并分配 Agent
     ↓
┌─────────────┐
│ IN PROCESS   │ ← Agent 执行 Epic 中的所有 Tasks
└────┬─────────┘
     │ Agent 完成后通知 Team Lead
     ↓
┌─────────────┐
│  IN REVIEW   │ ← Test Agent 执行测试
└────┬─────────┘
     │ 测试通过 → Done
     │ 测试失败 → 返回 IN PROCESS
     ↓
┌───────────┐
│   DONE     │ ← Team Lead 移动到这里
└───────────┘
```

### 开发规则（TDD）

1. 每个从 main 分支拉取 **feature branch**
2. 按照 Epic 中的 Tasks 顺序执行
3. 使用 TDD：先写测试 → 再写代码 → 重构
4. 完成后通知 Team Lead

## 阶段 5：Review & Merge

**测试通过后：**
1. Team Lead 将 Ticket 移到 Done
2. Team Lead 创建 Pull Request
3. Review Agent 代码审查（使用 `pragmatic-clean-code-reviewer` skill）
4. 合并到 main 分支

---

## 🤖 CC Agent Teams（开发组）

### 1. Team Lead（Orchestrator）

**职责：**
- 项目管理和调度
- 根据 Ticket Tags 创建和配置 Agents
- 通过 Vibe Kanban MCP 监控进度
- 更新 Ticket 状态
- 协调 Agents 之间的协作

### 2. Frontend Agent

**触发条件：** Ticket tag 包含 `frontend`

**工作目录：** `modules/frontend`

**职责：**
- UI 开发
- 组件开发
- 样式和布局
- 前端路由
- 前端单元测试

**需要的 Skills：**（CC 根据 Ticket 自动搜索安装）
- React/Next.js 相关技能
- CSS/Tailwind 技能
- 前端测试技能

### 3. Backend Agent

**触发条件：** Ticket tag 包含 `backend`

**工作目录：** `modules/backend`

**职责：**
- API 开发
- 数据库操作
- 业务逻辑实现
- 后端单元测试
- API 集成测试

**需要的 Skills：**（CC 根据 Ticket 自动搜索安装）
- Node.js/Python 技能
- 数据库技能
- API 设计技能

### 4. Test-Frontend Agent

**触发条件：** Ticket tag 包含 `test-frontend`

**工作目录：** `modules/test`

**职责：**
- 前端功能测试
- 前端 E2E 测试
- 前端组件测试

### 5. Test-Backend Agent

**触发条件：** Ticket tag 包含 `test-backend`

**工作目录：** `modules/test`

**职责：**
- 后端 API 测试
- 集成测试
- 后端功能测试

### 6. Review Agent

**触发条件：** 代码需要审查

**工作目录：** 对应的 repo

**职责：**
- 代码审查
- 质量检查
- 确保 SOLID/KISS/YAGNI/DRY 原则

---

## 🏷️ Ticket Tags 规范

### Priority Tags
- `P0` - 紧急重要
- `P1` - 重要
- `P2` - 一般

### Module Tags
- `frontend` - 前端开发
- `backend` - 后端开发
- `test-frontend` - 前端测试
- `test-backend` - 后端测试
- `infrastructure` - 基础设施

### Type Tags
- `feature` - 新功能
- `bug` - Bug 修复
- `refactor` - 重构

### 示例 Ticket Tags

```markdown
## Epic: Backend API for Job Search
Tags: P0, feature, backend, test-backend
→ 分配给：Backend Agent + Test-Backend Agent

## Epic: Frontend Job List Page
Tags: P0, feature, frontend, test-frontend
→ 分配给：Frontend Agent + Test-Frontend Agent
```

---

## 💎 关键原则

1. **企划阶段配置**：Agent 的技能、职责都在企划阶段确定，不在 prompt 里 hardcode
2. **Tag 驱动分配**：根据 Ticket 的 Tags 自动分配给对应的 Agent
3. **Epic 完整性**：整个 Epic 分配给一个 Agent（而不是拆散）
4. **TDD 开发**：所有开发使用测试驱动开发
5. **Feature Branch**：每个 Epic 在独立的 feature branch 上开发
6. **代码质量**：所有代码必须遵守 SOLID、KISS、YAGNI、DRY 原则

---

## 🚀 快速开始

### 启动 Vibe Kanban

```bash
npx -y vibe-kanban@latest
```

### 查看所有任务

```bash
curl -s "http://127.0.0.1:59375/api/tasks?project_id=b643874c-ca76-4705-864f-84a26a0740e9" | jq '.'
```

### 创建新任务

```bash
curl -s -X POST 'http://127.0.0.1:59375/api/tasks' \
  -H 'Content-Type: application/json' \
  -d '{
    "project_id": "b643874c-ca76-4705-864f-84a26a0740e9",
    "title": "任务标题",
    "description": "任务描述\n\n🏷️ 标签: P0, feature, frontend"
  }'
```

---

**最后更新**: 2026-02-09
**维护者**: hulinpiao
