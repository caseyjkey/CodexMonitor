# CodexMonitor

[![gitcgr](https://gitcgr.com/badge/Dimillian/CodexMonitor.svg)](https://gitcgr.com/Dimillian/CodexMonitor)

![CodexMonitor](screenshot.png)

CodexMonitor is a desktop app for running and monitoring Codex across one or more workspaces. It gives you a project sidebar, thread history, approvals, Git context, prompts, and workspace management in one place.

## What This README Covers

This README is organized for two different audiences:

- **Use CodexMonitor**: install it, connect it to Codex, and start using it locally or in server/client mode.
- **Develop CodexMonitor**: build from source, run validation, and navigate the codebase.

If you just want to use the app, read through **Choose Your Setup** and stop after **Troubleshooting**.

## Features

### Workspaces & Threads

- Add and persist workspaces, group/sort them, and jump into recent agent activity from the home dashboard.
- Spawn one `codex app-server` per workspace, resume threads, and track unread/running state.
- Worktree and clone agents for isolated work; worktrees live under the app data directory (legacy `.codex-worktrees` supported).
- Thread management: pin/rename/archive/copy, per-thread drafts, and stop/interrupt in-flight turns.
- Optional remote backend (daemon) mode for running Codex on another machine.
- Remote setup helpers for self-hosted connectivity, including Tailscale-assisted TCP setup.

### Composer & Agent Controls

- Compose with image attachments, drag/drop, paste, and configurable follow-up behavior (`Queue` vs `Steer` while a run is active).
- Use `Shift+Cmd+Enter` (macOS) or `Shift+Ctrl+Enter` (Windows/Linux) to send the opposite follow-up action for a single message.
- Autocomplete for skills (`$`), prompts (`/prompts:`), reviews (`/review`), and file paths (`@`).
- Model picker, collaboration modes (when enabled), reasoning effort, access mode, and context usage ring.
- Dictation with hold-to-talk shortcuts and live waveform (Whisper).
- Render reasoning/tool/diff items and handle approval prompts.

### Git & GitHub

- Diff stats, staged/unstaged file diffs, revert/stage controls, and commit log.
- Branch list with checkout/create plus upstream ahead/behind counts.
- GitHub Issues and Pull Requests via `gh` (lists, diffs, comments) and open commits/PRs in the browser.
- PR composer: "Ask PR" to send PR context into a new agent thread.

### Files & Prompts

- File tree with search, file-type icons, and Reveal in Finder/Explorer.
- Prompt library for global/workspace prompts: create/edit/delete/move and run in current or new threads.

### UI & Experience

- Resizable sidebar/right/plan/terminal/debug panels with persisted sizes.
- Responsive layouts for desktop, tablet, and phone.
- Sidebar usage and credits meter for account rate limits plus a home usage snapshot.
- Terminal dock with multiple tabs for background commands (experimental).
- In-app updates with toast-driven download/install, debug panel copy/clear, sound notifications, plus platform-specific window effects.

## Choose Your Setup

There are two main ways to use CodexMonitor:

### 1. Local

Use this when CodexMonitor and your `codex` CLI run on the same machine.

Typical example:
- You install CodexMonitor on your laptop or desktop.
- Your repos are on that same machine.
- `codex`, `git`, and any related tooling are installed on that same machine.

### 2. Server / Client

Use this when one machine runs the Codex backend and one or more other machines connect to it remotely.

Typical example:
- A server machine runs CodexMonitor's daemon and has access to your repos and `codex` CLI.
- A client machine runs the CodexMonitor app in `Remote (daemon)` mode.
- Both machines are connected over Tailscale or another network path you manage.

This is the right model if you want to keep the actual Codex work on a dedicated machine and use another laptop as a remote control surface.

## User Prerequisites

These are the things you need for normal usage. You do **not** need to read the development sections below unless you are building the app yourself.

### Required

- Codex CLI installed and available as `codex` in `PATH`, unless you set a custom Codex binary in settings.
- Git installed and available in `PATH`.
- A machine with access to the repos you want CodexMonitor to manage.

### Optional

- GitHub CLI (`gh`) for GitHub Issues and Pull Request features.
- Tailscale if you want to use server/client mode over your tailnet.

### If You Are Building from Source

You also need:

- Node.js + npm
- Rust toolchain (stable)
- CMake
- `pkg-config` on Linux
- LLVM/Clang on Windows

On Debian/Ubuntu, a practical starting point is:

```bash
sudo apt update
sudo apt install -y \
  build-essential pkg-config libssl-dev cmake curl git \
  libgtk-3-dev libayatana-appindicator3-dev librsvg2-dev \
  libsoup-3.0-dev libwebkit2gtk-4.1-dev libasound2-dev libclang-dev
```

If you hit native build errors, run:

```bash
npm run doctor
```

## Quick Start: Local

Use this if you want CodexMonitor and Codex on the same machine.

### If You Are Using a Release Build

1. Download CodexMonitor from the GitHub releases page.
2. Install and open the app.
3. Make sure `codex` works in your shell on that machine.
4. In CodexMonitor, add a workspace that points at one of your repos.
5. Start a thread and send a message.

### If You Are Building from Source

Clone the repo:

```bash
git clone git@github.com:caseyjkey/CodexMonitor.git
cd CodexMonitor
```

Install dependencies:

```bash
npm install
```

Run the app:

```bash
npm run tauri:dev
```

Then:

1. Open CodexMonitor.
2. Add a workspace.
3. Confirm the workspace can reach `codex`.
4. Start a thread.

## Quick Start: Server / Client

Use this if you want one machine to do the Codex work and another machine to connect remotely.

### Server Overview

The **server** machine is the one that:

- has access to the repos,
- has `codex` installed,
- runs the daemon,
- stays online while clients are connected.

### Client Overview

The **client** machine is the one that:

- runs the CodexMonitor app,
- connects to the server over TCP,
- does not need to host the repos locally if it is only acting as a remote client.

### Recommended Network Path

Tailscale is the recommended way to connect server and client. If both machines are already on the same tailnet, connectivity is usually the simple part. You still need:

- a reachable server host and port,
- a shared remote backend token,
- the daemon running on the server.

### Server Setup

If you are using a release build on the server:

1. Open CodexMonitor on the server.
2. Go to `Settings > Server`.
3. Set a `Remote backend token`.
4. Start the daemon from `Mobile access daemon`.
5. In `Tailscale helper`, detect or copy the suggested host, for example `server-name.tailnet.ts.net:4732`.

If you are building from source on the server:

Clone the repo:

```bash
git clone git@github.com:caseyjkey/CodexMonitor.git
cd CodexMonitor
```

Install dependencies and build the daemon tools:

```bash
npm install
cd src-tauri
cargo build --bin codex_monitor_daemon --bin codex_monitor_daemonctl
```

Start the daemon:

```bash
./target/debug/codex_monitor_daemonctl start \
  --listen 0.0.0.0:4732 \
  --token 'replace-this-with-a-strong-token' \
  --daemon-path ./target/debug/codex_monitor_daemon
```

Check daemon status:

```bash
./target/debug/codex_monitor_daemonctl status \
  --listen 0.0.0.0:4732 \
  --token 'replace-this-with-a-strong-token'
```

Notes:

- `0.0.0.0:4732` is appropriate when clients need to connect from other machines.
- If you omit `--daemon-path`, `codex_monitor_daemonctl` will try to resolve the daemon binary automatically.
- `--data-dir <path>` can be used if you want the daemon to read a specific `settings.json` / `workspaces.json`.

### Client Setup

On the client machine:

1. Install or run CodexMonitor.
2. Open `Settings > Server`.
3. Set `Backend mode` to `Remote (daemon)`.
4. Enter the server host and port, for example `server-name.tailnet.ts.net:4732`.
5. Enter the same remote backend token configured on the server.
6. Click `Connect & test`.
7. Confirm that the server workspaces load.

### Success Criteria

Your server/client setup is working when:

- `Connect & test` succeeds on the client,
- the client can see workspaces from the server,
- you can start or resume threads from the client,
- the server stays reachable while the client is connected.

## Troubleshooting

### The app starts but cannot run Codex

- Make sure `codex` is installed and available in `PATH`.
- If needed, configure a custom Codex binary path in app or workspace settings.

### A workspace does not behave correctly

- Make sure the repo path is valid on the machine actually running Codex.
- In server/client mode, that means the path must exist on the **server**, not the client.

### `Connect & test` fails

- Make sure the daemon is running on the server.
- Make sure the host and port are correct.
- Make sure the token matches exactly.
- Make sure the server and client can reach each other over your network or Tailscale.

### The daemon is running but clients still cannot connect

- Confirm the daemon is listening on a network-reachable address such as `0.0.0.0:4732`.
- Confirm that the hostname you entered resolves to the server you expect.
- Confirm nothing on the server is blocking the TCP port.

### I only want to use the app, not contribute to it

You can stop here. The rest of this README is for building, validating, releasing, or modifying CodexMonitor itself.

## Development

This section is for people building or contributing to CodexMonitor.

## Development Prerequisites

- Node.js + npm
- Rust toolchain (stable)
- CMake (required for native dependencies; dictation/Whisper uses it)
- LLVM/Clang (required on Windows to build dictation dependencies via bindgen)
- Codex CLI installed and available as `codex` in `PATH`
- Git CLI
- GitHub CLI (`gh`) for GitHub Issues/PR integrations (optional)

## Getting Started for Development

Install dependencies:

```bash
npm install
```

Run in dev mode:

```bash
npm run tauri:dev
```

## iOS Support (WIP)

iOS support is currently in progress.

- Current status: mobile layout runs, remote backend flow is wired, and iOS defaults to remote backend mode.
- Current limits: terminal and dictation remain unavailable on mobile builds.
- Desktop behavior is unchanged: macOS/Linux/Windows remain local-first unless remote mode is explicitly selected.

### iOS + Tailscale Setup (TCP)

Use this when connecting the iOS app to a desktop-hosted daemon over your Tailscale tailnet.

1. Install and sign in to Tailscale on both desktop and iPhone (same tailnet).
2. On desktop CodexMonitor, open `Settings > Server`.
3. Set a `Remote backend token`.
4. Start the desktop daemon with `Start daemon` (in `Mobile access daemon`).
5. In `Tailscale helper`, use `Detect Tailscale` and note the suggested host (for example `your-mac.your-tailnet.ts.net:4732`).
6. On iOS CodexMonitor, open `Settings > Server`.
7. Enter the desktop Tailscale host and the same token.
8. Tap `Connect & test` and confirm it succeeds.

Notes:

- The desktop daemon must stay running while iOS is connected.
- If the test fails, confirm both devices are online in Tailscale and that host/token match desktop settings.

### Headless Daemon Management (No Desktop UI)

Use the standalone daemon control CLI when you want remote mode without keeping the desktop app open.

Build binaries:

```bash
cd src-tauri
cargo build --bin codex_monitor_daemon --bin codex_monitor_daemonctl
```

Examples:

```bash
# Show current daemon status
./target/debug/codex_monitor_daemonctl status

# Start daemon using host/token from settings.json
./target/debug/codex_monitor_daemonctl start

# Stop daemon
./target/debug/codex_monitor_daemonctl stop

# Print equivalent daemon start command
./target/debug/codex_monitor_daemonctl command-preview
```

Useful overrides:

- `--data-dir <path>`: app data dir containing `settings.json` / `workspaces.json`
- `--listen <addr>`: bind address override
- `--token <token>`: token override
- `--daemon-path <path>`: explicit `codex-monitor-daemon` binary path
- `--json`: machine-readable output

### iOS Prerequisites

- Xcode + Command Line Tools installed.
- Rust iOS targets installed:

```bash
rustup target add aarch64-apple-ios aarch64-apple-ios-sim
# Optional (Intel Mac simulator builds):
rustup target add x86_64-apple-ios
```

- Apple signing configured (development team).
  - Set `bundle.iOS.developmentTeam` and `identifier` in `src-tauri/tauri.ios.local.conf.json` (preferred for local machine setup), or
  - set values in `src-tauri/tauri.ios.conf.json`, or
  - pass `--team <TEAM_ID>` to the device script.
  - `build_run_ios*.sh` and `release_testflight_ios.sh` automatically merge `src-tauri/tauri.ios.local.conf.json` when present.

### Run on iOS Simulator

```bash
./scripts/build_run_ios.sh
```

Options:

- `--simulator "<name>"` to target a specific simulator.
- `--target aarch64-sim|x86_64-sim` to override architecture.
- `--skip-build` to reuse the current app bundle.
- `--no-clean` to preserve `src-tauri/gen/apple/build` between builds.

### Run on USB Device

List discoverable devices:

```bash
./scripts/build_run_ios_device.sh --list-devices
```

Build, install, and launch on a specific device:

```bash
./scripts/build_run_ios_device.sh --device "<device name or identifier>" --team <TEAM_ID>
```

Additional options:

- `--target aarch64` to override architecture.
- `--skip-build` to reuse the current app bundle.
- `--bundle-id <id>` to launch a non-default bundle identifier.

First-time device setup usually requires:

1. iPhone unlocked and trusted with this Mac.
2. Developer Mode enabled on iPhone.
3. Pairing/signing approved in Xcode at least once.

If signing is not ready yet, open Xcode from the script flow:

```bash
./scripts/build_run_ios_device.sh --open-xcode
```

### iOS TestFlight Release (Scripted)

Use the end-to-end script to archive, upload, configure compliance, assign beta group, and submit for beta review.

```bash
./scripts/release_testflight_ios.sh
```

The script auto-loads release metadata from `.testflight.local.env` (gitignored).
For new setups, copy `.testflight.local.env.example` to `.testflight.local.env` and fill values.

## Release Build

Build the production Tauri bundle:

```bash
npm run tauri:build
```

Artifacts will be in `src-tauri/target/release/bundle/` (platform-specific subfolders).

### Windows (opt-in)

Windows builds are opt-in and use a separate Tauri config file to avoid macOS-only window effects.

```bash
npm run tauri:build:win
```

Artifacts will be in:

- `src-tauri/target/release/bundle/nsis/` (installer exe)
- `src-tauri/target/release/bundle/msi/` (msi)

Note: building from source on Windows requires LLVM/Clang (for `bindgen` / `libclang`) in addition to CMake.

## Type Checking

Run the TypeScript checker (no emit):

```bash
npm run typecheck
```

Note: `npm run build` also runs `tsc` before bundling the frontend.

## Validation

Recommended validation commands:

```bash
npm run lint
npm run test
npm run typecheck
cd src-tauri && cargo check
```

## Codebase Navigation

For task-oriented file lookup ("if you need X, edit Y"), use:

- `docs/codebase-map.md`

## Project Structure

```
src/
  features/         feature-sliced UI + hooks
  features/app/bootstrap/      app bootstrap orchestration
  features/app/orchestration/  app layout/thread/workspace orchestration
  features/threads/hooks/threadReducer/  thread reducer slices
  services/         Tauri IPC wrapper
  styles/           split CSS by area
  types.ts          shared types
src-tauri/
  src/lib.rs        Tauri app backend command registry
  src/bin/codex_monitor_daemon.rs  remote daemon JSON-RPC process
  src/bin/codex_monitor_daemon/rpc/  daemon RPC domain handlers
  src/shared/       shared backend core used by app + daemon
  src/shared/git_ui_core/      git/github shared core modules
  src/shared/workspaces_core/  workspace/worktree shared core modules
  src/workspaces/   workspace/worktree adapters
  src/codex/        codex app-server adapters
  src/files/        file adapters
  tauri.conf.json   window configuration
```

## Notes

- Workspaces persist to `workspaces.json` under the app data directory.
- App settings persist to `settings.json` under the app data directory (theme, backend mode/provider, remote endpoints/tokens, Codex path, default access mode, UI scale, follow-up message behavior).
- Feature settings are supported in the UI and synced to `$CODEX_HOME/config.toml` (or `~/.codex/config.toml`) on load/save. Stable: Collaboration modes (`features.collaboration_modes`), personality (`personality`), and Background terminal (`features.unified_exec`). Experimental: Apps (`features.apps`). Steering capability still follows Codex `features.steer`, but follow-up default behavior is controlled in Settings -> Composer.
- On launch and on window focus, the app reconnects and refreshes thread lists for each workspace.
- Threads are restored by filtering `thread/list` results using the workspace `cwd`.
- Selecting a thread always calls `thread/resume` to refresh messages from disk.
- CLI sessions appear if their `cwd` matches the workspace path; they are not live-streamed unless resumed.
- The app uses `codex app-server` over stdio; see `src-tauri/src/lib.rs` and `src-tauri/src/codex/`.
- The remote daemon entrypoint is `src-tauri/src/bin/codex_monitor_daemon.rs`; RPC routing lives in `src-tauri/src/bin/codex_monitor_daemon/rpc.rs` and domain handlers in `src-tauri/src/bin/codex_monitor_daemon/rpc/`.
- Shared domain logic lives in `src-tauri/src/shared/` (notably `src-tauri/src/shared/git_ui_core/` and `src-tauri/src/shared/workspaces_core/`).
- Codex home resolves from workspace settings (if set), then legacy `.codexmonitor/`, then `$CODEX_HOME`/`~/.codex`.
- Worktree agents live under the app data directory (`worktrees/<workspace-id>`); legacy `.codex-worktrees/` paths remain supported, and the app no longer edits repo `.gitignore` files.
- UI state (panel sizes, reduced transparency toggle, recent thread activity) is stored in `localStorage`.
- Custom prompts load from `$CODEX_HOME/prompts` (or `~/.codex/prompts`) with optional frontmatter description/argument hints.

## Tauri IPC Surface

Frontend calls live in `src/services/tauri.ts` and map to commands in `src-tauri/src/lib.rs`. The current surface includes:

- Settings/config/files: `get_app_settings`, `update_app_settings`, `get_codex_config_path`, `get_config_model`, `file_read`, `file_write`, `codex_doctor`, `menu_set_accelerators`.
- Workspaces/worktrees: `list_workspaces`, `is_workspace_path_dir`, `add_workspace`, `add_clone`, `add_worktree`, `worktree_setup_status`, `worktree_setup_mark_ran`, `rename_worktree`, `rename_worktree_upstream`, `apply_worktree_changes`, `update_workspace_settings`, `remove_workspace`, `remove_worktree`, `connect_workspace`, `list_workspace_files`, `read_workspace_file`, `open_workspace_in`, `get_open_app_icon`.
- Threads/turns/reviews: `start_thread`, `fork_thread`, `compact_thread`, `list_threads`, `resume_thread`, `archive_thread`, `set_thread_name`, `send_user_message`, `turn_interrupt`, `respond_to_server_request`, `start_review`, `remember_approval_rule`, `get_commit_message_prompt`, `generate_commit_message`, `generate_run_metadata`.
- Account/models/collaboration: `model_list`, `account_rate_limits`, `account_read`, `skills_list`, `apps_list`, `collaboration_mode_list`, `codex_login`, `codex_login_cancel`, `list_mcp_server_status`.
- Git/GitHub: `get_git_status`, `list_git_roots`, `get_git_diffs`, `get_git_log`, `get_git_commit_diff`, `get_git_remote`, `stage_git_file`, `stage_git_all`, `unstage_git_file`, `revert_git_file`, `revert_git_all`, `commit_git`, `push_git`, `pull_git`, `fetch_git`, `sync_git`, `list_git_branches`, `checkout_git_branch`, `create_git_branch`, `get_github_issues`, `get_github_pull_requests`, `get_github_pull_request_diff`, `get_github_pull_request_comments`.
- Prompts: `prompts_list`, `prompts_create`, `prompts_update`, `prompts_delete`, `prompts_move`, `prompts_workspace_dir`, `prompts_global_dir`.
- Terminal/dictation/notifications/usage: `terminal_open`, `terminal_write`, `terminal_resize`, `terminal_close`, `dictation_model_status`, `dictation_download_model`, `dictation_cancel_download`, `dictation_remove_model`, `dictation_request_permission`, `dictation_start`, `dictation_stop`, `dictation_cancel`, `send_notification_fallback`, `is_macos_debug_build`, `local_usage_snapshot`.
- Remote backend helpers: `tailscale_status`, `tailscale_daemon_command_preview`, `tailscale_daemon_start`, `tailscale_daemon_stop`, `tailscale_daemon_status`.
