# Claude Code Telegram Integration (OpenClaw-like Setup)

Anthropic has released Claude Code Channels, which allows connecting Claude Code to Telegram. With this update, we get very close to an [OpenClaw]-like implementation using an official Anthropic Claude subscription — without relying on third-party wrappers.

However, out of the box, the Telegram channel integration requires some fixes and automation to work smoothly as a persistent, headless AI assistant. This project provides those fixes.

---

## Fix: Auto-Approval of Claude Code Permission Prompts

Claude Code occasionally displays interactive permission prompts (e.g., "Do you want to make this edit to settings.json?" with options like Yes / Yes, and allow for this session / No). These prompts block the session and are not forwarded to Telegram by default, causing the bot to appear stuck.

**This fix:** The daemon monitors the tmux session for these permission prompts. When one is detected, it is forwarded to Telegram as an FYI notification and automatically approved by sending a keypress to select the default option, unblocking the session immediately.

---

## Features

### 1. Inactivity Timeout with Memory Persistence
Sessions have a configurable inactivity timer (default: 1 hour). After the timeout is reached, the daemon asks Claude to save any important information to its persistent memory files, waits 5 minutes for completion, then gracefully exits the session. This prevents stale sessions from running indefinitely while preserving context across sessions.

### 2. Automatic Session Spawning
When a new message is sent on Telegram and no active session exists, the daemon automatically spins up a fresh Claude Code session. This reduces the risk of context window clogging from long-running persistent sessions and ensures each conversation starts clean.

### 3. Headless Operation via tmux
Claude Code runs inside a tmux session, allowing it to operate entirely headlessly. Even if no terminal is active on the client side, the bot continues to respond and work. Sessions can be spawned, monitored, and terminated without any interactive terminal access.

### 4. Telegram-Initiated Session Exit
You can tell Claude on Telegram to exit a session (e.g., "save memories and exit"). Claude sends a farewell message, then types `/exit` in its output. The daemon detects `/exit` combined with an idle prompt (no active thinking or tool execution), and terminates the session. This is confirmed with a "Session ended. Send a message to start a new one." notification on Telegram, so the user knows the session has closed and the next message will start a fresh one.

### 5. Session Lifecycle Notifications
The daemon sends Telegram notifications for key lifecycle events — session ended due to voluntary exit, crash, or inactivity timeout — so the user always knows the current state of their assistant.

---

## Architecture

- **Daemon script** (`claude-telegram-daemon.sh`) — Runs as a systemd service; polls Telegram when idle, spawns sessions on demand, monitors inactivity, handles graceful shutdown.
- **Expect wrapper** (`start-claude-telegram.sh`) — Bypasses the interactive "trust this folder" prompt on startup.
- **Permission auto-approval** — Forwards prompts to Telegram and auto-approves to prevent blocking.
- **Exit detection** — Monitors tmux pane for `/exit` + idle prompt to cleanly terminate sessions.
- **systemd service** (`claude-telegram.service`) — Ensures the daemon starts on boot and restarts on crash.

---

## Requirements

- Linux VPS (Ubuntu 22.04+)
- Claude Pro subscription (or higher)
- Telegram bot (created via BotFather)
- Claude Code installed
- tmux, expect, jq, curl
