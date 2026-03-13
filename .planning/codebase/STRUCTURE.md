# Codebase Structure

**Analysis Date:** 2026-03-13

## Directory Layout

```
[project-root]/
├── .claude/                    # Claude Code configuration
│   ├── agents/                 # AI agent definitions
│   ├── commands/gsd/           # Slash command definitions
│   ├── get-shit-done/          # Core framework files
│   │   ├── bin/                # CLI tools
│   │   │   ├── gsd-tools.cjs   # Main CLI entry point
│   │   │   └── lib/            # Library modules
│   │   │       ├── commands.cjs
│   │   │       ├── config.cjs
│   │   │       ├── core.cjs
│   │   │       ├── frontmatter.cjs
│   │   │       ├── init.cjs
│   │   │       ├── milestone.cjs
│   │   │       ├── phase.cjs
│   │   │       ├── roadmap.cjs
│   │   │       ├── state.cjs
│   │   │       ├── template.cjs
│   │   │       └── verify.cjs
│   │   ├── references/         # Documentation references
│   │   ├── templates/          # Document templates
│   │   │   ├── codebase/       # Codebase analysis templates
│   │   │   └── research-project/ # Research templates
│   │   └── workflows/          # Workflow definitions
│   ├── hooks/                  # Runtime hooks (JS)
│   ├── settings.json           # Claude Code settings
│   └── gsd-file-manifest.json  # File integrity manifest
├── .codex/                     # Codex configuration
│   └── skills/gsd-*/           # Skill definitions
├── .gemini/                    # Gemini configuration
│   ├── commands/gsd/           # Command mappings
│   └── get-shit-done/          # Framework copy
├── .opencode/                  # OpenCode configuration
│   ├── commands/gsd/           # Command mappings
│   ├── get-shit-done/          # Framework copy
│   └── settings.json
└── .planning/                  # Runtime project state
    ├── codebase/               # Codebase analysis documents
    ├── config.json             # Project configuration
    ├── phases/                 # Phase directories
    │   └── {NN}-{slug}/        # Individual phase
    │       ├── {NN}-{MM}-PLAN.md
    │       ├── {NN}-{MM}-SUMMARY.md
    │       ├── {NN}-CONTEXT.md
    │       ├── {NN}-RESEARCH.md
    │       └── {NN}-VERIFICATION.md
    ├── PROJECT.md
    ├── REQUIREMENTS.md
    ├── ROADMAP.md
    └── STATE.md
```

## Directory Purposes

**`.claude/`:**
- Purpose: Primary Claude Code configuration and framework
- Contains: Agents, commands, CLI tools, templates, workflows
- Key files: `gsd-tools.cjs`, agent definitions, command definitions
- Sync: Source of truth (other platforms mirror from here)

**`.claude/agents/`:**
- Purpose: AI agent role definitions with capabilities and constraints
- Contains: 12 agent definition files (Markdown with YAML frontmatter)
- Key files: `gsd-planner.md`, `gsd-executor.md`, `gsd-codebase-mapper.md`
- Pattern: Frontmatter with name, description, tools, skills; body with role XML

**`.claude/commands/gsd/`:**
- Purpose: Slash command definitions for `/gsd:*` commands
- Contains: 30+ command definitions
- Key files: `plan-phase.md`, `execute-phase.md`, `map-codebase.md`
- Pattern: Role definition, process steps, templates for output

**`.claude/get-shit-done/bin/`:**
- Purpose: CLI tooling for framework operations
- Contains: Main entry point and library modules
- Key files: `gsd-tools.cjs`, `lib/*.cjs`
- Language: Node.js CommonJS

**`.claude/get-shit-done/bin/lib/`:**
- Purpose: Core library functionality
- Contains: 11 modules handling specific concerns
- Key modules:
  - `core.cjs`: Utilities, constants, model profiles, phase helpers
  - `state.cjs`: STATE.md operations, progression engine
  - `phase.cjs`: Phase CRUD, lifecycle, decimal calculation
  - `roadmap.cjs`: ROADMAP.md parsing and updates
  - `verify.cjs`: Verification suite, health checks
  - `template.cjs`: Template selection and fill
  - `frontmatter.cjs`: YAML frontmatter parsing/serialization

**`.claude/get-shit-done/templates/`:**
- Purpose: Document generation templates
- Contains: 30+ templates for various document types
- Key templates:
  - `summary-standard.md`, `summary-minimal.md`, `summary-complex.md`
  - `plan.md`, `phase-prompt.md`
  - `codebase/*.md` (architecture.md, stack.md, etc.)

