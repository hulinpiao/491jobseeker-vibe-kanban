# EPIC-3: 前后端集成

## 标签
`#frontend` `#backend` `#integration` `#api`

## 概述
将前端与后端 API 集成，实现搜索、筛选和分页功能。

## 成功指标
- [ ] 前端可以获取真实数据
- [ ] 搜索和筛选功能正常工作
- [ ] 分页功能正常工作
- [ ] 加载状态和错误处理正确

## Repository
`modules/frontend`, `modules/backend`

## Feature Branch
继续使用 EPIC-1 和 EPIC-2 的分支

## 所需 Skills
无特殊要求

## 执行顺序
TASK-3-1 → TASK-3-2 → TASK-3-3

---

## TASK-3-1: 集成 React Query

**标签**: `#frontend` `#integration`

**描述**: 使用 React Query 替换 Mock 数据，集成真实 API

**详细步骤**:
1. 在 `src/main.tsx` 中配置 QueryClientProvider
2. 创建 `src/hooks/useJobs.ts`
3. 创建 `src/hooks/useJob.ts`
4. 在组件中使用 Hooks 替换 Mock 数据

**验收标准**:
- [ ] 前端显示 MongoDB 真实数据
- [ ] React Query 缓存正常工作
- [ ] 刷新页面数据保持一致

**预计时间**: 45 分钟

---

## TASK-3-2: 搜索和筛选功能

**标签**: `#frontend` `#feature`

**描述**: 实现搜索栏和筛选面板

**详细步骤**:
1. 创建 `src/components/job-list/SearchBar.tsx`
2. 创建 `src/components/job-list/FilterPanel.tsx`
3. 创建 `src/hooks/useSearch.ts` 管理搜索状态
4. 将搜索参数传递给 useJobs Hook

**验收标准**:
- [ ] 关键词搜索实时更新列表
- [ ] 地点搜索正确工作
- [ ] 工作类型筛选正确工作
- [ ] 工作方式筛选正确工作

**预计时间**: 60 分钟

---

## TASK-3-3: 分页功能

**标签**: `#frontend` `#feature`

**描述**: 实现分页组件

**详细步骤**:
1. 创建 `src/components/job-list/Pagination.tsx`
2. 将分页状态添加到 useSearch Hook
3. 将分页参数传递给 useJobs Hook

**验收标准**:
- [ ] 分页组件正确显示
- [ ] 点击翻页按钮更新数据
- [ ] 页码显示正确

**预计时间**: 30 分钟
