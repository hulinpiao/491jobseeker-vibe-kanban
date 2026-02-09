# Task Plan - Job Listing 功能

## Epic 概览

| Epic ID | Epic 名称 | 描述 | 优先级 |
|---------|-----------|------|--------|
| EPIC-1 | 后端 API 基础 | Express.js 服务器搭建、MongoDB 连接、API 路由 | P0 |
| EPIC-2 | 前端基础布局 | React + Vite + Tailwind CSS 初始化、主从视图布局 | P0 |
| EPIC-3 | 前后端集成 | API 集成、搜索和筛选功能实现 | P0 |
| EPIC-4 | 测试与优化 | E2E 测试、单元测试、性能优化 | P1 |

---

## EPIC-1: 后端 API 基础

**Repository**: `modules/backend`
**Feature Branch**: `feature/job-listing-api`
**所需 Skills**: `python-testing`, `code-quality-principles`

### 成功指标
- [ ] Express 服务器运行在 http://localhost:3000
- [ ] MongoDB 连接成功
- [ ] GET /api/jobs 返回分页数据
- [ ] GET /api/jobs/:id 返回单个职位
- [ ] API 集成测试通过

### 执行顺序
TASK-1-1 → TASK-1-2 → TASK-1-3 → TASK-1-4 → TASK-1-5

---

### TASK-1-1: 项目初始化

**标签**: `#backend #setup`

**描述**:
初始化 Express.js + TypeScript 项目结构

**详细步骤**:
1. 初始化 package.json
2. 安装依赖:
   ```bash
   npm install express mongoose cors dotenv
   npm install -D typescript @types/express @types/node @types/mongoose @types/cors tsx nodemon
   ```
3. 创建目录结构:
   ```
   src/
   ├── config/
   ├── models/
   ├── routes/
   ├── services/
   ├── types/
   ├── middleware/
   ├── utils/
   └── index.ts
   ```
4. 配置 tsconfig.json
5. 配置 .env 文件

**验收标准**:
- [ ] `npm run dev` 成功启动服务器
- [ ] 目录结构符合架构设计
- [ ] TypeScript 配置正确（strict mode）

**预计时间**: 30 分钟

---

### TASK-1-2: MongoDB 连接

**标签**: `#backend #database`

**描述**:
实现 MongoDB 连接和 Mongoose 配置

**详细步骤**:
1. 创建 `src/config/database.ts`:
   ```typescript
   import mongoose from 'mongoose';

   export async function connectDB() {
     const uri = process.env.MONGODB_URI || 'mongodb://localhost:27017/unified_jobs';
     await mongoose.connect(uri);
     console.log('MongoDB connected:', mongoose.connection.name);
   }
   ```
2. 创建环境变量文件 `.env`:
   ```bash
   MONGODB_URI=mongodb://localhost:27017/unified_jobs
   PORT=3000
   ```
3. 在 `src/index.ts` 中调用连接函数
4. 添加连接错误处理

**验收标准**:
- [ ] 服务器启动时成功连接 MongoDB
- [ ] 连接失败时显示错误信息
- [ ] 数据库连接状态可在日志中查看

**预计时间**: 20 分钟

---

### TASK-1-3: Job Model 创建

**标签**: `#backend #model`

**描述**:
创建 Mongoose Job Schema 和 TypeScript 类型

**详细步骤**:
1. 创建 `src/models/Job.ts`:
   ```typescript
   import mongoose, { Schema } from 'mongoose';

   const jobSchema = new Schema({
     dedup_key: { type: String, required: true, unique: true },
     apply_link: String,
     city: String,
     company_name_normalized: String,
     country: { type: String, default: 'AU' },
     created_at: { type: Date, default: Date.now },
     employment_type: String,
     job_description: String,
     job_location: String,
     job_title: String,
     sources: [{
       platform: String,
       job_posting_id: String,
       url: String
     }],
     state: String,
     updated_at: { type: Date, default: Date.now },
     work_arrangement: String
   });

   // 索引
   jobSchema.index({ job_title: 'text', job_description: 'text' });
   jobSchema.index({ created_at: -1 });

   export const Job = mongoose.model('Job', jobSchema);
   ```
2. 创建 `src/types/index.d.ts` 定义类型
3. 验证 Model 能正确读取 2026_02_07 collection

**验收标准**:
- [ ] Job Model 定义完整
- [ ] TypeScript 类型正确
- [ ] 可以从数据库读取示例数据

**预计时间**: 30 分钟

---

### TASK-1-4: API 路由实现

**标签**: `#backend #api`

**描述**:
实现 GET /api/jobs 和 GET /api/jobs/:id 路由

