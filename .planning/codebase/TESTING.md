# Testing Patterns

**Analysis Date:** 2026-03-13

## Test Framework

**Status:** No formal test framework detected

**No test files found:**
- No `*.test.*` files
- No `*.spec.*` files
- No `jest.config.*` or `vitest.config.*`

## Testing Strategy

**Approach:** Manual/integration testing via CLI commands

**Verification Methods:**
1. **Built-in verification commands** in `verify.cjs`
2. **Health checks** via `validate health` command
3. **Consistency checks** via `validate consistency` command

## Verification Commands

The codebase includes self-verification through `gsd-tools.cjs`:

```bash
# Verify plan structure
node gsd-tools.cjs verify plan-structure <file>

# Verify phase completeness
node gsd-tools.cjs verify phase-completeness <phase>

# Validate references
node gsd-tools.cjs verify references <file>

# Verify git commits
node gsd-tools.cjs verify commits <hash1> [hash2] ...

# Verify artifacts from must_haves
node gsd-tools.cjs verify artifacts <plan-file>

# Verify key links
node gsd-tools.cjs verify key-links <plan-file>

# Full consistency check
node gsd-tools.cjs validate consistency

# Health check with optional repair
node gsd-tools.cjs validate health [--repair]
```

## Verification Patterns in Code

**File Existence Checks:**
```javascript
function safeReadFile(filePath) {
  try {
    return fs.readFileSync(filePath, 'utf-8');
  } catch {
    return null;
  }
}
```

**Path Verification:**
```javascript
function cmdVerifyPathExists(cwd, targetPath, raw) {
  const fullPath = path.isAbsolute(targetPath) ? targetPath : path.join(cwd, targetPath);
  try {
    const stats = fs.statSync(fullPath);
    const type = stats.isDirectory() ? 'directory' : stats.isFile() ? 'file' : 'other';
    output({ exists: true, type }, raw, 'true');
  } catch {
    output({ exists: false, type: null }, raw, 'false');
  }
}
```

**Structure Validation:**
```javascript
// From verify.cjs - cmdVerifyPlanStructure
const required = ['phase', 'plan', 'type', 'wave', 'depends_on', 'files_modified', 'autonomous', 'must_haves'];
for (const field of required) {
  if (fm[field] === undefined) errors.push(`Missing required frontmatter field: ${field}`);
}
```

## Mocking Strategy

**Not applicable** - No unit tests with mocking. Integration tests use:
- Temporary directories
- Actual file system operations
- Real git commands via `execGit()` helper

## Test Data

**No fixtures directory found**

Test data created ad-hoc:
- Temporary files in `/tmp/` for large JSON payloads
- Sample YAML frontmatter in template strings
- `.gitkeep` files for empty directory tracking

## Coverage

**No coverage tool configured**

Code quality ensured via:
1. Self-validating commands
2. Health check system with repair capability
3. Consistency validation between ROADMAP.md and filesystem

## Manual Testing Workflow

**Recommended testing approach:**

```bash
# 1. Initialize a test project
cd /tmp
cd test-project
node /path/to/gsd-tools.cjs init new-project

# 2. Create a phase
node /path/to/gsd-tools.cjs phase add "Test Phase"

# 3. Fill a template
node /path/to/gsd-tools.cjs template fill plan --phase 1 --plan 01

# 4. Verify structure
node /path/to/gsd-tools.cjs verify plan-structure .planning/phases/01-test-phase/01-01-PLAN.md

# 5. Check health
node /path/to/gsd-tools.cjs validate health

# 6. Check consistency
node /path/to/gsd-tools.cjs validate consistency
```

## Common Issues Detected

The verification system checks for:

1. **Missing required frontmatter fields** (plan schema)
2. **Plans without summaries** (orphaned work)
3. **Invalid file references** (`@path` resolution)
4. **Missing git commits** (hash validation)
5. **Phase numbering gaps**
6. **ROADMAP.md / filesystem inconsistency**
7. **Invalid JSON in config.json**

## Health Check Error Codes

From `verify.cjs`:

| Code | Severity | Issue |
|------|----------|-------|
| E001 | Error | `.planning/` directory not found |
| E002 | Error | `PROJECT.md` not found |
| E003 | Error | `ROADMAP.md` not found |
| E004 | Error | `STATE.md` not found |
| E005 | Error | `config.json` parse error |
| W001 | Warning | `PROJECT.md` missing required section |
| W002 | Warning | `STATE.md` references non-existent phase |
| W003 | Warning | `config.json` not found |
| W004 | Warning | Invalid `model_profile` value |
| W005 | Warning | Phase directory naming doesn't follow `NN-name` format |
| W006 | Warning | Phase in ROADMAP but no directory on disk |
| W007 | Warning | Phase on disk but not in ROADMAP |
| W008 | Warning | `workflow.nyquist_validation` absent |
| W009 | Warning | Validation Architecture in RESEARCH but no VALIDATION.md |

## Adding Tests

**To add testing to this codebase:**

1. **Install a test runner** (Jest or Vitest recommended)
2. **Create test directory** at `.claude/get-shit-done/tests/`
3. **Test each command** in isolation using temporary directories
4. **Mock file system** using `mock-fs` or memfs
5. **Verify output** matches expected JSON structure

**Example test structure:**
```javascript
describe('cmdConfigSet', () => {
  it('should update config value', () => {
    // Setup temp directory
    // Call command
    // Assert file content
  });
});
```

---

*Testing analysis: 2026-03-13*
