# Solution Architecture - Job Listing 功能

## 1. 架构概览

### 1.1 技术栈
- **架构模式**: MERN (MongoDB + Express.js + React + Node.js)
- **项目结构**: 保持现有 modules 目录结构
- **代码质量原则**: SOLID, KISS, YAGNI, DRY

### 1.2 系统架构图

```
┌─────────────────────────────────────────────────────────────────┐
│                         Browser                                  │
├─────────────────────────────────────────────────────────────────┤
│  ┌─────────────────────────────────────────────────────────────┐ │
│  │              React Frontend (modules/frontend)              │ │
│  │  ┌─────────────────────┐  ┌──────────────────────────────┐ │ │
│  │  │   Job List Panel    │  │      Job Detail Panel        │ │ │
│  │  │   (Left 40%)        │  │      (Right 60%)             │ │ │
│  │  │                     │  │                              │ │ │
│  │  │  - Search Bar       │  │  - Full Job Details          │ │ │
│  │  │  - Filter Options   │  │  - Description               │ │ │
│  │  │  - Job List         │  │  - Company Info              │ │ │
│  │  │  - Pagination       │  │  - Apply Link                │ │ │
│  │  └─────────────────────┘  └──────────────────────────────┘ │ │
│  └─────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
                                 │
                                 │ HTTP REST API
                                 ▼
┌─────────────────────────────────────────────────────────────────┐
│              Express.js Backend (modules/backend)               │
├─────────────────────────────────────────────────────────────────┤
│  ┌─────────────────────────────────────────────────────────────┐ │
│  │                    API Routes                               │ │
│  │  GET  /api/jobs              # List with pagination         │ │
│  │  GET  /api/jobs/:id          # Get job detail               │ │
│  └─────────────────────────────────────────────────────────────┘ │
│  ┌─────────────────────────────────────────────────────────────┐ │
│  │                    Services                                 │ │
│  │  - JobService (business logic)                              │ │
│  │  - SearchService (search & filter)                          │ │
│  └─────────────────────────────────────────────────────────────┘ │
│  ┌─────────────────────────────────────────────────────────────┐ │
│  │                    Data Layer                               │ │
│  │  - Job Model (Mongoose Schema)                              │ │
│  └─────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
                                 │
                                 │ Mongoose
                                 ▼
┌─────────────────────────────────────────────────────────────────┐
│                    MongoDB Database                             │
├─────────────────────────────────────────────────────────────────┤
│  Database: unified_jobs                                         │
│  Collection: 2026_02_07                                         │
└─────────────────────────────────────────────────────────────────┘
```

---

## 2. 项目目录结构

### 2.1 Backend 结构

```
modules/backend/
├── src/
│   ├── config/
│   │   └── database.ts          # MongoDB 连接配置
│   ├── models/
│   │   └── Job.ts               # Mongoose Job Schema
│   ├── routes/
│   │   └── jobs.ts              # API 路由
│   ├── services/
│   │   ├── JobService.ts        # 业务逻辑
│   │   └── SearchService.ts     # 搜索逻辑
│   ├── types/
│   │   └── index.d.ts           # TypeScript 类型定义
│   ├── middleware/
│   │   └── errorHandler.ts      # 错误处理
│   ├── utils/
│   │   └── logger.ts            # 日志工具
│   └── index.ts                 # Express 入口
├── tests/
│   ├── unit/                    # 单元测试
│   └── integration/             # 集成测试
├── package.json
├── tsconfig.json
└── vite.config.ts
```

### 2.2 Frontend 结构

```
modules/frontend/
├── src/
│   ├── components/
│   │   ├── layout/
│   │   │   ├── MainLayout.tsx   # 主布局（左列表 + 右详情）
│   │   │   └── Loading.tsx      # 加载状态组件
│   │   ├── job-list/
│   │   │   ├── JobListPanel.tsx # 列表面板
│   │   │   ├── JobCard.tsx      # 职位卡片
│   │   │   ├── SearchBar.tsx    # 搜索栏
│   │   │   ├── FilterPanel.tsx  # 筛选面板
│   │   │   └── Pagination.tsx   # 分页组件
│   │   └── job-detail/
│   │       └── JobDetailPanel.tsx # 详情面板
│   ├── services/
│   │   └── api.ts               # API 客户端
│   ├── types/
│   │   └── job.ts               # 类型定义
│   ├── hooks/
│   │   ├── useJobs.ts           # 职位数据 Hook
│   │   └── useSearch.ts         # 搜索逻辑 Hook
│   ├── utils/
│   │   └── formatters.ts        # 格式化工具
│   └── main.tsx                 # 入口
├── tests/
│   ├── unit/                    # 组件单元测试
│   └── e2e/                     # E2E 测试
├── package.json
├── tsconfig.json
├── vite.config.ts
├── tailwind.config.js
└── index.html
```

---

## 3. API 设计

### 3.1 职位列表 API

**Endpoint**: `GET /api/jobs`

**Query Parameters**:

| 参数 | 类型 | 必填 | 说明 | 示例 |
|------|------|------|------|------|
| `q` | string | 否 | 搜索关键词 | `devops` |
| `location` | string | 否 | 地点筛选 | `Sydney` |
| `employment_type` | string | 否 | 工作类型 | `full_time` |
| `work_arrangement` | string | 否 | 工作方式 | `remote` |
| `page` | number | 否 | 页码，默认 1 | `1` |
| `limit` | number | 否 | 每页条数，默认 20 | `20` |
| `sort` | string | 否 | 排序字段，默认 created_at | `-created_at` |

