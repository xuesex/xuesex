# Iron Triangle CI/CD Hub

## 项目概述
铁三角架构的 CI/CD 编排中心。包含 GitHub Actions 自动化流水线和 AI 代码审查配置。

## 架构
- **CI 流水线** (`.github/workflows/ci.yml`): Lint → Test → Audit → Summary
- **AI 代码审查** (`.github/workflows/code-review.yml`): PR 触发 → 获取 diff → DeepSeek V4 Pro 审查 → 发布评论
- **Code Review Secret**: `DEEPSEEK_API_KEY` 需通过 `gh secret set` 配置

## 铁三角组件
- **Hermes Agent**: 规划控制层 (DeepSeek V4 Pro) — 意图理解/任务拆解/调度
- **Claude Code**: 专业代码层 (DeepSeek V4 Pro via Anthropic 端点) — 编程/审查/重构
- **ClawX**: 自动化执行层 (DeepSeek V4 Flash) — 测试/部署/文件操作/多代理编排

## 环境要求
- WSL2 (Ubuntu) 本地环境
- Node.js ≥ 20, Python ≥ 3.10
- Git 使用 SSH (`git@github.com:`)，HTTPS 自动改写
- DeepSeek API Key (`sk-f9b23...`)

## 关键命令

### Git 操作 (SSH 模式)
```bash
git clone git@github.com:xuesex/xuesex.git      # SSH clone
git remote -v                                      # 验证远程地址
```

### CI/CD
```bash
gh workflow list --repo xuesex/xuesex              # 列出 workflows
gh run list --repo xuesex/xuesex --limit 5         # 最近运行记录
gh secret set DEEPSEEK_API_KEY --body "$DEEPSEEK_API_KEY" --repo xuesex/xuesex
```

### 健康检查
```bash
ssh -T git@github.com                              # GitHub SSH 连通性
curl -s https://api.deepseek.com/v1/models -H "Authorization: Bearer $DEEPSEEK_API_KEY" | head -c 100
```

## 代码规范

### Workflow 文件
- 使用 `actions/checkout@v4`（注意: Node 20 支持截止 2026年9月）
- Secret 注入使用 `jq --arg` 而非 GitHub expression 直接嵌入
- AI 审查结果通过文件传递，避免 JSON 注入

### 安全要点 (来自 pre-commit 审查发现)
- ❌ `${{ steps.outputs.xxx }}` 直接嵌入 JS template literal → 用文件传递
- ❌ 用户输入的 diff 直接拼入 JSON body → 用 `jq --arg` 安全嵌入
- ❌ `grep -q '"lint"'` 匹配过于宽泛 → 用 `jq -e '.scripts.lint'` 精确匹配

### Git 提交规范
```
type: 简洁描述

feat: 新功能
fix: 修复
refactor: 重构
docs: 文档
chore: 杂项
```

## 配置文件位置
```
~/.iron-triangle/env.sh       # 统一环境变量
~/.claude/settings.json       # Claude Code 设置
~/.clawx/config               # ClawX 配置 (dotenv)
~/.hermes/config.yaml         # Hermes 配置
```

## 日志
```
~/.iron-triangle/logs/
├── claude-code/              # Claude Code 调用日志
├── clawx/                    # ClawX 调用日志
└── hermes/                   # Hermes 日志
```
