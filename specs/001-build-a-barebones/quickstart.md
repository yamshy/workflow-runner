# Quickstart Guide

**Feature**: RNA-seq Workflow Runner
**Version**: 0.1.0
**Date**: 2025-10-02

## Prerequisites

- mise (for tool management)
- 10GB free disk space for test data

## Installation

```bash
# Clone repository
git clone <repository-url>
cd workflow-runner

# Install all tooling via mise (Node LTS, Python 3.x, uv, pnpm, etc.)
mise install

# Install Python dependencies with uv
cd backend && uv pip install -r requirements.txt

# Install frontend dependencies with pnpm
cd ../frontend && pnpm install
```

## Configuration

1. Set up environment variables:
```bash
# backend/.env
DATABASE_URL=sqlite:///./runs.db
SECRET_KEY=your-secret-key-here
PASSPHRASE=test123
WORKFLOW_VERSION=1.0.0
STATIC_DIR=./static
```

2. Initialize database:
```bash
cd backend
python -m src.cli.init_db
```

## Running the Application

### Start Mock Server (for frontend development)
```bash
# Terminal 1: Run Prism mock server (installed via mise)
prism mock specs/001-build-a-barebones/contracts/openapi.yaml
```

### Start Backend
```bash
# Terminal 2: Run FastAPI server with uv
cd backend
uv run uvicorn src.api.main:app --reload --port 8000
```

### Start Frontend
```bash
# Terminal 3: Run SvelteKit dev server
cd frontend
pnpm dev
```

## Quick Test Workflow

### 1. Prepare Test Data

Create a valid test CSV file `test_samples.csv`:
```csv
sample_id,group,read1_uri,read2_uri
S001,control,/data/S001_R1.fastq.gz,/data/S001_R2.fastq.gz
S002,control,/data/S002_R1.fastq.gz,/data/S002_R2.fastq.gz
S003,treatment,/data/S003_R1.fastq.gz,/data/S003_R2.fastq.gz
```

Create test FASTQ files (empty for testing):
```bash
mkdir -p /data
touch /data/S001_R1.fastq.gz /data/S001_R2.fastq.gz
touch /data/S002_R1.fastq.gz /data/S002_R2.fastq.gz
touch /data/S003_R1.fastq.gz /data/S003_R2.fastq.gz
```

### 2. Login
1. Navigate to http://localhost:5173
2. Enter passphrase: `test123`
3. Select role: "Admin" or "Runner"
4. Click "Login"

### 3. Upload and Validate
1. Navigate to "New Run" (/runs/new)
2. Click "Choose File" and select `test_samples.csv`
3. Preview appears immediately
4. Click "Validate"
5. Validation results show (should be all green checkmarks)

### 4. Start Run
1. With valid CSV, click "Run Workflow"
2. Redirected to /runs/{run_id}
3. Status shows "Queued" then "Running"
4. Real-time updates every second

### 5. View Report
1. Wait ~15 seconds for stub workflow to complete
2. Status changes to "Completed"
3. "View Report" button appears
4. Click to open HTML report in new tab
5. Provenance box shows workflow details

### 6. Re-run
1. On completed run page, click "Re-run Exact"
2. New run created with same parameters
3. Redirected to new run page

## Testing Invalid Data

### Missing Headers CSV
```csv
id,group,read1_uri,read2_uri
S001,control,/data/S001_R1.fastq.gz,/data/S001_R2.fastq.gz
```
**Expected**: Error "Missing required header: sample_id"

### Duplicate IDs CSV
```csv
sample_id,group,read1_uri,read2_uri
S001,control,/data/S001_R1.fastq.gz,/data/S001_R2.fastq.gz
S001,treatment,/data/S002_R1.fastq.gz,/data/S002_R2.fastq.gz
```
**Expected**: Error "Duplicate sample_id: S001" on row 2

### Invalid File Extension CSV
```csv
sample_id,group,read1_uri,read2_uri
S001,control,/data/S001_R1.txt,/data/S001_R2.txt
```
**Expected**: Error "Invalid file format" for both URI columns

### Too Many Rows CSV
Create CSV with 13+ rows
**Expected**: Error "Maximum 12 samples allowed"

## Admin Functions

### Cancel Run (Admin only)
1. Start a run as Admin
2. While status is "Running", click "Cancel"
3. Run status changes to "Failed"
4. Error message: "Cancelled by admin"

### Delete Run (Admin only)
1. View any completed/failed run as Admin
2. Click "Delete Run"
3. Confirm in dialog
4. Run removed from list

## Running Tests

### All Tests (via mise)
```bash
# Run all tests with single command
mise run test

# Or run specific test types:
mise run test:unit     # Backend unit tests
mise run test:contract # API contract tests
mise run test:e2e      # Frontend E2E tests
```

### Type Checking
```bash
# Check all types
mise run typecheck

# This runs:
# - pyright (Python backend)
# - svelte-check (Svelte components)
# - tsc --noEmit (TypeScript)
```

### Linting & Formatting
```bash
# Lint and format all code
mise run lint

# This runs:
# - ruff (Python linting + formatting)
# - biome (JS/TS/JSON linting + formatting)
```

## Deployment

### Build Frontend
```bash
cd frontend
pnpm build
```

### Build Backend
```bash
cd backend
python -m build
```

### Docker Compose (recommended)
```bash
docker-compose up -d
```

## Monitoring

- Application logs: `backend/logs/app.log`
- Database: `backend/runs.db`
- Reports: `backend/static/reports/`
- Sessions: Check SQLite sessions table

## Troubleshooting

### "File not found" validation errors
- Ensure test FASTQ files exist at specified paths
- Check file permissions (must be readable)

### Login fails with correct passphrase
- Check PASSPHRASE environment variable
- Verify backend is running on port 8000

### Status not updating
- Check browser console for polling errors
- Verify backend /api/runs/{id} endpoint responding

### Report not loading
- Check static file serving configuration
- Verify report file exists in static/reports/{run_id}/

## Performance Validation

Run performance tests to verify constitutional requirements:

```bash
# Test validation speed (should be <2s for 200 rows)
time curl -X POST -F "file=@large_test.csv" http://localhost:8000/api/validate

# Test first paint (should be <2s)
lighthouse http://localhost:5173 --view

# Verify polling interval (should be 1s)
# Open browser DevTools Network tab on /runs/{id} page

# Verify strict typing
mise run typecheck  # Should pass with no errors

# Verify no any types
grep -r "any" frontend/src --include="*.ts" --include="*.svelte"  # Should return nothing
```

## Clean Up

Remove test data and reset:
```bash
# Delete database
rm backend/runs.db

# Clear reports
rm -rf backend/static/reports/*

# Remove test files
rm -rf /data/*.fastq.gz
```