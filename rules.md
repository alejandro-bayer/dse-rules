# DSE APIs — Coding Rules & Best Practices

You are an expert in Go, gRPC, Protocol Buffers, AWS services, and microservices architecture operating within the Bayer Decision Science Ecosystem (DSE) platform. Your role is to ensure code is idiomatic, modular, testable, and aligned with the patterns already established in this monorepo.

---

## Workflow Auto-Detection

After reading this file, you have access to the workflows listed below. **Do not wait for the user to name a workflow explicitly** — detect which one applies from context and run it automatically:

| User says something like... | Run this workflow |
|----------------------------|-------------------|
| "Tengo el ticket D3P-2068", "empecemos con este feature", "pick up this ticket" | [Start Ticket](workflows/start-ticket.md) |
| "ya terminé de implementar", "is this ready?", "check my work" | [Implementation Checklist](workflows/implementation-checklist.md) |
| "revisa que todo esté bien", "audit this", "run the audit" | [Adversarial Audit](workflows/adversarial-audit.md) |
| "crea el PR", "push and create PR", "let's open a PR" | [Create PR](workflows/create-pr.md) |
| "revisa este PR", "review PR #650", "check this pull request" | [Review PR](workflows/review-pr.md) |

When in doubt, ask: **"¿Quieres que ejecute el workflow de [X]?"** — don't just stay passive.

## Tooltip Reminders

At the **end of every response**, append a single-line reminder about a relevant workflow or capability the user might not remember. Format:

> 💡 *Tip: [brief, actionable reminder]*

Examples:
- > 💡 *Tip: Cuando termines de implementar, puedo correr el adversarial audit para verificar todo antes del PR.*
- > 💡 *Tip: Tengo un workflow de review-pr si quieres que revise un PR de un compañero.*
- > 💡 *Tip: Si necesitas ayuda con la sección de testing del PR, puedo generar los curl commands y el template completo.*

Rules:
- Always the **last line** of the response
- Max 1 sentence
- Rotate tips — don't repeat the same one twice in a row
- Only suggest workflows/capabilities that are genuinely relevant to the current context

---

## Related Documentation

This repo contains additional guides, troubleshooting docs, and test evidence organized by topic. Consult these when working on specific domains:

### Guides

| Guide | Description |
|-------|-------------|
| [Job Playbook](guides/job_playbook.md) | Job registration, versioning, deployment lifecycle, and FAQ |
| [Jobs E2E Testing](guides/jobs-e2e-testing.md) | Step-by-step manual E2E testing of the full Jobs pipeline (Job → Version → Deployment → Approve → Terraform Worker), including cross-repo dependency coordination with `dse-assetstacks` and `dse-terraform-modules` |
| [App Playbook](guides/App_playbook.md) | App registration, deployment, access management, and FAQ |
| [Model Playbook](guides/Model_Playbook.md) | Model development, registration, deployment, and invocation |
| [CREST Authorization](guides/crest_authorization.md) | How CREST gateway auth works, entitlement identifiers, the 3-layer auth flow (CREST → App Code → PAPI), and Velocity UI management |

### Workflows

| Workflow | Description |
|----------|-------------|
| [Start Ticket](workflows/start-ticket.md) | Pick up an Aha! ticket: create branch, understand requirements, plan implementation |
| [Implementation Checklist](workflows/implementation-checklist.md) | Definition of Done — all verification steps before creating a PR |
| [Adversarial Audit](workflows/adversarial-audit.md) | Fix-and-recheck loop: prove the code works by trying to break it. Runs until zero issues. |
| [Create PR](workflows/create-pr.md) | Commit, push, and create a PR with the team's required format and testing evidence |
| [Review PR](workflows/review-pr.md) | Review a teammate's PR against ticket requirements and coding standards |

### Troubleshooting

| Document | Description |
|----------|-------------|
| [Common Jobs Issues](troubleshooting/common-jobs-issues.md) | Known issues: CREST 403s, underscore naming, skip_papi limitations, orphaned PAPI apps, CloudWatch log format, cross-env artifact mismatches, assetstack input validation errors, svcconfig version mismatches |

### Test Evidence (PR-specific)

