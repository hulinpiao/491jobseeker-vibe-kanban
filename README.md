# 491JobSeeker 文档管理中心

这是 491JobSeeker 项目的**文档管理仓库**，用于集中管理所有企划、任务文档和相关规范。

## 📋 仓库定位

本仓库专门负责：
- 📝 **企划文档** - 产品规划、功能设计
- 🎫 **任务文档** - 详细的任务说明和规格
- 📚 **规范文档** - 开发规范、标签策略
- 🤖 **AI 协作配置** - Vibe Kanban 任务管理配置

## 🎯 Vibe Kanban 配置

- **项目名称**: 491JobSeeker
- **项目ID**: `b643874c-ca76-4705-864f-84a26a0740e9`
- **访问地址**: http://127.0.0.1:59375
- **管理范围**: 全部 5 个子模块（frontend, backend, test, brightdata, vibe-kanban）

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

## 🏷️ 标签策略

### 优先级标签

| 标签 | 含义 | 响应时间 |
|------|------|----------|
| **P0** | 紧急重要 | 立即处理 |
| **P1** | 重要 | 当天处理 |
| **P2** | 一般 | 本周处理 |
| **P3** | 低优先 | 有空处理 |

### 模块标签

- `frontend` - 前端相关
- `backend` - 后端相关
- `test` - 测试相关
- `brightdata` - 数据采集
- `infrastructure` - 基础设施

### 类型标签

- `feature` - 新功能
- `bug` - Bug 修复
- `refactor` - 代码重构
- `docs` - 文档更新
- `performance` - 性能优化
- `security` - 安全相关

### 特殊标签

- `api` - API 开发
- `cicd` - CI/CD 配置
- `integration` - 第三方集成
- `database` - 数据库相关

### 标签使用示例

```
🏷️ 标签: P0, feature, frontend
🏷️ 标签: P1, feature, backend, api, integration
🏷️ 标签: P0, bug, backend, security
```

**标签顺序**: `[优先级], [类型], [模块], [特殊标签]`

## 📂 当前任务列表

| 任务 | 状态 | 标签 |
|------|------|------|
| 🎨 设计前端页面架构 | todo | P0, feature, frontend |
| ⚙️ 搭建后端API基础架构 | todo | P0, feature, backend, api |
| 🧪 配置测试环境 | todo | P1, infrastructure, test, cicd |
| 💡 集成BrightData数据采集 | todo | P1, feature, brightdata, integration |

## 📁 文档结构

```
vibe-kanban/
├── README.md              # 本文件 - Vibe Kanban 使用指南
├── plans/                 # 企划文档（待创建）
│   ├── product-planning.md
│   └── feature-designs.md
├── tickets/               # 任务文档（待创建）
│   ├── ticket-001.md
│   └── ticket-002.md
└── guidelines/            # 规范文档（待创建）
    ├── coding-standards.md
    └── api-specs.md
```

## 🔗 相关链接

- [Vibe Kanban 官方文档](https://www.vibekanban.com/docs)
- [主项目 README](../README.md)
- [GitHub 仓库](https://github.com/hulinpiao/491jobseeker-vibe-kanban)

---

**最后更新**: 2026-02-09
**维护者**: hulinpiao
