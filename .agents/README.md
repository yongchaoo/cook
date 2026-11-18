# cook AI Agent 统一入口

> 参考 `~/www/lexi` 的指引体系，按本项目量级裁剪：保留「单入口 + 权威分级 + 强制条款 + 验证门禁」，不引入重型治理脚本。
> **多工具 SSOT**：所有 AI 工具从 [../AGENTS.md](../AGENTS.md) 开始，规则不单绑 `.cursor/` 或某个 IDE。

## 读序（改代码前）

1. [../AGENTS.md](../AGENTS.md) — 入口与红线
2. 本文件 — 纪律与链接
3. [ARCHITECTURE.md](./ARCHITECTURE.md) — 目录地图 / 技术栈 / 边界
4. [rules/ai-coding-rules.md](./rules/ai-coding-rules.md) — **强制执行条款**
5. [rules/verification-gate.md](./rules/verification-gate.md) — 交付前验证门禁
6. [../plans/](../plans/) — 草案，仅作背景

## 文档权威性分级（MUST 遵守）

> AI 引用文档前 MUST 先判断它在哪一级。把低权威文档当决策依据 = 越权。

| 级别 | 位置 | 性质 | 可否作决策依据 |
|------|------|------|----------------|
| **SSOT（稳定）** | `AGENTS.md`、`.agents/`、`docs/` | 已采纳的边界 / 约定 / 约束 | ✅ 可 |
| **事实来源** | 代码本身（`miniprogram/**`、HTML 原型） | 最终真相，WHY 看 SSOT | ✅（结构/实现）/ ❌（WHY 须查 SSOT） |
| **过渡 / 草案** | `plans/` | 调研、计划、方案对比、未定稿；可随时删、会被推翻 | ❌ **禁止** |
| **过程记录** | `.workbuddy/memory/` | 工作日志 / 上下文 | ❌（仅作背景） |

### plans/ 硬规则

- `plans/**` **MUST NOT** 作为架构决策、行为契约、不变量的依据。
- 调研 / 分析类产出按用户习惯写进 `plans/`；被采纳后 **MUST** 迁移到对应 SSOT 位置：
  - 边界 / 结构 → `.agents/ARCHITECTURE.md`
  - 强制约束 → `.agents/rules/ai-coding-rules.md`
  - 项目事实 / 状态 → `AGENTS.md` 的「当前状态」
- 未迁移 = 未生效。冲突时：**SSOT > 代码 > plans/ > agent 判断**。

## 目录

```text
cook/
├── AGENTS.md                     ← AI 入口（先读）
├── .agents/
│   ├── README.md                 ← 你在这里：读序 + 权威分级
│   ├── ARCHITECTURE.md           ← 目录地图 / 技术栈 / 边界
│   └── rules/
│       ├── ai-coding-rules.md    ← 强制执行条款（MUST / MUST NOT）
│       └── verification-gate.md  ← 变更验证门禁
├── miniprogram/                  ← 原生小程序项目根（TypeScript，project.config.json 在此）
│   ├── app.ts / app.wxss / app.json
│   ├── tsconfig.json / package.json ← TS 类型检查（noEmit；编译由工具内置插件完成）
│   └── pages/index/              ← hello 页面四件套 wxml / wxss / ts / json
├── docs/wxamp-info.md            ← AppID / AppSecret（敏感）
├── plans/                        ← 调研 / 计划草案（非权威）
│   ├── 记忆里的美食-主体架构方案.md  ← 未实施设计稿，勿据此改代码
│   ├── 记忆里的美食-数据模型设计.md  ← 未实施设计稿
│   └── preview-qr.png             ← 真机预览二维码（临时产物）
├── cooking-timer-prototype.html  ← 多菜谱计时 HTML 原型（待移植）
└── _downloads/                   ← 安装包归档
```

## 协作约定（来自用户长期习惯）

- **中文交流，结论先行，有数据 / 证据支撑**；偏好表格化、结构化答案。
- 偏好**一次性完成、全权委托**，不喜欢逐次小步迭代挤牙膏。
- 对数据 / 结论真实性敏感：交付前 MUST 实际验证，不许把「文件已写好」当成「已跑通」。
- 样式 / 配色类改动先出可视化方案预览，选定后再落盘。
- 「先不改代码」= 暂停键，只出方案不动文件。
