# DeepSeek Usage+ — 官方 API 用量页增强仪表盘

> 在 DeepSeek 官方 API 用量页（`platform.deepseek.com/usage`）之外补充输入/输出拆分、缓存命中、均价、预估可用、模型明细表、按 API Key 汇总的费用明细与结构图表，并在对话页提供用量入口。

一个油猴脚本（Userscript），无需后端、无需额外授权（`@grant none`），直接读取你已登录的 DeepSeek 官方页面数据，在页面上叠加一个增强分析仪表盘。

---

## 功能特性

### 用量页（`platform.deepseek.com/usage`）

在官方总览面板下方注入 `DeepSeek Usage+` 增强仪表盘：

**概览卡片**

- **今日消费**：当日已产生费用（CNY）
- **区间费用**：当前所选时间区间内的总费用（CNY）
- **平均单价**：区间内每 1M Token 的均价（CNY / 1M）
- **缓存命中**：Prompt Cache 命中率（%）
- **输入 Tokens**：区间输入 Token 拆分（含命中 / 未命中明细）
- **预估可用**：按当前区间均价与账户余额估算的可用 Token 数

**结构图表**（基于 ECharts 5，默认折叠，可展开记忆状态）

- 趋势折线图（line）：按日用量走势
- 结构饼图（pie）：输入 / 输出 / 缓存命中占比
- 对比柱状图（bar）：模型或 API Key 维度对比

**明细表**

- 模型明细表：各模型的输入 / 输出 / 缓存 / 费用
- 按 API Key 汇总：当前区间内每个 Key 的费用明细与结构

**时间区间**

- 今天 / 昨天 / 近 7 天 / 近 30 天 / 本月 / 上月 / 自定义
- 切换区间后仪表盘自动重新拉取并刷新

**体验细节**

- 自动适配 DeepSeek 官方明 / 暗主题（复用页面 CSS 变量）
- 面板折叠状态、额外图表开关持久化（`localStorage`）
- 路由切换 / 页面刷新自动重新挂载，无需手动操作

### 对话页（`chat.deepseek.com`）

- 在对话页注入「API 用量」入口按钮，点击直达用量页

---

## 安装

### 1. 安装一个用户脚本管理器

任选其一（浏览器扩展）：

- [Tampermonkey](https://www.tampermonkey.net/)（推荐，支持 Chrome / Edge / Firefox / Safari）
- [Violentmonkey](https://violentmonkey.github.io/)
- [GreaseMonkey](https://addons.mozilla.org/firefox/addon/greasemonkey/)（仅 Firefox）

### 2. 安装本脚本

点击下方链接，用户脚本管理器会弹出安装确认：

**[安装 DeepSeek Usage+](https://raw.githubusercontent.com/harewise/DeepSeek_Usage_Plus/main/deepseek.user.js)**

或手动复制 [deepseek.user.js](deepseek.user.js) 的内容，粘贴到你的脚本管理器中新建脚本保存。

脚本会自动从 jsDelivr CDN 加载 ECharts 5.6.0（`@require`），无需你手动处理依赖。

### 3. 使用

1. 登录 [DeepSeek 开放平台](https://platform.deepseek.com/usage)。
2. 打开「用量」页，增强仪表盘会自动出现在官方总览下方。
3. 在 [DeepSeek 对话页](https://chat.deepseek.com/) 侧边栏会出现「API 用量」入口按钮。

---

## 数据来源与隐私

- 脚本仅在你的浏览器内运行，**不向任何第三方服务器发送数据**。
- 用量数据来自 DeepSeek 官方页面已有的接口响应（复用你登录后的会话），脚本本身不存储或外传任何 token、用量、余额信息。
- `@grant none`：脚本不请求任何油猴特殊权限，仅以页面脚本身份运行。
- 偏好（面板折叠状态、额外图表开关）保存在浏览器 `localStorage`，清除站点数据即被清除。

---

## 兼容性

| 项目 | 说明 |
| --- | --- |
| 浏览器 | 任意支持用户脚本管理器的现代浏览器 |
| 生效页面 | `platform.deepseek.com/*`（用量页）与 `chat.deepseek.com/*`（对话页） |
| 依赖 | ECharts 5.6.0（由 `@require` 自动加载） |
| 运行时机 | `document-idle` |

> 由于 DeepSeek 官方页面结构可能随版本更新而调整，若仪表盘无法挂载或数据读不到，请在此仓库提 issue 反馈。

---

## 开发

本项目为单文件用户脚本 [`deepseek.user.js`](deepseek.user.js)，结构如下：

- 顶部 `// ==UserScript==` 元数据块：声明 `@match`、`@require`、`@downloadURL`、`@updateURL`、`@version` 等。
- IIFE 包裹的脚本主体：
  - 常量与 `state`：DOM id、存储键、Token 类型、日期区间标签、运行状态。
  - 数据层：从官方接口响应里健壮地提取用量 / 费用 / 模型 / API Key 数据（兼容多种字段命名）。
  - 渲染层：概览卡片、明细表、ECharts 图表。
  - 生命周期：MutationObserver / 路由轮询自动挂载与重渲染。

### 本地调试

1. 克隆仓库：`git clone https://github.com/harewise/DeepSeek_Usage_Plus.git`
2. 在 Tampermonkey 中新建脚本，把 `deepseek.user.js` 内容粘进去，或开启「允许访问文件 URL」后用本地文件安装。
3. 修改后刷新 DeepSeek 用量页即可看到效果。

### 发布新版本

1. 修改脚本顶部 `@version`（语义化版本号，如 `1.0.1`）。
2. 提交并推送到 `main` 分支。
3. 已安装该脚本的用户，其脚本管理器会按 `@updateURL` 检测到版本号变化并提示更新。

---

## License

[MIT](deepseek.user.js#L12)（见脚本头 `@license MIT`）。
