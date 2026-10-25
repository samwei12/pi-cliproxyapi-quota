# 本地 Patch 备忘

## pi-cliproxyapi-provider · swe-2 contextWindow 覆盖

**位置**: `/Users/songguiming/.pi/agent/npm/node_modules/@router-for-me/pi-cliproxyapi-provider/extensions/lib.ts`（`toPiModel` 函数，约 542 行）

**为什么打这个补丁**：
Devin 后端通过 `/v1/models?client_version=pi` 自报 `devin/swe-2` 的 `context_window=262000`，但实测模型接受 ≥950k prompt tokens（Kimi K3 基座，1M 窗口）。自报值是保守建议上限不是硬截断，pi 的 footer 和 auto-compact 按 262k 算会提前触发压缩。

**补丁内容**：`toPiModel` 里 `id.startsWith("devin/swe-2")` 时强制 `contextWindow = 1048576`，其他模型走原逻辑。带注释指向 upstream issue。

**Upstream**：
- Issue: https://github.com/router-for-me/pi-cliproxyapi-provider/issues/49
- PR: https://github.com/router-for-me/pi-cliproxyapi-provider/pull/50（fork: `songhuiming2007-coder/pi-cliproxyapi-provider`，分支 `fix/swe-2-context-window`）

**会被覆盖的场景**：
- `pi update @router-for-me/pi-cliproxyapi-provider` —— npm 重装会抹掉本地改动
- `pi remove` + `pi install` —— 同上
- `~/.pi/agent/npm/node_modules/` 被手动清理

**覆盖后怎么办**：
- upstream 已合并 PR → 什么都不用做，正版带修复
- upstream 还没合并 → 去 `/tmp/pi-cliproxyapi-provider`（如果还在）把 `extensions/lib.ts` 的那段 diff 重新打；或者把 fork 里的 `fix/swe-2-context-window` 分支内容覆盖回 `node_modules` 对应文件

**验证方式**：
```bash
python3 -c "import json; d=json.load(open('/Users/songguiming/.pi/agent/cliproxyapi-models.json')); print([m['contextWindow'] for m in d['models'] if m['id']=='devin/swe-2'])"
# 应输出 [1048576]；如果是 [262000] 说明补丁被覆盖了
```

**日期**: 2026-10-25
