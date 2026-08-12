# GAP ANALYSIS — CLAUDE CODE v2.1.224 → v2.1.228

**Covering:** Aug 7–11 2026 | **Generated:** Wed Aug 12 2026

**Current Setup:**
- Pi ARM64 | npm-installed Claude Code (not native binary)
- Telegram MCP via plugin (**no `alwaysLoad` set**)
- Hooks: SessionStart (`gsd-check-update.js` + `session_context.py`), PostToolUse (`gsd-context-monitor.js` type:command), PreToolUse (`gsd-prompt-guard.js`), statusLine (`gsd-statusline.js`)
- Plugins: superpowers + frontend-design
- No `cleanupPeriodDays` set | No `sandbox.network` config
- Automation scripts use raw curl for Telegram (not mcp_tool hooks)
- No custom themes

---

## GAPS — ACTION NEEDED (5)

### 1. Telegram MCP: `alwaysLoad` not set *(ongoing)*

Without `alwaysLoad`, Telegram MCP tools are deferred every session and require a `ToolSearch` call before they can be used. This adds latency and can cause silent failures when ToolSearch is skipped.

Note: v2.1.228 fixed the deferred-tools reminder being sent **twice** after a skill invocation — setting `alwaysLoad` avoids the deferral entirely and makes this moot.

**Fix:** In your Telegram MCP plugin config (settings.json, plugins section):
```json
"alwaysLoad": true
```

---

### 2. Self-hosted runner not configured on Oracle VM *(new in v2.1.224)*

`claude self-hosted-runner` turns any machine into a remote execution environment for Claude Code web/mobile/desktop sessions — **no manual SSH needed**. This directly addresses the MT5 Oracle VM SSH issue.

**Fix:** On Oracle VM, run:
```bash
claude self-hosted-runner --base-dir ~/claude-runner
```
Requires: Team or Enterprise plan.

---

### 3. `cleanupPeriodDays` not set *(ongoing)*

v2.1.228 fixed a critical bug where session cleanup deleted contents inside project memory folders. But without `cleanupPeriodDays` configured, cleanup behavior remains uncontrolled — sessions may accumulate indefinitely.

**Fix:** Add to `~/.claude/settings.json`:
```json
"cleanupPeriodDays": 30
```

---

### 4. `crossSessionInbound` / `dialogExpiry` not configured *(new in v2.1.225)*

Without this setting:
- Cross-session messages sent to headless/automation sessions (like Cipher's scheduled runs) can park silently without delivery or expiry.
- Messages to sessions with bypassed permissions auto-deliver without approval.

**Fix:** Add to `settings.json`:
```json
"crossSessionInbound": "hold",
"dialogExpiry": 3600
```

---

### 5. claude.ai-synced skills — review after v2.1.228 hardening

v2.1.228 hardened skills synced from claude.ai: they now block `!` commands and `@` file expansions in skill bodies, and sanitize descriptions. Skills that relied on these features will silently break.

**Fix:** Run `/skills` and test each claude.ai-sourced skill. Recreate locally any that no longer work.

---

## ALREADY FIXED — NO ACTION NEEDED

| Fix | Version | Notes |
|---|---|---|
| Session cleanup deleting memory folder contents | v2.1.228 | Run `npm update -g @anthropic-ai/claude-code` if not on v2.1.228+ |
| Plugin cache cleanup deleting symlinked plugin cache | v2.1.228 | Relevant if superpowers/frontend-design are symlinked |
| Marketplace entry inheriting wrong tier's custom headers | v2.1.228 | Marketplace entries now merge as whole entries |
| Deferred-tools reminder sent twice after skill invocation | v2.1.228 | Affected Telegram MCP plugin |
| MCP tools connecting mid-turn not announced to model | v2.1.225 | Directly affected Telegram MCP plugin |
| Plugin install records corrupted across multiple projects | v2.1.224 | Now fixed |
| 200-subagent-per-session cap removed | v2.1.224 | Good for Cipher automation sessions |
| Feature flags without subscription tier on expired login | v2.1.227 | Max plan users wrongly prompted for usage credits |
| General bug fixes and reliability | v2.1.226 | Automatic benefit on npm update |

---

## NOT APPLICABLE TO OUR SETUP

| Item | Version | Reason |
|---|---|---|
| Windows / Git Bash path fixes | v2.1.228 | Pi ARM64 Linux |
| `/tui` model revert bug | v2.1.228 | We don't use /tui |
| Vertex AI credential handling | v2.1.228 | We use Anthropic direct API |
| GitHub Actions `allowed_non_write_users` | v2.1.227 | Not using GH Actions runners |
| macOS keychain / MCP OAuth intermittent 401s | v2.1.225 | Linux only |
| Gateway spend-limit support | v2.1.225 | No gateway |
| Bedrock `ANTHROPIC_BEDROCK_REGION_PREFIX` | v2.1.224 | No Bedrock |
| VSCode extension fixes | v2.1.224, v2.1.225 | No VSCode |
| Sandbox credential-masking / `sandbox.network` | v2.1.224 | No sandbox.network config |
| Remote Control connection/auto-start | v2.1.224, v2.1.225, v2.1.228 | Not using Remote Control |
| Wayland clipboard copy-on-select | v2.1.224 | Headless Pi, no Wayland GUI |
| `claude agents` workspace trust prompt | v2.1.225 | Not our flow |

---

## QUICK-WIN PRIORITY ORDER

1. **`alwaysLoad: true`** on Telegram MCP plugin — 2 min, edit `settings.json`
2. **`cleanupPeriodDays: 30`** in `settings.json` — 1 min
3. **`crossSessionInbound: "hold"` + `dialogExpiry: 3600`** — 2 min
4. **`claude self-hosted-runner`** on Oracle VM — 15 min, solves MT5 SSH access
5. **Review claude.ai-synced skills** for breakage after hardening — 10 min

**Update Claude Code:**
```bash
npm update -g @anthropic-ai/claude-code
```

---

*Cipher Gap Analysis | Wed Aug 12 2026 | Delivery: committed to repo (Telegram + email blocked by network policy)*