**详细步骤**:
1. 创建 `src/services/JobService.ts`:
   ```typescript
   export class JobService {
     async getJobs(params: GetJobsParams) {
       // 实现分页、搜索、筛选逻辑
     }

     async getJobById(id: string) {
       return Job.findById(id);
     }
   }
   ```
2. 创建 `src/routes/jobs.ts`:
   ```typescript
   router.get('/', async (req, res) => {
     // 调用 JobService
   });

   router.get('/:id', async (req, res) => {
     // 获取单个职位
   });
   ```
3. 在 `src/index.ts` 中注册路由
4. 实现 GET /api/jobs 支持:
   - q (关键词搜索)
   - location (地点筛选)
   - employment_type (工作类型)
   - work_arrangement (工作方式)
   - page, limit (分页)

**验收标准**:
- [ ] GET /api/jobs 返回分页数据
- [ ] GET /api/jobs/:id 返回单个职位
- [ ] 搜索和筛选参数正确工作
- [ ] 不存在的 ID 返回 404

**预计时间**: 60 分钟

---

### TASK-1-5: 错误处理和测试

**标签**: `#backend #testing`

**描述**:
添加错误处理中间件和编写集成测试

**详细步骤**:
1. 创建 `src/middleware/errorHandler.ts`
2. 编写集成测试 `tests/integration/jobs.test.ts`:
   ```typescript
   describe('GET /api/jobs', () => {
     it('should return paginated jobs', async () => { /* ... */ });
     it('should filter by keyword', async () => { /* ... */ });
     it('should filter by location', async () => { /* ... */ });
   });
   ```
3. 运行测试验证

**验收标准**:
- [ ] 错误被正确捕获和返回
- [ ] 集成测试全部通过
- [ ] API 响应格式符合规范

**预计时间**: 45 分钟

---

## EPIC-2: 前端基础布局

**Repository**: `modules/frontend`
**Feature Branch**: `feature/job-listing-ui`
**所需 Skills**: `pragmatic-clean-code-reviewer`, `code-quality-principles`

### 成功指标
- [ ] Vite 开发服务器运行在 http://localhost:5173
- [ ] 主从视图布局正确显示
- [ ] Mock 数据可以展示

### 执行顺序
TASK-2-1 → TASK-2-2 → TASK-2-3 → TASK-2-4

---

### TASK-2-1: 项目初始化

**标签**: `#frontend #setup`

**描述**:
初始化 React + Vite + TypeScript + Tailwind CSS 项目

**详细步骤**:
1. 创建 Vite 项目:
   ```bash
   npm create vite@latest . -- --template react-ts
   ```
2. 安装依赖:
   ```bash
   npm install
   npm install -D tailwindcss postcss autoprefixer
   npm install react-router-dom
   npm install @tanstack/react-query
   npm install -D @types/react-router-dom
   ```
3. 初始化 Tailwind CSS:
   ```bash
   npx tailwindcss init -p
   ```
4. 配置 tailwind.config.js
5. 创建目录结构:
   ```
   src/
   ├── components/
   │   ├── layout/
   │   ├── job-list/
   │   └── job-detail/
   ├── services/
   ├── types/
   ├── hooks/
   ├── utils/
   └── main.tsx
   ```
6. 安装 shadcn/ui:
   ```bash
   npx shadcn-ui@latest init
   ```

**验收标准**:
- [ ] `npm run dev` 成功启动
- [ ] Tailwind CSS 样式生效
- [ ] shadcn/ui 组件可用
- [ ] 目录结构符合架构设计

**预计时间**: 30 分钟

---

### TASK-2-2: 类型定义和 API 客户端

**标签**: `#frontend #types #api`

**描述**:
定义 TypeScript 类型和 API 客户端

**详细步骤**:
1. 创建 `src/types/job.ts`:
   ```typescript
   export interface Job { /* ... */ }
   export interface JobsResponse { /* ... */ }
   export interface SearchParams { /* ... */ }
   ```
2. 创建 `src/services/api.ts`:
   ```typescript
   const API_URL = import.meta.env.VITE_API_URL || 'http://localhost:3000';

   export const api = {
     getJobs: (params: SearchParams) => fetch(/* ... */),
     getJobById: (id: string) => fetch(/* ... */),
   };
   ```
3. 创建环境变量 `.env`:
   ```bash
   VITE_API_URL=http://localhost:3000
   ```

**验收标准**:
- [ ] TypeScript 类型定义完整
- [ ] API 客户端可以调用后端接口

**预计时间**: 20 分钟

---

### TASK-2-3: 主布局组件

