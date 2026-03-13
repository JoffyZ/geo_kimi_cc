# Architecture

**Analysis Date:** 2026-03-13

## Pattern Overview

**Overall:** Multi-Platform AI Agent Framework with Plugin Architecture

The GSD (Get Shit Done) framework is a comprehensive project management system designed to orchestrate AI agents across multiple AI platforms (Claude, OpenCode, Gemini, Codex). It follows a **modular layered architecture** with clear separation between orchestration logic, agent definitions, and platform-specific configurations.

**Key Characteristics:**
- **Multi-platform support**: Runs on Claude, OpenCode, Gemini, and Codex with platform-specific configurations
- **Agent-based workflow**: Specialized agents for different tasks (planner, executor, researcher, verifier)
- **File-driven state**: Markdown-based state management with YAML frontmatter
- **CLI tooling**: Node.js CLI tool (`gsd-tools.cjs`) for operations
- **Template-driven**: Extensive template system for generating plans, summaries, and documentation

## Layers

**CLI Tools Layer:**
- Purpose: Low-level operations, file manipulation, state management
- Location: `.claude/get-shit-done/bin/`
- Contains: Node.js CommonJS modules for core operations
- Depends on: Node.js filesystem, git, process APIs
- Used by: Orchestrator commands, agent workflows

**Orchestration Layer:**
- Purpose: High-level workflow coordination, command routing
- Location: `.claude/commands/gsd/`
- Contains: Markdown-based command definitions (30+ commands)
- Depends on: CLI tools, agent definitions
- Used by: AI assistants (Claude, etc.)

**Agent Layer:**
- Purpose: Specialized AI agent definitions with roles and capabilities
- Location: `.claude/agents/`
- Contains: 12 agent definitions (gsd-planner, gsd-executor, gsd-codebase-mapper, etc.)
- Depends on: Tools API (Read, Write, Bash, Grep, Glob, etc.)
- Used by: Orchestrator commands

**Template Layer:**
- Purpose: Document generation and scaffolding
- Location: `.claude/get-shit-done/templates/`
- Contains: 30+ templates for plans, summaries, research, debugging
- Depends on: Frontmatter parsing
- Used by: Planner agent, init commands

**State Management Layer:**
- Purpose: Project state tracking, progress monitoring
- Location: `.planning/` (runtime), `.claude/get-shit-done/bin/lib/state.cjs` (logic)
- Contains: STATE.md, ROADMAP.md, phase directories
- Depends on: Frontmatter serialization, git
- Used by: All commands and agents

**Platform Abstraction Layer:**
- Purpose: Platform-specific configuration and command routing
- Location: `.claude/`, `.opencode/`, `.gemini/`, `.codex/`
- Contains: Settings, command mappings, agent definitions
- Depends on: AI platform APIs
- Used by: End users on each platform

## Data Flow

**Phase Planning Flow:**

1. User runs `/gsd:plan-phase <N>`
2. Orchestrator loads context from ROADMAP.md, STATE.md, existing research
3. If research enabled: Spawn `gsd-phase-researcher` agent
4. Spawn `gsd-planner` agent with research context
5. Planner creates PLAN.md files with tasks, dependencies, must-haves
6. Plan-checker validates plan structure
7. State updated with plan count

**Phase Execution Flow:**

1. User runs `/gsd:execute-phase <N>`
2. Orchestrator loads plans via `init execute-phase` command
3. For each plan: Spawn `gsd-executor` agent
4. Executor implements tasks from PLAN.md
5. Creates SUMMARY.md with completion details
6. Verifier validates work against must-haves
7. State advanced to next plan/phase

**Codebase Mapping Flow:**

1. User runs `/gsd:map-codebase`
2. Orchestrator determines focus area (tech, arch, quality, concerns)
3. Spawn `gsd-codebase-mapper` agent
4. Mapper explores codebase using Grep, Glob, Read tools
5. Writes analysis documents to `.planning/codebase/`
6. Returns confirmation to orchestrator

**State Management:**

- All state stored in markdown files with YAML frontmatter
- STATE.md tracks current phase, plan, progress, decisions, blockers
- ROADMAP.md contains phase definitions and progress table
- Phase directories contain PLAN.md and SUMMARY.md files
- Git integration for version control of planning documents

