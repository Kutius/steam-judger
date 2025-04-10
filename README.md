# Steam Judger

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

一个全栈项目，允许用户输入他们的 Steam ID，获取他们拥有的游戏列表，并利用 AI 大语言模型（当前配置为 DeepSeek）生成一份关于他们游戏品味和习惯的“毒舌”锐评。前端使用 Nuxt 3 构建，后端 API 使用 Hono 构建并部署在 Cloudflare Workers 上，利用 Cloudflare KV 进行数据缓存。

## ✨ 主要功能

- **获取 Steam 游戏库:** 输入 64 位 Steam ID，通过 Steam Web API 获取用户公开的游戏列表和游玩时长。
- **KV 缓存:** 将获取到的游戏数据缓存在 Cloudflare KV 中，有效期为 7 天，避免频繁请求 Steam API。
- **AI 驱动的玩家画像:** 调用 AI 大模型（通过 OpenAI 兼容接口），基于用户的游戏数据生成一份幽默、讽刺且富有洞察力的“锐评”。
- **流式响应:** AI 生成的评价内容通过 Server-Sent Events (SSE) 流式传输到前端，提供即时反馈。
- **模型信息查询:** 提供 API 端点以查询当前后端正在使用的 AI 模型名称。
- **全栈部署:** 前后端均设计为部署在 Cloudflare 平台上（Pages + Workers）。

## 🚀 技术栈

