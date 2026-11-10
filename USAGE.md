# 紫微斗数 · 部署与使用文档

本文档只讲一件事：**拿到这个仓库后，怎么把它跑起来、怎么用、要准备什么**。
项目背景、数据集、许可协议见 [README.md](./README.md)。

---

## 一、项目概览

| 项目 | 内容 |
|------|------|
| 应用类型 | Next.js 16（App Router）Web 应用，前后端同仓 |
| 语言 | TypeScript 5 |
| 关键依赖 | Next.js `16.4.3`、React `19.2.5`、Tailwind CSS `3.4`、Framer Motion `11` |
| 排盘内核 | 本仓库 `lib/ziwei/`（基于 [iztro](https://github.com/SylarLong/iztro) + `lunar-javascript`） |
| 包管理 | npm（仓库已提交 `package-lock.json`） |

**运行形态**：整套排盘（安命宫、十四主星、四化、大限流年、格局判定）都在**浏览器端**用 `lib/ziwei/` 完成，
因此**跑起来不依赖任何后端服务、不需要 API Key、不需要数据库**。

**默认端口**：**4008**（已写进 `package.json` 的 `dev` / `start` 脚本：`next dev -p 4008`、`next start -p 4008`）。

> 开源版**不含** `/api/` 后端路由。AI 解读（`/api/interpret`）与合盘 AI 分析（`/api/heming`）
> 需要你自己实现；未实现时界面会直接提示「开源版不含该接口」，配 Key 也不会变可用。

---

## 二、环境要求

| 要求 | 版本 | 依据 |
|------|------|------|
| Node.js | **≥ 20.9.0**（CI 用 22，开发机验证过 24） | `next@16.4.3` 的 `engines` 字段 |
| npm | 随 Node 附带即可（CI 用 npm 11） | — |
| 操作系统 | macOS / Linux / Windows 均可 | — |

无需 PostgreSQL、MySQL、Redis、MongoDB 等任何数据库或中间件。

---

## 三、部署启动

### 3.1 拉代码 + 装依赖

```bash
git clone https://github.com/Renhuai123/ziwei-doushu.git
cd ziwei-doushu

# 仓库有 package-lock.json：CI / 生产建议用 ci（严格按锁定版本装，不改 lock 文件）
npm ci

# 本地开发也可以用 install
npm install
```

> `.npmrc` 里设了 `legacy-peer-deps=true`，装依赖时不需要额外加参数。

### 3.2 配置环境变量（可选）

```bash
cp .env.example .env.local
```

**只想排盘看命盘的话，这一步可以整个跳过**——不配任何变量也能正常启动。
可配置项见[第四节](#四环境变量)。

### 3.3 开发模式启动

```bash
npm run dev
```

- 启动后访问 **http://localhost:4008**
- 换端口：`npm run dev -- -p 3000`（或 `PORT=3000 npm run dev`）——命令行参数优先于脚本里的 `-p 4008`

### 3.4 生产模式启动

```bash
npm run build     # 构建产物在 .next/
npm start         # 以生产模式启动
```

- 同样默认监听 **4008** 端口（脚本已带 `-p 4008`）
- 换端口：`npm start -- -p 3000` 或 `PORT=3000 npm start`
- `next.config.js` 未配置 `output: 'standalone'`，因此部署时需要主机上**保留 `node_modules/` 与 `package.json`**，
  用 `npm start` 拉起，而不是只拷 `.next/` 目录。

---

## 四、环境变量

来源：[`.env.example`](./.env.example)。变量分两类：

| 变量 | 是否必需 | 作用 |
|------|----------|------|
| `NEXT_PUBLIC_SITE_URL` | 建议填 | 站点根地址。用于 `sitemap`、`robots`、canonical、分享卡底部域名。不填则回退 `http://localhost:4008`（见 `lib/site.ts`），**fork 后务必改成自己的域名**，否则会指向别人或本机地址 |
| `NEXT_PUBLIC_GOOGLE_VERIFICATION` / `NEXT_PUBLIC_BING_VERIFICATION` / `NEXT_PUBLIC_BAIDU_VERIFICATION` / `NEXT_PUBLIC_360_VERIFICATION` | 可选 | 各搜索引擎站长平台的站点验证 meta |
| `AI_PROVIDER` / `DEEPSEEK_API_KEY` 等 AI 变量 | 可选 | **开源版代码里没有任何引用**。只有你自己实现了 `/api/interpret`、`/api/heming` 之后才会用到 |

注意：`NEXT_PUBLIC_*` 变量在**构建时**被打包进前端产物，生产环境必须先写好变量再 `npm run build`，改完要重新构建。

`.env*` 已被 `.gitignore` 忽略（`.env.example` 除外），不会误提交。

---

## 五、需要什么数据库？

**不需要任何数据库。**

- 排盘是纯前端计算：表单 → `lib/ziwei/` 引擎 → 命盘组件，全程在浏览器内存里完成，不落库。
- 古籍原文（`lib/classics/`）、SEO 知识图谱（`lib/seo/`）、名人样本（`lib/ziwei/famous.ts`）都是**随包发布的静态 TS 数据**。
- 没有用户系统、没有登录/会员/支付、没有服务端会话，因此没有持久化需求。

**关于 `package.json` 里的 `pg`、`ioredis`、`@anthropic-ai/sdk`**：
这三个包已在依赖列表里，但**本仓库的源码中没有任何引用**（全仓检索无 import / require）。
它们是为自建后端（数据库连接池、Redis 缓存、LLM 调用）预留的，你如果要自己实现 `/api/*`，
可以直接用它们，也可以按自己的技术选型替换。

---

## 六、如何使用

### 6.1 页面一览

启动后可用路由（均来自 `app/` 目录）：

| 路由 | 作用 |
|------|------|
| `/` | 首页落地页：星空背景 + 滚动引导，按钮跳转 `/chart`、`/heming`（**表单不在首页**） |
| `/chart` | **排盘页**：填生日、时辰、性别、出生地 → 出命盘方格 + 宫位详情 + 运限切换 |
| `/heming` | 合盘页：两张盘在本地区分开起盘，**AI 分析需自建 `/api/heming`** |
| `/knowledge` | 命理百科：14 主星 × 13 宫位知识 |
| `/knowledge/[star]/[topic]` | 星曜 × 主题详情页 |
| `/library` | 古籍阅读器（骨髓赋 / 紫微斗数全集 / 全书） |
| `/library/[book]`、`/library/[book]/[chapter]` | 古籍书目与章节 |
| `/library/search` | 古籍全文搜索 |
| `/preview` | 「紫微卷轴」开场动画预览（可重播） |
| `/privacy`、`/terms` | 隐私政策、服务条款 |

### 6.2 典型使用流程

1. 打开 http://localhost:4008，点按钮进入 **`/chart`（排盘）**
2. 在 `/chart` 的出生信息表单里填写：
   - **时辰是时辰序号或钟点**，注意晚子时（23:00–23:59）按次日排盘
   - 填出生地才会做**真太阳时校正**（按经度 + 均时差），不填则不做校正
   - 1986–1991 年出生会涉及中国夏令时：引擎**只判定、不自动扣**，需要由用户确认后用 `clock-adjust.ts` 换成标准时间
3. 出盘后切换 **本命 / 大限 / 流年**，点宫位看星曜详情
4. 想要 AI 解读，需先自行实现 `POST /api/interpret`（请求体 `{ chart, messages }`），
   合盘同理实现 `POST /api/heming`（请求体 `{ chartA, chartB, question? }`）；
   可参考 `lib/ziwei/patterns.ts`、`heming-knowledge.ts` 里的知识库自建 prompt

### 6.3 Demo 的能力边界

本仓库界面是**最小可运行示例**：能跑通出生信息表单、命盘方格与宫位详情、运限切换、
古籍阅读、命理百科、亮暗主题、移动端适配。
**断语内容本身不在开源范围**（`lib/ziwei/db-analysis.ts` 是占位实现），
所以画得出盘，但给不出线上那套 13 维解读。

---

## 七、构建校验与回归测试

```bash
npm run typecheck   # TypeScript 全量类型检查
npm test            # 排盘引擎回归（盘完整性、夏令时、立春边界、时辰序号边界、日期合法性等）
```

`npm test` 会依次跑 `scripts/regression/` 下的全部用例。每次 push / PR 在 GitHub Actions
（[`.github/workflows/ci.yml`](./.github/workflows/ci.yml)）上以 Node 22 跑同一套。

---

## 八、常见问题

| 现象 | 原因 / 处理 |
|------|-------------|
| 点「解读」提示「开源版不含 AI 解读接口（/api/interpret）」 | 正常。开源版不含后端接口，需要自己实现 |
| 合盘页提示「开源版不含合盘 AI 分析接口（/api/heming）」 | 同上 |
| `npm start` 报找不到 `.next` | 先执行 `npm run build` |
| 端口 4008 被占用 | `npm run dev -- -p 4000`，或 `PORT=4000 npm run dev`（生产同理 `npm start -- -p 4000`） |
| 装依赖报 peer 冲突 | 确认 `.npmrc` 的 `legacy-peer-deps=true` 生效（或加 `--legacy-peer-deps`） |
| sitemap / canonical 指向 localhost 或别人域名 | `.env.local` 里设置 `NEXT_PUBLIC_SITE_URL` 后重新构建 |
| 要不要起数据库？ | 不用。排盘全在浏览器端完成 |

---

## 九、许可提示（部署前必读）

| 内容 | 协议 |
|------|------|
| 代码（`lib/`、`app/`、`components/`） | MIT |
| 数据集（Releases，v3.0 / v4.0） | CC BY 4.0，需保留来源标注 |
| 古籍原文 | 公有领域 |
| 倪师原话（`docs/tianji-ziwei/`） | CC BY-NC-SA 4.0，**不许商用** |

详细条款见 [README.md 第九章](./README.md#九协议)。
