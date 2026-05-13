# AGENTS.md

## Cursor Cloud specific instructions

### Overview

Threat Designer is an AI-powered threat modeling application. The repo is a monorepo with:
- **Frontend**: React 18 + Vite SPA (root `package.json`)
- **Backend Lambda functions**: Python 3.12 in `backend/app/`, `backend/authorizer/`, `backend/stream_processor/`
- **AI agents**: Python 3.12 FastAPI services in `backend/threat_designer/`, `backend/sentry/`
- **CLI tool**: `cli/` (installable via `pip install ./cli`)
- **MCP server**: `mcp-server/`

The backend services are tightly coupled to AWS (DynamoDB, S3, Cognito, Bedrock AgentCore). There is no docker-compose or local mock infrastructure. End-to-end backend testing requires a deployed AWS environment.

### Running the frontend

```bash
npm run dev          # Vite dev server on port 5173
npm run build        # Production build
npm run lint         # ESLint (pre-existing errors in repo, exit code 1 is expected)
```

### Running backend tests

Backend tests are pure unit tests with mocked AWS services. They require `AWS_DEFAULT_REGION` to be set (any valid region works, e.g. `us-east-1`) because some modules create boto3 clients at import time.

```bash
export AWS_DEFAULT_REGION=us-east-1
python3 -m pytest test/ -v
```

Dependencies needed beyond `requirements.txt` files:
- `backend/app/requirements.txt` (boto3)
- `backend/dependencies/requirements-test.txt` (pytest, pytest-cov, pytest-mock, hypothesis)
- `backend/dependencies/requirements-authorizer.txt` (PyJWT, cryptography)
- `aws-lambda-powertools` (imported by backend modules but not in any requirements file)
- `pydantic` (imported by `attack_tree_service.py` but not in the app requirements file)

### Gotchas

- Hypothesis-based property tests (`test_fetch_all_pagination.py`, `test_pagination_infrastructure.py`) can consume significant memory. If the VM is memory-constrained, run test files individually or exclude property tests with `-k "not Property"`.
- The 5 failing tests in `test_threat_designer_route.py` and `test_threat_designer_service.py` are pre-existing in the repo (not environment-related).
- The ESLint config (`eslint.config.js`) reports ~251 problems (86 errors, 165 warnings) — these are pre-existing in the repo.
- `~/.local/bin` must be on `PATH` for `pytest` and other pip-installed scripts.
