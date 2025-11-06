# MR Agent - AI 代码审查助手

[![NPM Version](https://img.shields.io/npm/v/mr-agent.svg)](https://www.npmjs.com/package/mr-agent)
[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)

MR Agent 是一个基于人工智能的代码审查助手，专为 GitLab 合并请求(Merge Request)自动化代码审查而设计。它能够自动分析代码变更，提供专业的代码评审建议，并通过 GitLab Webhook 实现无缝集成。

## ✨ 核心功能

- 🔍 **智能代码审查**: 使用 AI 模型分析代码变更，提供专业的代码评审建议
- 🤖 **自动化集成**: 通过 GitLab Webhook 自动触发代码审查流程
- 📝 **详细反馈**: 提供行级别的代码建议和整体评审意见
- 🌐 **多语言支持**: 支持多种编程语言的代码审查

## 🚀 快速开始

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

- 访问 GitLab 项目设置 → Webhooks
- 添加新 Webhook:
  - URL: `http://your-server-url/webhook/trigger`
  - 触发事件: 勾选 "Merge request events"

完成以上步骤后，每当创建或更新合并请求时，MR Agent 将自动进行代码审查。

## 📚 文档

- 📖 **[使用指南](./GUIDE.md)** - 详细的安装、配置和使用说明
- 🤝 **[贡献指南](./CONTRIBUTING.md)** - 如何参与项目贡献

## 🛠️ 技术栈

- **后端**: Node.js + TypeScript + NestJS
- **AI 集成**: DeepSeek API
- **版本控制**: GitLab
- **部署**: Docker