**Response**:

```typescript
{
  "success": true,
  "data": {
    "jobs": Job[],
    "pagination": {
      "total": 223,
      "page": 1,
      "limit": 20,
      "totalPages": 12
    }
  }
}
```

### 3.2 职位详情 API

**Endpoint**: `GET /api/jobs/:id`

**Response**:

```typescript
{
  "success": true,
  "data": Job
}
```

---

## 4. 数据模型

### 4.1 Mongoose Schema

```typescript
// modules/backend/src/models/Job.ts
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
jobSchema.index({ city: 1, state: 1 });
jobSchema.index({ employment_type: 1 });
jobSchema.index({ work_arrangement: 1 });
```

### 4.2 前端类型定义

```typescript
// modules/frontend/src/types/job.ts
export interface Job {
  _id: string;
  dedup_key: string;
  apply_link: string;
  city: string;
  company_name_normalized: string;
  country: string;
  created_at: string;
  employment_type: EmploymentType;
  job_description: string;
  job_location: string;
  job_title: string;
  sources: JobSource[];
  state: string;
  updated_at: string;
  work_arrangement: WorkArrangement;
}

export type EmploymentType = 'full_time' | 'part_time' | 'contract' | 'casual' | 'internship';
export type WorkArrangement = 'onsite' | 'remote' | 'hybrid';

export interface JobSource {
  platform: 'linkedin' | 'indeed' | 'seek';
  job_posting_id: string;
  url?: string;
}

export interface JobsResponse {
  success: boolean;
  data: {
    jobs: Job[];
    pagination: {
      total: number;
      page: number;
      limit: number;
      totalPages: number;
    };
  };
}
```

---

## 5. 前端架构设计

### 5.1 组件层次

```
App
└── MainLayout
    ├── JobListPanel
    │   ├── SearchBar
    │   ├── FilterPanel
    │   ├── JobList
    │   │   └── JobCard[] (通过 map 渲染)
    │   └── Pagination
    └── JobDetailPanel
        ├── JobHeader
        ├── JobDescription
        └── JobMeta
```

### 5.2 状态管理

使用 React Query (TanStack Query) 进行服务端状态管理：

```typescript
// modules/frontend/src/hooks/useJobs.ts
export function useJobs(params: SearchParams) {
  return useQuery({
    queryKey: ['jobs', params],
    queryFn: () => api.getJobs(params),
    staleTime: 30000, // 30秒缓存
  });
}
```

### 5.3 样式系统

使用 Tailwind CSS + shadcn/ui：

```typescript
// tailwind.config.js
export default {
  content: ['./src/**/*.{js,jsx,ts,tsx}'],
  theme: {
    extend: {
      colors: {
        primary: { /* ... */ },
        secondary: { /* ... */ },
      },
    },
  },
  plugins: [require('tailwindcss-animate')],
};
```

---

## 6. 后端架构设计

### 6.1 分层架构

```
Controller Layer (routes/)
    ↓
Service Layer (services/)
    ↓
Data Access Layer (models/)
```

### 6.2 搜索实现

使用 MongoDB 文本搜索 + 组合查询：

```typescript
// modules/backend/src/services/SearchService.ts
async function searchJobs(params: SearchParams) {
  const query: any = {};

  // 文本搜索
  if (params.q) {
    query.$text = { $search: params.q };
  }

  // 地点筛选
  if (params.location) {
    query.$or = [
      { city: new RegExp(params.location, 'i') },
      { state: new RegExp(params.location, 'i') },
      { job_location: new RegExp(params.location, 'i') },
    ];
  }

  // 工作类型
  if (params.employment_type) {
    query.employment_type = params.employment_type;
  }

  // 工作方式
  if (params.work_arrangement) {
    query.work_arrangement = params.work_arrangement;
  }

  return Job.find(query)
    .sort(params.sort || { created_at: -1 })
    .skip((params.page - 1) * params.limit)
    .limit(params.limit);
}
```

---

## 7. 错误处理

### 7.1 后端错误处理

```typescript
// modules/backend/src/middleware/errorHandler.ts
export function errorHandler(err, req, res, next) {
  const status = err.status || 500;
  const message = err.message || 'Internal Server Error';

  res.status(status).json({
    success: false,
    error: { message, status },
  });
}
```

### 7.2 前端错误处理

使用 React Error Boundaries 和 Toast 通知：

```typescript
// modules/frontend/src/components/ErrorBoundary.tsx
export class ErrorBoundary extends Component {
  // ...
}
```

---

## 8. 测试策略

### 8.1 后端测试

- **单元测试**: Jest 测试 Services 和 Utils
- **集成测试**: Supertest 测试 API 端点

### 8.2 前端测试

- **单元测试**: React Testing Library 测试组件
- **E2E 测试**: Playwright 测试用户流程

---

## 9. 部署架构

### 9.1 开发环境

```
Frontend: Vite Dev Server (http://localhost:5173)
Backend: Express (http://localhost:3000)
Database: MongoDB (mongodb://localhost:27017)
```

### 9.2 环境变量

```bash
# Backend .env
MONGODB_URI=mongodb://localhost:27017/unified_jobs
PORT=3000
NODE_ENV=development

# Frontend .env
VITE_API_URL=http://localhost:3000
```

---

**文档版本**: 1.0
**创建日期**: 2026-02-09
**最后更新**: 2026-02-09
