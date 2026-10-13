# Pie 设计（M1，用户确认 2026-10-13）

Pie 是一个常驻、偏编码的 agent：网页派活、改代码开 PR、挂了能接着跑。底座是 pi-durable 1.1.0。代码由 SWE 写；架构变更走 PR，由用户审。

本文只写已定范围。公开仓库里不写内网地址、主机名、IP、密钥或 token。

## M1（无截止日期）

### 入口与界面
- **唯一入口**：最小网页。包含：输入框派活、任务列表、进度与结果页；告警也显示在网页上。
- **不做**：PWA、Telegram（含只发告警）、推送通知、向外部 CLI（Claude Code / Codex 等）派活。

### 底座与恢复
- 基于 **pi-durable 1.1.0**（精确版本）。
- 重启后：续跑，或明确报「中断」。
- **验收**：模型回答到一半时用 `kill -9` 杀进程，再重启；正常退出后重启不算通过。

### 自改代码与仓库流程
- Pie 在**单独 clone** 的 pie 仓库里改代码，与正在运行的那份代码分开。
- 只开 PR，**用户合并和部署**；Pie 不自行合并、不上线。
- 分支名：`pie/task-<id>`。开 PR 前执行 `gh pr list --head pie/task-<id>`，已有则沿用，避免重复开 PR。
- GitHub token：只限 pie 仓库的**细粒度 token**。

### 部署与数据
- 部署前备份 DB；镜像打版本号。
- Pie 自己的数据单独存，**不往 pi-durable 文档加字段**。
- 网页层要薄：直接读写 pi-durable，**不用实验性 server 包**。
- 对话记录预留 **profile** 字段，给 M2 用。

### 模型
- 模型只走 **new-api**。
- 自建 provider：参照 `packages/ai/src/providers/cerebras.ts`，用 `createProvider` + `openAICompletionsApi()`。
- 地址和 key 从环境变量读：`NEWAPI_BASE_URL`、`NEWAPI_API_KEY`。
- **不要**用 `openaiProvider()`（地址写死 OpenAI，且走 Responses API）。

### 分工
- 代码：SWE。
- 架构与设计文档：Architect（本文）。
- 合并与部署：用户。

## 后续分期

| 阶段 | 内容 |
|------|------|
| M2 | 多 profile |
| M3–M4 | subagent |
| 以后 | 远程机器，经 pi-env |

## 迁移条件

若 **Pi Mobile** 发布，且能自托管、支持 new-api，再考虑迁过去。此前不迁。

## M1 明确不做（相对旧稿）

- 主界面不是 PWA；不接 Telegram；不派活给外部 CLI。
- 不 fork pi-durable-telegram 当产品入口（底座能力来自 pi-durable 1.1.0；入口是自研薄网页）。
- 不依赖仅在 pi-durable main、尚未进 1.1.0 的能力（如嵌套工具调用、`abandonOnRestart`、`documentRetentionMs`）。

## 验收清单（M1）

1. 在网页派一件活：Pie 改仓库、开 PR（分支 `pie/task-<id>`，开前查重）；失败或卡住时网页上主动报告。
2. `kill -9` 中途杀进程后重启：续跑或明确报「中断」。
3. 模型仅经 new-api 自定义 provider；环境变量配置，无密钥进仓库。
4. 部署流程含 DB 备份与镜像版本号（用户侧执行；Pie 只开 PR）。

## 文档约定

- Issue 挂状态与验收；细节在 `docs/design.md`（本文）及后续 `docs/adr`、`docs/research`。
- 设计改动走 PR，用户审；issue 评论可直接更新状态。

