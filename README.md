# pi-cliproxyapi-quota — personal feature branch

English | [简体中文](README.zh-CN.md)

This branch builds on [songhuiming2007-coder/pi-cliproxyapi-quota](https://github.com/songhuiming2007-coder/pi-cliproxyapi-quota). It includes the Antigravity active-model pool fix plus optional compact formatting, macOS Keychain authentication, provider filtering, and startup/periodic refresh. These personal enhancements are separate from the upstream correctness PR.

```text
5h 67.2% (2h51m) · w 89.1% (6d21h)
```

Percentages mean remaining capacity; parentheses show time until reset. Gemini uses the Gemini pool; Claude and GPT-OSS routed through Antigravity use the Claude/GPT pool.

## Install on any machine

Requires Node.js 22.19+ and Pi. Install the Git branch, **not the upstream npm package**, to get these changes:

```sh
pi install git:github.com/samwei12/pi-cliproxyapi-quota@feature/personal-quota-footer
```

Remove any previous local `pi-cliproxy-quota` package entry to avoid duplicate footers, then `/reload`. Use `pi list` to confirm the Git source. To update this source:

```sh
pi update git:github.com/samwei12/pi-cliproxyapi-quota@feature/personal-quota-footer
```

This branch is not published to npm. It does not change inference providers or default models.

## Configure independently on each machine

Create `~/.pi/agent/cliproxyapi-quota.json` (inside `PI_CODING_AGENT_DIR` if set; legacy `PI_AGENT_DIR` is also supported):

```json
{
  "footerStyle": "compact",
  "footerProviders": ["cliproxy-antigravity"],
  "keychain": {
    "service": "CLIPROXY_MANAGEMENT_KEY"
  }
}
```

- `footerStyle`: `compact` uses the example above; omitted/`default` retains the upstream formatter.
- `footerProviders`: exact Pi provider names whose footer should be shown. Use the provider configured on that machine. Omit to allow the upstream-supported providers. `[]` disables automatic footer queries. `/quota` remains an explicit all-provider query.
- `keychain`: optional macOS login Keychain item. Account defaults to the current OS username; use `keychain.account` for a different item account. The item must already exist. No key is written to this repository or copied between machines. Failure of an explicitly configured Keychain source does not fall back to a plaintext key.
- `accountName`: optional exact authentication-file name. Multiple Antigravity credentials require explicit selection for a footer; this is not an aggregate or a guarantee about proxy load-balancing.

The proxy URL resolves from `CLIPROXYAPI_BASE_URL`, then `~/.pi/agent/cliproxyapi.json` `baseUrl`, then `http://127.0.0.1:8317`. Management key resolution is: `CLIPROXYAPI_MANAGEMENT_KEY` environment variable → explicitly configured Keychain → legacy `managementKey` JSON field → EasyCLIProxyAPI GUI config. Legacy paths are preserved for compatibility; do not commit credentials. On non-macOS systems use the environment variable and omit `keychain`. Management HTTP redirects are rejected.

A work machine must have its own reachable CLIProxy instance (or an explicitly configured accessible endpoint), management authentication, and the matching inference provider. Installing this package alone does not set up the proxy or transfer OAuth credentials. `localhost` always means the machine running Pi, not another computer.

## Usage

- Startup, model changes, settled turns and a 60-second timer refresh the footer (throttled/coalesced).
- Automatic queries access only the current provider, preserving unrelated account quotas and avoiding unnecessary requests.
- Switching away clears the footer. Old responses cannot overwrite a newer selection.
- `/cliproxy-quota`: refresh the current-model footer.
- `/quota` or `Ctrl+Shift+Q`: upstream full quota view across enabled credentials, including account labels.
- Missing data displays `n/a`; a failed query shows `Quota unavailable`. A missing/ambiguous Antigravity pool never falls back to another provider.
- Noninteractive sessions do not automatically query quota or read Keychain secrets.

The underlying upstream adapters support Claude, Antigravity/Gemini, Kimi, and best-effort Codex/xAI (the latter remain unverified as upstream declares). The personal live validation covers Antigravity, not every provider. Arbitrary model aliases and load-balancer account selection cannot be inferred.

## Validation and privacy

```sh
npm test
```

Offline tests use synthetic credentials and mocked HTTP. They cover routing, pool selection, multiple accounts, formatting, Keychain error redaction, provider filtering and model-switch races. Local live validation can reuse a Keychain item without printing it. Quota requests use CLIProxy management `api-call`; they do not invoke model inference.

No machine-specific paths, private account identifiers, tokens or quota snapshots are included in this fork. Authors and the GitHub fork owner remain public attribution. Shared remaining percentages are not individual model allowances, token counts or guarantees of upstream capacity.

## Rollback

Remove the Git source with `pi remove` using the same source string, then restore the previous local package entry and `/reload`. This does not delete the old checkout, local settings or Keychain item.

## License

MIT; upstream copyright and attribution are preserved in [LICENSE](LICENSE).
