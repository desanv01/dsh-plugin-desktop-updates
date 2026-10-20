# Changelog

All notable changes to this project are documented here. The format follows
[Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and this project adheres to
[Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

- Decide the future of the page bridge this plugin renders. The bridge belongs to the archived C# desktop
  app; the official DeepSeek Harness desktop app ships its own plugin manager and updates view.

## [1.3.0]

The version published to npm from `desanv01/deepseek-harness-desktop-app`.

### Added

- Plugin update check: each installed plugin is compared against its published version.
- A tray menu line and a window title suffix that report app and harness update state.

### Changed

- The panel reports a slot the shell refused, instead of dropping it silently.

## [1.0.0]

First release: a sidebar entry beside Settings and a Settings section, driven by the app's page bridge.