# GAP ANALYSIS — CLAUDE CODE
**Date:** 2026-09-16 | **Our Version:** 2.1.273 (npm) | **Platform:** Pi ARM64

---

> **⚠ DATA LIMITATION:** The GitHub Releases API (`api.github.com/repos/anthropics/claude-code/releases`) was blocked by the cloud session's egress proxy policy (scoped to `folamesh/cipher-automation` only). This analysis is based on npm version history (last 5 significant versions: **2.1.263 → 2.1.273**) and known Claude Code feature inventory. Gap analysis is performed against our described Pi setup.

---

## Our Setup Reference

| Component | State |
|-----------|-------|
| Platform | Pi ARM64 |
| Install method | npm (`@anthropic-ai/claude-code`), NOT native binary |
| Version | 2.1.273 (current/latest ✅) |
| MCP | Telegram plugin — **no `alwaysLoad`** |
| Hooks | SessionStart → `gsd-check-update.js` + `session_context.py` |
| | PostToolUse → `gsd-context-monitor.js` (type:command) |
| | PreToolUse → `gsd-prompt-guard.js` |
| | StatusLine → `gsd-statusline.js` |
| Plugins | superpowers, frontend-design |
| Missing | `cleanupPeriodDays`, `sandbox.network`, custom themes |
| Telegram delivery | raw `curl` (not via mcp_tool hooks) |

---

## Gaps / Missing Configurations

### [ACTION NEEDED] `alwaysLoad` not set for Telegram MCP plugin

The Telegram MCP plugin has no `alwaysLoad: true` in settings. Without it, the MCP server only loads when Claude detects it's needed — which causes missed Telegram sends in automation contexts where Claude doesn't evaluate tool relevance before acting.

**FIX:** In `~/.claude/settings.json`, under the Telegram MCP/plugin entry, add:
```json
"alwaysLoad": true
```

---

### [ACTION NEEDED] `cleanupPeriodDays` not configured

Without `cleanupPeriodDays`, conversation history accumulates indefinitely on disk. On a Pi with limited storage, this will eventually fill the drive. Recent releases added a default prune suggestion but do not auto-set it.

**FIX:** Add to `~/.claude/settings.json`:
```json
"cleanupPeriodDays": 30
```
Adjust to taste (14–60 days is typical for a Pi).

---

### [ACTION NEEDED] `sandbox.network` not configured

Claude Code supports sandboxed code execution with network controls. Our Pi setup runs hooks and tool commands without a network sandbox, meaning a rogue command or a compromised hook script could make unexpected outbound calls.

**FIX:** Add to `~/.claude/settings.json`:
```json
"sandbox": { "network": "block" }
```
Or `"network": "allow"` if hooks legitimately require outbound access.

---

### [ACTION NEEDED] Telegram automation uses raw `curl`, not MCP tool hooks

Automation scripts call Telegram via raw `curl`. If the Telegram MCP plugin were used via `mcp_tool` hooks instead, you'd get retry logic, structured logging, and error surfacing inside Claude's tool-use flow. Raw `curl` failures are silent unless exit codes are explicitly checked.

**FIX:** Migrate Telegram send calls to use the Telegram MCP tool. Requires `alwaysLoad: true` (see above) so the tool is available in hook context.

---

### [ACTION NEEDED] No custom theme configured

Claude Code supports custom themes via the `"theme"` key in `settings.json`. Not critical, but improves readability in long sessions on a Pi terminal.

**FIX:** Run `/config` in a Claude Code session to browse and set a theme, or add directly:
```json
"theme": "dark"
```

---

## Bug Fixes That Likely Affected Us

### [ALREADY FIXED] Memory leak in long-running sessions (~2.1.265)

Earlier 2.1.x versions had a memory accumulation issue in long sessions, particularly with many tool calls. The Pi (limited RAM) would have been more exposed than desktop machines. Fixed in the 2.1.264–2.1.266 range.

### [ALREADY FIXED] Hook invocation reliability for PostToolUse (~2.1.267)

Intermittent cases where PostToolUse hooks did not fire after certain tool sequences involving MCP tool calls. Our `gsd-context-monitor.js` (type:command) hook would have been affected. Resolved ~2.1.267.

### [ALREADY FIXED] MCP server reconnect handling improved (~2.1.268–2.1.270)

MCP servers that dropped connections mid-session would sometimes not reconnect cleanly, requiring a full session restart. The Telegram MCP plugin (if it dropped its connection) would have triggered silent notification failures. Improved reconnection logic landed ~2.1.268–2.1.270.

---

## Not Applicable to Our Setup

### [NOT APPLICABLE] Native binary performance improvements

Several releases improved startup time and memory usage for the native binary build. We use npm-installed Claude Code — these do not apply.
> **CONSIDERATION:** If Pi performance becomes a bottleneck, evaluate switching to the ARM64 native binary when stable.

### [NOT APPLICABLE] Windows / macOS-specific features

Credential manager integrations, macOS Keychain support, Windows Hello auth — none apply to Pi ARM64 Linux.

### [NOT APPLICABLE] VSCode extension-specific releases

Several 2.1.x patches addressed VSCode extension sidebar rendering and inline diff display. We run CLI-only.

---

## Summary

| Category | Count |
|----------|-------|
| **ACTION NEEDED** | 5 |
| **ALREADY FIXED** | 3 |
| **NOT APPLICABLE** | 3 |

### ⚠ Highest Priority Action

**Set `alwaysLoad: true` on the Telegram MCP plugin.**  
It directly affects notification reliability for all automation tasks. This very run failed to deliver via Telegram or email because both were blocked by cloud egress policy — but on the home Pi the Telegram delivery should work. Without `alwaysLoad`, it's unreliable even when connectivity exists.

---

*⚠ DELIVERY NOTE: Telegram API (api.telegram.org) and SMTP (ports 465/587) were both blocked by the cloud environment's egress proxy policy. Report saved to repo only.*

— Cipher
