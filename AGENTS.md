# AI Agent Entry — cook

> **多工具 SSOT**：Cursor / Claude Code / Codex / WorkBuddy 等一律从本文件开始，规则不单绑某个工具目录。
> 项目定位：「记忆里的美食」微信小程序（结构化菜谱内容库 + 消费端/生产端双端，计时器只是消费端的「做菜模式」）。当前处于**骨架跑通阶段**：先让原生小程序 hello 页面在真机可见，不实现业务功能。
> `plans/记忆里的美食-*.md` 是**未实施的架构设计稿**（草案、非权威），**禁止**据此改代码或当作现行约束；被采纳后须迁移到 `.agents/`。

## 读序（改代码前，按序）

1. 本文件 — 入口与红线
2. [.agents/README.md](./.agents/README.md) — 文档权威性分级、读序、目录
3. [.agents/ARCHITECTURE.md](./.agents/ARCHITECTURE.md) — 目录地图、技术栈、边界
4. [.agents/rules/ai-coding-rules.md](./.agents/rules/ai-coding-rules.md) — **强制执行条款（必读必守）**
5. [.agents/rules/verification-gate.md](./.agents/rules/verification-gate.md) — 交付前验证门禁
6. [plans/](./plans/) — 调研 / 计划草案，**仅作背景，不是决策依据**

## Hard red lines（不可违反）

1. **AppSecret 等凭证禁止写入小程序前端**（`miniprogram/**` 任何文件），凭证只留在服务端。见 [docs/wxamp-info.md](./docs/wxamp-info.md)。
2. **改样式 / 配色前先出可视化方案预览**；用户未明确选定前不得落盘改文件。
3. **用户说「先不改代码」「不要直接改」时**：只给方案，不改任何文件。
4. `plans/**` 是草案，**禁止**作为架构决策 / 行为契约 / 不变量的依据；被采纳后须迁移到 SSOT。
5. **先跑通再逐步加功能**：不要一次性把 `cooking-timer-prototype.html` 全量搬进小程序。本阶段只交付最简 hello 页面。
6. 不实现用户没要求的功能；不顺手重构没坏的东西。

## 当前状态（2026-11-18）

| 资产 | 路径 | 状态 |
|------|------|------|
| 原生小程序骨架（**TypeScript**） | `miniprogram/` | 单一 hello 页面；`npm run typecheck` 0 error、工具编译通过、真机预览二维码已生成（`plans/preview-qr.png`） |
| 类型检查依赖 | `miniprogram/package.json` | devDependencies：`typescript` + `miniprogram-api-typings`；`node_modules/` 已忽略，不进小程序包 |
| 多菜谱计时 HTML 原型 | `cooking-timer-prototype.html` | 12 菜 / 58 步，元气橙蓝主题；**尚未移植** |
| 小程序凭证 | `docs/wxamp-info.md` | AppID + AppSecret（敏感） |
| 架构设计稿 | `plans/记忆里的美食-*.md` | **未实施**，草案非权威，勿据此改代码 |
| 调研 / 计划文档 | `plans/` | 草案，非权威；含真机预览二维码 `preview-qr.png` |
| 微信开发者工具 | `/Applications/wechatwebdevtools.app` | CLI：`Contents/MacOS/cli` |

## 常用命令

```bash
# TypeScript 类型检查（0 error 才可交付）
cd /Volumes/dufa-mac/www/cook/miniprogram && npm run typecheck

# 打开项目（工具需已登录、开启服务端口）
/Applications/wechatwebdevtools.app/Contents/MacOS/cli open --project /Volumes/dufa-mac/www/cook/miniprogram

# 生成真机预览二维码（服务端口须已开启，见 .agents/rules/verification-gate.md）
/Applications/wechatwebdevtools.app/Contents/MacOS/cli preview --project /Volumes/dufa-mac/www/cook/miniprogram --qr-format image --qr-output /Volumes/dufa-mac/www/cook/plans/preview-qr.png --lang zh
```

完整索引：[.agents/README.md](./.agents/README.md)
