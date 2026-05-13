# Go Backend Playbook

Compressed from `samber/cc-skills-golang` into one downstream reference. This file is documentation, not a standalone skill: it intentionally avoids a nested `SKILL.md` so installing the CTO skill does not add dozens of Go-specific skill entries to the agent context.

Use this when the generated project includes Go services, CLIs, workers, GraphQL/gRPC edges, or Go-backed infrastructure. Load it once, then rely on the current codebase, official Go docs, and project tests for details.

## Default Stance

- Prefer the standard library first; add dependencies only when they remove real complexity.
- Keep packages small and boring. Avoid `utils`, `common`, and framework-shaped layers that hide ownership.
- Push boundaries into `internal/` unless the package is intentionally public.
- Pass `context.Context` as the first argument for request-scoped work.
- Return errors instead of panicking. Use `errors.Is`, `errors.As`, and `%w` wrapping.
- Define interfaces where they are consumed. Return concrete types unless abstraction is already needed.
- Keep goroutine ownership explicit. Every goroutine needs cancellation, bounded lifetime, and a shutdown path.
- Make production behavior observable from the first implementation: structured logs, metrics, traces, and health endpoints.

## Project Shape

Use the layout as an architecture contract, not decoration:

```text
cmd/<app>/          entrypoints
internal/<domain>/  private application packages
api/                OpenAPI, protobuf, GraphQL schema files
migrations/         database migrations
configs/            sample config only, no secrets
pkg/                public libraries only when external reuse is real
```

For monorepos, keep each deployable unit with its own `cmd/`, `internal/`, `go.mod` if module boundaries are real, and clear ownership in README or architecture docs. Use `go.work` for local development across modules.

## Code Shape

- Run `gofmt` / `go fmt` and `go vet`; prefer `golangci-lint` for team policy.
- Name packages by what they provide, not who uses them. Avoid `helpers`, `models`, and `types` catchalls.
- Use MixedCaps. Do not use ALL_CAPS constants or Java-style getters.
- Design zero values to be usable when practical.
- Prefer simple constructors. Add functional options only when there are several optional knobs.
- Use generics for reusable type-safe operations; use `any` only at true dynamic boundaries.
- Keep comments focused on exported API contracts or non-obvious why, not restating code.

## Context And Lifecycle

- Never store `context.Context` on structs.
- Do not pass nil contexts. Use `context.Background()` at process roots and derive below.
- Put request deadlines at the edge; pass cancellation through database, HTTP, gRPC, and worker calls.
- Use context values only for request metadata such as trace IDs, auth claims, or logging fields.
- On shutdown, stop accepting work, cancel background loops, wait for in-flight work, and close resources.

## Errors

- Add context when crossing boundaries: `fmt.Errorf("load account %s: %w", id, err)`.
- Use sentinel errors for package-level classification and custom error types when callers need structured fields.
- Handle each error exactly once: return it, wrap it, translate it, or log it at the boundary.
- Use `panic` only for impossible programmer errors or startup invariants.
- In HTTP/gRPC adapters, translate domain errors to stable status codes and user-safe messages.

## Concurrency

- Prefer sequential code until concurrency is needed.
- Use `errgroup` for fan-out with cancellation.
- Use worker pools for bounded parallelism.
- Use `singleflight` for duplicate suppression.
- Use mutexes for protecting shared memory; use channels for ownership transfer and coordination.
- The sender closes channels. Receivers range until closed.
- Run `go test -race ./...` for code touching shared state.

## Persistence

- Prefer explicit SQL through `database/sql`, `pgx`, `sqlx`, or `sqlc`. Avoid ORMs for complex domains.
- Use parameterized queries only; never concatenate user input into SQL.
- Treat `sql.ErrNoRows` as a domain decision, not an infrastructure failure.
- Model nullable columns deliberately with pointers, `sql.Null*`, or package-specific nullable types.
- Keep transactions short, context-aware, and owned by the service method that defines consistency.
- Configure connection pools intentionally: max open, max idle, lifetime, idle time.
- Use additive migrations first, backfill separately, then cut over and remove old paths later.

## APIs And Edges