**`.claude/get-shit-done/workflows/`:**
- Purpose: Workflow process definitions
- Contains: 30+ workflow definitions
- Key workflows: `plan-phase.md`, `execute-phase.md`, `new-project.md`

**`.codex/skills/`:**
- Purpose: Codex-specific skill definitions
- Contains: GSD skill wrappers
- Pattern: Codex skill format (different from Claude agents)

**`.gemini/` and `.opencode/`:**
- Purpose: Platform-specific configurations
- Contains: Mirrored framework files, command mappings
- Note: Copies of `.claude/` content adapted for each platform

**`.planning/` (runtime):**
- Purpose: Project-specific planning state
- Created by: `/gsd:new-project` command
- Contains: Configuration, roadmap, state, phases
- Git: Should be committed (unless in `.gitignore`)

## Key File Locations

**Entry Points:**
- `.claude/get-shit-done/bin/gsd-tools.cjs`: Main CLI entry
- `.claude/commands/gsd/*.md`: Command definitions
- `.claude/agents/*.md`: Agent definitions

**Configuration:**
- `.claude/settings.json`: Claude Code permissions
- `.planning/config.json`: Project GSD configuration
- `~/.gsd/defaults.json`: User-level defaults (optional)

**Core Logic:**
- `.claude/get-shit-done/bin/lib/core.cjs`: Shared utilities
- `.claude/get-shit-done/bin/lib/state.cjs`: State management
- `.claude/get-shit-done/bin/lib/phase.cjs`: Phase operations

**Templates:**
- `.claude/get-shit-done/templates/codebase/*.md`: Codebase analysis
- `.claude/get-shit-done/templates/summary-*.md`: Summary formats

## Naming Conventions

**Files:**
- Agents: `gsd-{role}.md` (e.g., `gsd-planner.md`)
- Commands: `{command-name}.md` (e.g., `plan-phase.md`)
- Library modules: `{module}.cjs` (e.g., `core.cjs`)
- Plans: `{NN}-{MM}-PLAN.md` (e.g., `01-01-PLAN.md`)
- Summaries: `{NN}-{MM}-SUMMARY.md` (e.g., `01-01-SUMMARY.md`)
- Research: `{NN}-RESEARCH.md`
- Context: `{NN}-CONTEXT.md`

**Directories:**
- Phases: `{NN}-{slug}` (e.g., `01-setup`, `02-auth`)
  - NN: Zero-padded phase number (01, 02, ..., 99)
  - slug: Lowercase, hyphenated description
- Decimal phases: `{NN}.{M}-{slug}` (e.g., `05.1-hotfix`)

**Phase Numbering:**
- Integer phases: `01`, `02`, `03`...
- Letter suffixes: `12A`, `12B` (for parallel tracks)
- Decimal phases: `05.1`, `05.2` (inserted work)

## Where to Add New Code

**New Agent:**
- Create: `.claude/agents/gsd-{name}.md`
- Add to: `core.cjs:MODEL_PROFILES` if model selection needed
- Update: Platform mirrors in `.gemini/`, `.opencode/`, `.codex/`

**New Command:**
- Create: `.claude/commands/gsd/{command-name}.md`
- Add workflow: `.claude/get-shit-done/workflows/{command-name}.md`
- Add CLI support (if needed): Extend `gsd-tools.cjs`
- Update: Manifest in `gsd-file-manifest.json`

**New Library Module:**
- Create: `.claude/get-shit-done/bin/lib/{module}.cjs`
- Export: Add to module.exports
- Import: Add to `gsd-tools.cjs` router

**New Template:**
- Create: `.claude/get-shit-done/templates/{name}.md`
- Register: Add to `template.cjs` if selectable
- Update: Manifest hash in `gsd-file-manifest.json`

## Special Directories

**`.claude/get-shit-done/bin/lib/`:**
- Purpose: Core library modules
- Generated: No
- Committed: Yes

**`.planning/phases/`:**
- Purpose: Phase work directories
- Generated: Yes (by commands)
- Committed: Optional (user preference)

**`.planning/codebase/`:**
- Purpose: Codebase analysis outputs
- Generated: Yes (by `map-codebase`)
- Committed: Recommended

**`.claude/hooks/`:**
- Purpose: Runtime automation scripts
- Contains: JavaScript hooks for monitoring/updating
- Key files: `gsd-context-monitor.js`, `gsd-statusline.js`

---

*Structure analysis: 2026-03-13*
