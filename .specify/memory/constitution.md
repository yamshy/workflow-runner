<!--
Sync Impact Report
------------------
Version: 1.0.0 → 1.1.0 (MINOR bump - new principle added)
Ratified: 2025-10-02
Last Amended: 2025-10-02

Principles:
  1. Code Quality & Tooling
  2. Type Safety Everywhere
  3. Linting & Formatting
  4. Testing & Contracts
  5. UX Consistency
  6. Performance
  7. Styling System
  8. Reproducibility & Security
  9. Scope Control
  10. Commit Standards (NEW - Conventional Commits enforcement)

Template Sync Status:
  ✅ plan-template.md - Updated Constitution Check section and version reference
  ✅ spec-template.md - No changes required (commit standards are development workflow, not spec)
  ✅ tasks-template.md - No changes required (commit standards are development workflow, not task structure)

Follow-up TODOs: None
-->

# Workflow Runner Constitution

## Core Principles

### I. Code Quality & Tooling
All tooling is managed by mise with a committed `.mise.toml` configuration file. This file MUST pin exact versions for Node, Python, uv, pnpm, pyright, ruff, and Biome. No tool installations outside of mise are permitted for project development dependencies.

**Rationale**: Reproducible builds and consistent developer environments require locked tool versions. mise provides a single source of truth for all tooling.

### II. Type Safety Everywhere
Frontend code MUST use strict TypeScript with svelte-check enabled. Backend code MUST use pyright for type checking. The use of `any` types in TypeScript or `# type: ignore` in Python is prohibited unless accompanied by an inline comment justifying the exception.

**Rationale**: Type safety catches errors at build time, improves code comprehension, and enables reliable refactoring.

### III. Linting & Formatting
Python code MUST pass ruff (lint + format). TypeScript, JavaScript, and JSON files MUST pass Biome checks. A single command `mise run lint` MUST validate the entire codebase and MUST pass before any merge to the main branch.

**Rationale**: Automated formatting eliminates style debates. Linting catches common bugs and enforces best practices consistently.

### IV. Testing & Contracts
Unit tests are REQUIRED for CSV validation logic. Contract tests MUST be enforced against the OpenAPI specification. A Playwright smoke test MUST validate the complete upload → report happy path. All tests MUST pass before merge. CI MUST run on every pull request.

**Rationale**: Tests are executable documentation and the only reliable way to prevent regressions. Contract testing ensures frontend-backend compatibility.

### V. UX Consistency
The user interface follows a single linear path: Upload → Validate → Run → Report. Error messages MUST include row and column numbers for data errors. All forms MUST be accessible with proper labels, focus management, and full keyboard navigation support.

**Rationale**: A predictable flow reduces cognitive load. Specific error messages enable self-service debugging. Accessibility is a requirement, not a feature.

### VI. Performance
Initial page load (first paint) MUST occur in under 2 seconds on broadband connections. CSV validation for 200 rows MUST complete in 2 seconds or less. Status polling MUST default to a 10-second interval with exponential backoff to reduce server load.

**Rationale**: Performance is a feature. Fast feedback loops improve user satisfaction and reduce perceived friction.

### VII. Styling System
Utility CSS frameworks (Tailwind, UnoCSS, etc.) are prohibited. Styling MUST use Vanilla Extract for design tokens and theming. Component-specific styles MUST use Svelte scoped styles. The build process MUST use Lightning CSS for optimized output.

**Rationale**: Vanilla Extract provides type-safe styles with zero runtime cost. Scoped styles prevent cascade conflicts. This stack avoids utility class bloat while maintaining performance.

### VIII. Reproducibility & Security
Workflow definitions MUST specify pinned versions. Container images MUST use digests (sha256), never tags like `:latest`. Each run MUST produce an immutable manifest. A "Re-run exact" feature MUST reproduce the same parameters and digests byte-for-byte. Secrets MUST NOT be committed to the repository.

**Rationale**: Scientific reproducibility requires deterministic execution. Digest pinning prevents supply-chain attacks. Immutable manifests enable auditing and rollback.

### IX. Scope Control
Version 0 (v0) supports exactly one workflow with one preset configuration. Any request for additional workflows, presets, or major features REQUIRES a new feature specification document before implementation.

