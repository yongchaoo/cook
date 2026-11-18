# AI Coding Rules（Enforced）

> 硬性纪律，不是建议。违反任一条 = **回滚到违规前，不许原地修**。
> 冲突时优先级：**本文件 > 其余 SSOT > 任务描述 > agent 判断**。
> 适用：所有 AI 在本项目内的代码与文档改动。参考 `~/www/lexi/docs/ai/ai-coding-rules.md`，按项目量级裁剪。

## How to work（行为）

1. **Ask, don't guess.** 不确定就问，不许猜；改前先陈述假设，困惑时停下并说清「不清楚的是什么」。
2. **Define verifiable success, then loop.** 先定义可验证的成功标准（本阶段 = 编译无错 + 真机预览看到页面），循环到通过或卡住就停下问人。
3. **Pick one side, don't blend.** 两套冲突方案选一个（更新的 / 更被验证的），说明理由，标记另一个待清理，不许融合成中间态。
4. **Fail loud.** 静默跳过 = 错，假「完成」= 错。测试 pass 但有跳过 = 错。默认暴露不确定性。
5. **Don't continue from a state you can't describe back.** 交付前 MUST 能描述：当前状态、已验证什么、还剩什么。
6. **一次做完。** 用户偏好一次性完成、全权委托；不把可一次交付的任务拆成多轮挤牙膏式确认。

## What to write（质量）

7. **Surgical changes only.** 只动必须动的；不顺手改相邻代码、注释、格式；匹配现有风格。
8. **Read before you write.** 加代码前先读现有文件、调用方与共享约定；「看起来正交」是危险的。
9. **Nothing speculative.** 不加未要求的功能，不造只用一次的抽象，用解决问题的最少代码。
10. **Conform, surface disagreement.** 仓库一致性 > 个人品味；真心觉得有害就提出，不许静默 fork 自己的风格。
11. **Search before you cite.** 声称某文件 / API / 符号存在前 MUST 先搜（Grep / Glob），不许凭印象引用。

## Verify, don't assume（实战条款）

12. **静态通过 ≠ working.** JSON 合法、文件齐全仍可能在开发者工具里编译失败或真机白屏；交付前 MUST 按 [verification-gate.md](./verification-gate.md) 做运行时验证。
13. **禁止 silent done.** 「文件已创建」「命令已执行」不等于「服务已启动 / 页面可见」；必须给出实际证据（进程、编译结果、预览二维码、真机截图）。
14. **数据 / 结论 MUST 可复现**：涉及数字或性能结论时给出口径与来源，不用估算冒充实测。

## 用户交互硬约束（本项目特有）

15. **「先不改代码」= 暂停键。** 用户说「先不改代码」「不要直接改」时，只输出方案 / 预览，不改任何文件，等明确确认。
16. **样式 / 配色改动先出可视化预览。** 用对比方案让用户选定后才落盘；未选定前绝不覆盖原型文件。
17. **结论先行、中文、结构化。** 答案先给结论与证据，偏好表格；不要长篇铺陈过程。
18. **调研 / 分析类产出写入 `plans/`**（用户习惯），但 `plans/` 不是权威文档，被采纳后须迁移到 SSOT。
19. **不着急实现原型。** 用户明确说「先跑通」「不着急实现原型」时，只交付最小可运行骨架，不提前搬功能。

## 小程序专有约束（见 [ARCHITECTURE.md](../ARCHITECTURE.md)）

20. **凭证红线**：AppSecret **MUST NOT** 出现在 `miniprogram/**` 或任何会入库的文件；AppID 只写在 `project.config.json`。
21. **MUST NOT** 引入 Web API（document / window / localStorage / Web Audio / Notification / vibrate）、前端框架或 npm 构建链。TS 的 devDependencies（`typescript` / `miniprogram-api-typings`）**仅供类型检查**，不产出运行时代码、不参与小程序编译，不属构建链。
22. **MUST** 页面四件套齐全（`wxml` / `wxss` / **`ts`** / `json`）+ `app.json` 注册；**MUST** 用 `setData()` 数据驱动，不手动操作 DOM。
23. **MUST** 切后台计时用 `endAt` 时间戳 + `onShow` 重算，不靠 `setInterval` 累计。
24. **MUST NOT** 一次性把 `cooking-timer-prototype.html` 全量搬进小程序；按功能分批移植，每批过验证门禁。
25. **TypeScript 强制**：业务 / 页面逻辑 **MUST** 写 `.ts`，**MUST NOT** 新建 `.js` 或把编译产物落回源码目录（工具内置插件按需编译，项目内只留 `.ts`）。
26. **类型检查是独立门禁**：交付前 **MUST** `npm run typecheck`（`tsc --noEmit`）0 error。工具内置 TS 编译插件由 Babel 实现，**只剥离类型、不做类型检查**，类型错误照样能编译 / 预览成功——**MUST NOT** 把 `open` / `preview` 成功当成类型安全。

## 变更规模

| 规模 | 例子 | 要求 |
|------|------|------|
| **Small** | 文案、样式微调、注释 | 静态校验 + typecheck + 编译验证 |
| **Medium** | 新增页面、单个交互逻辑 | typecheck + 编译 + 真机预览 + 更新 AGENTS.md「当前状态」与 ARCHITECTURE.md |
| **Large** | 原型整体移植、多页面架构 | 先出方案到 `plans/` 让用户确认 → 分批实现 → 每批过门禁 |
| **Bug fix** | 修错误行为 | 先复现并说明现象 → 修 → 真机复验 |

流程统一收口到 [verification-gate.md](./verification-gate.md)。
