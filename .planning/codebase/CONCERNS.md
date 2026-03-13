# Codebase Concerns

**Analysis Date:** 2026-03-13

## Tech Debt

### Error Handling Patterns

**Issue:** Silent/Empty Catch Blocks
- **Files:** `verify.cjs` (10 occurrences), `init.cjs` (13 occurrences), `phase.cjs` (6 occurrences), `core.cjs` (4 occurrences), `state.cjs` (3 occurrences), `milestone.cjs` (3 occurrences), `commands.cjs` (4 occurrences), `config.cjs` (1 occurrence), `frontmatter.cjs` (2 occurrences), `roadmap.cjs` (1 occurrence)
- **Pattern:** `} catch {}` blocks that swallow errors silently
- **Impact:** Silent failures make debugging difficult; errors may propagate or cause undefined behavior downstream
- **Fix approach:** Add error logging or re-throw strategy; at minimum log to stderr

**Specific examples:**
- `.claude/get-shit-done/bin/lib/verify.cjs` lines 429, 491, 511, 587, 629, 640, 659, 677, 699
- `.claude/get-shit-done/bin/lib/init.cjs` lines 156, 180, 269, 310, 439, 474, 476, 523, 525, 534, 573, 649, 657
- `.claude/get-shit-done/bin/lib/phase.cjs` lines 402, 471, 539, 599, 812, 829

### String-Based Error Handling

**Issue:** Error messages returned as strings instead of structured errors
- **Files:** `core.cjs`, `commands.cjs`, `verify.cjs`
- **Impact:** Cannot programmatically distinguish error types; brittle error handling
- **Fix approach:** Use error codes or custom Error classes

### Regex Complexity

**Issue:** Heavy reliance on regex for parsing (44 RegExp constructions across 7 files)
- **Files:** `verify.cjs`, `state.cjs`, `roadmap.cjs`, `phase.cjs`, `milestone.cjs`, `frontmatter.cjs`, `core.cjs`
- **Impact:** Regex-based parsing is fragile; markdown/YAML frontmatter parsing may break on edge cases
- **Fix approach:** Consider using a proper YAML parser for frontmatter; add more test coverage for edge cases

### File Size and Complexity

**Large Files:**
- `.claude/get-shit-done/bin/lib/phase.cjs` (901 lines) - Phase CRUD operations
- `.claude/get-shit-done/bin/lib/verify.cjs` (820 lines) - Verification suite
- `.claude/get-shit-done/bin/lib/state.cjs` (721 lines) - State management
- `.claude/get-shit-done/bin/lib/init.cjs` (710 lines) - Workflow initialization

**Impact:** Large files make code harder to maintain and test; single responsibility principle violations
**Fix approach:** Consider breaking into smaller modules by feature

## Known Bugs

### Phase Numbering Edge Cases

**Issue:** Decimal phase comparison may have edge cases with letter suffixes
- **File:** `.claude/get-shit-done/bin/lib/core.cjs` lines 186-212
- **Pattern:** Complex comparison logic for phase numbers with decimals and letters
- **Risk:** Sorting and comparison may behave unexpectedly with certain phase formats

### Frontmatter Array Parsing

**Issue:** YAML frontmatter parsing is custom-built and may not handle all YAML features
- **File:** `.claude/get-shit-done/bin/lib/frontmatter.cjs` lines 11-84
- **Pattern:** Custom parser for YAML frontmatter instead of using a library
- **Risk:** Edge cases in YAML syntax may not be handled correctly

### Git Ignore Detection

**Issue:** Character filtering in gitignore check may be overly aggressive
- **File:** `.claude/get-shit-done/bin/lib/core.cjs` line 140
- **Pattern:** `targetPath.replace(/[^a-zA-Z0-9._\-//]/g, '')`
- **Risk:** Paths with special characters may be incorrectly processed

## Security Considerations

### Command Injection Risk

**Issue:** Shell command construction through string concatenation
- **Files:** `core.cjs` (line 156), `init.cjs` (line 174)
- **Pattern:** `execSync('git ' + escaped.join(' '))` and similar patterns
- **Current mitigation:** Arguments are escaped with basic regex testing
- **Risk:** Moderate - custom escaping may not cover all edge cases
- **Recommendations:** Use array-based exec APIs or more robust escaping

### Path Traversal

**Issue:** File path resolution without validation
- **Files:** Throughout the codebase
- **Pattern:** `path.join(cwd, userProvidedPath)` without validation
- **Risk:** Paths like `../../../etc/passwd` could be constructed
- **Current mitigation:** Limited by Claude Code sandboxing
- **Recommendations:** Add path validation to ensure resolved paths stay within project directory

### API Key Exposure

**Issue:** Brave Search API key read from file
- **File:** `.claude/get-shit-done/bin/lib/config.cjs` lines 30-32
- **Pattern:** Reads `~/.gsd/brave_api_key` if env var not set
- **Risk:** File permissions not checked; key file may be world-readable
- **Recommendations:** Check file permissions (should be 0600); warn if too permissive

## Performance Bottlenecks

### Synchronous File Operations

