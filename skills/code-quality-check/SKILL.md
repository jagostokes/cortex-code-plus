---
name: code-quality-check
description: Run comprehensive code quality checks including security scans, linting, type checking, tests, and build verification before completing tasks or creating PRs.
allowed-tools: "*"
---

# Code Quality & Pre-Flight Check

**When to invoke:** Use before completing coding tasks, creating PRs, or when asked to "check code quality", "run tests", "verify code", or "pre-flight check".

## Role
You are a code quality assurance expert ensuring code meets quality standards before completion or deployment.

## Workflow

### 0. Determine Fix Mode

Before running quality checks, determine if auto-fix mode should be enabled:

1. **Check for config file** `.cortex/code-quality-config.json`:
   - If exists and `auto_fix.enabled: true` → Enable AUTO_FIX mode
   - If not exists → Check user intent

2. **Check user prompt for fix keywords**:
   - User says: "fix", "apply fixes", "auto-fix", "fix automatically" → Enable AUTO_FIX mode
   - Otherwise → REPORT_ONLY mode (default)

3. **Set mode variable** for later steps:
   - `MODE = AUTO_FIX` or `MODE = REPORT_ONLY`

**Default behavior:** REPORT_ONLY (safe, conservative approach per Anthropic guidelines)

### 1. Project Detection & Setup
- Detect project type by checking for:
  - `package.json` (Node/JS/TS)
  - `pyproject.toml`, `setup.py`, `requirements.txt` (Python)
  - `Cargo.toml` (Rust)
  - `go.mod` (Go)
  - `pom.xml`, `build.gradle` (Java)
- Check README or AGENTS.md for custom quality commands
- Detect Python environment (uv, poetry, venv, conda) with `find_custom_python_environment`

### 2. Security Scan
**CRITICAL - Run first before any other checks:**
- Scan for exposed secrets/credentials:
  - `.env` files in git staging
  - API keys, tokens, passwords in code
  - Database credentials
  - Private keys
  - Snowflake account identifiers
- Check gitignore for sensitive files
- **BLOCK completion if secrets found**

### 3. Linting
Run project-specific linters:
- **JavaScript/TypeScript:** `npm run lint` or `eslint .`
- **Python:** `ruff check .` or `pylint` or `flake8`
- **Rust:** `cargo clippy`
- **Go:** `golangci-lint run`
- **SQL:** Check for Snowflake SQL best practices
- Parse output and identify actionable issues

### 4. Type Checking
- **TypeScript:** `npm run typecheck` or `tsc --noEmit`
- **Python:** `mypy .` or `pyright`
- **Rust:** `cargo check`
- **Go:** Built into compilation
- Report type errors with file:line references

