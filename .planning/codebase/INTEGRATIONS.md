# External Integrations

**Analysis Date:** 2026-03-13

## APIs & External Services

**Web Search (Optional):**
- Brave Search API - Research capability for project/domain exploration
  - Endpoint: `https://api.search.brave.com/res/v1/web/search`
  - Auth: `BRAVE_API_KEY` environment variable or `~/.gsd/brave_api_key` file
  - Features: Query, result limiting, freshness filtering (day/week/month)
  - Code location: `.claude/get-shit-done/bin/lib/commands.cjs` (lines 321-380)

**npm Registry:**
- Used for version checking (`npm view get-shit-done-cc version`)
- No auth required for public package queries
- Called in background via `gsd-check-update.js` hook

## Data Storage

**Databases:**
- None detected - file-based storage only

**File Storage:**
- Local filesystem for all state
- Temporary files written to `os.tmpdir()` for large JSON payloads
- Cache files: `~/.claude/cache/gsd-update-check.json`

**Caching:**
- Update check cache: `~/.claude/cache/gsd-update-check.json`
- Context metrics bridge: `/tmp/claude-ctx-{session_id}.json`
- Warning debounce state: `/tmp/claude-ctx-{session_id}-warned.json`

## Authentication & Identity

**Auth Provider:**
- None required - framework runs within Claude Code/OpenCode/Gemini/Codex context
- No user authentication system
- No session management

**Permission System:**
- Claude Code: Configured via `.claude/settings.json` with tool allowlists
- OpenCode: Configured via `.opencode/opencode.json` with directory permissions
- Permissions restrict Bash tool access to safe commands only

## Monitoring & Observability

**Error Tracking:**
- None detected (no external error tracking services)

**Logs:**
- Console output via `process.stdout` and `process.stderr`
- Silent failure pattern used throughout (try/catch with empty handlers)
- No structured logging framework

**Metrics:**
- Context window usage tracked via statusline hook
- Progress calculated from file existence (plans vs summaries)
- Update availability checked via npm registry query

## CI/CD & Deployment

**Hosting:**
- Distributed via npm registry as `get-shit-done-cc` package
- Installed to user's home directory
- Multi-platform support: `.claude`, `.gemini`, `.opencode`, `.codex` variants

**CI Pipeline:**
- None detected

**Update Mechanism:**
- Background update check via `gsd-check-update.js` hook
- Runs on session start, caches result for quick statusline display
- Manual update via `/gsd:update` command

## Environment Configuration

**Required env vars:**
- None strictly required for basic operation

**Optional env vars:**
- `BRAVE_API_KEY` - Enables web search functionality
- `CLAUDE_CONFIG_DIR` - Custom Claude config directory (supports multi-account)
- `HOME` - Used for tilde expansion in path references

**Secrets location:**
- `~/.gsd/brave_api_key` - File-based API key storage for Brave Search
- No `.env` files detected

## Webhooks & Callbacks

**Incoming:**
- None (framework runs entirely client-side)

**Outgoing:**
- None (no webhook callbacks registered)

**Internal Hooks:**
- `SessionStart` - Update check hook
- `PostToolUse`/`AfterTool` - Context monitoring hook
- `StatusLine` - Real-time status display hook

## Multi-Platform Support

**Supported AI Platforms:**
| Platform | Config Dir | Settings File | Hook Type |
|----------|-----------|---------------|-----------|
| Claude Code | `.claude` | `settings.json` | PostToolUse |
| OpenCode | `.opencode` | `opencode.json` | Command hooks |
| Gemini CLI | `.gemini` | `settings.json` | AfterTool |
| Codex CLI | `.codex` | `config.toml` | TOML agents |

**Platform Detection:**
- Automatic detection in `gsd-check-update.js` via config directory existence
- Priority: env override > `.config/opencode` > `.opencode` > `.gemini` > `.claude`

---

*Integration audit: 2026-03-13*