**Issue:** Heavy use of synchronous file operations
- **Pattern:** `fs.readFileSync`, `fs.writeFileSync`, `fs.readdirSync` throughout
- **Files:** All modules in `bin/lib/`
- **Impact:** Blocking operations may cause latency in large projects
- **Note:** Acceptable for CLI tool but may limit scalability

### Repeated File Reads

**Issue:** Same files read multiple times in single operations
- **File:** `.claude/get-shit-done/bin/lib/verify.cjs` - health validation reads config multiple times
- **Impact:** Unnecessary I/O overhead
- **Fix approach:** Cache file contents within operation scope

### Large JSON Payload Handling

**Issue:** Large JSON payloads written to temp files
- **File:** `.claude/get-shit-done/bin/lib/core.cjs` lines 42-48
- **Pattern:** When JSON > 50KB, writes to temp file
- **Risk:** Temp files may not be cleaned up on crash
- **Fix approach:** Add cleanup handler or use streaming

## Fragile Areas

### Frontmatter Schema Validation

**Issue:** Schema validation uses hardcoded field lists
- **File:** `.claude/get-shit-done/bin/lib/frontmatter.cjs` lines 227-231
- **Pattern:** `FRONTMATTER_SCHEMAS` object with required fields
- **Risk:** Adding new fields requires code changes
- **Safe modification:** Add new schemas incrementally; maintain backward compatibility

### Phase Renumbering Logic

**Issue:** Complex phase renumbering on remove
- **File:** `.claude/get-shit-done/bin/lib/phase.cjs` lines 488-699
- **Pattern:** Recursive renaming of directories and files
- **Risk:** Data loss if operation interrupted mid-way
- **Safe modification:** Test thoroughly with backup; consider atomic operations

### State.md Field Extraction

**Issue:** Regex-based field extraction from markdown
- **File:** `.claude/get-shit-done/bin/lib/state.cjs` lines 12-20, 184-194
- **Pattern:** Multiple regex patterns to find field values
- **Risk:** May fail with unexpected formatting
- **Test coverage:** Add tests for various STATE.md formats

### Model Resolution

**Issue:** Hardcoded model profile mappings
- **File:** `.claude/get-shit-done/bin/lib/core.cjs` lines 18-31
- **Pattern:** `MODEL_PROFILES` object maps agent types to models
- **Risk:** New agent types not in map default to 'sonnet'
- **Fix approach:** Make profile lookup more flexible or validate agent types

## Scaling Limits

### Large Payload Handling

**Current:** JSON output limited to 50KB before temp file fallback
- **File:** `.claude/get-shit-done/bin/lib/core.cjs` lines 42-48
- **Limit:** Large history digests may trigger this
- **Scaling path:** Streaming JSON or pagination

### Phase Count Scalability

**Current:** Phase operations O(n) where n = phase count
- **Files:** `phase.cjs`, `roadmap.cjs`
- **Limit:** Operations scanning all phases may slow with hundreds of phases
- **Scaling path:** Index or caching of phase metadata

### File Count in Phases

**Current:** No pagination for file listing
- **Files:** `phase.cjs`, `verify.cjs`
- **Limit:** Phases with many plans may cause memory issues
- **Scaling path:** Implement streaming or pagination

## Dependencies at Risk

### Node.js Built-ins Only

**Current:** No external runtime dependencies
- **Risk:** Low - uses only Node.js built-in modules
- **Impact:** No npm audit concerns, but limited functionality
- **Trade-off:** Intentional design choice for portability

### Child Process Spawn Patterns

**Issue:** Spawn used for background update check
- **File:** `.claude/hooks/gsd-check-update.js` lines 44-81
- **Pattern:** Inline JavaScript passed to `node -e`
- **Risk:** May break with Node.js version changes
- **Fix approach:** Move inline script to separate file

## Missing Critical Features

### No Automated Tests

**Issue:** No test suite detected
- **Impact:** Changes may introduce regressions
- **Risk:** High - complex logic without test coverage
- **Priority:** Critical

### No Type Safety

**Issue:** JavaScript without TypeScript or JSDoc types
- **Files:** Entire `bin/` directory
- **Impact:** Type errors only caught at runtime
- **Risk:** Moderate - complex data structures in frontmatter parsing

### Configuration Schema Validation

**Issue:** Config loading accepts any JSON shape
- **File:** `.claude/get-shit-done/bin/lib/core.cjs` lines 68-130
- **Impact:** Invalid config values may cause subtle bugs
- **Fix approach:** Add JSON schema validation

## Test Coverage Gaps

### Regex Parsing

**Untested:** Complex regex patterns for phase parsing, frontmatter extraction
- **Files:** `core.cjs`, `frontmatter.cjs`, `phase.cjs`
- **Risk:** Edge cases in phase numbering, markdown format variations

### Error Paths

**Untested:** Empty catch blocks, error conditions
- **Files:** All modules
- **Risk:** Silent failures go unnoticed

### Cross-Platform Behavior

**Untested:** Path handling on Windows
- **Files:** All modules using `path.join`
- **Risk:** Windows path separator issues

---

*Concerns audit: 2026-03-13*
