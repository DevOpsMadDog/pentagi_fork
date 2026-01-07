# FixOps Compatibility (MPTEngine ↔ FixOps)

This repository contains MPTEngine (formerly PentAGI). FixOps already expects a small set of REST endpoints and an image that can be dropped into its stack.

## What FixOps expects (from CI + conventions)

- **Docker image override**
  - The repo’s GHCR publish workflow prints:
    - `export PENTAGI_IMAGE=ghcr.io/<org>/<repo>:latest`
    - `make up-pentagi`
  - **Implication**: FixOps is wired to an environment variable named `PENTAGI_IMAGE` and a make target named `up-pentagi`. We should keep that integration stable even as we rebrand to MPTEngine.

- **REST endpoints (baseline contract)**
  - CI checks for “Flows” and “Tasks” APIs. In MPTEngine these are implemented under `basePath: /api/v1`:
    - `GET /api/v1/flows/`
    - `POST /api/v1/flows/`
    - `GET /api/v1/flows/{flowID}`
    - `PUT /api/v1/flows/{flowID}` (actions like `stop`, `finish`, `input`)
    - `DELETE /api/v1/flows/{flowID}`
    - `GET /api/v1/flows/{flowID}/tasks/`
    - `GET /api/v1/flows/{flowID}/tasks/{taskID}`
    - `GET /api/v1/flows/{flowID}/tasks/{taskID}/graph`
    - `GET /api/v1/flows/{flowID}/tasks/{taskID}/subtasks/`
    - `GET /api/v1/flows/{flowID}/tasks/{taskID}/subtasks/{subtaskID}`

  - The full set of documented endpoints lives in:
    - `backend/pkg/server/docs/swagger.yaml`

## Current state in this repo

- **Routes exist**: `backend/pkg/server/router.go` wires `/flows` and `/flows/:flowID/tasks` under `/api/v1`.
- **Auth model**: the REST API uses session cookies (`gin-contrib/sessions`) and `/api/v1/auth/login` to create an authenticated session.

## Recommended compatibility guarantees

- **Keep `/api/v1` stable**: treat it as the FixOps contract surface.
- **Keep FixOps integration knobs stable**:
  - Continue printing `PENTAGI_IMAGE=...` in GH workflow output (even if the image is branded MPTEngine).
  - Continue exposing the flows/tasks REST endpoints and their response envelopes.
- **Versioned evolution**:
  - Add new features behind new endpoints or new fields, but avoid breaking response schemas consumed by FixOps.