### 5. Testing
- **Run existing tests** (don't write new ones unless explicitly asked):
  - JavaScript: `npm test` or `yarn test`
  - Python: `pytest` or `python -m pytest`
  - Rust: `cargo test`
  - Go: `go test ./...`
- Report failures with clear context
- Check test coverage if available

### 6. Build Verification
- **Attempt build** if applicable:
  - `npm run build`
  - `cargo build`
  - `go build`
- Ensure no build-time errors
- Check output artifacts exist

### 7. Dependency Check
- Look for unused dependencies
- Check for outdated critical packages
- Verify lockfiles are up to date (`package-lock.json`, `Cargo.lock`, etc.)

### 8. Git Status Check
- Verify no unintended files staged (node_modules, .env, etc.)
- Check commit message follows conventions (if applicable)
- Ensure GPG signing is working (if configured)

### 9. Metrics Tracking

**Track quality metrics over time using JSON format** (per Anthropic best practices):

1. **Get git context**:
   ```bash
   git rev-parse HEAD  # Current commit SHA
   git log -1 --format='%H %ai %an'  # Commit info
   ```

2. **Create/update** `.cortex/quality-history.json`:
   ```json
   {
     "project_name": "<project_name>",
     "history": [
       {
         "timestamp": "2025-12-12T17:00:00Z",
         "commit_sha": "abc123def456",
         "commit_author": "John Doe",
         "results": {
           "security": {"status": "PASS", "issues_count": 0},
           "linting": {"status": "WARNINGS", "errors": 0, "warnings": 5},
           "type_check": {"status": "PASS", "errors": 0},
           "tests": {"status": "PASS", "passing": 45, "failing": 0, "coverage": 0.85},
           "build": {"status": "PASS"},
           "dependencies": {"status": "WARNINGS", "outdated": 3}
         },
         "verdict": "READY"
       }
     ]
   }
   ```

3. **Append new entry** to history array
4. **Keep last 30 runs** (trim older entries)

**Trend Analysis:**
- Compare to previous run (if exists)
- Calculate: issues increasing/decreasing
- Note: quality improving/degrading

### 10. Auto-Fix Execution (Conditional)

**Only execute if MODE = AUTO_FIX:**

1. **List all fixable issues**:
   ```
   Found fixable issues:
   - 12 linting errors (eslint --fix)
   - 8 formatting issues (prettier --write)
   - 3 Python style violations (ruff format)
   ```

2. **Ask for confirmation**:
   ```
   Apply these fixes automatically? [y/n]
   - Linting: 12 fixes
   - Formatting: 8 fixes
   
   Note: Dependency updates will NOT be applied (too risky)
   ```

3. **If user confirms (y)**:
   - Apply linting fixes:
     ```bash
     # JavaScript/TypeScript
     eslint --fix .
     prettier --write .
     
     # Python
     ruff check --fix .
     ruff format .
     
     # Rust
     cargo clippy --fix --allow-dirty
     cargo fmt
     ```
   - Re-run quality checks to verify fixes
   - Update metrics with post-fix results

4. **If user declines (n)**:
   - Skip auto-fix
   - Provide fix commands in report

5. **Never auto-fix**:
   - ❌ Dependency updates (suggest only)
   - ❌ Security issues (manual review required)
   - ❌ Test failures (need investigation)
   - ❌ Type errors (may require logic changes)

**Fix suggestions format:**
```
To fix these issues manually:
  eslint --fix src/app.js
  ruff format src/utils.py
  npm update lodash@latest
```

## Output Format

```
✅ CODE QUALITY REPORT

PROJECT: <type> (<language>)
ENVIRONMENT: <detected_env>
MODE: <REPORT_ONLY | AUTO_FIX>
COMMIT: <sha> (<author>, <date>)

🔒 SECURITY SCAN: <PASS/FAIL>
   <findings>

📋 LINTING: <PASS/FAIL/WARNINGS>
   <summary of issues>
   Fix: eslint --fix src/app.js

🔤 TYPE CHECK: <PASS/FAIL>
   <type errors>

🧪 TESTS: <PASS/FAIL> (<X/Y passing>)
   <failures>
   Coverage: <percentage>%

🔨 BUILD: <PASS/FAIL/SKIPPED>
   <build output>

📦 DEPENDENCIES: <OK/WARNINGS>
   <issues>
   Outdated: lodash@4.17.20 → 4.17.21

📝 GIT STATUS: <CLEAN/ISSUES>
   <staging area status>

📈 TRENDS: (vs last run)
   Linting errors: 12 → 8 (↓33% improvement)
   Test coverage: 82% → 85% (↑3% improvement)
   Quality trend: IMPROVING ✅

VERDICT: ✅ READY / ⚠️  NEEDS ATTENTION / ❌ BLOCKED

Metrics saved to: .cortex/quality-history.json
```

## Quality Gates

**MUST PASS to complete task:**
- ❌ No exposed secrets
- ❌ No critical linting errors
- ❌ No type errors
- ❌ All tests passing

**SHOULD WARN but not block:**
- ⚠️  Linting warnings
- ⚠️  Missing tests
- ⚠️  Outdated dependencies

## Best Practices

- Run checks in parallel where possible (multiple bash calls in one message)
- Always read full error output (use `tail -100` if truncated)
- Provide specific fix suggestions with file:line references
- Don't just report errors - explain what needs fixing
- Remember to activate Python environments correctly
- Check for project-specific scripts in `package.json` or `Makefile`
- Respect project conventions (some projects allow warnings)

## Common Patterns

**Finding quality commands:**
```bash
# Check package.json
jq '.scripts | keys[]' package.json

# Check Python project
cat pyproject.toml | grep -A 5 "\[tool"

# Check for Makefile targets
make -qp | grep "^[a-z].*:" | cut -d: -f1
```

**Python environment activation:**
```bash
# UV project
uv run pytest

# Poetry project  
poetry run pytest

# venv
source venv/bin/activate && pytest
```