## Key Abstractions

**Phase:**
- Purpose: High-level work unit in a milestone
- Examples: `01-setup`, `02-authentication`, `03-api-design`
- Pattern: `{NN}-{slug}` directory naming
- Contains: PLAN.md files, SUMMARY.md files, CONTEXT.md, RESEARCH.md

**Plan:**
- Purpose: Executable task specification
- Location: `.planning/phases/{NN-slug}/{NN}-{MM}-PLAN.md`
- Pattern: Frontmatter with wave, depends_on, autonomous, must_haves
- Contains: Objective, context references (@file), XML task definitions

**Summary:**
- Purpose: Work completion record
- Location: `.planning/phases/{NN-slug}/{NN}-{MM}-SUMMARY.md`
- Pattern: Frontmatter with tech-stack, key-files, key-decisions
- Contains: Accomplishments, file changes, decisions, next-phase readiness

**Task (XML Format):**
```xml
<task type="code">
  <name>[Task name]</name>
  <files>[file paths]</files>
  <action>[What to do]</action>
  <verify>[How to verify]</verify>
  <done>[Definition of done]</done>
</task>
```

**Must-Haves:**
- Purpose: Verification criteria for plan success
- Location: PLAN.md frontmatter under `must_haves`
- Contains: truths (observable facts), artifacts (files to create), key_links (relationships)

## Entry Points

**CLI Entry Point:**
- Location: `.claude/get-shit-done/bin/gsd-tools.cjs`
- Triggers: Direct node execution, orchestrator subprocess calls
- Responsibilities: Command routing, argument parsing, library delegation

**Command Entry Points:**
- Location: `.claude/commands/gsd/*.md` (30 commands)
- Examples: `new-project.md`, `plan-phase.md`, `execute-phase.md`
- Pattern: Markdown with role definition, process steps, output format
- Responsibilities: Workflow orchestration, agent spawning, state updates

**Agent Entry Points:**
- Location: `.claude/agents/*.md` (12 agents)
- Examples: `gsd-planner.md`, `gsd-executor.md`, `gsd-codebase-mapper.md`
- Pattern: Frontmatter with name, description, tools, skills
- Responsibilities: Task execution, document generation, verification

**Library Entry Points:**
- Location: `.claude/get-shit-done/bin/lib/*.cjs` (11 modules)
- Examples: `core.cjs`, `state.cjs`, `phase.cjs`, `verify.cjs`
- Pattern: CommonJS modules with exported command functions
- Responsibilities: File operations, state management, validation

## Error Handling

**Strategy:** Graceful degradation with structured error output

**Patterns:**
- CLI tools return JSON with `{ error, reason }` structure
- Missing files handled with fallback defaults
- Git operations wrapped with exit code checking
- Validation produces structured `{ valid, errors[], warnings[] }` output
- Large JSON payloads written to temp files with `@file:` prefix

**Error Types:**
- File not found: Returns `{ exists: false }` instead of throwing
- Invalid config: Falls back to hardcoded defaults
- Git failures: Captures stdout/stderr, returns exit code
- Parse errors: Returns empty object/array instead of crashing

## Cross-Cutting Concerns

**Logging:** Console output via `process.stdout.write()` and `process.stderr.write()`
- Large outputs: Written to temp file, path returned with `@file:` prefix
- Raw mode: Plain text output for shell parsing
- JSON mode: Structured output for programmatic use

**Validation:** Multi-layer validation system
- Plan structure: Required frontmatter fields, task element presence
- Phase completeness: All plans have summaries
- References: @-references resolve to existing files
- Health checks: ROADMAP/STATE consistency, phase numbering

**Git Integration:** Automatic commit capability
- Configurable via `commit_docs` setting
- Respects `.gitignore` (skips if `.planning` ignored)
- Commit messages derived from workflow context
- Hash tracking in SUMMARY.md files

**Model Profile Resolution:** Tiered model selection
- Profiles: `quality` (opus), `balanced` (sonnet), `budget` (haiku)
- Agent-specific mappings in `core.cjs:MODEL_PROFILES`
- Per-agent overrides supported in config.json
- Inheritance: `opus` maps to `inherit` (use parent model)

---

*Architecture analysis: 2026-03-13*
