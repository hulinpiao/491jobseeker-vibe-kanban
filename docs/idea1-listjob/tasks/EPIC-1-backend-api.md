# EPIC-1: 后端 API 基础

## 标签
`#backend` `#api` `#express` `#mongodb`

## 概述
搭建 Express.js 后端服务器，连接 MongoDB 数据库，实现职位列表和详情 API。

## 成功指标
- [ ] Express 服务器运行在 http://localhost:3000
- [ ] MongoDB 连接成功
- [ ] GET /api/jobs 返回分页数据
- [ ] GET /api/jobs/:id 返回单个职位
- [ ] API 集成测试通过

## Repository
`modules/backend`

## Feature Branch
`feature/job-listing-api`

## 所需 Skills
- `python-testing` - 测试框架参考
- `code-quality-principles` - 代码质量原则

## 执行顺序
TASK-1-1 → TASK-1-2 → TASK-1-3 → TASK-1-4 → TASK-1-5

---

## TASK-1-1: 项目初始化

**标签**: `#backend` `#setup`

**描述**: 初始化 Express.js + TypeScript 项目结构

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

## TASK-1-2: MongoDB 连接

**标签**: `#backend` `#database`

**描述**: 实现 MongoDB 连接和 Mongoose 配置

**详细步骤**:
1. 创建 `src/config/database.ts`
2. 创建环境变量文件 `.env`
3. 在 `src/index.ts` 中调用连接函数
4. 添加连接错误处理

**验收标准**:
- [ ] 服务器启动时成功连接 MongoDB
- [ ] 连接失败时显示错误信息

**预计时间**: 20 分钟

---

## TASK-1-3: Job Model 创建

**标签**: `#backend` `#model`

**描述**: 创建 Mongoose Job Schema 和 TypeScript 类型

**详细步骤**:
1. 创建 `src/models/Job.ts`
2. 创建 `src/types/index.d.ts` 定义类型
3. 验证 Model 能正确读取 2026_02_07 collection

**验收标准**:
- [ ] Job Model 定义完整
- [ ] TypeScript 类型正确
- [ ] 可以从数据库读取示例数据

**预计时间**: 30 分钟

---

## TASK-1-4: API 路由实现

**标签**: `#backend` `#api`

**描述**: 实现 GET /api/jobs 和 GET /api/jobs/:id 路由

**详细步骤**:
1. 创建 `src/services/JobService.ts`
2. 创建 `src/routes/jobs.ts`
3. 在 `src/index.ts` 中注册路由
4. 实现搜索和筛选参数支持

**验收标准**:
- [ ] GET /api/jobs 返回分页数据
- [ ] GET /api/jobs/:id 返回单个职位
- [ ] 搜索和筛选参数正确工作

**预计时间**: 60 分钟

---

## TASK-1-5: 错误处理和测试

**标签**: `#backend` `#testing`

**描述**: 添加错误处理中间件和编写集成测试

**详细步骤**:
1. 创建 `src/middleware/errorHandler.ts`
2. 编写集成测试 `tests/integration/jobs.test.ts`
3. 运行测试验证

**验收标准**:
- [ ] 错误被正确捕获和返回
- [ ] 集成测试全部通过

**预计时间**: 45 分钟
