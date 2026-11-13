# n8n Image MCP for Claude

在 Claude（手机 / 网页 / 电脑）里直接用 OpenAI 生图，每张图后显示 **token 用量、本次花费、累计花费、估算余额**。
Generate images with OpenAI from inside Claude, and see token usage, cost per image and an estimated balance after every generation.

一张图大约 USD 0.01–0.025，OpenAI 最低充值 USD 5。适合主力用 Claude、偶尔生图（封面、配图）的人。

## 怎么装 / Setup

**最简单：** 把 [`SETUP_WITH_AI.md`](SETUP_WITH_AI.md) 丢给你的 Claude 或 ChatGPT，说"一步一步带我装"。
**Easiest:** give [`SETUP_WITH_AI.md`](SETUP_WITH_AI.md) to your AI assistant and ask it to guide you.

你需要 / You need:
- 一个 n8n（n8n Cloud 或自建，需要公网 HTTPS）
- OpenAI API key（至少充值 USD 5）
- 支持自定义连接器的 Claude 账号

## 文件 / Files

| File | 用途 |
|---|---|
| `workflows/1-openai-image-core.json` | 出图 + 查花费 + 生成用量摘要 |
| `workflows/2-claude-mcp-image-gen.json` | MCP Server Trigger，给 Claude 连接 |
| `SETUP_WITH_AI.md` | 给 AI 看的逐步安装指南 |

## 注意 / Notes

- **MCP 链接等于密码**，别公开、别出现在截图或视频里。
- 余额是估算值：OpenAI 没有查余额的接口，这里用"基准日余额 − 官方统计花费"算，官方统计可能有延迟。
- 模型名默认 `gpt-image-2.6-sunburst`，OpenAI 更新后在 Generate Image 节点里改。
- 不存储任何图片。

## License

MIT
