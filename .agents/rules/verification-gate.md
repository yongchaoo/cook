# 变更验证门禁

> 交付前 **MUST** 按序通过。只写完文件不算完成（见 [ai-coding-rules.md](./ai-coding-rules.md) §13 禁止 silent done）。

```text
静态校验（JSON 合法 / 四件套 wxml·wxss·ts·json 齐全 / 无残留 .js / 凭证无泄漏）
  → TypeScript 类型检查（npm run typecheck 0 error）
    → 开发者工具编译（CLI open 后无编译错误，TS 插件生效）
      → 真机预览（生成二维码 → 手机微信扫码看到页面）
        → 更新 AGENTS.md「当前状态」
```

## 1. 静态校验（机器可跑）

```bash
cd /Volumes/dufa-mac/www/cook
# 所有 JSON 合法
for f in miniprogram/*.json miniprogram/pages/*/*.json; do
  /Users/dufa/.workbuddy/binaries/python/versions/3.13.12/bin/python3 -m json.tool "$f" >/dev/null || echo "INVALID: $f"
done
# app.json 里每个 page 的四件套（ts）是否齐全 + 源码内不得残留 .js
/Users/dufa/.workbuddy/binaries/python/versions/3.13.12/bin/python3 - <<'PY'
import json, pathlib
root = pathlib.Path('miniprogram')
cfg = json.loads((root/'app.json').read_text())
ok = True
for p in cfg['pages']:
    for ext in ('wxml','wxss','ts','json'):
        f = root / f'{p}.{ext}'
        if not f.exists():
            ok = False; print(f'MISSING: {f}')
leftover = [str(f) for f in root.rglob('*.js') if 'node_modules' not in f.parts]
if leftover:
    ok = False; print('LEFTOVER .js:', leftover)
print('PAGES OK' if ok else 'PAGES FAILED', len(cfg['pages']), 'pages')
PY
```

## 2. TypeScript 类型检查（MUST）

```bash
cd /Volumes/dufa-mac/www/cook/miniprogram && npm run typecheck   # tsc --noEmit，必须 0 error
```

判据：`tsc --noEmit` 退出码 0，且 `--listFiles` 确实覆盖 `app.ts` / 各页面 `.ts`（防空跑）。工具内置 TS 编译插件**不做类型检查**，`open` / `preview` 成功不能替代本步。

门禁有效性 MUST 能反向验证：临时注入一个类型错误（如 `const x: number = "bad"`），确认 `tsc` 报 `TS2322` 且退出码非 0，删除探针后复检为 0。

## 3. 编译验证

```bash
CLI=/Applications/wechatwebdevtools.app/Contents/MacOS/cli
$CLI open --project /Volumes/dufa-mac/www/cook/miniprogram
```

前提：开发者工具已启动、**已微信扫码登录**、设置 → 安全设置 → **服务端口已开启**。

本机已于 2026-11-18 开启：用户目录 `…/微信开发者工具/<hash>/Default/.ide-status` = `On`，端口写在同目录 `.ide`，CLI 自动读取，**无需传 `--port`**。若状态文件缺失导致 `IDE service port disabled`：先退出 IDE，把 `WeappLocalData/localstorage_*.json` 里的 `security.enableServicePort` 设为 `true` 后重启 IDE，由 IDE 自己写 `.ide-status`；CLI 提示的 `enter y` 在非交互终端不生效，须走这条路径。

判据：工具打开项目且模拟器渲染出页面，编译输出 0 error。`open` 返回成功 ≠ 编译成功，MUST 看工具实际状态。

**编译缓存注意**：`preview` 可能复用上一次编译产物（实测改 `.ts` 后包大小未变）。改动源码后 MUST 先 `close --project` + `open --project` 强制重载（或重启 IDE）再 `preview`；判据是上传包大小 / 内容确实变化，而不是只看 `✔ preview`。

**TS 插件生效证据**（2026-11-18 实测）：在 `.ts` 里写 `const probe: string = "..."` 能 preview 成功，且工具编译缓存里该语句被转译为 `var probe = "..."`（类型注解被剥离）——证明插件生效；若按纯 JS 解析会直接语法报错。验证后 MUST 删除探针并强制重载重新出二维码。

## 4. 真机预览（本阶段最终判据）

```bash
$CLI preview --project /Volumes/dufa-mac/www/cook/miniprogram \
  --qr-format image \
  --qr-output /Volumes/dufa-mac/www/cook/plans/preview-qr.png \
  --lang zh
```

判据：
- 生成二维码 PNG，且能被微信扫码打开小程序；
- **手机屏幕上确实出现页面内容**（本阶段 = hello 文案），不是白屏 / 报错页；
- 二维码有有效期，过期须重新生成。

注意 `--qr-output` **必须指向项目内已存在的目录**：写 `/tmp/...` 会报 `code 17 二维码输出路径无效或不存在`。

若 CLI 预览不可用（服务端口未开 / 未登录 / AppID 无权限）：**MUST 明确报出阻塞原因**，并给出手动路径：开发者工具顶部「预览」→ 扫码。不许把「命令执行了」写成「手机能看到」。

## 5. 证据要求

| 宣称 | 最低证据 |
|------|----------|
| 项目已跑通 | typecheck 0 error + 编译 0 error + 真机预览页面可见 |
| 类型安全生效 | `npm run typecheck` 0 error + 反向注入类型错误能报错 |
| TS 编译插件生效 | 探针类型注解被剥离且 preview 成功 |
| 服务已启动 | 进程存在（PID）+ 前台窗口 |
| 新增页面生效 | `app.json` 注册 + 模拟器 / 真机可见 |
| 样式改动生效 | 真机截图或预览二维码 |

**禁止**：以 `apply 成功` / `文件已写入` / `typecheck 通过` / `preview ✔` 冒充运行时验收。

## 6. 交付收尾

- Medium 及以上改动 **MUST** 同步更新 [../../AGENTS.md](../../AGENTS.md)「当前状态」与 [../ARCHITECTURE.md](../ARCHITECTURE.md) 目录地图。
- 未通过的门禁项 **MUST** 在回复里显式列为「未验证 / 阻塞」，不许略过。
