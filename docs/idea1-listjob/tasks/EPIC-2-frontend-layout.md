# EPIC-2: 前端基础布局

## 标签
`#frontend` `#react` `#vite` `#tailwind`

## 概述
初始化 React + Vite + TypeScript 项目，配置 Tailwind CSS + shadcn/ui，实现主从视图布局。

## 成功指标
- [ ] Vite 开发服务器运行在 http://localhost:5173
- [ ] 主从视图布局正确显示
- [ ] Mock 数据可以展示

## Repository
`modules/frontend`

## Feature Branch
`feature/job-listing-ui`

## 所需 Skills
- `pragmatic-clean-code-reviewer` - 代码审查
- `code-quality-principles` - 代码质量原则

## 执行顺序
TASK-2-1 → TASK-2-2 → TASK-2-3 → TASK-2-4

---

## TASK-2-1: 项目初始化

**标签**: `#frontend` `#setup`

**描述**: 初始化 React + Vite + TypeScript + Tailwind CSS 项目

**详细步骤**:
1. 创建 Vite 项目
2. 安装依赖
3. 初始化 Tailwind CSS
4. 安装 shadcn/ui
5. 创建目录结构

**验收标准**:
- [ ] `npm run dev` 成功启动
- [ ] Tailwind CSS 样式生效
- [ ] shadcn/ui 组件可用

**预计时间**: 30 分钟

---

## TASK-2-2: 类型定义和 API 客户端

**标签**: `#frontend` `#types` `#api`

**描述**: 定义 TypeScript 类型和 API 客户端

**详细步骤**:
1. 创建 `src/types/job.ts`
2. 创建 `src/services/api.ts`
3. 创建环境变量 `.env`

**验收标准**:
- [ ] TypeScript 类型定义完整
- [ ] API 客户端可以调用后端接口

**预计时间**: 20 分钟

---

## TASK-2-3: 主布局组件

**标签**: `#frontend` `#layout` `#component`

**描述**: 实现主从视图布局（左列表 + 右详情）

**详细步骤**:
1. 创建 `src/components/layout/MainLayout.tsx`
2. 创建 `src/components/job-list/JobListPanel.tsx` (骨架)
3. 创建 `src/components/job-detail/JobDetailPanel.tsx` (骨架)
4. 使用 shadcn/ui 组件

**验收标准**:
- [ ] 页面显示为左右两栏布局
- [ ] 响应式设计（平板适配）

**预计时间**: 45 分钟

---

## TASK-2-4: 列表和详情组件（Mock 数据）

**标签**: `#frontend` `#component`

**描述**: 使用 Mock 数据实现列表和详情组件

**详细步骤**:
1. 创建 `src/components/job-list/JobCard.tsx`
2. 在 JobListPanel 中使用 Mock 数据渲染
3. 创建 `src/components/job-detail/JobDetailPanel.tsx`
4. 添加选中状态样式

**验收标准**:
- [ ] 列表显示 Mock 职位数据
- [ ] 点击职位卡片在右侧显示详情
- [ ] 选中卡片有视觉反馈

**预计时间**: 60 分钟
