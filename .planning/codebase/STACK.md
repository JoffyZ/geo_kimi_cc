# Technology Stack

**Analysis Date:** 2026-03-13

## Languages

**Primary:**
- JavaScript (Node.js) - Entire codebase written in CommonJS
- Markdown - Documentation, templates, and agent prompts

**Secondary:**
- JSON - Configuration files and manifests
- TOML - Codex agent configuration (`.codex/config.toml`, `.codex/agents/*.toml`)

## Runtime

**Environment:**
- Node.js v22.22.0 (detected in environment)
- CommonJS module system (all `.cjs` files)

**Package Manager:**
- npm 10.9.4
- Lockfile: Not present (framework distributed as standalone files)

## Frameworks

**Core:**
- Custom Node.js CLI framework - Built-in command system in `gsd-tools.cjs`
- Multi-agent orchestration system - Native implementation without external frameworks

**Testing:**
- Not detected (no test files found)

**Build/Dev:**
- None (framework ships as source, no build step required)

## Key Dependencies

**Critical (Built-in Node.js modules only):**
- `fs` - File system operations for state management
- `path` - Cross-platform path handling
- `os` - Operating system utilities (tmpdir, homedir)
- `child_process` - Git operations via `execSync` and `spawn`
- `https`/`fetch` - Web search API calls (Brave Search)

**Infrastructure:**
- Git - Required for version control integration
- No external npm dependencies detected

## Configuration

**Environment:**
- `BRAVE_API_KEY` - Optional API key for web search via Brave Search API
- `CLAUDE_CONFIG_DIR` - Optional override for custom Claude config directory
- Home directory detection via `os.homedir()` for config paths

**Config Files:**
- `.claude/settings.json` - Claude Code permissions configuration
- `.gemini/settings.json` - Gemini CLI hooks and statusline config
- `.opencode/opencode.json` - OpenCode permission settings
- `.codex/config.toml` - Codex multi-agent configuration
- `.planning/config.json` - Per-project GSD configuration (generated)

**Key Configuration Options:**
- `model_profile`: 'quality' | 'balanced' | 'budget' - Model selection strategy
- `commit_docs`: boolean - Whether to auto-commit planning documents
- `branching_strategy`: 'none' | 'branch-per-phase' | 'branch-per-milestone'
- `research`: boolean - Enable research phase before planning
- `verifier`: boolean - Enable verification phase after execution
- `brave_search`: boolean - Enable web search capability

## Platform Requirements

**Development:**
- Node.js >= 18.0.0 (uses native `fetch` API)
- Git >= 2.0 (for git operations and check-ignore)
- Unix-like shell (uses `find`, `grep` commands via execSync)

**Production:**
- Deployed locally to user's home directory (`~/.claude`, `~/.gemini`, etc.)
- Self-updating via npm: `get-shit-done-cc` package
- Version tracked in `get-shit-done/VERSION` file (currently 1.22.4)

## Architecture Pattern

**Agent-based System:**
- 12 specialized agent types defined in `core.cjs` MODEL_PROFILES
- Agents spawned as sub-processes with markdown prompt files
- Model profile resolution: quality/balanced/budget mapped to Claude models (opus/sonnet/haiku)

**File-based State Management:**
- No database - all state stored in markdown files with frontmatter
- Key files: `STATE.md`, `ROADMAP.md`, `PLAN.md`, `SUMMARY.md`
- JSON bridge files for inter-process communication (context metrics)

---

*Stack analysis: 2026-03-13*
