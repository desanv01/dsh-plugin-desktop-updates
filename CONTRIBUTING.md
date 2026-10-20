# Contributing

## What this plugin is

A UI plugin with two halves: `index.js`, an empty host `apply` that keeps the bundle row alive, and `client.js`,
a hand-written module-face bundle that registers four slots and renders them from the desktop app's page
bridge.

## Ground rules

1. **A refused registration must stay visible.** The shell drops a slot it dislikes without reporting it, so a
   missing UI is indistinguishable from a plugin that never loaded. Anything you register must also be recorded
   in `window.__dshDesktopUpdates`, refusals included.
2. **Degrade honestly.** When `window.__dshDesktop` is absent, say so. A panel that looks live and does nothing
   is worse than a panel that says the app is unavailable.
3. **Opening the panel is a layout action.** Use `ctx.layout.selectPanel`; fall back to the app's native window
   only when there is no panel slot. Do not make opening depend on the bridge.
4. **List slots need an `id`, the keyed `main` slot needs a `key`.** The shell rejects a registration that
   omits either.
5. **The plugin never performs an update itself.** Buttons post over the bridge; the app does the work.

## Local checks

There is no build step for the plugin. Before opening a pull request:

```powershell
node --input-type=module -e "import('./index.js').then(m => console.log(m.name, typeof m.apply))"
npm pack --dry-run     # confirm cordis.patch.yml and client.js are in the payload
```

The real conformance check is `tools/client-plugin-smoke.mjs` in
[the app repository](https://github.com/desanv01/deepseek-harness-desktop-app): it renders `client.js` under a
minimal React and a fake shell that enforces the slot rules. Run it there if you change `client.js`.

## Pull requests

- One logical change per pull request.
- Commit subjects follow the conventional form: `fix(scope):`, `feat(scope):`, `docs(scope):`.
- Bump the version in `package.json` in the same pull request as a user-visible change; the release workflow
  refuses a tag that does not match the manifest.

## Reporting a bug

Include: the app version, the harness version, whether the sidebar row appears at all, and the contents of
`window.__dshDesktopUpdates` from the page console. That marker lists what registered and what was refused, which
is usually the whole answer.

## License

Contributions are accepted under the MIT license in [LICENSE](LICENSE).