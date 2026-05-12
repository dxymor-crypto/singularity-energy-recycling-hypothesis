# 使用 CodexBridge 对接微信（实践指南）

> 目标：把 Codex（或你自己的 AI 服务）通过 [Gan-Xing/CodexBridge](https://github.com/Gan-Xing/CodexBridge) 接到微信，实现微信收发消息与 AI 自动回复。

## 1. 总体架构

微信（个人号/企业微信/公众号）
→ 消息回调（Webhook）
→ CodexBridge（消息适配与会话管理）
→ 你的 AI 服务（OpenAI 兼容接口 / 自建服务）
→ CodexBridge
→ 回发到微信

## 2. 前置准备

- 一台可公网访问的服务器（推荐 Linux）。
- 可用域名 + HTTPS 证书（微信回调通常要求 HTTPS）。
- 微信侧可配置回调入口（根据你使用的微信通道类型）。
- AI 服务 Key（如 OpenAI 兼容接口）。

## 3. 部署 CodexBridge（通用步骤）

1. 拉取项目代码：
   - `git clone https://github.com/Gan-Xing/CodexBridge`
2. 进入项目，按其 README 安装依赖。
3. 复制示例配置文件（通常是 `.env.example` 或 `config.example.*`）并填写：
   - 微信回调鉴权参数（token / encoding key / appid 等）
   - AI 服务地址与 Key
   - 回调域名、端口、签名密钥
4. 启动服务并确认日志中出现监听地址（如 `0.0.0.0:xxxx`）。
5. 配置反向代理（Nginx/Caddy）把 HTTPS 请求转发到 CodexBridge。

## 4. 微信侧配置要点

根据你的接入形态（公众号、企业微信、个人号桥接方案）设置：

- **回调 URL**：填写你的公网 HTTPS 地址，例如：
  - `https://bot.example.com/wechat/callback`
- **Token / EncodingAESKey / AppID**：必须与 CodexBridge 配置一致。
- **IP 白名单**（若有）：放行反向代理出口 IP。

验证通过后，微信会向回调地址发起校验请求。

## 5. 对接你的 AI（Codex / OpenAI 兼容）

在 CodexBridge 配置中通常需要：

- `BASE_URL`：AI 网关地址（OpenAI 兼容）。
- `API_KEY`：你的密钥。
- `MODEL`：默认模型名。
- 可选：
  - 超时、重试
  - system prompt
  - 会话记忆长度
  - 群聊触发关键词（如 `@机器人`）

## 6. 最小联调清单

1. 微信发一条消息到机器人。
2. 在 CodexBridge 日志确认：
   - 收到 webhook
   - 鉴权通过
   - 成功请求 AI 接口
   - 成功回发消息
3. 故障排查优先级：
   - 回调 URL 不通（80/443、防火墙、证书）
   - token 不一致（签名失败）
   - API_KEY 或模型名错误
   - 请求超时（网络或代理）

## 7. 生产建议

- 用 systemd / Docker 保活。
- 打开结构化日志与请求 ID，便于追踪单条消息。
- 配置限流与敏感词过滤。
- 按需开启消息去重（防止重试导致重复回复）。
- 配置监控告警（可用性、错误率、延迟）。

## 8. 你可以直接给我的信息（我可继续帮你落地）

如果你愿意，我可以下一步按你的环境给出**可直接执行**的部署命令。请提供：

- 服务器系统（Ubuntu / CentOS / Docker 环境）
- 你使用的微信类型（公众号/企微/其他）
- 你的域名
- 你计划使用的模型与接口地址

