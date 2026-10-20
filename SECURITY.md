# Security

## Reporting a vulnerability

Open a private security advisory through this repository's **Security** tab, or contact the maintainer through
the GitHub profile `desanv01`. Please do not open a public issue for anything that could be exploited.

Include the version, the desktop app version, reproduction steps, and what an attacker gains.

## What this plugin does with data

| Property | Value |
| --- | --- |
| Network access | **none** from the plugin |
| Reads credentials | **no** |
| Writes files | **no** |
| Reads | app and harness version state, and the plugin list, over the app's page bridge |
| Writes to the page | `window.__dshDesktopUpdates`, a registration marker |

## The trust boundary worth understanding

This plugin is a **UI over the app**. It does not download, verify, install or remove anything itself: each
action posts a message across `window.__dshDesktop`, and the desktop app performs the work with its own
guarantees — checksum verification of downloads, and the harness's own plugin manager for installs.

That means the security properties of an update or an install are the **app's**, not this plugin's. If a
release is bad, this plugin cannot make it better or worse; it can only ask.

## Installing plugins

Installing any plugin runs host code outside the workspace sandbox. This one does not, because its host half is
an empty `apply` — but the harness cannot know that in advance, and neither should you assume it of the next
plugin you install. Install from the npm package or from source you have read.

## Supported versions

The current major version. Security fixes are released as patch or minor versions, and the release workflow
refuses a tag that does not match `package.json`.