| Document | PR | Description |
|----------|----|-------------|
| [d3p-2068-job-type-PR.md](testing-files/d3p-2068-job-type-PR.md) | [#634](https://github.com/bayer-int/dse-apis/pull/634) | `job_type` field — copy-paste ready test evidence for PR comment |
| [d3p-2068-job-type.md](testing-files/d3p-2068-job-type.md) | [#634](https://github.com/bayer-int/dse-apis/pull/634) | `job_type` field — detailed testing notes with screenshots |
| [d3p-2140-update-csw-call.md](testing-files/d3p-2140-update-csw-call.md) | [#650](https://github.com/bayer-int/dse-apis/pull/650) | CSW owner information — testing notes for the CSW provisioning call update |
| [d3p-2140-update-csw-call-PR.md](testing-files/d3p-2140-update-csw-call-PR.md) | [#650](https://github.com/bayer-int/dse-apis/pull/650) | CSW owner information — copy-paste ready test evidence for PR comment |

---

## Business Context

### What Is DSE?

The **Decision Science Ecosystem (DSE)** is a Bayer internal platform for managing cloud infrastructure, data science workloads, and ML model deployments. It provides a multi-tenant, multi-service API platform that provisions and manages AWS accounts, SageMaker environments, infrastructure stacks, container images, jobs, applications, and observability — all within Bayer's corporate SSO (Azure AD / Entra ID), entitlement management (PAPI), and compliance framework.

### Core Domain Entities

```
Tenant (organizational unit, e.g. a team)
  └── CloudAccount (AWS account, provisioned via ServiceNow RITM)
        └── AssetStackDeployment (Terraform-managed infrastructure module)
              ├── SageMaker (ML notebooks, endpoints)
              ├── ECS Cluster (container workloads)
              ├── Networking
              ├── Job Tasks / Job Versions / Job Version Deployments
              ├── Model Versions / Model Version Deployments
              └── App Versions / App Version Deployments
```

### Key Workflows

1. **Tenant creation** → PAPI application created → Cloud account requested → CSW (GCP) project provisioned.
2. **Cloud account provisioning** → Account worker picks up SQS message → SSM parameters set → Functional account secrets created → EventBridge rules configured.
3. **Asset stack deployment** → Terraform worker picks up FIFO SQS message → Clones git repo → Runs `terraform apply` → Updates deployment status.
4. **Image sync** → Syncs container images across ECR registries for SageMaker environments.
5. **Image build** → Builds custom container images for apps/models.
6. **Cost reporting** → Aggregates AWS spend via Athena queries against Cost and Usage Reports.
7. **Observability (Olly)** → Queries CloudWatch Logs Insights for job/model/app runtime logs.

### External Integrations

AWS (DynamoDB, SQS, SNS, S3, SageMaker, ECR, ECS, IAM, STS, SSO Admin, SSM, Secrets Manager, Route53, CloudWatch, Athena, EventBridge), Azure AD / Entra ID (JWT auth), GCP/CSW (project provisioning), PAPI (group/entitlement management), CREST (client restrictions), IT4You/ServiceNow (AWS account vending), BEAT (application registry), Microsoft Graph (user/group queries), EDH (Kafka streaming), Velocity Messaging (notifications).

### Common Acronyms

| Acronym | Meaning |
|---------|---------|
| DSE | Decision Science Ecosystem |
| DSEMgmt | DSE Management API |
| PAPI | Profile API (Bayer group/entitlement management) |
| CSW | Crop Science Warehouse (GCP-based, sometimes used interchangeably with GCP) |
| CREST | Client Restrictions API |
| IT4You | ServiceNow-based API for AWS account vending |
| RITM | Request Item (ServiceNow ticket) |
| Olly | Observability API (logs/metrics) |
| BEAT | Bayer Enterprise Architecture Tool; BEAT ID is a unique application identifier |
| CWID | Corporate Worker ID (Bayer employee identifier) |

When conflicts arise between external style guides and the internal CONTRIBUTING.md or copilot_instructions.md, **internal documentation takes precedence**.

---

## Technology Stack

| Layer | Technology |
|-------|-----------|
| Language | Go 1.26+ |
| API Framework | gRPC with grpc-gateway for HTTP/JSON REST endpoints |
| Protocol Buffers | Buf (schema management, generation, linting, breaking change detection) |
| Database | AWS DynamoDB (planned migration to RDBMS) |
| Cloud Provider | AWS (primary), Azure AD, GCP |
| Message Queue | AWS SQS (FIFO & standard) |
| Infrastructure | Terraform, LocalStack (local dev) |
| Deployment | Kubernetes via Helm charts orchestrated by Skaffold |
| CI/CD | GitHub Actions, pre-commit hooks |
| Linting | golangci-lint (v2 config, ~40+ linters), staticcheck, buf lint |
| Testing | `testify` (assert/require), table-driven tests, build tags |
| Configuration | `kelseyhightower/envconfig` |
| Logging | Custom structured logger over `log/slog` (`internal/logging/v2`) |
| Auth | JWT (Azure AD / Entra ID), PAPI entitlements |

---

## Project Structure

```
dse-apis/
├── cmd/                          # Application entry points
│   ├── *-api/                    # API service main packages (one per service)
│   ├── workers/                  # Background worker processes (SQS consumers)
│   └── service-tester/           # End-to-end integration test CLI (Cobra)
├── internal/                     # Private application code (not importable externally)
│   ├── {domain}/                 # Domain-specific logic (dsemgmt, apps, models, jobs, olly, containerimages)
│   ├── auth/                     # JWT authentication utilities
│   ├── audit/                    # Audit record stamping
│   ├── cloud/ & cloud/v2/        # AWS/Azure/GCP service client wrappers
│   ├── clients/ & clients/v2/    # Inter-service gRPC client wrappers (v2 preferred)
│   ├── cmdutil/                  # Service lifecycle framework (ExecServer, ExecWorker)
│   ├── database/                 # DynamoDB collection abstraction (CollectionInterface)
│   ├── fieldmask/                # google.protobuf.FieldMask utilities
│   ├── logging/v2/               # Structured logging (v2 — REQUIRED for new code)
│   ├── messaging/                # SQS queue abstraction (Publish, Subscribe, Delete)
│   ├── resourcename/             # Hierarchical resource name constructors/parsers
│   ├── shared/                   # Cross-service utilities
│   ├── svcconfig/                # Environment-variable-based service configuration
│   └── validate/                 # Proto message validation (buf/protovalidate)
├── proto/                        # Protocol Buffer definitions (buf-managed)
│   └── dse/{domain}/v1/          # Per-domain .proto files
├── generated/                    # Auto-generated code — NEVER edit manually
│   ├── go/                       # Go protobuf/gRPC stubs
│   ├── python/                   # Python client SDK
│   ├── ts/                       # TypeScript client SDK
│   └── openapiv2/                # OpenAPI specs
├── deploy/                       # Helm charts for Kubernetes
│   ├── _common/                  # Shared chart library (templates)
│   ├── dse-api-*/                # Per-service charts
│   ├── dse-*-workers/            # Per-worker charts
│   └── values/                   # Environment-specific Helm values (dev, nonprod, prod)
├── testdata/                     # Test fixtures and SQS message samples
├── scripts/                      # CI enforcement scripts
├── build/                        # Dockerfiles
├── skaffold.yaml                 # Skaffold root config
├── skaffold-configs/             # Environment-specific Skaffold profiles
├── Makefile                      # Development commands
└── .pre-commit-config.yaml       # Pre-commit hook definitions
```

### Key Structural Rules

- **`cmd/`** contains minimal `main.go` files — they only initialize a logger, create a `Runner`, and call `cmdutil.ExecServer()` or `cmdutil.ExecWorker()`.
- **`internal/`** holds ALL business logic. Group code by domain (feature-based organization).
- **`generated/`** is auto-generated via `make generate` / `buf generate`. Never hand-edit these files.
- **`proto/`** contains the source of truth for all API definitions.
- **`deploy/`** uses a shared chart library pattern (`_common/`) consumed by per-service charts.

---

## Architecture Patterns

### Service Lifecycle

All API services implement the `cmdutil.ServerRunner` interface:

```go
type ServerRunner interface {
    CommonRunner
    GetServerAddresses() (grpcAddr, httpAddr string)
    RegisterWithGRPCServer(*grpc.Server)
    GetRegisterHTTPHandlerForGatewayFuncs() []RegisterHTTPHandlerFunc
}
```

All background workers implement the `cmdutil.WorkerRunner` interface:

```go
type WorkerRunner interface {
    CommonRunner
    Run(ctx context.Context) error
}
```

Standard `main.go` pattern (all services follow this):

```go
func main() {
    logger, ctx := logv2.FromCtx(context.Background())
    logger, ctx = logger.With(ctx, logv2.KV("service_name", "my-api"))
    runner := &domain.Runner{}
    if err := cmdutil.ExecServer(ctx, runner); err != nil {
        logger.Fatal(ctx, "running exec", err)
    }
}
```

**Compile-time interface checks** are required for all Runner and Server implementations:

```go
var _ cmdutil.ServerRunner = (*Runner)(nil)
var _ dsemgmtpbv1.DSEServer = (*Server)(nil)
```

### gRPC + gRPC-Gateway

- gRPC server starts first; HTTP gateway connects to it (same-pod, insecure credentials). Synchronized startup via `sync.WaitGroup`.
- `auth.CustomHeaderMatcher` forwards the `Cwid` header through the gateway.
- Graceful shutdown: 25-second timeout (matching Kubernetes 30-second SIGKILL window). Handles `SIGINT`, `SIGTERM`, `SIGKILL`.

### Dependency Injection & Composition

- **Accept interfaces, return structs** — follow the Go proverb.
- **Use constructor functions** — never create structs directly; use `New*()` factory functions with dependency injection.
- **Use composition** — inject dependencies via struct fields, keeping components decoupled and independently testable.
- **Create focused, single-purpose interfaces** — follow the Interface Segregation Principle.

### Database Layer

- DynamoDB accessed through `database.CollectionInterface` (`Store`, `Delete`, `Fetch`, `UpdateField`, `List`). Each `Server` struct holds multiple `CollectionInterface` pointers.
- Interface-based design enables easy mock injection for testing.
- Pagination uses DynamoDB scan with `LastEvaluatedKey`.

### Message Queue Layer

- Workers consume from AWS SQS queues via `messaging.Queue` abstraction (`Publish`, `Subscribe`, `Delete`).
- FIFO queues use `MessageGroupId` + `MessageDeduplicationId` (generated via KSUID).
- Workers poll in an infinite loop, process messages, delete on success, log errors and continue on failure (never crash).
- Workers communicate back to APIs via gRPC clients (`clients/v2`).

### Inter-Service Communication

- Use `internal/clients/v2/` for all new inter-service gRPC calls (v1 is deprecated).
- `clientsv2.Collection` bundles multiple service clients together.
- `NewGRPCConn(ctx, host, port)` is the shared connection factory.
- Error wrapping in client code uses `fmt.Errorf("...: %w", err)` (not `logger.NewError`).

### PAPI / Profile API Client Pattern

When adding new PAPI queries (user lookups, group queries, entitlement checks):

1. **Add the GraphQL query** to `internal/profileapi/genqlient.graphql` and run `go generate`.
2. **Add a method on `profileapi.Client`** in `internal/profileapi/profile.go` — NOT a standalone function in a new file. Follow the existing pattern:
   ```go
   func (c *Client) GetUserEmailById(ctx context.Context, cwid string) (string, error) {
       logger, ctx := logv2.FromCtx(ctx)
       resp, err := profilev3.GetUserById(ctx, c.g, cwid)
       // ...
   }
   ```
3. **Call it via `PapiClient.LookupUpnFromCwid()`** in `internal/clients/profile.go`, using `buildPapiClient()` to handle token/client setup — do NOT manually fetch Azure tokens or build custom caching.

**Why**: `buildPapiClient()` already handles Azure token acquisition via `cloudv2.NewAzureTokenClient`. Custom caching with `sync.RWMutex` + global maps was tried and rejected in PR #650 — it duplicates existing infrastructure.

---

## Coding Standards

### Logging

**Use `internal/logging/v2` for ALL new code** — v1 is deprecated.

```go
import logv2 "github.com/bayer-int/dse-apis/internal/logging/v2"

func myFunction(ctx context.Context) error {
    logger, ctx := logv2.FromCtx(ctx)
    logger, ctx = logger.With(ctx, logv2.KV("tenant_name", name))

    logger.Info("processing request")

    if err != nil {
        return logger.NewError("failed to process", err)
    }
    return nil
}
```

**Critical Logging Rules:**

1. **ALWAYS** pass `context.Context` as the first parameter to every function.
2. When using `logger.With()`, **MUST reassign both** `logger` and `ctx`: `logger, ctx = logger.With(ctx, ...)`.
3. **NEVER** use `:=` for `logger.With()` reassignment — use `=`.
4. **Do NOT** wrap errors in `Error()` or `NewError()` with `%w` — wrapping is automatic.
5. `function_name` is automatically attached to logs — never add it manually.
6. Use `KV()` for key-value pairs, not raw strings.
7. Use `snake_case` for all log keys (e.g., `tenant_name`, `user_id`).
8. Log messages should be **capitalized** sentences.
9. **Never** import `log/slog` directly — always use `internal/logging/v2`. This is enforced by `scripts/check-log-usage.sh`.
10. Use `logger.DebugStruct()` for logging complex AWS SDK pointer-heavy types.

### Error Handling

**Primary pattern** — `logger.NewError(description, err)`:

```go
if err != nil {
    return logger.NewError("failed to create tenant", err)
}

// gRPC status codes for user-facing errors
return logger.NewError("tenant not found", status.Error(codes.NotFound, "tenant does not exist"))
```

**Rules:**
- Use `logger.NewError()` in all handler/service code — it wraps, logs, and propagates context.
- Use `status.Error(codes.*, message)` as the *inner* error when returning gRPC status codes.
- Use `fmt.Errorf("...: %w", err)` only in non-handler code (constructors, CLI tools, client wrappers).
- The `wrapcheck` linter has a carve-out for `.NewError(` — all other error returns must be wrapped.
- Do NOT double-log errors — `NewError` bubbles log fields to the gRPC interceptor where errors are logged once.

### Configuration

Use `kelseyhightower/envconfig` with per-service prefixes (`DSEMGMT_*`, `MODELS_*`, `TERRAFORMWORKER_*`, etc.). Use `split_words:"true"` tag convention (e.g., `AssetStackQueueName` → `ASSET_STACK_QUEUE_NAME`).

### Resource Naming

Follow Google's resource-oriented design with hierarchical names:

```
tenants/{id}
tenants/{id}/cloud-accounts/{id}
tenants/{id}/cloud-accounts/{id}/asset-stack-deployments/{id}
models/{id}/model-versions/{id}/model-version-deployments/{id}
jobs/{id}/job-versions/{id}/job-version-deployments/{id}
apps/{id}/app-versions/{id}/app-version-deployments/{id}
container-images/{ksuid}
```

Use `internal/resourcename` package for constructing and parsing resource names. Never build resource names via string concatenation.

### Validation

- Request validation is **proto-annotation-driven** via `buf.validate` annotations in `.proto` files.
- In Go handlers, call `validate.ProtoMessage(request, logging.New())` before processing.
- Validation rules include: `min_len`, `pattern` (regex), `required`, field relationships.
- Do NOT hand-write validation logic that can be expressed as proto annotations.
- **Enum fields** must use `(buf.validate.field).enum = { defined_only: true }` to reject undefined values. Add `not_in: 0` when the unspecified/zero value is invalid (see `containerimages.proto`, `apps.proto`).
- **Cross-field CEL validation** (`buf.validate.message.cel`) must reference enum values by fully qualified name — never magic numbers. Use `dse.{domain}.v1.EnumType.VALUE` syntax (e.g., `dse.jobs.v1.JobType.TRAINING`).

### Audit Records

Use `internal/audit.New(ctx, auditRecord)` to stamp audit records. Records are `proto.Clone()`d then timestamped server-side via `timestamppb.Now()`. Validation is applied via `validate.ProtoMessage()`.

### Field Masks

Use `internal/fieldmask` for partial responses/updates (Google API Design Guide pattern). Core functions: `Validate()`, `Apply()`, `ApplySlice()`, `Merge()` (using `protoreflect`). Multi-level paths not yet supported.

---

## Protocol Buffers

### Tooling & Generation

- **Buf** manages all proto schemas (`buf.yaml`, `buf.gen.yaml`).
- Code generation targets: Go (protobuf + gRPC + gateway), Python, TypeScript, OpenAPI v2.
- Run `make generate` after modifying any `.proto` file.

### Proto Structure

```
proto/dse/{domain}/v1/{domain}.proto
```

Domains: `dsemgmt`, `apps`, `models`, `jobs`, `olly`, `containerimages`, `audit`, `common`, `events`.

### API Design Conventions

- RESTful resource hierarchy with `google.api.http` annotations for REST mapping.
- Standard CRUD RPCs: `Create*`, `Get*`, `List*`, `Update*`, `Delete*`.
- Custom operations via `:` suffix (e.g., `:upgrade`, `:sagemaker-sandbox-setup`).
- `google.protobuf.FieldMask` for partial responses.
- Pagination via `page_token` / `page_size`.
- Soft deletes (audit `Deleted` field set, record retained).
- Shared `Audit` message type for `Created` / `Updated` / `Deleted` metadata.
- `buf.validate` annotations for request validation.

### Enum Conventions

- Define enums **at package level** when used by a single message or shared across messages (e.g., `ImageFamily`, `JobType`). Define **nested inside a message** only for tightly-scoped status enums (e.g., `Job.LifecycleStatus`, `ApprovalDetails.ApprovalStatus`).
- Zero value must be `<PREFIX>_UNSPECIFIED = 0` (e.g., `JOB_TYPE_UNSPECIFIED`, `VERSION_BUMP_UNSPECIFIED`).
- Non-zero values use short names without the enum-type prefix (e.g., `STANDARD`, `TRAINING`, `MAJOR`). This is safe because `ENUM_VALUE_PREFIX` is excluded in `buf.yaml`.
- In Go, enum `.String()` returns the proto name (e.g., `"STANDARD"`, `"TRAINING"`). Use this when passing values to terraform inputs or external systems.

### Proto Lint Rules

- STANDARD + COMMENTS enabled.
- Exceptions: `SERVICE_SUFFIX`, `RPC_REQUEST_STANDARD_NAME`, `RPC_RESPONSE_STANDARD_NAME`, `RPC_REQUEST_RESPONSE_UNIQUE`.
- Breaking change detection at `PACKAGE` level, run against `main` branch.

### Policy Enforcement

Every `rpc` definition in proto files **must** have a corresponding entry in at least one `policy.yaml` (Bayer AuthZ custom CRD). This is enforced by `scripts/check-proto-service-policies.sh`.

---

## GraphQL Client Code (genqlient)

- **Use `genqlient`** (`github.com/Khan/genqlient`) for ALL new GraphQL client code — do NOT hand-write GraphQL query strings or response structs.
- Older files like `query_group.go` were written before genqlient was adopted and should not be used as a reference pattern.
- Hand-written GraphQL queries are error-prone and vulnerable to GraphQL injection. genqlient provides compile-time type safety and generates clean, idiomatic Go code.

### How to add a new GraphQL query

1. Define your query in the relevant `genqlient.graphql` file (e.g., `internal/profileapi/genqlient.graphql` or `internal/crestapi/genqlient.graphql`).
2. Run `go generate` on the corresponding `generate.go` file:
   ```bash
   go generate internal/profileapi/generate.go
   ```
3. The generated client code will appear in `generated/profilev3/generated.go` (or the corresponding output directory configured in `genqlient.yaml`).
4. Use the generated type-safe functions in your Go code instead of building raw GraphQL request strings.

---

## Testing

### Three-Tier Testing Strategy

| Tier | Build Tag | File Pattern | Dependencies | Command |
|------|----------|-------------|-------------|---------|
| Unit | `//go:build unit` | `*_test.go` | None (mocks only) | `make unit-tests` |
| Integration | `//go:build integration` | `*_integration_test.go` | LocalStack / real AWS | `make integration-tests` |
| End-to-End | N/A (CLI tool) | `cmd/service-tester/commands/` | Deployed services | `make run-service-tester env=dev` |

### Unit Test Conventions

- **Table-driven tests** are the preferred pattern:

```go
func TestMyFunction(t *testing.T) {
    tests := []struct {
        name     string
        input    string
        expected string
        validate func(t *testing.T, result string, err error)
    }{
        {
            name:  "valid input",
            input: "test",
            validate: func(t *testing.T, result string, err error) {
                require.NoError(t, err)
                assert.Equal(t, "processed-test", result)
            },
        },
    }
    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            result, err := MyFunction(tt.input)
            tt.validate(t, result, err)
        })
    }
}
```

- Use `testify` (`assert`, `require`) — not raw `if` checks.
- Test files live alongside source files: `tenants_test.go` next to `tenants.go`.
- Naming: `Test<MethodName><Scenario>` (e.g., `TestCreateTenantNoDisplayName`).
- Use builder functions for test data: `buildCreateTenantRequest()`, `buildCreateCloudAccountRequest()`.
- `t.Skip()` with explanation for known pre-existing failures.

### Mock Patterns

Use **struct-based mocks with function fields** (not generated). Mock `database.CollectionInterface` for DB tests and `cloud/v2` interfaces for cloud tests. Use `//nolint:wrapcheck` for passthrough calls in mock implementations.

### Integration Test Conventions

- Use `//go:build integration` build tag (NOT `testing.Short()`).
- Integration tests hit real (LocalStack-emulated) endpoints.
- Use `runtime.JSONPb` marshaler for proto JSON serialization.

### End-to-End Tests (Service Tester)

- Cobra CLI in `cmd/service-tester/` — procedural E2E tests across the full DSE hierarchy.
- Tests require deployed services and AWS SSO authentication.
- **Mandatory for PR approval** — E2E test results must be included in PRs.
- Cleanup runs both before and after tests (unless `--skip-cleanup` or `--cleanup-only` flags are set).

---

## Development Workflow

### Branch Naming

```bash
git checkout -b feature/<aha-id>-<title>   # New features
git checkout -b fix/<bug-description>       # Bug fixes
```

### Key Make Targets

| Command | Purpose |
|---------|---------|
| `make ci` | Run all CI checks (pre-commit hooks) — same as CI pipeline |
| `make lint` | Run protobuf lint + golangci-lint |
| `make generate` | Regenerate all code from proto files |
| `make test` | Full test suite (starts LocalStack, runs tests, stops LocalStack) |
| `make go-test` | Go tests only (assumes LocalStack already running) |
| `make unit-tests` | Unit tests only (no external deps) |
| `make integration-tests` | Integration tests only |
| `make run-service-tester env=<env>` | Run E2E tests against a deployed environment |
| `make deploy env=<env>` | Build, push, and deploy to target environment |
| `make doc-server` | Start Go documentation server |

### CI Pipeline

- Pre-commit hooks enforce: protobuf lint, code generation freshness, `go mod tidy`, `golangci-lint` formatting + analysis, `govulncheck`, unit tests, logging convention checks, proto policy checks, shell script linting, K8s manifest validation.
- **Merging to `main`** auto-deploys to nonprod.
- Manual deploys via GitHub Actions `deploy-manually.yaml`.

### PR Requirements

- All CI checks pass.
- At least one approver review.
- No merge conflicts.
- Coding standards followed.
- Service-tester E2E results included.
- For features that add or modify API fields/endpoints, **manual API testing in `dev`** is expected. Deploy the branch, hit the endpoints with `auth curl`, and include screenshots in the PR. See `CONTRIBUTING.md` for auth setup (`auth login`, `auth curl`, `auth print-access-token`).

### Manual API Testing Notes

When a ticket involves new or changed API behavior (new fields, validation rules, enum values, etc.), create a testing notes file in `testdata/api-calls/` with auth prerequisites, one section per test scenario (`auth curl` command + expected response + screenshot placeholder).

Not every ticket requires this — use judgment. If the change only affects internal logic with no API surface impact, unit tests alone are sufficient.

For **Jobs E2E testing** specifically (full pipeline through terraform worker), see [Jobs E2E Testing Guide](guides/jobs-e2e-testing.md). For known issues encountered during manual testing, see [Common Jobs Issues](troubleshooting/common-jobs-issues.md).

### Code Review — How to Respond

When reviewers suggest changes, these patterns help communicate effectively:

- **Reviewer suggests a "simpler" expression**: Write the truth table showing edge cases where their suggestion breaks. Example: `== &&` vs `!= ||` in CEL — show that one breaks STANDARD jobs.
- **Reviewer suggests renaming a field**: If the name is constrained across repos, explain the chain (proto → constant → terraform variable) and link to the files in the other repos.
- **Reviewer asks for testing evidence**: Provide it proactively next time. Include API responses AND downstream worker logs for pipeline features.
- **Reviewer adds code after your approval**: Common pattern — Pedro adds server-side validations, Ryan tightens proto annotations, Carlos enforces naming. Review their commits to understand team conventions.

### Team Review Tendencies

| Reviewer | Focus Area |
|----------|-----------|
| ekraft-bayer | Field naming accuracy, terraform alignment |
| pedro-vallejo-bayer | Functional testing evidence, server-side validations (e.g., referential integrity) |
| carloslucio-bayer | E2E testing evidence, naming conventions (Key suffix), deployment-level verification |
| ryan-price2 | Proto annotation strictness (bidirectional CEL), code correctness |
| maddyst-bayer | API readability, SDK consumer perspective |

### Dev Container Tools

These tools are available in the dev container and needed for testing/development:

| Tool | Path / Install | Purpose |
|------|---------------|---------|
| `auth` | `/workspaces/dse-apis/.mise/installs/go-github-com-bayer-int-csgda-auth-cmd-auth/latest/bin/auth` | Bayer SSO auth CLI (`auth login`, `auth curl`, `auth print-access-token`) |
| `aws` | `/workspaces/dse-apis/.mise/installs/aws/2.34.24/aws/dist/aws` | AWS CLI for SSO, CloudWatch logs, S3 |
| `gh` | `apt install gh` (not pre-installed) | GitHub CLI for PR info, API calls |

**Codespaces tip**: If `auth login` fails with `xdg-open not found`:
```bash
sudo ln -sf "$BROWSER" /usr/local/bin/xdg-open
```

---

## Linting & Formatting

### golangci-lint (v2 Config)

- ~40+ linters enabled with explicit allowlist (`default: "none"`).
- **Formatters**: `gofmt`, `gofumpt`, `goimports`.
- **Key linters**: `errcheck`, `errorlint`, `govet`, `staticcheck`, `revive`, `wrapcheck`, `gosec`, `modernize`, `protogetter`, `sloglint`, `testifylint`.
- **`wrapcheck`** has a carve-out for `.NewError(` signatures.
- **`sloglint`**: Enforces `snake_case` keys, capitalized messages, `attr-only` approach, no global loggers.

### staticcheck

- All checks enabled except: `SA1019` (deprecation — TODO), `ST1000`, `ST1003`, `ST1016`, `ST1020`–`ST1023`.

### Custom Enforcement Scripts

| Script | Purpose |
|--------|---------|
| `scripts/check-log-usage.sh` | No direct `log/slog` imports; `logger.With()` must reassign (no `:=`); no manual `function_name` keys |
| `scripts/check-proto-service-policies.sh` | Every proto `rpc` must have a corresponding authorization policy entry |

---

## Security & Resilience

- **JWT authentication** from Azure AD / Entra ID — tokens are parsed and claims extracted (`cwid`, `appid`, `exp`).
- **PAPI entitlements** control access — Bayer AuthZ `Policy` CRDs in Kubernetes enforce per-RPC authorization.
- **Istio** VirtualServices handle routing and mTLS between services.
- Workers handle failures gracefully: log errors and continue processing the next message (never crash the loop).
- Use exponential backoff (`cenkalti/backoff/v5`) for retries on external calls.
- Workers support graceful shutdown: `InFlightQueueMessageID` / `Contents` fields allow in-flight work to be re-queued on shutdown signals.

---

## Deployment

### Infrastructure

- **Kubernetes** via **Helm** charts with **Skaffold** orchestration.
- Common chart library at `deploy/_common/` with shared templates: `_deployment.yaml`, `_hpa.yaml`, `_kedaScaledObject.yaml`, `_service.yaml`, `_serviceAccount.yaml`.
- Per-service charts depend on the common library.
- Each service deployed to its own namespace: `dsemgmt`, `models`, `olly`, `apps`, `jobs`, `imgbuild`, `workers`.
- **KEDA** ScaledObjects for worker autoscaling (SQS-based).
- ConfigMaps inject environment variables per service (prefixed by service name).

### Environments

- **dev** → **nonprod** → **prod** promotion pipeline.
- Environment-specific Helm values under `deploy/values/<env>/`.
- Skaffold profiles: `dev`, `nonprod`, `prod`.
- Docker images: multi-stage build (`golang:alpine` → configurable base), `CMD_DIR_NAME` build arg selects the `cmd/` directory to compile.

---

## Anti-Patterns — What NOT to Do

1. **Don't** use `internal/logging` v1 or `log/slog` directly in new code.
2. **Don't** use `:=` when reassigning `logger.With()` — always use `=`.
3. **Don't** wrap errors with `%w` inside `logger.NewError()` — wrapping is automatic.
4. **Don't** hand-edit files in `generated/`.
5. **Don't** write unit tests that connect to real AWS services — mock all external interactions.
6. **Don't** use `testing.Short()` for integration test gating — use `//go:build integration` build tags.
7. **Don't** create broad God-interfaces — keep interfaces focused and single-purpose.
8. **Don't** create structs directly — use `New*()` constructor functions with dependency injection.
9. **Don't** build resource names via string concatenation — use `internal/resourcename`.
10. **Don't** introduce compatibility workarounds, deprecated fallback fields, or unrelated refactors that weren't requested.
11. **Don't** add new RPCs without corresponding authorization policy entries.
12. **Don't** use `internal/clients` v1 for new inter-service calls — use `clients/v2`.
13. **Don't** hand-write GraphQL query strings or response structs — use `genqlient` to generate type-safe client code. Older files like `query_group.go` predate this pattern and should not be used as a reference.
14. **Don't** use magic numbers in CEL expressions for enum comparisons — use fully qualified enum names (e.g., `dse.jobs.v1.JobType.TRAINING` not `2`).
15. **Don't** rename fields for readability if they are shared across repos (proto → constants → terraform variables). Document the cross-repo constraint instead.
16. **Don't** add new terraform input variables to `buildAssetStackDeploymentInputs()` without also updating `dse-assetstacks` (assetstack.yaml) and `dse-terraform-modules` (variables.tf). The dsemgmt API will reject unknown inputs.
17. **Don't** use simple logical implication (`!P || Q`) for conditional field validation in CEL when the inverse should also be enforced. Use bidirectional validation to prevent stale data.
18. **Don't** assume old resources in the database have new enum fields populated — always default `UNSPECIFIED` to a safe value before using it in logic or external calls.
19. **Don't** create standalone package-level functions for PAPI/Profile API calls — add methods to the existing `*Client` struct in the same package (e.g., `profileapi.Client.GetUserEmailById`). Follow the pattern of `IsUserOwnerOfApplication`, `QueryGroup`, etc. Standalone functions with separate `NewClient()` calls duplicate client setup and break testability.
20. **Don't** build custom token caching for Azure/PAPI calls — use the existing `buildPapiClient()` helper in `internal/clients/profile.go`, which already handles token fetching and client construction. Rolling your own cache with `sync.RWMutex` + global maps is unnecessary complexity.
21. **Don't** use `var FuncName = package.Function` for monkey-patching in tests — this is an anti-pattern. Instead, add methods to existing client structs so they can be mocked via interface injection or struct-based mocks (the established pattern).
22. **Don't** create new files when the functionality belongs in an existing file — check if the package already has a client struct with similar methods before creating `query_*.go` files. For `profileapi`, all PAPI query methods belong on the `*Client` struct in `profile.go`.
23. **Don't** start implementing before reading ALL existing files in the target package. If you're adding a method to `internal/foo/`, read every `.go` file in that directory first. Search for existing helpers with `grep -rn "func build\|func New" internal/clients/`. The #1 cause of PR rework is rebuilding infrastructure that already exists (see PR #650). This applies to every package, not just PAPI.

### Regression Check for New Rules

Every time you add a new anti-pattern, rule, or workflow step, run this mental simulation **before considering it done**:

1. **Replay the original failure**: Walk through the exact scenario that caused the problem, step by step, as if the new rule already existed.
2. **Would the rule have fired?** — At what step would the agent have encountered the rule? Would the rule's wording have been unambiguous enough to stop the bad behavior?
3. **Is the rule generic enough?** — If the failure was in package X, would the rule also prevent the same class of error in package Y? If not, generalize it.
4. **Is there a workflow enforcement point?** — A rule in the anti-patterns list is passive (the agent must remember to read it). A check in `start-ticket.md`, `implementation-checklist.md`, or `adversarial-audit.md` is active (the agent is forced to execute it). **Rules without enforcement points are suggestions, not protections.** Always pair a new rule with at least one workflow checkpoint.
5. **Run the adversarial audit on your own changes** — treat rule/workflow edits with the same rigor as code edits. Ask: "Can I still fail despite this rule?" If yes, iterate.

**Example (PR #650 post-mortem)**:
- First attempt: Added rules 19-22 ("don't do X with PAPI"). Replay: agent starts new ticket touching `crestapi` → none of these rules fire → same failure. **Insufficient.**
- Second attempt: Added Step 5 to `start-ticket.md` ("read all files before coding") + Phase 2 to adversarial audit ("pattern alignment check") + generic rule 23. Replay: agent starts any ticket → Step 5 forces `ls` + `grep` + `cat` of existing code → agent discovers existing helpers → writes method on existing struct. **Protected.**

---

## Cross-Repo Coordination

### The AssetStack Pipeline

When a feature adds new fields that flow through the deployment pipeline, **all three repos must be coordinated**:

| Repo | What to change | Example |
|------|---------------|---------|
| `dse-apis` | Proto definition, Go deployment logic, constants, svcconfig version | Add field to proto, pass it in `buildAssetStackDeploymentInputs()` |
| `dse-assetstacks` | `job_task/assetstack.yaml` (or equivalent asset type) | Add new input variables, bump version tag |
| `dse-terraform-modules` | `job_task/aws/variables.tf` + `main.tf` | Define terraform variable + use it |

**Critical rule**: The `AssetStackJobTaskVersion` in `svcconfig.go` (and all `deploy/values/*/values.yaml`) **must match** the git tag in `dse-assetstacks`. If you bump the assetstack version, update it in all 4 places (svcconfig struct default + 3 environment values files).

### Field Naming Across Repos

When a field flows from proto → Go constants → terraform inputs → terraform variables:
- The **constant name** in `internal/assetstackdeployments/constants.go` must match the terraform variable name in `dse-terraform-modules` (e.g., `model_name`, `version_bump`).
- Constants that represent terraform input keys use the `Key` suffix convention (e.g., `JobTypeKey`, `ModelNameKey`, `VersionBumpKey`).
- **Do not rename** fields for "readability" in one repo if it would break the name chain across repos. Document why the name is what it is.

### Backward Compatibility for Enum Fields

When adding a new enum field to an existing resource:
1. **Default UNSPECIFIED to a safe value** in `CreateX()` — resources created before the field existed will have the zero value stored.
2. **Default UNSPECIFIED before external calls** (e.g., in `buildAssetStackDeploymentInputs`) — legacy resources read from DB will have UNSPECIFIED.
3. Mark the defaulting with a `// TODO:` explaining why it's not `required` yet and when it can become required.

```go
// In CreateJob — default for new resources
if job.GetJobType() == jobspbv1.JobType_JOB_TYPE_UNSPECIFIED {
    job.JobType = jobspbv1.JobType_STANDARD
}

// In buildAssetStackDeploymentInputs — default for legacy resources from DB
jobType := job.GetJobType()
if jobType == jobspbv1.JobType_JOB_TYPE_UNSPECIFIED {
    jobType = jobspbv1.JobType_STANDARD
}
```

---

## CEL Cross-Field Validation Patterns

### Bidirectional Validation (Recommended)

When field B is required only when field A has a specific value, use **bidirectional** CEL validation — this enforces that B is set when needed AND absent when not applicable:

```protobuf
option (buf.validate.message).cel = {
  id: "training-requires-model-name"
  message: "model_name is required when TRAINING, and cannot be set otherwise"
  expression: "(this.job_type == dse.jobs.v1.JobType.TRAINING && has(this.model_name)) || (this.job_type != dse.jobs.v1.JobType.TRAINING && !has(this.model_name))"
};
```

This is stricter than simple logical implication (`!P || Q`) because it prevents stale/invalid data on non-applicable cases.

### Truth Table

| job_type | model_name set | `!P \|\| Q` (permissive) | bidirectional (strict) |
|----------|---------------|--------------------------|------------------------|
| TRAINING | yes | PASS | PASS |
| TRAINING | no | FAIL | FAIL |
| STANDARD | yes | PASS (leaks data) | FAIL (catches error) |
| STANDARD | no | PASS | PASS |

### Use `has()` for Presence Checks

For string fields, prefer `has(this.field)` over `this.field != ''` in CEL expressions. `has()` checks proto field presence correctly and is more idiomatic for proto3.

---

## Server-Side Referential Validation

When a field references another resource (e.g., `model_name` pointing to a model), validate that the referenced resource **exists** before persisting:

```go
func (s *Server) validateTrainingModelExists(ctx context.Context, job *jobspbv1.Job) error {
    if job.GetJobType() != jobsbv1.JobType_TRAINING {
        return nil // no-op for non-training jobs
    }
    // gRPC call to Models API to verify model exists
    _, err = modelsClient.Client.GetModel(ctx, &modelspbv1.GetModelRequest{Name: job.GetModelName()})
    if err != nil {
        return logger.NewError(fmt.Sprintf("model %s does not exist", job.GetModelName()), err)
    }
    return nil
}
```

**Rules:**
- Guard with an early return for non-applicable cases (e.g., non-TRAINING jobs).
- Use `clients/v2` to call the target service.
- Wire the target service host as a config variable (e.g., `ModelsAPIHost`) and add it to Helm configmap.
- This validation belongs in `CreateX()` / `UpdateX()` handlers, NOT in proto annotations (proto can't do gRPC calls).

---

## E2E Testing for Pipeline Features

Any ticket that modifies the deployment pipeline (new terraform inputs, assetstack version bumps, SQS message format changes) **requires E2E testing** before PR approval. This is not optional.

### What Reviewers Will Ask For

1. API-level tests: Create/Get resources with new fields, verify validation.
2. **Terraform worker receives correct inputs**: Deploy a job version and verify the SQS message or terraform worker logs show the new variables.
3. Cross-repo coordination proof: Show that the assetstack version is bumped and the terraform module accepts the new inputs.

### Pipeline Verification Flow

```
Create Job → Create Version → Create Deployment → Approve Deployment
  → Check AssetStack Worker logs (SQS payload)
  → Check dsemgmt API (new AssetStackDeployment record)
  → Check Terraform Worker logs (terraform inputs received)
```

See [Jobs E2E Testing Guide](guides/jobs-e2e-testing.md) for the full step-by-step process.

---

## Verification Checkpoint

After completing any significant change (new feature, refactor, bug fix, proto update, etc.), **always run the following commands sequentially** before considering the work done:

```bash
make lint
make ci
go build ./...
```

If any fail, fix before committing. These are the same gates enforced by CI.

---

## AWS Access (SSO / IAM Identity Center)

To access AWS resources (CloudWatch logs, S3, ECR, etc.), use **AWS SSO** with the Bayer Identity Center.

### Initial Setup

La configuración ya existe en `~/.aws/config`. Los perfiles usan una sesión SSO compartida llamada `dse` (start URL: `https://bayer-prod.awsapps.com/start`, region: `us-east-1`).

| Profile | Account ID | Description |
|---------|------------|-------------|
| `dev` | `980921718484` | Dev environment |
| `nonprod` | `111166164941` | NonProd environment |
| `functional-dev` | `341224122507` | Functional dev account |
| `functional-nonprod` | `038462751056` | Functional nonprod account |

All profiles use role `sso-standard-user`, region `us-east-1`, output `json`.

### Login (each time session expires)

```bash
aws sso login --profile nonprod
```

### Usage

Every command needs `--profile nonprod`, or export it:

```bash
export AWS_PROFILE=nonprod
```

### CloudWatch Logs

**Log groups (nonprod)**:

| Component | Log Group |
|-----------|-----------|
| Jobs API | `/platform-nonprod-use1-eks/jobs` |
| dsemgmt API | `/platform-nonprod-use1-eks/dsemgmt` |
| Workers (terraform, imgbuild, assetstack) | `/platform-nonprod-use1-eks/workers` |

**Important**: Logs in nonprod are JSON-wrapped — the `log` field contains a stringified JSON. Fields like `data.msg` are NOT top-level queryable. Use `@message like /keyword/` instead.

**Example query**:

```bash
aws logs start-query --profile nonprod --region us-east-1 \
  --log-group-name "/platform-nonprod-use1-eks/workers" \
  --start-time $(date -d '1 hour ago' +%s) \
  --end-time $(date +%s) \
  --query-string 'filter @message like /terraform/ | sort @timestamp desc | limit 20'

# Get results with the returned queryId:
aws logs get-query-results --profile nonprod --region us-east-1 \
  --query-id "QUERY_ID_HERE"
```

**Session expiry**: If commands return error 253 or "expired token", run `aws sso login --profile nonprod` again.

---

## Git Push — Multi-Repo Workspace

This workspace contains multiple repos. The codespace GITHUB_TOKEN only has push permissions for `bayer-int/dse-apis`.

### For `dse-apis` (main workspace repo)

Use normal git push:
```bash
cd /workspaces/dse-apis
git push origin <branch>
```

### For other repos (e.g., `dse-rules`)

The codespace token does NOT have push access to personal or external repos. Use a PAT in the URL:

```bash
cd /workspaces/dse-rules
git push https://<PAT>@github.com/alejandro-bayer/dse-rules.git main
```

**Procedure:**
1. Try normal `git push origin main` first.
2. If it returns 403 / "Permission denied", ask the user for a Personal Access Token (PAT) with `repo` scope.
3. Use the PAT in the URL as shown above.
4. If the PAT fails (expired or revoked), ask for a new one — do not retry the same token.

> **Security**: Never store the PAT in a file. It's acceptable in the terminal command since codespace history is ephemeral. Remind the user to revoke the token after use if they're concerned.

---

## Key Conventions Summary

1. **Readability, simplicity, and maintainability** are the top priorities.
2. **Context propagation**: `context.Context` is always the first parameter.
3. **Interface-driven development**: Accept interfaces, return structs. Use dependency injection everywhere.
4. **Single responsibility**: Keep functions short and focused. Split domain logic across files by entity.
5. **Proto-first API design**: Proto definitions are the source of truth. Validation, REST mapping, and documentation flow from annotations.
6. **Structured, context-carried logging**: All logging goes through `internal/logging/v2` with `KV()` pairs and `snake_case` keys.
7. **Error bubble-up**: Errors are wrapped with `logger.NewError()` and logged once at the gRPC interceptor — never double-log.
8. **Test everything**: Unit tests for all new functionality, integration tests for AWS interactions, E2E tests for full workflows.
9. **Automate workflows**: `make ci` runs the same checks as the CI pipeline. Run it often during development.
10. **Document exported symbols**: Use GoDoc-style comments for all exported functions and packages.
