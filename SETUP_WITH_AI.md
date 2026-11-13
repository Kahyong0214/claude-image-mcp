# Setup with your AI assistant / 让你的 AI 带你安装

> **用法：** 把这个文件（或它的 GitHub 链接）丢给你的 AI（Claude、ChatGPT 都可以），然后说："请一步一步带我完成这个安装。"
> **How to use:** Give this file (or its link) to Claude / ChatGPT and say: "Please guide me through this setup step by step."

---

## Instructions for the AI assistant

You are helping a user install an **n8n-based MCP server** that lets Claude generate images with the OpenAI API. After every image it shows tokens used, cost of this image, cumulative spend and an estimated remaining balance.

It consists of two n8n workflows in the `workflows/` folder:
1. `1-openai-image-core.json` — sub-workflow: calls OpenAI, reads the Costs API, builds the usage summary.
2. `2-claude-mcp-image-gen.json` — main workflow: an MCP Server Trigger exposing one tool that calls workflow 1.

**How to guide the user**
- The user may be a beginner. Go **one step at a time**; after each step ask for confirmation or a screenshot before continuing.
- Reply in the language the user writes in.
- Never ask the user to paste API keys or their MCP URL into the chat.
- n8n's UI changes between versions. If a button isn't where you describe it, ask for a screenshot and adapt.
- If something fails, ask for the exact error (n8n → Executions shows it) and fix that first.

**Prerequisites**
- An n8n instance with the **MCP Server Trigger** node (n8n Cloud, or self-hosted on a recent version). It must be reachable from the internet over HTTPS, so localhost won't work with Claude.
- An OpenAI Platform account with at least USD 5 of credit (platform.openai.com → Billing).
- A Claude plan that supports custom connectors.

### Step 1 — Create the OpenAI keys
1. platform.openai.com → **API keys** → Create new secret key (starts with `sk-`). Used for image generation.
2. Optional but needed for the balance line: Settings → **Admin keys** → create one with read access to usage/costs (starts with `sk-admin-`).
Keys are shown only once; the user should save them somewhere safe.

### Step 2 — Create the credentials in n8n
1. **OpenAI** credential: Credentials → Create → type **OpenAI** → paste the `sk-` key. Name it e.g. `OpenAI Image`.
2. **Header Auth** credential (for the admin key): Credentials → Create → type **Header Auth**.
   - Name: `Authorization`
   - Value: `Bearer sk-admin-...` (the word Bearer, one space, then the key). Missing "Bearer " causes a 401 error.
   Name the credential e.g. `OpenAI Admin Costs`.

### Step 3 — Import workflow 1 (OpenAI Image Core)
1. n8n → Create workflow → ⋯ menu → **Import from File** → choose `workflows/1-openai-image-core.json`.
2. Open node **Generate Image** → select the OpenAI credential.
3. Open node **Get Costs** → select the Header Auth credential.
   In its `start_time` query parameter, change the date `2026-01-01` to **today's date** (UTC). This is the "base date".
4. Open node **Build Usage Summary** → at the top set `BASE_BALANCE` to the balance shown today on platform.openai.com → Billing.
5. Save, then **Publish** / activate.
6. Note: OpenAI model names change. If generation fails with a model error, edit the `model` value in **Generate Image** (default `gpt-image-2.6-sunburst`) to the current image model.

### Step 4 — Import workflow 2 (Claude MCP for Image gen)
1. Import `workflows/2-claude-mcp-image-gen.json` the same way.
2. Open node **Generate Image Tool** → in **Workflow**, select `OpenAI Image Core` from the list. If the inputs show a warning, click the refresh icon next to Workflow Inputs.
3. Save, then **Publish** / activate.
4. Open node **MCP Server Trigger** → copy the **Production URL**.

**Explain clearly:** this URL works like a password. Anyone with it can spend the user's OpenAI credit. Never post it, show it in screenshots or videos, or commit it anywhere. To rotate it, change the trigger's Path to a new random value and republish. (Advanced: the trigger also supports Bearer/Header authentication.)

### Step 5 — Connect Claude
Claude (web or desktop) → Settings → Connectors → **Add custom connector** → paste the Production URL → save. It then also works in the Claude mobile app.
Test in a new chat: "Generate a 16:9 image of a cup of coffee on a wooden table."
Expected: an image plus four lines (tokens, this cost, cumulative spend, estimated balance).

### Step 6 — After topping up
When the user adds credit, update `BASE_BALANCE` (node Build Usage Summary) and the date in `start_time` (node Get Costs) to the new balance and today's date, then save and republish workflow 1.

### Troubleshooting
| Symptom | Likely cause |
|---|---|
| Claude can't find the tool | Connector not refreshed: disconnect and reconnect it in Claude settings; check both workflows are published |
| Error 401 in **Generate Image** | Wrong OpenAI key or credential not selected |
| Balance line says "查询失败" / lookup failed; 401 in **Get Costs** | Header Auth value missing `Bearer `, or not an Admin key |
| Error about model | Model name changed (Step 3.6) |
| Error about size | Size must be WxH, multiples of 16, longest side ≤ 3840, ratio ≤ 3:1 |
| Workflow 2 errors "workflow not found" | Step 4.2: re-select OpenAI Image Core, save, republish |
| Image doesn't appear in Claude | Some clients don't render large images; the summary text still shows |

Notes for the user: the balance is an **estimate** (OpenAI has no balance API; official spend stats can lag). Images are not stored anywhere by this setup.
