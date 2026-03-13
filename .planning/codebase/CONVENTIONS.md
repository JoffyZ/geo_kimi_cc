# Coding Conventions

**Analysis Date:** 2026-03-13

## Language & Environment

**Primary Language:** JavaScript (Node.js)

**File Extensions:** `.cjs` for CommonJS modules, `.js` for scripts/hooks

**Runtime:** Node.js (no explicit version lock, uses modern features like `fetch`)

## Naming Patterns

**Files:**
- Library modules: `kebab-case.cjs` (e.g., `frontmatter.cjs`, `commands.cjs`)
- Hook scripts: `gsd-kebab-case.js` (e.g., `gsd-check-update.js`)
- Executable: `gsd-tools.cjs` (main CLI entry point)

**Functions:**
- Public command handlers: `cmdCamelCase` (e.g., `cmdConfigSet`, `cmdStateUpdate`)
- Internal helpers: `camelCase` (e.g., `extractFrontmatter`, `normalizePhaseName`)
- Error handling: `error()` function from `core.cjs` for consistent error output

**Variables:**
- Constants: `UPPER_SNAKE_CASE` (e.g., `MODEL_PROFILES`, `FRONTMATTER_SCHEMAS`)
- Configuration objects: `camelCase` (e.g., `config`, `options`)
- File paths: prefer `path` from Node.js, use `fullPath`, `configPath` patterns

## Code Style

**Formatting:**
- No explicit formatter configuration found
- Uses 2-space indentation consistently
- No semicolons at end of statements (optional style)
- Single quotes for strings

**Line Length:**
- Generally keeps lines under 100-120 characters
- Breaks long regex patterns and template strings across lines

## Import Organization

**Order:**
1. Node.js built-ins (`fs`, `path`, `child_process`, etc.)
2. Local library modules (from `./lib/`)
3. No external npm dependencies (self-contained)

**Example:**
```javascript
const fs = require('fs');
const path = require('path');
const { execSync } = require('child_process');
const { output, error } = require('./core.cjs');
const { extractFrontmatter } = require('./frontmatter.cjs');
```

**Pattern:** All library modules export an object with named functions:
```javascript
module.exports = {
  cmdStateLoad,
  cmdStateGet,
  cmdStatePatch,
  // ...
};
```

## Error Handling

**Strategy:** Centralized via `error()` function in `core.cjs`

**Pattern:**
```javascript
const { error } = require('./core.cjs');

function cmdSomething(cwd, arg) {
  if (!arg) {
    error('argument required for something');
  }
  // ... continue execution
}
```

**Silent Failures:**
- Use empty catch blocks for non-critical failures: `catch {}`
- Return `null` or safe defaults when file reads fail

## Output Patterns

**JSON vs Raw Output:**
- All commands support `--raw` flag for plain text output
- Default is JSON output via centralized `output()` function
- Large payloads (>50KB) written to temp file with `@file:` prefix

**Pattern:**
```javascript
const { output } = require('./core.cjs');

function cmdExample(cwd, raw) {
  const result = { success: true, data: 'value' };
  output(result, raw, 'value'); // JSON if !raw, else rawValue
}
```

## Documentation

**File Headers:**
```javascript
/**
 * ModuleName — Brief description
 */
```

**Section Dividers:**
```javascript
// ─── Section Name ────────────────────────────────────────────────────────────
```

## Function Design

**Parameter Style:**
- Prefer destructured options objects for >3 parameters
- `cwd` as first parameter for file system operations
- `raw` flag parameter for output formatting

**Return Pattern:**
- Commands call `output()` or `error()` and exit (no return values)
- Internal helpers return objects or primitives

## Configuration Patterns

**Config Loading:**
- Config file: `.planning/config.json`
- Default values in `loadConfig()` function in `core.cjs`
- Supports nested objects accessed via dot notation (e.g., `workflow.research`)

## Regex Patterns

**Escaping:**
- Use `escapeRegex()` helper from `core.cjs` for dynamic patterns:
```javascript
const escaped = escapeRegex(phaseNum);
const pattern = new RegExp(`Phase\s+${escaped}:`, 'i');
```

## File System Patterns

**Safe File Operations:**
```javascript
function safeReadFile(filePath) {
  try {
    return fs.readFileSync(filePath, 'utf-8');
  } catch {
    return null;
  }
}
```

**Path Normalization:**
- Use `toPosixPath()` helper for consistent forward slashes
- Always resolve relative paths against `cwd`

## Comments

**When to Comment:**
- Section headers with decorative dividers
- Complex regex explanations
- Why-not-what for non-obvious logic

**Avoid:**
- JSDoc (not used in this codebase)
- Inline comments for obvious code

---

*Convention analysis: 2026-03-13*
