# cook 架构边界（AI 用）

> **本文件是边界 SSOT**。只答「改哪里、什么禁止、深主题去哪查」。完整入口见 [README.md](./README.md)。

## 两条线（MUST 分清）

| 线 | 载体 | 定位 | 当前阶段 |
|----|------|------|----------|
| **原型线** | `cooking-timer-prototype.html` | 单文件 HTML 交互原型，12 菜 / 58 步；浏览器直接打开 | v7，设计已定稿，**冻结待移植** |
| **小程序线** | `miniprogram/` | 微信原生小程序（WXML / WXSS / **TypeScript** / JSON），真机运行 | **骨架阶段**：只有 hello 页面 |

两条线独立演进。原型是**设计参考与数据来源**，不是可直接运行的小程序代码。**MUST NOT** 把 HTML 原型的 DOM / Web API 原样搬进小程序。

## 目录地图

```text
miniprogram/
├── project.config.json     ← 项目配置（AppID + setting.useCompilerPlugins: ["typescript"]）
├── tsconfig.json           ← TS 类型检查配置（noEmit: true，只检查不产出 js）
├── package.json            ← devDependencies：typescript / miniprogram-api-typings
├── app.ts                  ← App() 全局生命周期（TS）
├── app.json                ← 全局配置：pages 注册、window、tabBar 等
├── app.wxss                ← 全局样式
├── sitemap.json            ← 索引配置
└── pages/
    └── index/              ← 页面固定四件套：wxml / wxss / ts / json
        ├── index.wxml      ← 结构
        ├── index.wxss      ← 样式
        ├── index.ts        ← Page() 逻辑（TS）
        └── index.json      ← 页面配置
```

**源码目录只保留 `.ts`，不生成同名 `.js`**；JS 由微信开发者工具内置编译插件按需生成，不落盘。

### 改动路由

| 改什么 | 落点 |
|--------|------|
| 新增页面 | `miniprogram/pages/<name>/` 四件套（`.ts`） + `app.json` 的 `pages` 注册 |
| 全局窗口 / 导航 / tabBar | `app.json` |
| 全局样式 / 主题变量 | `app.wxss` |
| 全局生命周期 / 全局数据 | `app.ts` |
| 页面结构 | `pages/<name>/<name>.wxml` |
| 页面逻辑 | `pages/<name>/<name>.ts` |
| 项目编译设置 / AppID / 编译插件 | `project.config.json` |
| TS 类型检查规则 / 依赖 | `tsconfig.json`、`package.json` |
| 原型交互 / 菜谱数据 | `cooking-timer-prototype.html`（冻结，改动须先确认） |

## TypeScript 约定（2026-11-18 起）

- **编译**：`project.config.json` → `setting.useCompilerPlugins: ["typescript"]`（工具 1.05.2109101+ 支持）。项目内只写 `.ts`，工具用 `@babel/plugin-transform-typescript` 按需转译，**不生成同名 `.js`**；`miniprogram-ci` 1.6.1+ 同样支持。
- **关键限制**：该插件**只剥离类型注解，不做类型检查**，类型错误照样编译成功、真机照跑，只在编辑器画红线。
- **因此** `tsc --noEmit`（`npm run typecheck`）是**独立且 MUST** 的门禁。`tsconfig.json`：`noEmit: true`、`strict: true`、`module: Preserve` + `moduleResolution: bundler`（TypeScript 7 已移除 `node10`）、`types: ["miniprogram-api-typings"]`。
- 依赖 `typescript` / `miniprogram-api-typings` 仅作 devDependencies，**不进小程序运行时包**（`node_modules/` 已忽略）。
- 交付前 MUST 按 [rules/verification-gate.md](./rules/verification-gate.md) 顺序：静态校验 → typecheck → 工具编译 → 真机预览。

## 技术栈边界（MUST NOT）

- **原生小程序 + TypeScript**，不引入 Taro / uni-app / mpvue / Vant / npm **构建链**。TS 编译由工具内置插件完成；npm 依赖只用于类型声明与 `tsc` 检查，不产出运行时代码、不参与小程序编译。
- **不引入任何 Web API**：`document` / `window` / `localStorage` / `Web Audio` / `Notification` / `navigator.vibrate` 在小程序运行时不存在。对应关系：

  | HTML 原型 | 小程序替代 |
  |-----------|------------|
  | `localStorage` / `sessionStorage` | `wx.setStorageSync` / `wx.getStorageSync` |
  | `new Audio()` / Web Audio | `wx.createInnerAudioContext()` |
  | `navigator.vibrate` | `wx.vibrateShort()` / `wx.vibrateLong()` |
  | `Notification` | 订阅消息（需后端 + 用户授权） |
  | `setInterval` 前台计时 | `setInterval` + `wx.setStorageSync('endAt')`，`onShow` 时按 `endAt - Date.now()` 重算 |
  | DOM 事件 / `innerHTML` | `bindtap` + `setData()` 数据驱动 |

- **切后台计时 MUST 用时间戳重算**，不得依赖 `setInterval` 持续累计（小程序后台会被挂起）。
- 页面 **MUST** 保持四件套齐全（`wxml` / `wxss` / `ts` / `json`）；新增页面 **MUST** 同步注册到 `app.json.pages`。
- 配色沿用原型已定的「元气橙蓝」主题（主色 `#E85D04` / `#FF7A00`，撞色 `#2E90FA`，底色 `#FFF4E3`），不擅自换色。

## 敏感信息边界（Hard red line）

| 项 | 允许位置 | 禁止位置 |
|----|----------|----------|
| AppID `wxef484150791ff62b` | `project.config.json`、服务端配置 | — |
| **AppSecret** | **仅服务端 / 环境变量** | `miniprogram/**` 任何文件、`plans/**`、提交到版本库的任何文件 |

凭证原文存放在 `docs/wxamp-info.md`（仅本地，已被 `.gitignore` 忽略）。AI **MUST NOT** 把 AppSecret 复制进代码、注释、配置样例或日志。

## 维护规则

- 改目录结构 / 新增模块 **MUST** 同次更新本文件与 [../AGENTS.md](../AGENTS.md) 的「当前状态」。
- 边界变更 **MUST** 先改本文件，再改代码。
- 发现文档与代码漂移 = bug，应立即修一边使其一致。
