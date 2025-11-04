# Steam Judger - AI 游戏库锐评

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

一个**前后端分离**的全栈项目。用户输入其 Steam ID 后，后端从 Steam Web API 获取公开的游戏列表，将数据存储在 **Cloudflare D1** 数据库中，并利用 AI 大语言模型（当前配置为 DeepSeek）生成一份关于用户游戏品味和习惯的“毒舌”锐评。前端使用 Nuxt 3 构建，负责用户交互和展示结果；后端 API 使用 Hono 构建，部署在 Cloudflare Workers 上，负责数据获取、存储和 AI 分析。

关联仓库：

- [Steam Judger - Steam 游戏库吐槽姬 (Frontend)](https://github.com/kutius/steam-judger)
- [Steam Judger - Steam 游戏库吐槽姬 (Backend)](https://github.com/kutius/steam-judger-backend)

## ✨ 主要功能

- **获取 Steam 游戏库:** 输入 64 位 Steam ID，通过 Steam Web API 获取用户公开的游戏列表和游玩时长。
- **D1 数据存储:** 将获取到的 Steam 游戏数据存储在 Cloudflare D1 数据库中，用于后续分析，并设置有效期以定期更新，避免频繁请求 Steam API。
- **AI 驱动的玩家画像:** 调用 AI 大模型（通过 OpenAI 兼容接口），基于用户的游戏数据生成一份幽默、讽刺且富有洞察力的“锐评”。
- **流式响应:** AI 生成的评价内容通过 Server-Sent Events (SSE) 流式传输到前端，提供即时反馈。
- **模型信息查询:** 提供 API 端点以查询当前后端正在使用的 AI 模型名称和版本。
- **用户 Steam 信息查询:** 提供 API 端点根据 Steam ID 查询用户的公开 Steam 个人资料信息（昵称、头像等）。
- **全栈 Cloudflare 部署:** 前后端均设计为可便捷地部署在 Cloudflare 平台上（Pages + Workers + D1）。

## 🚀 技术栈

- **前端:** [Nuxt 3](https://nuxt.com/)
- **后端 API:** [Hono](https://hono.dev/)
- **部署平台:** [Cloudflare Workers](https://workers.cloudflare.com/) (后端), [Cloudflare Pages](https://pages.cloudflare.com/) (前端)
- **数据库:** [Cloudflare D1](https://developers.cloudflare.com/d1/)
- **AI 模型:** [DeepSeek API](https://platform.deepseek.com/) (或其他 OpenAI 兼容模型，通过 OpenAI SDK 调用)
- **数据源:** [Steam Web API](https://steamcommunity.com/dev)

## 🔧 环境准备

在开始之前，请确保你已安装以下工具和拥有必要的账户/密钥：

- [Node.js](https://nodejs.org/) (建议使用 LTS 版本)
- [pnpm](https://pnpm.io/) (或 npm/yarn)
- [Cloudflare Account](https://dash.cloudflare.com/sign-up)
- [Wrangler CLI](https://developers.cloudflare.com/workers/wrangler/install-and-update/) (已登录并授权，用于部署 Cloudflare Worker 和管理 D1)
- 有效的 [Steam Web API Key](https://steamcommunity.com/dev/apikey)
- 有效的 [DeepSeek API Key](https://platform.deepseek.com/) (或其他 OpenAI 兼容模型的 API Key)
- 一个 Cloudflare D1 数据库实例（可在 Cloudflare Dashboard 创建）
- 一个拥有公开游戏库的 Steam 账户（用于测试）

## ⚙️ 安装与配置

1.  **克隆仓库:**
    项目包含前端和后端两个独立部分。

    ```bash
    # 克隆前端仓库 (Nuxt)
    git clone https://github.com/kutius/steam-judger.git steam-judger-frontend

    # 克隆后端仓库 (Hono on Workers)
    git clone https://github.com/kutius/steam-judger-backend.git steam-judger-backend
    ```

2.  **安装依赖:**
    分别进入前后端项目目录安装依赖。

    ```bash
    cd steam-judger-frontend
    pnpm install

    cd ../steam-judger-backend
    pnpm install
    ```

3.  **配置后端 (`steam-judger-backend`):**

    - **`wrangler.toml` 配置:**
      编辑 `wrangler.toml` 文件，配置 D1 数据库绑定。你需要先在 Cloudflare Dashboard 创建一个 D1 数据库，获取其 `database_name` 和 `database_id`。

      ```toml
      # backend/wrangler.toml (示例片段)
      name = "steam-judger-backend" # Worker 名称
      main = "src/index.ts"
      compatibility_date = "2024-03-18" # 使用适合你项目的日期

      # D1 数据库绑定
      [[d1_databases]]
      binding = "DB" # 在代码中访问数据库时使用的绑定名称 (例如 c.env.DB)
      database_name = "your-d1-database-name" # 你在 Cloudflare 上创建的 D1 数据库名称
      database_id = "your_d1_database_uuid_from_cloudflare" # D1 数据库的 UUID
      # preview_database_id = "your_preview_d1_database_uuid" # 可选，用于本地预览环境
      ```

    - **环境变量配置:**
      - **本地开发:** 在 `steam-judger-backend` 目录下创建一个 `.dev.vars` 文件，并填入以下内容：
        ```ini
        STEAM_API_KEY="YOUR_STEAM_API_KEY"
        OPENAI_API_KEY="YOUR_DEEPSEEK_API_KEY"
        # D1 绑定在 wrangler dev 运行时会根据 wrangler.toml 自动模拟或连接
        # OPENAI_BASE_URL="OPTIONAL_CUSTOM_OPENAI_COMPATIBLE_ENDPOINT" # 如果使用非官方 DeepSeek 或 OpenAI 端点
        ```
      - **Cloudflare 部署:**
        - 登录 Cloudflare Dashboard。
        - 导航到 Workers & Pages -> 你的 Worker (`steam-judger-backend`) -> Settings -> Variables。
        - 在 "Secret variables" 下添加 `STEAM_API_KEY` 和 `OPENAI_API_KEY`，并填入你的密钥值。如果需要自定义 OpenAI 端点，也在此处添加 `OPENAI_BASE_URL`。
        - D1 数据库绑定已在 `wrangler.toml` 中配置，部署时会自动关联。

<!-- 4.  **配置前端 (`steam-judger-frontend`) (可选):**
    如果前端需要明确知道后端的 URL（尤其是在生产环境），可以在 `steam-judger-frontend` 目录下创建 `.env` 文件：

    ```env
    # frontend/.env

    # 本地开发时指向 Wrangler dev 启动的后端地址
    NUXT_PUBLIC_API_BASE_URL=http://localhost:8787

    # 生产环境部署时，可以在 Cloudflare Pages 的环境变量中设置此值
    # 指向你部署的 Cloudflare Worker 的 URL
    # NUXT_PUBLIC_API_BASE_URL=https://your-worker-name.your-account.workers.dev
    ```

    在 Nuxt 代码中可以通过 `useRuntimeConfig().public.apiBaseUrl` 来获取这个基础 URL。如果前端和后端部署在同一个 Cloudflare 账户下，通常可以直接使用相对路径 `/api/...` 调用后端 API，无需此配置。 -->

## ▶️ 本地运行

1.  **启动后端 Worker (Hono):**
    在 `steam-judger-backend` 项目目录下运行：

    ```bash
    # 首次运行或 D1 schema 变更后，需要迁移数据库结构
    # npx wrangler d1 execute your-d1-database-name --local --file=./schema.sql

    # 启动本地开发服务器
    pnpm run dev
    ```

    Wrangler 会启动一个本地服务器（通常监听 `http://localhost:8787`），并模拟 D1 数据库环境。

2.  **启动前端应用 (Nuxt):**
    在 `steam-judger-frontend` 项目目录下运行：
    ```bash
    pnpm run dev
    ```
    Nuxt 会启动一个开发服务器（通常监听 `http://localhost:3000`）。在浏览器中打开此地址即可访问前端页面。

## ☁️ 部署到 Cloudflare

1.  **部署后端 Worker:**
    确保 `wrangler.toml` 配置正确，并且你已经在 Cloudflare 创建了对应的 D1 数据库。
    在 `steam-judger-backend` 目录下运行：

    ```bash
    # 部署前，确保生产环境的 D1 数据库结构是最新的
    # npx wrangler d1 execute your-d1-database-name --file=./schema.sql

    # 部署 Worker
    pnpm run deploy
    ```

    Wrangler 会将你的 Hono 应用部署到 Cloudflare Workers。记下部署后的 Worker URL（例如 `https://steam-judger-backend.your-account.workers.dev`）。

2.  **部署前端 Nuxt 应用:**
    - 将你的 `steam-judger-frontend` 代码推送到 Git 仓库 (GitHub, GitLab 等)。
    - 登录 Cloudflare Dashboard，导航到 Workers & Pages -> Create application -> Pages -> Connect to Git。
    - 选择你的前端仓库和分支。
    - 配置构建设置：
      - **Framework preset:** Nuxt
      - **Build command:** `pnpm build` (或 `npm run build`)
      - **Build output directory:** `.output/public`
    - (可选) 配置环境变量：
      - 添加 `NUXT_PUBLIC_API_BASE_URL` 并将其值设置为你部署的后端 Worker URL（如果前端需要显式指定）。
    - 点击 "Save and Deploy"。

## 🎮 使用方法

1.  访问部署好的前端应用 URL（Cloudflare Pages 的 URL）。
2.  在输入框中输入一个有效的 64 位 Steam ID。
3.  点击提交按钮。
4.  前端调用后端 `/api/analyze/:steamid` 接口。
5.  后端首先检查 D1 数据库中是否有该 Steam ID 的有效游戏数据记录。
6.  如果记录不存在或已过期，后端调用 Steam API 获取最新数据，并存入 D1。
7.  后端使用 D1 中的数据调用 AI 大模型生成分析评价。
8.  AI 生成的锐评通过 SSE 流式传输回前端并显示在页面上。
9.  页面同时会尝试调用 `/api/user/:steamid` 获取并展示用户的 Steam 昵称和头像。

## 📝 API 端点 (后端 Hono 应用)

所有 API 路径建议以 `/api` 开头，方便前端代理或区分。

- `GET /api/analyze/:steamid`

  - 接收 64 位 Steam ID 作为路径参数。
  - 检查 D1 数据库中是否有该 Steam ID 的有效游戏数据记录 (`dataId`)。
  - 如果不存在或已过期，调用 Steam API 获取最新数据，存入 D1（设置 TTL），并获取 `dataId`。
  - 从 D1 中获取 `dataId` 对应的游戏数据。
  - 如果数据不存在（异常情况），返回错误。
  - 调用配置的 AI 大模型生成分析评价。
  - 通过 `text/event-stream` 流式返回 AI 生成的内容。

- `GET /api/user/:steamid`

  - 接收 64 位 Steam ID 作为路径参数。
  - 调用 Steam Web API 获取用户的公开个人资料信息（昵称、头像 URL 等）。
  - 返回用户信息 JSON 对象。

- `GET /api/model`
  - 返回当前后端配置用于分析的 AI 模型名称和版本信息。
  - **响应示例:** `{ "modelName": "deepseek-chat", "version": "v1.0" }` (版本信息可根据实际情况添加)

## 🤝 贡献

欢迎通过提交 Pull Requests 或创建 Issues 来改进项目！无论是功能建议、代码优化还是 Bug 修复，都非常感谢。

## 🙏 致谢与赞助

**特别致谢** `小黑盒`的朋友对本项目的支持！

特别感谢所有为本项目提供灵感、帮助和反馈的朋友们！

如果你觉得这个项目对你有帮助或启发，并且希望支持我继续开发和维护类似的项目，欢迎考虑：

- 给我一个 Star ⭐！
- [Buy Me a Coffee](https://vapor.onex.email/sponsor)

你的支持是我前进的最大动力！

## 📄 License

本项目采用 [MIT License](LICENSE)。
