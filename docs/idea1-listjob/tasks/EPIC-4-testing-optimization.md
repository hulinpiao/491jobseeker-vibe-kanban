# EPIC-4: 测试与优化

## 标签
`#testing` `#e2e` `#optimization` `#review`

## 概述
编写 E2E 测试和单元测试，进行性能优化和代码审查。

## 成功指标
- [ ] E2E 测试覆盖主要用户流程
- [ ] 单元测试覆盖率 > 80%
- [ ] Lighthouse 性能分数 > 90
- [ ] ESLint 零警告

## Repository
`modules/frontend`, `modules/backend`, `modules/test`

## Feature Branch
继续使用现有分支

## 所需 Skills
- `test-driven-development` - TDD 方法
- `subagent-testing` - 测试子代理
- `pragmatic-clean-code-reviewer` - 代码审查

## 执行顺序
TASK-4-1 → TASK-4-2 → TASK-4-3

---

## TASK-4-1: E2E 测试

**标签**: `#testing` `#e2e`

**描述**: 使用 Playwright 编写 E2E 测试

**详细步骤**:
1. 在 `modules/test/` 初始化 Playwright
2. 创建测试文件 `tests/e2e/job-listing.spec.ts`
3. 测试用例:
   - 显示职位列表
   - 按关键词搜索
   - 显示职位详情
4. 配置 playwright.config.ts

**验收标准**:
- [ ] E2E 测试全部通过
- [ ] 覆盖主要用户流程

**预计时间**: 60 分钟

---

## TASK-4-2: 单元测试

**标签**: `#testing` `#unit`

**描述**: 编写前端和后端单元测试

**详细步骤**:
1. 后端单元测试 (使用 Jest):
   - 测试 JobService 方法
   - 测试 SearchService 方法
2. 前端单元测试 (使用 React Testing Library):
   - 测试 JobCard 组件
   - 测试 SearchBar 组件
   - 测试 Pagination 组件
   - 测试自定义 Hooks

**验收标准**:
- [ ] 单元测试覆盖率 > 80%
- [ ] 所有测试通过

**预计时间**: 90 分钟

---

## TASK-4-3: 性能优化和代码审查

**标签**: `#optimization` `#review`

**描述**: 优化性能并进行代码审查

**详细步骤**:
1. 运行 Lighthouse 分析
2. 优化措施:
   - 添加虚拟滚动（如果列表很长）
   - 图片懒加载
   - 代码分割
3. 使用 `pragmatic-clean-code-reviewer` skill 进行代码审查
4. 修复 ESLint 警告

**验收标准**:
- [ ] Lighthouse 性能分数 > 90
- [ ] ESLint 零警告
- [ ] 代码审查通过

**预计时间**: 60 分钟