**标签**: `#frontend #layout #component`

**描述**:
实现主从视图布局（左列表 + 右详情）

**详细步骤**:
1. 创建 `src/components/layout/MainLayout.tsx`:
   ```typescript
   export function MainLayout() {
     return (
       <div className="flex h-screen">
         <JobListPanel className="w-2/5" />
         <JobDetailPanel className="w-3/5" />
       </div>
     );
   }
   ```
2. 创建 `src/components/job-list/JobListPanel.tsx` (骨架组件)
3. 创建 `src/components/job-detail/JobDetailPanel.tsx` (骨架组件)
4. 使用 shadcn/ui 的 Card、ScrollArea 等组件

**验收标准**:
- [ ] 页面显示为左右两栏布局
- [ ] 左侧占 40%，右侧占 60%
- [ ] 响应式设计（平板适配）

**预计时间**: 45 分钟

---

### TASK-2-4: 列表和详情组件（Mock 数据）

**标签**: `#frontend #component`

**描述**:
使用 Mock 数据实现列表和详情组件

**详细步骤**:
1. 创建 `src/components/job-list/JobCard.tsx`:
   ```typescript
   interface JobCardProps {
     job: Job;
     onClick: () => void;
   }

   export function JobCard({ job, onClick }: JobCardProps) {
     return (
       <Card onClick={onClick}>
         <h3>{job.job_title}</h3>
         <p>{job.company_name_normalized}</p>
         {/* ... */}
       </Card>
     );
   }
   ```
2. 在 JobListPanel 中使用 Mock 数据渲染 JobCard 列表
3. 创建 `src/components/job-detail/JobDetailPanel.tsx` 显示完整职位信息
4. 添加选中状态样式

**验收标准**:
- [ ] 列表显示 Mock 职位数据
- [ ] 点击职位卡片在右侧显示详情
- [ ] 选中卡片有视觉反馈

**预计时间**: 60 分钟

---

## EPIC-3: 前后端集成

**Repository**: `modules/frontend`, `modules/backend`
**Feature Branch**: 继续使用 EPIC-1 和 EPIC-2 的分支
**所需 Skills**: 无特殊要求

### 成功指标
- [ ] 前端可以获取真实数据
- [ ] 搜索和筛选功能正常工作
- [ ] 分页功能正常工作
- [ ] 加载状态和错误处理正确

### 执行顺序
TASK-3-1 → TASK-3-2 → TASK-3-3

---

### TASK-3-1: 集成 React Query

**标签**: `#frontend #integration`

**描述**:
使用 React Query 替换 Mock 数据，集成真实 API

**详细步骤**:
1. 在 `src/main.tsx` 中配置 QueryClientProvider
2. 创建 `src/hooks/useJobs.ts`:
   ```typescript
   export function useJobs(params: SearchParams) {
     return useQuery({
       queryKey: ['jobs', params],
       queryFn: () => api.getJobs(params),
     });
   }
   ```
3. 创建 `src/hooks/useJob.ts`:
   ```typescript
   export function useJob(id: string) {
     return useQuery({
       queryKey: ['job', id],
       queryFn: () => api.getJobById(id),
       enabled: !!id,
     });
   }
   ```
4. 在组件中使用 Hooks 替换 Mock 数据

**验收标准**:
- [ ] 前端显示 MongoDB 真实数据
- [ ] React Query 缓存正常工作
- [ ] 刷新页面数据保持一致

**预计时间**: 45 分钟

---

### TASK-3-2: 搜索和筛选功能

**标签**: `#frontend #feature`

**描述**:
实现搜索栏和筛选面板

**详细步骤**:
1. 创建 `src/components/job-list/SearchBar.tsx`:
   ```typescript
   export function SearchBar({ value, onChange }: SearchBarProps) {
     return (
       <Input
         placeholder="搜索职位..."
         value={value}
         onChange={(e) => onChange(e.target.value)}
       />
     );
   }
   ```
2. 创建 `src/components/job-list/FilterPanel.tsx`:
   ```typescript
   export function FilterPanel({ filters, onChange }: FilterPanelProps) {
     return (
       <div>
         <CheckboxGroup
           value={filters.employment_type}
           onChange={(v) => onChange({ ...filters, employment_type: v })}
         >
           <Checkbox value="full_time">全职</Checkbox>
           <Checkbox value="contract">合同</Checkbox>
         </CheckboxGroup>
         {/* work_arrangement 筛选 */}
       </div>
     );
   }
   ```
3. 创建 `src/hooks/useSearch.ts` 管理搜索状态
4. 将搜索参数传递给 useJobs Hook

