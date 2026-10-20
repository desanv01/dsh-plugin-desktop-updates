# dsh-plugin-desktop-updates

**A DeepSeek Harness plugin that renders the Updates UI of the Windows desktop app: a row beside Settings, an Updates panel, and an Updates section, driven by the app's native page bridge.**

![MIT](https://img.shields.io/badge/license-MIT-green)
![platform](https://img.shields.io/badge/platform-Web%20UI-1f6feb)
![dsh](https://img.shields.io/badge/dsh-0.1.x-1f6feb)
![npm](https://img.shields.io/npm/v/dsh-plugin-desktop-updates)
![status](https://img.shields.io/badge/status-maintained%20for%20an%20archived%20app-orange)

It shows the desktop app's version, the installed and published harness versions, and the plugins this home
runs — with the actions to check, download, install, restart, repair the harness, and enable, disable, install
or remove plugins.

> **Project status:** 1.3.0, published on npm. Verified against the DeepSeek Harness **0.1.5-rc.x** runtime
> that the Windows desktop app bundles. **The desktop app it serves is archived**, and this plugin depends on
> that app's own page bridge (`window.__dshDesktop`), which the official DeepSeek Harness desktop app does not
> provide. In the official app the plugin loads and renders, but reports that the bridge is unavailable —
> because it is. See §Why this exists.

## Screenshots

The UI appears in the desktop app that bundles it. Screenshots of the app window, including the Updates panel,
are in
[the app repository](https://github.com/desanv01/deepseek-harness-desktop-app#screenshots).

## Why this exists

The app needed a settings surface — the app's version, whether the harness was current, what plugins this home
runs, and buttons to act on all of it — without rebuilding a native window for every change. The app already
rendered the harness web UI in WebView2, so the surface became a plugin: sidebar row, panel and Settings
section, all in the page the user is already looking at.

That decision had one consequence worth stating. The facts and the actions live in the **app**, not the page,
so the page needs a channel back to it. The app injects a versioned bridge, `window.__dshDesktop`, into every
page it loads, and this plugin is the only consumer of it.

**That bridge is the reason this plugin is tied to one app.** The official DeepSeek Harness desktop app ships
its own plugin manager and its own updates view; it does not expose `__dshDesktop`. Installed there, this plugin
registers its slots and then reports that it cannot reach the app — which is accurate, and better than a panel
that looks live and does nothing.

## What it does

- **A row beside Settings** in the sidebar — a labelled entry (an icon in the 56 px rail) that opens the Updates
  panel, with a dot when a newer app build exists.
- **A panel**, full view, in the app's own window: the facts plus check, download, install, restart and
  repair, and the plugin list with enable, disable, install and remove.
- **A Settings section** with the same facts and actions in a narrower form.
- **Reports what the shell refused.** The shell drops a slot registration it dislikes without telling anyone, so
  a plugin whose UI never appears looks exactly like one that never loaded. This plugin publishes what it
  registered — and every refusal — to `window.__dshDesktopUpdates` and to the app's log.
- **Degrades honestly.** With no bridge, it says the app is unavailable instead of pretending the panel works.

## How it works

### The four slots it contributes

| Slot | Entry | What it renders |
| --- | --- | --- |
| `sidebar.footer.action` | id `desktop-updates` | A labelled row beside Settings (an icon in the 56 px rail) that opens the Updates panel; a dot when a newer app build exists |
| `sidebar.panellist` | id `desktop-updates` | The sidebar row for the Updates panel, addressed by the same id as the body below |
| `main` | key `desktop-updates` | The Updates panel itself: the full view, in the app's own window |
| `settings.section` | id `desktop-updates` | The concise view inside Settings, with the same facts and actions |

The three list slots require an `id` and the keyed `main` slot requires a `key`; the shell rejects a
registration that omits either.

### Opening the panel is a layout action, not a bridge call

The plugin selects its own panel through `ctx.layout.selectPanel`, so the view appears in the window the user
is already looking at, and opening it cannot fail because the bridge is unavailable. When the shell has no
panel slot to select, it falls back to the app's native updates window — the same window the tray menu opens.

### The registration marker

`client.js` writes `window.__dshDesktopUpdates` as it applies: the slots it registered and every contribution
the shell refused. The app reads that marker after each page load and logs it. The Updates section repeats any
refusal, because that is the only place a user would see it.

### Version skew

The app leaves an installed version alone when it is newer than the copy inside its executable, so a newer npm
release is not undone by the next launch. `DeepSeekHarness.exe --install-plugin` restores the bundled copy
deliberately.

## Requirements

- A DeepSeek Harness **0.1.x** composition with the `layout` service — the runtime the Windows desktop app
  bundles.
- `@deepseek-ai/dsh-client-ui-layout`, declared in `dsh.client.inject`, which the host must provide.
- For anything to be actionable: the desktop app's `window.__dshDesktop` bridge.

## Quick start

### Install from npm

```powershell
dsh plugin --profile web add dsh-plugin-desktop-updates
```

### Install into the desktop app

The app installs its bundled copy by itself, into the harness home it owns. From inside the app, use
**Settings → Updates → the plugin manager**, which accepts a package name, a git spec or a local folder.
Installing from npm replaces the bundled copy with the published one.

### Build from source

Nothing to build — the client half is a hand-written module-face bundle:

```powershell
git clone https://github.com/desanv01/dsh-plugin-desktop-updates.git
cd dsh-plugin-desktop-updates
dsh plugin --profile web add .
```

### Smoke test

The client half can be rendered headlessly against a fake shell; that is how the desktop app's own CI checks
it, with `tools/client-plugin-smoke.mjs` in
[the app repository](https://github.com/desanv01/deepseek-harness-desktop-app). It loads the real `client.js`,
enforces the real slot rules (a list entry needs an `id`, the panel slot needs a `key`) and fails if a
registration is dropped.

## Configuration

None. The plugin reads no `Config` values; it renders what the bridge reports.

## Slots used

See the four-slot table in §How it works. `main` is keyed; the other three are addressed by `id`.

## Where things live

| Piece | Path |
| --- | --- |
| Host half | `index.js` — an empty `apply`, so the row exists in the profile's bundle stack and the loader keeps it |
| Browser half | `client.js` — a module-face bundle (`window.__ModuleLoader__.load({ id, factory })`) |
| Loader row | `cordis.patch.yml` — inserts the row `desktop-updates` |
| Registration marker | `window.__dshDesktopUpdates`, plus the app's log |
| Bridge it consumes | `window.__dshDesktop`, injected by the app |

## Troubleshooting

| Symptom | Cause | Fix |
| --- | --- | --- |
| No sidebar row at all | The bundle row is missing from the profile, or the client half was not served | Check `cordis.patch.yml` is installed, then read `window.__dshDesktopUpdates` in the page |
| Row appears, panel says the app is unavailable | No `window.__dshDesktop` — this is the expected result outside the Windows desktop app | Use the app that provides the bridge |
| Row appears, panel is empty | The bridge exists but reported no state | Check the app's log for the marker it logs after each page load |
| Settings section missing but the panel works | The shell refused the `settings.section` registration | The section repeats refusals; the marker lists them |
| A newer npm version did not take effect | The app keeps an installed version newer than its bundled copy | See §Version skew |

## Current status

Working, for the app it was built for.

Verified: the four slot registrations against a fake shell that enforces the real rules; the layout-action path
and the native-window fallback; the marker and its logging; and honest degradation when the bridge is absent.
Published on npm from 1.0.0 through 1.3.0.

**Not verified: the official DeepSeek Harness desktop app.** That app provides no `__dshDesktop` bridge, so
this plugin's value there is zero even if every slot registers. The plugin is maintained as part of the
archived Windows desktop app, not as a plugin for the official one.

## Roadmap

- [x] Sidebar row, panel and Settings section
- [x] Check, download, install, restart and repair the app and the harness
- [x] Plugin list with enable, disable, install and remove
- [x] Registration marker, so a refused slot is visible instead of silent
- [x] Honest degradation without the bridge
- [ ] Decide whether to re-target the official app's own plugin settings surface, or retire

## Security

This plugin is a **UI that asks the app to act**. It performs no update, install or removal itself: every
button posts a message over the app's bridge, and the app does the work with its own verification — checksum
verification of downloads, and the harness's own plugin manager for installs.

It makes no network calls, reads no credentials, and writes no files. It does read app and harness version
state through the bridge, which is not sensitive.

Installing *any* plugin runs host code outside the workspace sandbox; that is a property of the harness, not of
this plugin. Install from the npm package or from source you have read.

Report a vulnerability through the process in [SECURITY.md](SECURITY.md).

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md). The review bar: does a refused registration stay visible, and does the
plugin behave honestly when the bridge is absent?

## License and attribution

MIT — see [LICENSE](LICENSE).

Extracted from `desanv01/deepseek-harness-desktop-app`, where it shipped inside the executable and on npm.
