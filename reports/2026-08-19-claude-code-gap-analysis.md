# GAP ANALYSIS — CLAUDE CODE v2.1.231–v2.1.235

**Date:** 2026-08-19  
**Setup:** Pi ARM64 | npm-installed Claude Code | Telegram MCP via plugin (no alwaysLoad) | Hooks: SessionStart (gsd-check-update.js + session_context.py), PostToolUse (gsd-context-monitor.js), PreToolUse (gsd-prompt-guard.js), statusLine (gsd-statusline.js) | Plugins: superpowers + frontend-design | No cleanupPeriodDays | No sandbox.network | Raw curl for Telegram in automation

---

## v2.1.235 — Aug 18, 2026

**[ACTION NEEDED - OPTIONAL]** Spellcheck in prompt input  
New optional setting. Add `"spellcheck": true` to `~/.claude/settings.json`. Requires `aspell`, `hunspell`, or `ispell` on Pi. Install: `sudo apt install aspell`

**[ALREADY FIXED]** Improved memory/CPU for background cloud sessions — event streams no longer re-scanned on every update. Benefits long automation runs.

**[ALREADY FIXED]** Improved embedded grep on Linux — pathological patterns fail fast. Our ARM64 npm build includes this.

**[ALREADY FIXED]** Fixed prompt input highlights in multi-line prompts; fixed Shift+Tab in permission prompt; improved context-limit error shows auto-compact state.

**[NOT APPLICABLE]** VSCode focus fix, Remote Control enterprise checks — Terminal only.

---

## v2.1.234 — Aug 17, 2026

**[ACTION NEEDED]** Auto-resume after usage limit resets  
New: Claude Code continues sessions when claude.ai usage limit resets. Check `/config` for the "Continue after usage limit" toggle. Enable for unattended Pi automation runs.

**[ACTION NEEDED - OPTIONAL]** `CLAUDE_CODE_PROJECT_DIR_NAME` env var  
Per-project transcript directory naming. Add to Pi `.env`:  
```
export CLAUDE_CODE_PROJECT_DIR_NAME=cipher-automation
```

**[ALREADY FIXED - CRITICAL]** Fixed MCP diagnostics printing resolved secrets  
Previous versions **exposed our Telegram bot token** in MCP diagnostic output. Fixed in 2.1.234+.  
**Immediate action:** `npm update -g @anthropic-ai/claude-code` if not already on 2.1.234+.

**[ALREADY FIXED]** Fixed session-scoped permission answers dropped on subagent prompts — affects automation workflows.

**[ALREADY FIXED]** Reduced `claude-api` skill context cost 200k → 25k tokens. Directly benefits automation sessions.

**[ALREADY FIXED]** Background task notifications now in `<system-reminder>` tags. Scheduled task handling improved.

**[NOT APPLICABLE]** GitLab MR badge, Windows startup fix.

---

## v2.1.233 — Aug 14, 2026

**[ACTION NEEDED - CRITICAL]** Todo/TaskCreate tools disabled on newer models  
`TaskCreate`, `TaskGet`, `TaskUpdate`, `TaskList`, `TodoWrite` are **no longer available** on Sonnet 5, Opus 5, Fable 5, Mythos 5+.  
If automation scripts or hooks use these tools on those models, they will silently fail.  
**Fix:** Add to `~/.claude/settings.json` or Pi `.env`:
```
export CLAUDE_CODE_ENABLE_TODO_TOOLS=1
```

**[ACTION NEEDED - OPTIONAL]** Memory limit for Bash tool on Linux  
New env var: `CLAUDE_CODE_TOOL_MEMORY_LIMIT` (e.g., `"256M"`). Pi has limited RAM — set this to prevent OOM kills:
```
export CLAUDE_CODE_TOOL_MEMORY_LIMIT=256M
```

**[ACTION NEEDED - OPTIONAL]** WebFetch cache TTL  
`CLAUDE_CODE_WEBFETCH_CACHE_TTL_MS` env var (default: 15 minutes). Tune for automation sessions that re-fetch URLs.