**Rationale**: Feature creep is the enemy of shipping. Forcing specification documents creates an intentional decision point and prevents scope drift.

### X. Commit Standards
All commit messages MUST follow the Conventional Commits specification. Format: `type(scope): description` where type is one of: `feat`, `fix`, `docs`, `style`, `refactor`, `test`, `chore`, `perf`, `ci`, `build`, or `revert`. Breaking changes MUST include `BREAKING CHANGE:` in the commit body or append `!` after the type/scope. The description MUST be lowercase and not end with a period.

**Rationale**: Conventional Commits enable automated changelog generation, semantic versioning, and clear communication of intent. Standardized messages improve git history readability and enable tooling to parse changes programmatically.

## Enforcement

### Development Workflow
1. All code changes MUST pass `mise run lint` before commit
2. All tests MUST pass locally before opening a pull request
3. CI MUST validate linting, type checking, and tests on every PR
4. No PR may be merged with failing CI checks
5. Commit messages MUST follow Conventional Commits format

### Quality Gates
Each pull request MUST verify:
- [ ] Type checking passes (pyright + svelte-check)
- [ ] Linting passes (ruff + Biome)
- [ ] All tests pass (unit + contract + Playwright)
- [ ] No TypeScript `any` or Python `# type: ignore` without justification
- [ ] Performance budgets met (first paint < 2s, validation < 2s)
- [ ] OpenAPI contract tests updated for API changes
- [ ] Commit messages follow Conventional Commits format

### Complexity Review
Any deviation from these principles MUST be documented in the feature's `plan.md` under "Complexity Tracking" with:
1. Which principle is violated
2. Why the violation is necessary
3. What simpler alternative was considered and why it was rejected

Unjustified violations MUST be rejected at code review.

## Governance

This constitution supersedes all other development practices and conventions. Amendments require:
1. A written proposal with rationale
2. Approval from project maintainers
3. Updates to all dependent templates and documentation
4. Version increment following semantic versioning rules

All code reviews MUST verify constitutional compliance. Complexity MUST be justified; unjustified complexity MUST be simplified before merge.

**Version**: 1.1.0 | **Ratified**: 2025-10-02 | **Last Amended**: 2025-10-02

---

## Verification Checklist

Use this checklist during `/specify` and `/plan` to verify adherence:

**Code Quality**
- [ ] .mise.toml exists with pinned versions for Node, Python, uv, pnpm, pyright, ruff, Biome
- [ ] No tool versions specified outside mise

**Type Safety**
- [ ] TypeScript strict mode enabled
- [ ] svelte-check configured for frontend
- [ ] pyright configured for backend
- [ ] No unjustified `any` or `# type: ignore`

**Linting & Formatting**
- [ ] `mise run lint` command exists
- [ ] ruff configured for Python (lint + format)
- [ ] Biome configured for TS/JS/JSON

**Testing**
- [ ] CSV validation has unit tests
- [ ] OpenAPI contract tests exist
- [ ] Playwright smoke test covers Upload → Report path
- [ ] CI configured to run tests on every PR

**UX**
- [ ] Flow follows Upload → Validate → Run → Report
- [ ] Error messages include row + column for data errors
- [ ] Forms have labels, focus management, keyboard navigation

**Performance**
- [ ] First paint measured and < 2s budgeted
- [ ] Validation measured and < 2s for 200 rows
- [ ] Status polling at 10s with exponential backoff

**Styling**
- [ ] No Tailwind/UnoCSS/utility frameworks
- [ ] Vanilla Extract used for tokens/themes
- [ ] Svelte scoped styles used for components
- [ ] Lightning CSS in build pipeline

**Reproducibility**
- [ ] Workflow version pinned
- [ ] Container images use sha256 digests
- [ ] Run manifest is immutable
- [ ] "Re-run exact" feature specified
- [ ] No secrets in repository, no `:latest` tags

**Scope**
- [ ] Only one workflow + one preset in v0
- [ ] New features require new spec document

**Commit Standards**
- [ ] Commit messages follow Conventional Commits format (type(scope): description)
- [ ] Valid types: feat, fix, docs, style, refactor, test, chore, perf, ci, build, revert
- [ ] Breaking changes marked with `!` or `BREAKING CHANGE:` in body
- [ ] Descriptions are lowercase and do not end with period
