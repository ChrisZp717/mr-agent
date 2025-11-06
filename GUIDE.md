# MR Agent 使用指南

本指南将帮助您快速了解和使用 MR Agent，一个基于人工智能的代码审查助手。

## 目录

1. [项目简介](#项目简介)
2. [快速开始](#快速开始)
3. [系统架构](#系统架构)
4. [安装与配置](#安装与配置)
5. [使用流程](#使用流程)
6. [API 参考](#api-参考)
7. [最佳实践](#最佳实践)
8. [故障排除](#故障排除)

## 项目简介

MR Agent 是一个专为 GitLab 合并请求(Merge Request)设计的自动化代码审查工具。它通过 AI 模型分析代码变更，提供专业的代码评审建议，帮助开发团队提高代码质量。

### 核心功能

- **智能代码审查**: 使用 AI 模型分析代码变更，提供专业的代码评审建议
- **自动化集成**: 通过 GitLab Webhook 自动触发代码审查流程
- **详细反馈**: 提供行级别的代码建议和整体评审意见
- **多语言支持**: 支持多种编程语言的代码审查

## 快速开始

### 前置要求

- Node.js 18+
- GitLab 账户和项目访问权限
- DeepSeek API Key

### 5分钟快速部署

```bash
# 1. 克隆并安装
git clone <repository-url> && cd mr-agent && npm install

# 2. 配置环境变量
cp .env.example .env
# 编辑 .env 文件，添加必要的配置

# 3. 启动服务
npm run start
```

### 配置 GitLab Webhook

1. 访问 GitLab 项目设置 → Webhooks
2. 添加新 Webhook:
   - URL: `http://your-server-url/webhook/trigger`
   - 触发事件: 勾选 "Merge request events"
   - 添加令牌: 填入访问令牌

完成以上步骤后，每当创建或更新合并请求时，MR Agent 将自动进行代码审查。

## 系统架构

### 整体架构图

```
┌─────────────┐     ┌──────────────┐     ┌─────────────┐
│   GitLab    │────▶│   MR Agent   │────▶│  DeepSeek   │
│             │     │              │     │    API      │
│  - MR事件   │     │  - Webhook   │     │             │
│  - 代码存储 │     │  - 代码分析  │     │  - AI分析   │
└─────────────┘     │  - 结果发布  │     │             │
                    └──────────────┘     └─────────────┘
                           │
                           ▼
                    ┌──────────────┐
                    │   GitLab     │
                    │              │
                    │ - 发布评审意见│
                    │ - 添加评论   │
                    └──────────────┘
```

### 核心组件

1. **Webhook 控制器** (`webhook.controller.ts`)
   - 接收 GitLab Webhook 事件
   - 验证请求合法性
   - 触发代码审查流程

2. **GitLab 服务** (`git-provide.service.ts`)
   - 与 GitLab API 交互
   - 获取合并请求信息和代码变更
   - 发布评审结果

3. **AI 服务** (`agent.service.ts`)
   - 与 DeepSeek API 交互
   - 处理代码分析请求
   - 解析 AI 响应

4. **发布服务** (`publish.service.ts`)
   - 格式化评审结果
   - 发布到 GitLab 合并请求
   - 处理发布异常

## 安装与配置

### 环境要求

- Node.js 18.0 或更高版本
- npm 8.0 或更高版本
- GitLab 实例访问权限
- DeepSeek API 访问权限

### 环境变量配置

创建 `.env` 文件并配置以下变量:

```env
# GitLab 配置
GITLAB_TOKEN=your_gitlab_token
GITLAB_BASE_URL=https://gitlab.example.com/api/v4

# DeepSeek API 配置
DEEPSEEK_API_KEY=your_deepseek_api_key
DEEPSEEK_BASE_URL=https://api.deepseek.com
```

### Docker 部署

```bash
# 构建镜像
docker build -t mr-agent .

# 运行容器
docker run -d \
  --name mr-agent \
  -p 3000:3000 \
  --env-file .env \
  mr-agent
```

## 使用流程

### 基本工作流程

1. **开发人员创建合并请求**
   - 在 GitLab 中创建新的合并请求
   - 添加代码变更

2. **MR Agent 自动触发**
   - GitLab 发送 Webhook 事件到 MR Agent
   - MR Agent 接收事件并开始处理

3. **代码分析**
   - MR Agent 获取合并请求的代码变更
   - 将代码发送给 DeepSeek AI 进行分析
   - 接收并解析 AI 分析结果

4. **发布评审结果**
   - MR Agent 将评审结果格式化
   - 在合并请求中添加评审评论
   - 发送通知 (可选)

### 详细流程图

```
开发人员创建MR
       │
       ▼
GitLab发送Webhook
       │
       ▼
MR Agent接收Webhook
       │
       ▼
验证请求合法性
       │
       ▼
获取MR详细信息
       │
       ▼
提取代码变更
       │
       ▼
发送代码到AI分析
       │
       ▼
接收AI分析结果
       │
       ▼
格式化评审结果
       │
       ▼
发布评审到GitLab
       │
       ▼
发送完成通知
```

## API 参考

### Webhook 端点

#### POST /webhook

接收 GitLab Webhook 事件并触发代码审查流程。

**请求头:**

```
Content-Type: application/json
X-Gitlab-Token: <webhook-secret>
```

**请求体:**
GitLab Webhook 标准请求体，详见 [GitLab Webhook 文档](https://docs.gitlab.com/ee/user/project/integrations/webhooks.html)。

**响应:**

```json
{
  "status": "success",
  "message": "Webhook received and processing started"
}
```

### 内部 API

#### GitLab 服务

##### getMrInfo(projectId: string, mrId: string): Promise<MrInfo>

获取合并请求详细信息。

##### getFullDiff(projectId: string, mrId: string): Promise<ExtendedDiffFile[]>

获取合并请求的完整代码差异。

#### AI 服务

##### getPrediction(diffContent: string): Promise<Review[]>

分析代码差异并生成评审建议。

## 最佳实践

### 代码审查最佳实践

1. **设置合适的触发条件**
   - 仅对特定分支启用自动审查
   - 设置文件大小限制，避免处理过大的文件
   - 配置支持的文件类型，专注于代码文件

2. **优化 AI 提示词**
   - 根据项目特点定制提示词模板
   - 添加项目特定的编码规范
   - 调整评审的严格程度

3. **处理评审结果**
   - 定期检查评审质量
   - 根据反馈调整 AI 参数
   - 建立评审结果反馈机制

## 故障排除

### 常见问题

#### 1. Webhook 未触发

**症状**: 创建合并请求后，MR Agent 没有响应。

**解决方案**:

1. 检查 GitLab Webhook 配置中的 URL 是否正确
2. 确认 MR Agent 服务正在运行
3. 检查网络连接和防火墙设置
4. 查看 MR Agent 日志中的错误信息

#### 2. 404 Not Found 错误

**症状**: 日志显示 "获取MR变更文件失败: 404 Not Found"

**解决方案**:

1. 验证 GitLab Token 是否具有足够的权限
2. 检查项目ID和合并请求ID是否正确
3. 确认 GitLab API URL 配置正确
4. 检查 Webhook 请求体中的参数

#### 3. AI 分析失败

**症状**: 代码分析过程中出现错误或超时。

**解决方案**:

1. 验证 DeepSeek API Key 是否有效
2. 检查网络连接
3. 减少请求内容大小
4. 增加超时时间

#### 4. 评审结果未发布

**症状**: AI 分析完成，但 GitLab 中没有评审评论。

**解决方案**:

1. 确认 GitLab Token 具有写入权限
2. 检查发布服务配置
3. 查看错误日志
4. 验证网络连接