**[ALREADY FIXED - CRITICAL]** Fixed idle Linux sessions keeping 1 CPU core at 100% with sandboxing enabled  
This was likely hammering our Pi CPU. Fixed in 2.1.233+. Update immediately.

**[ALREADY FIXED]** Fixed MCP v2 connections endlessly reopening subscriptions — our Telegram MCP was potentially affected.

**[ALREADY FIXED]** Fixed bundled skill aliases (`/checkup`, `/review`) in `-p` mode — our scheduled automation uses `-p` mode.

**[NOT APPLICABLE]** GitLab MR URL support, Notification hooks for Desktop/VSCode — Terminal only.

---

## v2.1.232 — Aug 13, 2026

**[ACTION NEEDED]** Bash input redirections now permission-checked  
Commands using `< file` (e.g., `curl ... < input.txt`, `python script.py < data.json`) now trigger permission prompts.  
Our `gsd-prompt-guard.js` (PreToolUse hook) may block automation scripts using this pattern.  
**Review:** `grep -r "< " ~/cipher-automation/scripts/` to find affected commands.  
Add pre-approval patterns to the hook if needed.

**[ACTION NEEDED]** `sandbox.ripgrep` no longer honored from project settings  
Must now be in **user** settings, not `.claude/settings.json`.  
Move if present:  
```
From: .claude/settings.json → "sandbox": {"ripgrep": ...}
To:   ~/.claude/settings.json → "sandbox": {"ripgrep": ...}
```

**[ACTION NEEDED]** Set `alwaysLoad: true` for Telegram MCP plugin  
This release improved plugin loading consistency. Without `alwaysLoad`, the Telegram MCP plugin may not load in `-p` (non-interactive) automation sessions.  
**Fix:** In `~/.claude/settings.json`, set `"alwaysLoad": true` for the Telegram plugin entry.

**[ALREADY FIXED]** Fixed MCP connections hanging 30 seconds on server probe failure — our Telegram MCP was causing startup delays.

**[ALREADY FIXED]** Hardened Linux filesystem sandbox against protected-path bypass — security fix.

**[ALREADY FIXED]** Fixed nested git repositories inheriting parent trust.

**[NOT APPLICABLE]** PowerShell bypass fixes, GitLab marketplace, Remote Control cloud sessions — Linux terminal only.

---

## v2.1.231 — Aug 13, 2026

**[NOT APPLICABLE]** Fixed MCP OAuth redirect URI mismatch (Slack MCP) — our Telegram MCP uses a bot token, not OAuth.

---

## ✅ ACTION SUMMARY (Priority Order)

| # | Priority | Action |
|---|---|---|
| 1 | 🔴 CRITICAL | `npm update -g @anthropic-ai/claude-code` (fixes MCP token leak, Pi CPU spike, MCP 30s hangs) |
| 2 | 🔴 CRITICAL | `export CLAUDE_CODE_ENABLE_TODO_TOOLS=1` (TaskCreate/TaskGet broken on Sonnet 5/Opus 5 without this) |
| 3 | 🟠 HIGH | Set `"alwaysLoad": true` for Telegram MCP plugin in `~/.claude/settings.json` |
| 4 | 🟠 HIGH | Review `gsd-prompt-guard.js` for Bash `< redirection` patterns — new permission prompts |
| 5 | 🟡 MEDIUM | Move `sandbox.ripgrep` from project → user settings if present |
| 6 | 🟡 MEDIUM | Check `/config` for "Continue after usage limit" auto-resume toggle |
| 7 | 🟢 LOW | `export CLAUDE_CODE_TOOL_MEMORY_LIMIT=256M` (Pi RAM protection) |
| 8 | 🟢 OPTIONAL | Add `"spellcheck": true` to settings.json (needs `sudo apt install aspell`) |

---

> ⚠️ **Delivery note:** Telegram (api.telegram.org) and SMTP email (server355.web-hosting.com:465) are blocked by the remote execution environment's egress proxy (403/timeout). Report saved to repo instead. The Pi-side cron job running Cipher directly will not have this restriction.

*Generated by cipher-automation | 2026-08-19*
