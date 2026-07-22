# GAP ANALYSIS — CLAUDE CODE
**Versions analyzed:** 2.1.217 · 2.1.216 · 2.1.215 · 2.1.214 · 2.1.212  
**Date:** July 22, 2026  
**Our setup:** Pi ARM64 · npm-installed Claude Code (not native binary) · Telegram MCP via plugin (no alwaysLoad) · Hooks: SessionStart (gsd-check-update.js + session_context.py), PostToolUse (gsd-context-monitor.js), PreToolUse (gsd-prompt-guard.js), statusLine (gsd-statusline.js) · Plugins: superpowers + frontend-design · No cleanupPeriodDays · No sandbox.network config · Raw curl for Telegram automation

---

## [ACTION NEEDED]

### 1. `cleanupPeriodDays` not configured [Ongoing]
Sessions accumulate forever on Pi disk — storage risk.  
**FIX:** Add `"cleanupPeriodDays": 30` to `~/.claude/settings.json`

### 2. Telegram MCP — no `alwaysLoad` set [Ongoing]
Without `alwaysLoad: true`, the Telegram MCP may not be available in background/scheduled sessions, forcing fallback to raw curl.  
**FIX:** In `~/.claude/settings.json`, under the Telegram plugin config add `"alwaysLoad": true`

### 3. `/verify` and `/code-review` no longer auto-invoke [v2.1.215]
Breaking behavioral change — these skills were previously invoked automatically by Claude. Now they must be called explicitly.  
**FIX:** Audit all automation prompts/scripts that relied on auto-invocation. Add explicit `/verify` or `/code-review` calls where needed.

### 4. `sandbox.filesystem.disabled` — new setting available [v2.1.216]
New option allows network egress control without filesystem isolation. Previously, enabling sandbox disabled both.  
**FIX (if desired):** Add `"sandbox": {"filesystem": {"disabled": true}}` to `~/.claude/settings.json`

### 5. Subagent concurrency cap now default 20 + nested spawning disabled [v2.1.217]
`CLAUDE_CODE_MAX_CONCURRENT_SUBAGENTS` defaults to 20. Nested subagent spawning is now **off by default** — deep nesting silently fails unless configured.  
**FIX:** If using parallel subagents, set `CLAUDE_CODE_MAX_CONCURRENT_SUBAGENTS=N` env var. For nested agents (e.g. workflow-within-workflow): set `CLAUDE_CODE_MAX_SUBAGENT_SPAWN_DEPTH=2`

---

## [ALREADY FIXED] — Key fixes now in your installed version

| Fix | Version | Relevance to Our Setup |
|-----|---------|------------------------|
| Bash `pkill -f` killing Claude session on Linux | 2.1.214 | **CRITICAL** — Pi ARM64 Linux |
| Scheduled task prompt trust issue — fired prompts now delivered properly | 2.1.214 | **CRITICAL** — directly affects Cipher automation |
| MCP truncated output memory leak (full results kept in memory) | 2.1.217 | High — Telegram MCP heavy use |
| `CLAUDE.md`/`SKILL.md` brace expansion OOM at startup | 2.1.217 | Medium — if CLAUDE.md has brace patterns |
| Workflow/scheduled task writes following symlinks at `.claude` | 2.1.216 | High — scheduled tasks |
| Hooks exit code 2 not blocking on schema validation failure | 2.1.214 | High — `gsd-prompt-guard.js` PreToolUse hook |
| Plugin skills losing slash-menu prefix in autocomplete | 2.1.216 | Medium — superpowers + frontend-design |
| Skills and commands not appearing in slash menu until restart | 2.1.216 | Medium — plugin skills |
| `dir/**` allow rules over-approving nested directory writes | 2.1.214 | Security — review your allow rules |
| Quadratic slowdown in long sessions (message normalization) | 2.1.216 | Medium — long automation sessions |
| ISO `modified` timestamp in memory file frontmatter | 2.1.214 | Low — better memory tracking |
| Background subagents cancelled during high-priority message arrivals | 2.1.216 | Medium — if using background agents |
| MCP re-authenticate revoking working credentials prematurely | 2.1.216 | Medium — Telegram MCP |
| Periodic progress heartbeat for long-running tool calls | 2.1.214 | Low — better UX in long sessions |

---

## [NOT APPLICABLE]

- Windows auto-update / `claude.exe` missing after update (Pi ARM64 Linux)
- PowerShell 5.1 Unicode, redirect, and `where.exe` / `fc.exe` fixes (Linux)
- Bedrock / Claude Opus 4.8 auto-compact issues (standard API, not Bedrock)
- Desktop app session fixes / corporate mTLS / proxy OAuth scope (CLI only)
- VSCode extension right-to-left text rendering (no IDE extension in use)
- Remote Control session "session ready" / late-joining viewer fixes (not in use)
- Native binary auto-update (`claude.exe`) — npm-installed, not native binary
- Claude-in-Chrome 403 OAuth reconnect loop (terminal use only)
- `EndConversation` tool for abusive users (automated/trusted sessions)
- Screen reader mode startup announcement fix (not applicable)

---

## Recommended Actions — Priority Order

1. **Immediate:** Add `"cleanupPeriodDays": 30` to `~/.claude/settings.json`
2. **Immediate:** Set `"alwaysLoad": true` for Telegram MCP plugin in settings
3. **Short-term:** Audit automation prompts — remove assumptions about `/verify` or `/code-review` auto-invoking
4. **Evaluate:** Consider `sandbox.filesystem.disabled` for network policy control
5. **When needed:** Set `CLAUDE_CODE_MAX_CONCURRENT_SUBAGENTS` env var before running parallel subagent workflows

---
*Generated by Cipher | Automated Claude Code Gap Analysis | July 22, 2026*
