# Fork notes — DSH 0.2.1-rc.2 compatibility

This is a personal fork of [THEWOLFWALKER/dsh-notifier](https://github.com/THEWOLFWALKER/dsh-notifier)
(MIT), owned by `xiaobizahizhi`, used to run the plugin on **DeepSeek Harness 0.2.1-rc.2**.

## Why this fork exists

Upstream `dsh-notifier@0.13.1` declares `@deepseek-ai/dsh-session` peer support only for the
`0.1.7-*` family. The DSH plugin manager therefore refuses to install or activate it on
`0.2.1-rc.2` unless a per-version risk exemption is granted. This fork carries the minimal
metadata patch so the plugin can be installed and smoke-tested from source control.

## Base and branch

- Upstream base: `main` @ `9d69dd7` (v0.13.1). The upstream `dev` branch is a **v1 rewrite**
  (different tree, `1.0.0-dev.0`) and is intentionally not used as this patch's base.
- Work branch: `dsh-021-compat`.

## Changes (metadata only — no runtime source changes)

1. `package.json` — `peerDependencies["@deepseek-ai/dsh-session"]` also accepts `0.2.1-rc.2`;
   `dshWorkshop.compatibility.dshVersions` mirrors it.
2. `package.json` — `@tencent-connect/qqbot-connector@^1.2.0` moved into `optionalDependencies`
   (it is documented as the QQ QR-login optional dependency but was missing from the manifest,
   so QR login silently degraded to manual AppID/AppSecret).
3. `docs/compatibility-matrix.md` — fork-local `0.2.1-rc.2` row + a clear "fork claim, host
   smoke pending" note.
4. `CHANGELOG.md` — Unreleased entry.

## Verification status (honest)

- Static seam audit: `inject = [tools, agents, connection]` exist; host declares
  `agent/error`, `agent/status`, `approval/asked`, `approval/decided`, `approval/request`,
  `session/event`, `user-questions/request`; required client packages exist.
- Headless load smoke: entry imports under Node 24; the DSH version gate passes after this
  patch; the row stays `pending (waiting for service: connection)` because headless has no
  web connection service — expected, not a failure.
- Real web-host activation on 2026-10-25: installed into the live `web` profile and the host
  hot-loaded it without a process restart. `dsh --profile web --dump-config` composes the
  `dsh-notifier` row; the plugin's admin server answers `200` on `http://127.0.0.1:8104/`
  (title `dsh-notifier 管理台`) and `401` for unauthenticated `/api/*`. Native “通知与控制”
  panel rendering and real QQ provider flows are still pending.

## 方案 0：隐形模式（declared 2026-10-26）

本 fork 的 `dsh-021-compat` 分支在 2026-10-26 追加「隐形模式」：

- `client.js` 不再注册 `main`、`sidebar.panellist`、`plugins.bundle.activation`，只保留
  `plugins.bundle.config`（Plugins 页里一张状态卡）。插件在后台跑通知/审批/提问/QQ 回传，
  **不会接管主界面，也就不存在「进得去退不出来」的返回键问题**。
- `cordis.patch.yml` 默认 `admin.enabled: false`，不再监听 `127.0.0.1:8104`。
- 配置方式改为「配置即代码」：在 profile 的 `cordis.patch.yml` 覆盖该行
  `config.channels`（QQ 渠道 appId/appSecret/targetType/userId），字段表见
  `src/config.mjs` 的 FIELD_HINTS；QQ 官方扫码用
  `node node_modules/dsh-notifier/scripts/channel-login.mjs qq`。

## Install

```powershell
dsh plugin --profile web add "github:xiaobizahizhi/dsh-notifier#dsh-021-compat"
# 0.2.1-rc.2 hot-loaded the row on install; if a host does not hot-reload, restart dsh web.
```

## Upstream relationship

MIT license and attribution are retained. Prefer upstream once it declares 0.2.x support;
this branch is a temporary compatibility shim, not a maintained fork.
