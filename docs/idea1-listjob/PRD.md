# PRD - Job Listing 功能

## 1. 项目概述

### 1.1 项目名称
**Job Listing & Search System**

### 1.2 项目背景
用户需要在本地 localhost 上查看 MongoDB 数据库中的职位信息，并进行搜索和筛选。

### 1.3 目标用户
- 求职者：快速浏览和搜索职位
- 开发者：验证数据采集系统的输出

---

## 2. 核心功能需求

### 2.1 功能列表

| 功能 ID | 功能名称 | 优先级 | 描述 |
|---------|----------|--------|------|
| F1 | 职位列表展示 | P0 | 在首页展示所有职位，默认按日期倒序排列 |
| F2 | 搜索功能 | P0 | 支持按关键词和地点搜索职位 |
| F3 | 筛选功能 | P1 | 按工作类型（全职/兼职/合同）和工作方式（远程/现场/混合）筛选 |
| F4 | 职位详情 | P0 | 点击职位后在详情区域显示完整信息 |
| F5 | 分页功能 | P1 | 每页显示 20 条职位 |

### 2.2 功能详细说明

#### F1: 职位列表展示
- **位置**: 左侧列表区域
- **显示内容**:
  - 职位标题 (job_title)
  - 公司名称 (company_name_normalized)
  - 地点 (city, state)
  - 工作类型 (employment_type)
  - 工作方式 (work_arrangement)
  - 发布日期 (created_at)
- **排序**: 默认按 created_at 倒序

#### F2: 搜索功能
- **关键词搜索**: 搜索 job_title 和 job_description
- **地点搜索**: 搜索 city 和 state
- **交互**: 实时搜索（输入后自动触发）

#### F3: 筛选功能
- **工作类型**: full_time, part_time, contract, casual, internship
- **工作方式**: onsite, remote, hybrid
- **交互**: 勾选框形式，多选

#### F4: 职位详情
- **位置**: 右侧详情区域
- **显示内容**:
  - 所有列表字段
  - 完整职位描述 (job_description)
  - 申请链接 (apply_link)
  - 数据来源 (sources)

#### F5: 分页功能
- **每页条数**: 20
- **导航**: 上一页/下一页/页码跳转

---

## 3. 非功能需求

### 3.1 性能要求
- 列表首次加载时间 < 1s
- 搜索响应时间 < 500ms
- 支持 1000+ 职位数据流畅浏览

### 3.2 兼容性要求
- 现代浏览器（Chrome, Firefox, Safari, Edge 最新版）
- 响应式设计，支持桌面和平板

### 3.3 代码质量要求
- TypeScript 严格模式
- 单元测试覆盖率 > 80%
- ESLint 零警告

---

## 4. 数据源

### 4.1 数据库
- **类型**: MongoDB
- **地址**: mongodb://localhost:27017/unified_jobs
- **Collection**: 2026_02_07 (当前)

### 4.2 数据结构

```typescript
interface Job {
  _id: ObjectId;
  dedup_key: string;
  apply_link: string;
  city: string;
  company_name_normalized: string;
  country: string;
  created_at: Date;
  employment_type: string;
  job_description: string;
  job_location: string;
  job_title: string;
  sources: Array<{platform: string; job_posting_id: string; url?: string}>;
  state: string;
  updated_at: Date;
  work_arrangement: string;
}
```

---

## 5. 成功指标

| 指标 | 目标 | 测量方法 |
|------|------|----------|
| 功能完整性 | 100% | 所有 F1-F5 功能实现 |
| 测试覆盖率 | >80% | Jest/Playwright 报告 |
| 性能达标 | 100% | Lighthouse Score >90 |
| Bug 数量 | 0 | Test Agent 验证 |

---

## 6. 技术栈

### 6.1 后端
- Node.js + Express.js
- MongoDB (Mongoose)
- TypeScript

### 6.2 前端
- React + Vite
- TypeScript
- Tailwind CSS + shadcn/ui
- React Router (如需要)

### 6.3 测试
- Jest (单元测试)
- Playwright (E2E 测试)

---

## 7. 开发方法

- **TDD**: 测试驱动开发
- **Agent Teams**: 多 Agent 协作开发
- **Git Workflow**: Feature Branch + PR

---

**文档版本**: 1.0
**创建日期**: 2026-02-09
**最后更新**: 2026-02-09