- HTTP handlers should parse, validate, call application code, and translate responses. Keep business logic out of handlers.
- For REST, document request/response shapes with OpenAPI when external consumers exist.
- For gRPC, keep protobuf packages stable, use interceptors for auth/logging/metrics, and test with `bufconn`.
- For GraphQL, keep resolvers thin and batch data loading to avoid N+1 queries.
- For CLIs, use Cobra when command trees are non-trivial; keep stdout for machine output and stderr for diagnostics.
- For config, prefer env vars and typed config structs. Viper is useful for layered config but isolate it from business packages.

## Testing

- Use table-driven tests with named cases and `t.Run`.
- Test observable behavior, not implementation details.
- Keep unit tests fast and deterministic; put integration tests behind build tags such as `//go:build integration`.
- Use `httptest` for HTTP edges and fakes for interfaces owned by the consumer.
- Test cancellation, timeouts, partial failures, transaction rollback, and duplicate requests.
- Use `testify/require` for setup preconditions and `assert` for multiple independent checks.
- Add benchmarks only when there is a performance question; compare with `benchstat`, not a single run.

## Observability And Operations

- Use `log/slog` for structured logs. Include request ID, trace ID, actor, operation, and stable error fields.
- Use OpenTelemetry for traces across service boundaries.
- Use Prometheus-compatible metrics for request rate, latency, errors, queue depth, goroutines, GC, and DB pools.
- Expose liveness and readiness separately.
- Keep `pprof` available behind an internal-only or authenticated surface.
- Prefer graceful shutdown with signal handling and time-bounded cleanup.
- Use GoReleaser or equivalent release automation only when releases are frequent enough to justify it.

## Security

- Validate and canonicalize all untrusted input at boundaries.
- Protect file paths with clean/join checks and explicit base directories.
- Avoid shelling out; if unavoidable, pass arguments without shell interpolation.
- Store secrets in environment or a secret manager, never committed config.
- Use `crypto/rand` for tokens and standard crypto primitives only.
- Set HTTP timeouts on servers and clients.
- Use TLS/mTLS where transport trust matters.
- Keep dependencies updated and run `govulncheck` in CI for Go services.

## Dependency Choices

- `google/wire`: preferred when compile-time dependency injection is worth the setup.
- Manual constructors: default for most services.
- `uber/dig` or `uber/fx`: use mainly when already established in the codebase.
- `samber/lo`: acceptable for clear collection transforms; avoid turning simple loops into clever chains.
- `samber/oops`: useful for structured, user-safe errors when the service needs error codes and attributes.
- `samber/hot`: consider for in-memory caching with explicit eviction and metrics needs.
- `samber/ro`: reserve for real streaming/reactive pipelines, not ordinary request flows.
- `spf13/cobra`: good for multi-command CLIs.
- `spf13/viper`: useful for layered config, but keep it at the edge.
- `stretchr/testify`: useful test helpers; do not let mocks replace behavior tests.

## Troubleshooting

Follow evidence before changing code:

1. Read the exact error or symptom.
2. Reproduce with the narrowest command.
3. Form one hypothesis at a time.
4. Add logging, tests, or probes that distinguish causes.
5. Fix the root cause, then remove temporary instrumentation.

Useful commands:

```bash
go test ./...
go test -race ./...
go test -run TestName ./path
go test -bench=. -benchmem ./path
go test -cpuprofile cpu.out -memprofile mem.out ./path
go tool pprof cpu.out
go tool trace trace.out
go build -gcflags="-m" ./...
govulncheck ./...
```

## Handoff Checklist

- [ ] Package boundaries are clear and enforced by `internal/` where useful.
- [ ] Context, shutdown, and goroutine lifetimes are explicit.
- [ ] Errors are wrapped, classified, and translated at edges.
- [ ] Database access is parameterized, transaction boundaries are clear, and migrations are safe.
- [ ] Tests cover success, failure, cancellation, and integration boundaries.
- [ ] Race detection is run for concurrent changes.
- [ ] Logs, metrics, traces, health checks, and profiling surfaces exist for production services.
- [ ] Secrets, input validation, dependency scanning, and HTTP timeouts are covered.
