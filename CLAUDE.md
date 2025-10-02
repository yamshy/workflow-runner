# workflow-runner Development Guidelines

Auto-generated from all feature plans. Last updated: 2025-10-02

## Active Technologies
- Python 3.x (backend), TypeScript 5.x with Node LTS (frontend) + FastAPI, SvelteKit, SQLite, Vanilla Extract, zod, papaparse, @tanstack/svelte-query (001-build-a-barebones)
- SQLite for runs database, static file server for HTML reports (001-build-a-barebones)
- Tool management: mise, uv (Python), pnpm workspaces (frontend)
- Linting/Formatting: ruff (Python), Biome (JS/TS/JSON)
- Type checking: pyright (Python), svelte-check, tsc (TypeScript)

## Project Structure
```
backend/
├── src/
│   ├── api/
│   ├── models/
│   └── services/
└── tests/

frontend/
├── src/
│   ├── routes/
│   └── lib/
└── tests/
```

## Commands
```bash
# Development
mise install          # Install all tools
mise run lint         # Run ruff + biome
mise run typecheck    # Run pyright + svelte-check + tsc
mise run test         # Run all tests

# Backend
cd backend
uv pip install -r requirements.txt
uv run uvicorn src.api.main:app --reload

# Frontend
cd frontend
pnpm install
pnpm dev
```

## Code Style
- Python: ruff for linting/formatting, strict pyright typing
- TypeScript: Biome for linting/formatting, strict mode
- CSS: Vanilla Extract for tokens, Svelte scoped styles, Lightning CSS build
- No utility CSS frameworks (Tailwind prohibited by constitution)

## Recent Changes
- 001-build-a-barebones: Initial setup with constitutional-compliant styling (Vanilla Extract + Svelte styles)

<!-- MANUAL ADDITIONS START -->
<!-- MANUAL ADDITIONS END -->
