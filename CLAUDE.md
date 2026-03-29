# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This repository is a GitHub Actions workflow orchestrator for E2E testing a full-stack "Todo AI Workshop" application. It contains no application code—only a single workflow file that pulls and tests three external repositories together.

**Repositories under test:**
- Backend: `marcin119a/todo_ai_worshop` (Python/FastAPI + Alembic + SQLite)
- Frontend: `marcin119a/todo_workshop_frontend` (Node.js/Vite, port 5173)
- E2E Tests: `marcin119a/todo_selenium` (Python/Selenium + Chrome)

## Workflow: `.github/workflows/e2e-tests.yml`

**Triggers:** Push/PR to `main`/`master`, manual dispatch.

**Job pipeline:**
1. `test-backend` and `build-frontend` run in parallel
2. `e2e-tests` runs only after both pass (`needs: [test-backend, build-frontend]`)

**Job details:**
- `test-backend`: Python 3.11, runs `alembic upgrade head` then `pytest --tb=short -v`
- `build-frontend`: Node.js 20, runs `npm ci` then `npm run build`; uploads `frontend-dist` artifact (1-day retention)
- `e2e-tests`: Checks out all three repos, starts backend (port 8000) and frontend (port 5173) as background processes, waits 30s for each, then runs Selenium pytest; uploads JUnit XML and failure screenshots (7-day retention)

**Key environment detail:** `OPENAI_API_KEY` is set to an empty string in the E2E job, which causes the backend to use its mock AI service.

## Making Changes

This is a YAML-only repo. When editing the workflow:
- Validate the workflow structure with `gh workflow view` or push to a branch and check Actions tab
- Service startup order matters: backend must be healthy before frontend starts, both must be healthy before Selenium runs
- The 30-second `sleep` waits are hardcoded—increase them if services are slow to initialize
- Screenshots on failure are captured by the Selenium test repo's pytest fixtures, not this workflow
