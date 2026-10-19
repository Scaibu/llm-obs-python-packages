---
description: "Rules and invariants for creating or refactoring server main.py files based on 07-main-file.md"
globs: "**/main.py, **/src/main.py"
always_on: true
---

# Server Main Entrypoint Specification (07-main-file.md)

Whenever generating, scaffolding, or refactoring a Python service `src/main.py` entrypoint, the implementation must adhere strictly to the 9-phase lifecycle architecture described in `policies/rules/folder-structure/07-main-file.md`:

## Core Architectural Invariants
1. **Component Contract**: Every long-lived component exposes: `start(ctx, deadline)`, `stop(deadline)`, and `health()`.
2. **Started-Components Stack**: Pushed in start order; shutdown pops in strict reverse topological order.
3. **Single Fatal Channel**: Background tasks report failure into one `asyncio.Queue` owned by `main`. The first fatal error triggers clean shutdown.
4. **Error Classification**:
   - `Fatal`: Hard dependency lost, listener died -> begin shutdown, nonzero exit.
   - `Degrade`: Soft dependency or optional component failed -> mark degraded, keep serving.
   - `Ignore`: Expected transient (e.g. single probe fail) -> count in metrics only.

## 9-Phase Lifecycle Structure
- **Phase 0 (Bootstrap)**: Parse CLI flags (`--config`, `--validate-only`), install OS signal handlers (`SIGTERM`, `SIGINT`), create root cancellation context (`asyncio.Event` / `asyncio.TaskGroup`), generate instance ID & bootstrap logger.
- **Phase 1 (Config)**: Multi-layer config hierarchy (defaults -> file -> env -> secrets), validate schema and cross-field CEL rules, freeze config into immutable snapshot (`ServerPolicySnapshot`), reconfigure logger with config fingerprint.
- **Phase 2 (Construct Components)**: Inert dependency injection; strictly zero I/O, no network, and no background task spawning in constructors.
- **Phase 3 (Validate Desired State)**: Descriptor schema verification, cycle detection in topological start order. If `--validate-only`, report and exit 0.
- **Phase 4 (Admin Surface Up)**: Bind internal admin listener on separate port for health (`/live`, `/ready`), diagnostics, and metrics before starting heavy services.
- **Phase 5 (Reconcile & Start Services)**: Start components in topological order with bounded deadlines; push started components to the shutdown stack.
- **Phase 6 (Data Plane)**: Bind public listener (e.g., Uvicorn ASGI with `AsgiPipelineAdapter`) only after dependencies and routes are verified. Flip readiness to `True`.
- **Phase 7 (Run)**: Block `main` exclusively on the root cancellation signal or the first fatal error from the queue.
- **Phase 8 (Shutdown)**: Flip readiness to `False`, propagation wait, stop accepting connections, drain in-flight requests with deadline, pop and stop components in reverse order, flush logs/metrics/telemetry, and exit with designated code.

## Strict Anti-Patterns (What `main.py` Must Never Contain)
- Zero business logic, request handling, or per-request code.
- Zero provider, vendor, model, or protocol-specific names.
- Zero retry, circuit breaker, or rate-limiting logic (handled by pipeline stages).
- Zero raw `os.environ` access outside Phase 1.
- Zero global mutable state accessible from outside modules.