- **前端:** [Nuxt 3](https://nuxt.com/)
- **后端 API:** [Hono](https://hono.dev/)
- **部署平台:** [Cloudflare Workers](https://workers.cloudflare.com/) (后端), [Cloudflare Pages](https://pages.cloudflare.com/) (前端)
- **缓存:** [Cloudflare KV](https://developers.cloudflare.com/kv/)
- **AI 模型:** [DeepSeek API](https://platform.deepseek.com/) (通过 OpenAI SDK 调用)
- **数据源:** [Steam Web API](https://steamcommunity.com/dev)

## 🔧 环境准备

在开始之前，请确保你已安装以下工具和拥有必要的账户/密钥：

- [Node.js](https://nodejs.org/) (建议使用 LTS 版本)
- [pnpm](https://pnpm.io/) (或 npm/yarn)
- [Cloudflare Account](https://dash.cloudflare.com/sign-up)
- [Wrangler CLI](https://developers.cloudflare.com/workers/wrangler/install-and-update/) (用于部署 Cloudflare Worker)
- 有效的 [Steam Web API Key](https://steamcommunity.com/dev/apikey)
- 有效的 [DeepSeek API Key](https://platform.deepseek.com/) (或其他 OpenAI 兼容模型的 API Key)
- 一个拥有公开游戏库的 Steam 账户（用于测试）

## ⚙️ 安装与配置

1.  **克隆仓库:**

    ```bash
    # 克隆前端仓库
    git clone https://github.com/kutius/steam-judger.git

    # 克隆后端仓库
    git clone https://github.com/kutius/steam-judger-backend.git
    ```

2.  **安装依赖:**

    ```bash
    cd steam-judger
    pnpm install

    cd .. && cd steam-judger-backend
    pnpm install
    ```

3.  **配置后端环境变量:**

    - **本地开发:** 在 _后端项目_ 目录下创建一个 `.dev.vars` 文件，并填入以下内容：
      ```ini
      STEAM_API_KEY="YOUR_STEAM_API_KEY"
      OPENAI_API_KEY="YOUR_DEEPSEEK_API_KEY"
      # MY_KV 绑定将在运行 wrangler dev 时自动模拟或需要本地配置
      ```
    - **Cloudflare 部署:**
      - 登录 Cloudflare Dashboard。
      - 导航到 Workers & Pages -> 你的 Worker -> Settings -> Variables。
      - 在 "Secret variables" 下添加 `STEAM_API_KEY` 和 `OPENAI_API_KEY`，并填入你的密钥值。
      - KV Namespace Binding (`MY_KV`) 需要在 `wrangler.toml` 文件中配置，并在 Cloudflare 上创建对应的 KV 命名空间。

4.  **配置 `wrangler.toml` (后端):**
    确保 `backend/wrangler.toml` 文件配置正确，特别是 `kv_namespaces` 部分，需要指定 `binding` 为 `"MY_KV"` 并提供 Cloudflare 上的 `id`。

    ```toml
    # backend/wrangler.toml (示例片段)
    name = "steam-analyzer-backend"
    main = "src/index.ts"
    compatibility_date = "2023-10-30" # Or your desired date

    # KV Namespace Binding
    [[kv_namespaces]]
    binding = "MY_KV"
    id = "your_kv_namespace_id_from_cloudflare"
    # preview_id = "your_preview_kv_namespace_id" # Optional for wrangler dev preview
    ```

; 5. **配置前端环境变量 (可选):**
; 如果前端需要知道后端的 URL，可以在 `frontend/` 目录下创建 `.env` 文件，例如：
; `env
;     # frontend/.env
;     # 本地开发时指向 Wrangler dev 启动的地址
;     NUXT_PUBLIC_API_BASE_URL=http://localhost:8787
;     # 生产环境可以在 Cloudflare Pages 的环境变量中设置
;     # NUXT_PUBLIC_API_BASE_URL=https://your-worker-url.workers.dev
;     `
; 并在 Nuxt 代码中使用 `useRuntimeConfig().public.apiBaseUrl` 获取。

## ▶️ 本地运行

1.  **启动后端 Worker (Hono):**
    在 `steam-judger-backend` 项目下运行：

    ```bash
    pnpm run dev
    ```

    Wrangler 会启动一个本地服务器，通常监听 `http://localhost:8787`。

2.  **启动前端应用 (Nuxt):**
    在 `steam-judger` 项目下运行：
    ```bash
    pnpm run dev
    ```
    Nuxt 会启动一个开发服务器，通常监听 `http://localhost:3000`。在浏览器中打开此地址。

## ☁️ 部署到 Cloudflare

1.  **部署后端 Worker:**
    在 `backend/` 目录下运行：

    ```bash
    pnpm run deploy
    ```

    Wrangler 会将你的 Hono 应用部署到 Cloudflare Workers。确保你已经在 Cloudflare Dashboard 配置了 Secrets 和 KV Namespace。记下部署后的 Worker URL。

2.  **部署前端 Nuxt 应用:**
    - 将你的代码推送到 Git 仓库 (GitHub, GitLab 等)。
    - 登录 Cloudflare Dashboard，导航到 Workers & Pages -> Create application -> Pages -> Connect to Git。
    - 选择你的仓库和分支。
    - 配置构建设置：
      - **Framework preset:** Nuxt
      - **Build command:** `pnpm run build` (或 `npm run build`)
      - **Build output directory:** `.output/public`
    - (可选) 配置环境变量，例如 `NUXT_PUBLIC_API_BASE_URL` 指向你部署的 Worker URL。
    - 点击 "Save and Deploy"。

## 🎮 使用方法

1.  访问部署好的前端应用 URL。
2.  在输入框中输入一个有效的 64 位 Steam ID。
3.  点击提交按钮。
4.  应用会首先调用后端 `/games/:steamid` 接口获取或检查缓存的游戏数据。
5.  然后调用 `/analyze/data/:dataId` 接口。
6.  AI 生成的锐评会流式显示在页面上。

## 📝 API 端点 (后端)

- `GET /games/:steamid`

  - 接收 64 位 Steam ID 作为路径参数。
  - 检查 KV 缓存。如果命中且未过期，返回缓存数据的 ID (`dataId`)。
  - 如果未命中或已过期，调用 Steam API 获取最新数据，存入 KV (7 天 TTL)，并返回新数据的 ID (`dataId`)。
  - **响应示例 (缓存命中):** `{ "dataId": "steamgames:76561198...", "source": "cache" }`
  - **响应示例 (API 获取):** `{ "dataId": "steamgames:76561198...", "source": "api", "gamesFound": 150 }`

- `GET /analyze/data/:dataId`

  - 接收 `/games` 接口返回的 `dataId` 作为路径参数。
  - 从 KV 中获取 `dataId` 对应的游戏数据。
  - 如果数据不存在，返回 404。
  - 调用配置的 AI 大模型（如 DeepSeek）生成分析评价。
  - 通过 `text/event-stream` 流式返回 AI 生成的内容。

- `GET /model`
  - 返回当前后端配置用于分析的 AI 模型名称。
  - **响应示例:** `{ "modelName": "deepseek-chat" }`

## 🤝 贡献

欢迎提交 Pull Requests 或创建 Issues 来改进项目！

##📄 License

本项目采用 [MIT License](LICENSE)。
