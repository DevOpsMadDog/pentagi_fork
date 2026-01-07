# MPTEngine remodel: “world-class micro pentesting engine”

MPTEngine’s direction is to be a **micro** pentesting engine: small, composable, automation-first, and easy to embed into other platforms (like FixOps) via stable APIs.

## Design goals (what “world-class” means here)

- **Micro by default**: minimal services for core execution (API + worker + DB), optional add-ons for graph/observability/search.
- **Deterministic runs**: reproducible plans, tool invocations, artifacts, and evidence capture.
- **Safe execution**: strict sandboxing, network policy, secrets isolation, and least-privilege Docker control.
- **Composable**: clear “engine core” with pluggable tools, providers, and policy hooks.
- **Integration-first**: a stable REST/GraphQL contract and clean event stream for external orchestrators (FixOps, CI, ticketing).
- **Measurable quality**: built-in evaluation harness (already present: `ctester`, `ftester`, `etester`) + regression baselines.

## Proposed architecture refinements (incremental)

- **Core engine boundary**
  - Treat “Flow/Task/Subtask + Tool Execution + Artifact Store” as the engine core.
  - Keep `/api/v1` as the stable integration surface; use `/api/v2` only for breaking changes.

- **Plugin contract**
  - Formalize a tool/provider plugin interface with:
    - capability declaration (inputs/outputs, permissions, network requirements)
    - policy gates (allow/deny/require-approval)
    - standard artifact schema (logs, screenshots, reports, raw tool output)

- **Micro deployment profile**
  - Add a minimal compose profile that runs only:
    - backend API
    - postgres/pgvector
    - (optional) a single worker queue implementation
  - Keep existing “full stack” compose files as add-ons.

- **Integration hardening for FixOps**
  - Document and freeze the FixOps-used endpoints (`/flows`, `/flows/{id}/tasks`, etc.).
  - Add contract tests (schema snapshots) to CI instead of grep-only checks.

## Execution plan (practical sequencing)

1. **Rebrand safely**
   - User-visible name becomes MPTEngine (UI, Swagger title/description, docs).
   - Keep internal identifiers and integration knobs that FixOps depends on (`/api/v1`, `PENTAGI_IMAGE`, etc.).

2. **Lock the FixOps contract**
   - Publish a compatibility doc and a small “contract test” suite for required endpoints.

3. **Micro compose profile**
   - Add a minimal deployment option and document the “micro” vs “full” stacks.

4. **Plugin + policy**
   - Introduce explicit tool permissions and policy gates (filesystem, network egress, docker privileges).

5. **Quality loop**
   - Treat `ctester/ftester/etester` outputs as performance and correctness baselines.

