# pi-cliproxyapi-quota — 个人增强分支

[English](README.md) | 简体中文

本分支基于 [songhuiming2007-coder/pi-cliproxyapi-quota](https://github.com/songhuiming2007-coder/pi-cliproxyapi-quota)，包含 Antigravity 当前模型额度池修复，以及可选的紧凑显示、macOS 钥匙串认证、provider 筛选和启动/定时刷新。这些个人增强与上游正确性 PR 分开维护。

```text
5h 67.2% (2h51m) · w 89.1% (6d21h)
```

百分比表示剩余额度，括号表示距重置的时间。Gemini 使用 Gemini 池；通过 Antigravity 使用的 Claude 和 GPT-OSS 使用 Claude/GPT 池。

## 在本机或公司安装

需要 Node.js 22.19+ 和 Pi。必须安装这个 Git 分支，而不是上游 npm 包，才能获得这些改动：

```sh
pi install git:github.com/samwei12/pi-cliproxyapi-quota@feature/personal-quota-footer
```

移除旧的本地 `pi-cliproxy-quota` 包配置以避免重复显示，再执行 `/reload`。用 `pi list` 确认来源为这个 Git 分支。更新命令：

```sh
pi update git:github.com/samwei12/pi-cliproxyapi-quota@feature/personal-quota-footer
```

本分支没有发布 npm，不会修改推理 provider 或默认模型。

## 每台机器分别配置

创建 `~/.pi/agent/cliproxyapi-quota.json`。设置了 `PI_CODING_AGENT_DIR` 时使用该目录，也兼容旧的 `PI_AGENT_DIR`：

```json
{
  "footerStyle": "compact",
  "footerProviders": ["cliproxy-antigravity"],
  "keychain": {
    "service": "CLIPROXY_MANAGEMENT_KEY"
  }
}
```

- `footerStyle`：`compact` 使用上面的简洁样式；省略或设置为 `default` 保留上游样式。
- `footerProviders`：需要显示额度的 Pi provider 精确名称，应按该机器实际配置填写。省略则允许上游支持的通道；`[]` 禁止自动查询。`/quota` 仍可手动查询全部供应商。
- `keychain`：可选的 macOS 登录钥匙串条目，账号名默认取当前系统用户；条目使用其他账号名时设置 `keychain.account`。条目必须已经存在，密钥不会写入仓库或自动传到另一台机器。明确配置钥匙串后，读取失败不会回退到明文密钥。
- `accountName`：可选的认证文件精确名称。存在多个 Antigravity 凭证时，必须明确选择才能显示底栏；不是全部账号汇总，也不保证代理负载均衡会选择该账号。

代理地址依次读取 `CLIPROXYAPI_BASE_URL`、`~/.pi/agent/cliproxyapi.json` 的 `baseUrl`，默认 `http://127.0.0.1:8317`。管理密钥优先级为：环境变量 `CLIPROXYAPI_MANAGEMENT_KEY` → 显式配置的钥匙串 → 旧配置的 `managementKey` 字段 → EasyCLIProxyAPI GUI 配置。旧方式仅作兼容，不能把凭证提交到仓库。非 macOS 使用环境变量并省略 `keychain`。管理请求不跟随 HTTP 重定向。

公司端还需要可访问的 CLIProxy、管理认证和对应推理 provider。安装插件不会部署代理或复制 OAuth 登录。`localhost` 永远指运行 Pi 的那台机器，不会自动指向家里的电脑。

## 使用

- 在启动、切换模型、任务结束和每 60 秒定时刷新；限制频率并合并重复请求。
- 自动刷新只查询当前供应商，不额外读取无关账号额度。
- 切换到不匹配的 provider 后清除底栏，旧响应不会覆盖新模型。
- `/cliproxy-quota`：刷新当前模型底栏。
- `/quota` 或 `Ctrl+Shift+Q`：上游完整额度视图，展示各启用凭证，包含账号标签。
- 缺失字段显示 `n/a`，查询失败显示 `Quota unavailable`。Antigravity 对应池缺失或账号不明确时，不回退显示其他供应商的额度。
- 非交互会话不自动查询额度或读取钥匙串。

沿用上游适配器：Claude、Antigravity/Gemini、Kimi，以及尽力支持的 Codex/xAI；后两项保留上游“未验证”标记。个人分支的真实验证覆盖 Antigravity，并不代表全部供应商均已验证。扩展无法推断任意模型别名或负载均衡的实际账号选择。

## 验证与隐私

```sh
npm test
```

离线测试只使用合成凭证和模拟 HTTP，覆盖路由、额度池、多账号、显示格式、钥匙串错误脱敏、provider 筛选和切换竞争。真实查询可复用本机钥匙串，无需打印密钥。额度经管理 API 的 `api-call` 查询，不调用模型生成。

仓库不包含本机专用路径、私人账号标识、token 或实际额度快照。原作者及 GitHub fork 所有者仍保留公开署名。共享池百分比不等于单模型独享额度、token 数或上游可用性保证。

## 回退

使用同一来源字符串执行 `pi remove`，恢复旧本地包配置，再 `/reload`。这不会删除旧源码目录、本机配置或钥匙串条目。

## 许可证

MIT，保留上游版权和署名，见 [LICENSE](LICENSE)。
