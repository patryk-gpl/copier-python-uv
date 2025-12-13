---
applyTo: "**"
---

# Ruff Documentation

Comprehensive reference for the `ruff` Python code linter and formatter.

## Documentation Links
- **Ruff Overview:** https://docs.astral.sh/ruff/
- **Tutorial:** https://docs.astral.sh/ruff/tutorial/
- **The Ruff Linter:** https://docs.astral.sh/ruff/linter/
- **The Ruff Formatter:** https://docs.astral.sh/ruff/formatter/
- **Configuration:** https://docs.astral.sh/ruff/configuration/
- **Rules Reference:** https://docs.astral.sh/ruff/rules/
- **Settings Reference:** https://docs.astral.sh/ruff/settings/
- **FAQ:** https://docs.astral.sh/ruff/faq/

## Key Concepts

### Linting and Formatting
- Use `ruff check` to lint code and identify issues
- Use `ruff check --fix` to automatically fix fixable violations
- Use `ruff format` to auto-format code (Black-compatible formatter)
- Both tools respect `pyproject.toml`, `ruff.toml`, and `.ruff.toml` configuration files

### Rule Selection Strategy
- **Prefer `lint.select` over `lint.extend-select`** to make rule sets explicit
- Add rules incrementally (e.g., `UP` for pyupgrade, `B` for flake8-bugbear, `I` for isort)
- Avoid `ALL` rule code as it implicitly enables new rules on version upgrades
- Target-version is auto-detected from `requires-python` in `pyproject.toml` if not specified

### Safe vs. Unsafe Fixes
- Safe fixes are enabled by default and preserve runtime behavior and code intent
- Unsafe fixes may change runtime behavior or remove comments (disabled by default)
- Enable unsafe fixes with `--unsafe-fixes` flag or set `unsafe-fixes = true` in config
- Use `lint.extend-safe-fixes` and `lint.extend-unsafe-fixes` to adjust fix safety per-rule

### Fix Control
- `lint.fixable`: Explicitly list rules eligible for `--fix` (replaces defaults)
- `lint.extend-fixable`: Add rules to the default fixable set
- `lint.unfixable`: Prevent fixes for specific rules even when `--fix` is enabled
- Configure per-file fixes using section-specific settings

### Error Suppression
- **Line-level:** `# noqa: CODE` or blanket `# noqa` to suppress violations
- **File-level:** `# ruff: noqa` or `# ruff: noqa: CODE` at top of file
- **Block-level:** `# ruff: disable[CODE]` ... `# ruff: enable[CODE]` (preview mode)
- Use `RUF100` rule to detect unused suppression comments
- Use `--add-noqa` flag to automatically add suppression comments to existing violations

### Formatter-Linter Compatibility
- Ruff formatter is a Black-compatible formatter designed to work with the linter
- Avoid enabling these rules when using the formatter: `W191`, `E111`, `E114`, `E117`, `D206`, `D300`, `Q000`, `Q001`, `Q002`, `Q003`, `COM812`, `COM819`, `ISC002`
- Formatter respects `# fmt: off`, `# fmt: on`, `# fmt: skip`, and YAPF pragmas
- Run imports before formatting: `ruff check --select I --fix && ruff format`

## Common Tasks

### Basic Linting
```bash
ruff check .                 # Lint current directory
ruff check --fix .           # Lint and fix all fixable violations
ruff check --fix --unsafe-fixes .  # Include unsafe fixes
ruff check --watch .         # Watch mode, re-run on file changes
ruff check path/to/code/     # Lint specific directory
ruff check path/to/file.py   # Lint specific file
```

### Formatting
```bash
ruff format .                # Format all files
ruff format --check .        # Check formatting without modifying
ruff format --diff .         # Show formatting diff without modifying
```

### Code Quality Checks
```bash
ruff check --statistics .    # Show count for each rule
ruff check --show-files .    # List files Ruff will analyze
ruff check --show-settings . # Display effective configuration
ruff check path/file.py --add-noqa  # Auto-add noqa directives
ruff check --select RUF100 --fix .  # Remove unused noqa comments
```

### Configuration
- Configuration files: `pyproject.toml`, `ruff.toml`, `.ruff.toml` (discovery order by directory)
- Use `[tool.ruff]` section for shared settings (line-length, target-version, exclude)
- Use `[tool.ruff.lint]` section for linter-specific settings (select, ignore, per-file-ignores)
- Use `[tool.ruff.format]` section for formatter settings (quote-style, indent-style, line-length)
- Omitting configuration uses sensible defaults compatible with Black

## Best Practices

### Rule Set Recommendations
Start with sensible defaults and expand incrementally:
```toml
[tool.ruff.lint]
select = [
    "E", "F",    # pycodestyle errors + Pyflakes
    "UP",        # pyupgrade (modern Python syntax)
    "B",         # flake8-bugbear (bug detection)
    "I",         # isort (import sorting)
]
```

### Per-File Configuration
Handle different requirements for different file types:
```toml
[tool.ruff.lint.per-file-ignores]
"__init__.py" = ["F401", "F403"]  # Allow unused imports in __init__
"tests/**/*.py" = ["D", "ANN"]    # No docstrings or type hints in tests
"**/*.pyi" = ["F401"]             # Allow unused imports in stubs
```

### Formatter Configuration
Align with Black conventions (Ruff's default):
```toml
[tool.ruff.format]
quote-style = "double"      # Black's default
indent-style = "space"      # Black's default
skip-magic-trailing-comma = false  # Black's default
line-ending = "auto"        # Detect platform line endings
```

## IMPORTANT Notes for Copilot

### Before Running Tests
1. **Ensure configuration is correct:** Verify `[tool.ruff]` section exists and is properly formatted
2. **Check rule conflicts:** If tests fail, check for disabled formatter-incompatible rules
3. **Run linting first:** Always run `ruff check` before `ruff format` to catch logic errors
4. **Test with fixes:** Use `ruff check --fix` to identify auto-fixable issues early

### Conflict Resolution
- If formatter warns about conflicting lint rules, disable them via `lint.ignore` or `lint.per-file-ignores`
- E501 (line-too-long) can coexist with formatter but may still report violations
- Always verify `ruff format` output has no warnings before deployment

### Performance
- Ruff is extremely fast (~100x faster than Flake8)
- Use `--no-cache` to disable caching if debugging, but keep caching enabled normally
- Integrate into pre-commit hooks and CI/CD pipelines for fast feedback

### Migration from Other Tools
- **From Flake8:** Ruff is a drop-in replacement; enable equivalent rules via `select`
- **From Black:** Ruff formatter is 99.9%+ compatible; test before full adoption
- **From isort:** Use `"I"` rule in `select` to enable import sorting via linter