**验收标准**:
- [ ] 关键词搜索实时更新列表
- [ ] 地点搜索正确工作
- [ ] 工作类型筛选正确工作
- [ ] 工作方式筛选正确工作

**预计时间**: 60 分钟

---

### TASK-3-3: 分页功能

**标签**: `#frontend #feature`

**描述**:
实现分页组件

**详细步骤**:
1. 创建 `src/components/job-list/Pagination.tsx`:
   ```typescript
   export function Pagination({ page, totalPages, onPageChange }: PaginationProps) {
     return (
       <div className="flex gap-2">
         <Button onClick={() => onPageChange(page - 1)} disabled={page === 1}>
           上一页
         </Button>
         <span>第 {page} / {totalPages} 页</span>
         <Button onClick={() => onPageChange(page + 1)} disabled={page === totalPages}>
           下一页
         </Button>
       </div>
     );
   }
   ```
2. 将分页状态添加到 useSearch Hook
3. 将分页参数传递给 useJobs Hook

**验收标准**:
- [ ] 分页组件正确显示
- [ ] 点击翻页按钮更新数据
- [ ] 页码显示正确

**预计时间**: 30 分钟

---

## EPIC-4: 测试与优化

**Repository**: `modules/frontend`, `modules/backend`, `modules/test`
**Feature Branch**: 继续使用现有分支
**所需 Skills**: `test-driven-development`, `subagent-testing`

### 成功指标
- [ ] E2E 测试覆盖主要用户流程
- [ ] 单元测试覆盖率 > 80%
- [ ] Lighthouse 性能分数 > 90
- [ ] ESLint 零警告

### 执行顺序
TASK-4-1 → TASK-4-2 → TASK-4-3

---

### TASK-4-1: E2E 测试

**标签**: `#testing #e2e`

**描述**:
使用 Playwright 编写 E2E 测试

**详细步骤**:
1. 在 `modules/test/` 初始化 Playwright:
   ```bash
   npm init -y
   npm install -D @playwright/test
   npx playwright install
   ```
2. 创建测试文件 `tests/e2e/job-listing.spec.ts`:
   ```typescript
   test.describe('Job Listing', () => {
     test('should display job list', async ({ page }) => {
       await page.goto('http://localhost:5173');
       await expect(page.locator('.job-card').first()).toBeVisible();
     });

     test('should search jobs by keyword', async ({ page }) => {
       await page.goto('http://localhost:5173');
       await page.fill('[placeholder="搜索职位"]', 'devops');
       await expect(page.locator('.job-card').first()).toContainText('devops');
     });

     test('should display job detail', async ({ page }) => {
       await page.goto('http://localhost:5173');
       await page.click('.job-card:first-child');
       await expect(page.locator('.job-detail')).toBeVisible();
     });
   });
   ```
3. 配置 playwright.config.ts

**验收标准**:
- [ ] E2E 测试全部通过
- [ ] 覆盖主要用户流程

**预计时间**: 60 分钟

---

### TASK-4-2: 单元测试

**标签**: `#testing #unit`

**描述**:
编写前端和后端单元测试

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

### TASK-4-3: 性能优化和代码审查

**标签**: `#optimization #review`

**描述**:
优化性能并进行代码审查

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

---

## 附录

### A. 依赖安装命令

**Backend**:
```bash
cd modules/backend
npm install express mongoose cors dotenv
npm install -D typescript @types/express @types/node @types/mongoose @types/cors tsx nodemon
npm install -D jest @types/jest ts-jest supertest @types/supertest
```

**Frontend**:
```bash
cd modules/frontend
npm create vite@latest . -- --template react-ts
npm install react-router-dom @tanstack/react-query
npm install -D tailwindcss postcss autoprefixer
npm install -D @vitest/ui @testing-library/react @testing-library/jest-dom
npm install -D @playwright/test
```

### B. 环境变量模板

**Backend `.env`**:
```bash
MONGODB_URI=mongodb://localhost:27017/unified_jobs
PORT=3000
NODE_ENV=development
```

**Frontend `.env`**:
```bash
VITE_API_URL=http://localhost:3000
```

### C. Git Workflow

每个 Epic 创建一个 feature branch:

```bash
# Epic 1
git checkout -b feature/job-listing-api

# Epic 2
git checkout -b feature/job-listing-ui

# 开发完成后
git add .
git commit -m "feat: implement job listing api"
git push origin feature/job-listing-api
gh pr create --title "feat: job listing api" --body "..."


```

**文档版本**: 1.0
**创建日期**: 2026-02-09
**最后更新**: 2026-02-09
