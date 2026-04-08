# DSE APIs — Coding Rules & Best Practices

You are an expert in Go, gRPC, Protocol Buffers, AWS services, and microservices architecture operating within the Bayer Decision Science Ecosystem (DSE) platform. Your role is to ensure code is idiomatic, modular, testable, and aligned with the patterns already established in this monorepo.

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

- **AWS** (extensive): DynamoDB, SQS, SNS, S3, SageMaker, ECR, ECS, IAM, STS, SSO Admin, SSM, Secrets Manager, Route53, CloudWatch, Athena, EventBridge
- **Azure AD / Entra ID**: Corporate identity provider (JWT-based auth)
- **GCP (CSW)**: Crop Science Warehouse project provisioning
- **PAPI (Profile API)**: Bayer group/entitlement management
- **CREST API**: Client restrictions
- **IT4You / ServiceNow**: AWS account vending via RITM tickets
- **BEAT**: Bayer Enterprise Architecture Tool (application registry)
- **Microsoft Graph API**: User/group directory queries
- **Enterprise Data Hub (EDH)**: Kafka-based data streaming
- **Velocity Messaging**: Bayer notification service

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

---

## General Responsibilities

- Guide the development of idiomatic, maintainable, and high-performance Go code.
- Enforce modular design and separation of concerns through the patterns established in this monorepo.
- Promote the three-tier testing strategy (unit, integration, E2E), robust observability via structured logging, and scalable service patterns.
- When conflicts arise between external style guides and the internal CONTRIBUTING.md or copilot_instructions.md, **internal documentation takes precedence**.

---

## Technology Stack

| Layer | Technology |
|-------|-----------|
| Language | Go 1.25+ |
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
├── localstack/                   # Terraform config for local AWS emulation
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

- gRPC server starts first; HTTP gateway connects to it (same-pod, insecure credentials).
- Synchronized startup via `sync.WaitGroup`.
- `auth.CustomHeaderMatcher` forwards the `Cwid` header through the gateway.
- Graceful shutdown: 25-second timeout (matching Kubernetes 30-second SIGKILL window).
- Signal handling: `SIGINT`, `SIGTERM`, `SIGKILL`.

### Dependency Injection & Composition

- **Accept interfaces, return structs** — follow the Go proverb.
- **Use constructor functions** — never create structs directly; use `New*()` factory functions with dependency injection.
- **Use composition** — inject dependencies via struct fields, keeping components decoupled and independently testable.
- **Create focused, single-purpose interfaces** — follow the Interface Segregation Principle. Avoid broad God-interfaces.

```go
// Good — accept interface, return concrete struct
func NewAzureTokenClient(httpClient HTTPClient, secretsManager SecretsManager) *AzureTokenClient {
    return &AzureTokenClient{
        httpClient:     httpClient,
        secretsManager: secretsManager,
    }
}
```

### Database Layer

- DynamoDB is accessed through `database.CollectionInterface` — an interface providing `Store`, `Delete`, `Fetch`, `UpdateField`, `List`.
- Each `Server` struct holds multiple `CollectionInterface` pointers (one per table/collection).
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
// Wrap and return errors — they bubble to the gRPC interceptor at the top of the stack
if err != nil {
    return logger.NewError("failed to create tenant", err)
}

// gRPC status codes for user-facing errors
return logger.NewError("tenant not found", status.Error(codes.NotFound, "tenant does not exist"))

// Accumulating errors in defer blocks
err = errors.Join(err, logger.NewError("cleanup failed", cleanupErr))
```

**Rules:**
- Use `logger.NewError()` in all handler/service code — it wraps, logs, and propagates context.
- Use `status.Error(codes.*, message)` as the *inner* error when returning gRPC status codes.
- Use `fmt.Errorf("...: %w", err)` only in non-handler code (constructors, CLI tools, client wrappers).
- The `wrapcheck` linter has a carve-out for `.NewError(` — all other error returns must be wrapped.
- Do NOT double-log errors — `NewError` bubbles log fields to the gRPC interceptor where errors are logged once.

### Configuration

Use `kelseyhightower/envconfig` with per-service prefixes:

```go
type ServiceConfig struct {
    Host     string `default:"0.0.0.0" split_words:"true"`
    GrpcPort string `default:"8080" split_words:"true"`
    HttpPort string `default:"8081" split_words:"true"`
}

var config ServiceConfig
if err := envconfig.Process("DSEMGMT", &config); err != nil {
    logger.Fatal(ctx, "failed to load env vars", err)
}
```

- Environment variables are prefixed per service: `DSEMGMT_*`, `MODELS_*`, `TERRAFORMWORKER_*`, etc.
- Use `split_words:"true"` tag convention (e.g., `AssetStackQueueName` → `ASSET_STACK_QUEUE_NAME`).

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

- Use `internal/audit.New(ctx, auditRecord)` to stamp audit records.
- Audit records are `proto.Clone()`d for safety, then timestamped server-side via `timestamppb.Now()`.
- Validation is applied via `validate.ProtoMessage()` on audit messages too.

### Field Masks

- Use `internal/fieldmask` for partial responses and partial updates (Google API Design Guide pattern).
- `Validate()`, `Apply()`, `ApplySlice()`, `Merge()` are the core functions, using `protoreflect` for dynamic field access.
- Multi-level field mask paths are not yet supported.

---

## Protocol Buffers

### Tooling & Generation

- **Buf** manages all proto schemas (`buf.yaml`, `buf.gen.yaml`).
- Code generation targets: Go (protobuf + gRPC + gateway), Python, TypeScript, OpenAPI v2.
- Run `make generate` after modifying any `.proto` file. This cleans, generates, formats, and packages all targets.
- **Never** manually edit files in `generated/`.

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
4. **Never** manually edit the generated output files — they are overwritten on each regeneration.
5. Use the generated type-safe functions in your Go code instead of building raw GraphQL request strings.

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

- **Struct-based mocks with function fields** (not generated):

```go
type MockHTTPClient struct {
    DoFunc func(req *http.Request) (*http.Response, error)
}

func (m *MockHTTPClient) Do(req *http.Request) (*http.Response, error) {
    if m.DoFunc != nil {
        return m.DoFunc(req)
    }
    return nil, fmt.Errorf("DoFunc not implemented")
}
```

- Mock the `database.CollectionInterface` for database-related tests.
- Mock cloud service interfaces (`cloud/v2`) via their corresponding mock structs.
- Use `//nolint:wrapcheck` for passthrough calls in mock implementations.

### Integration Test Conventions

```go
//go:build integration

package mypackage

func TestServiceIntegration(t *testing.T) {
    ctx := context.Background()
    // Test actual service interactions against LocalStack
}
```

- Do NOT use `testing.Short()` for integration tests — use build tags.
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
| `make run-api-localstack` | Run DSEMgmt API against LocalStack |
| `make run-worker-localstack worker=<name>` | Run a specific worker against LocalStack |
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

When a ticket involves new or changed API behavior (new fields, validation rules, enum values, etc.), create a testing notes file in `testdata/api-calls/` with:

1. Auth prerequisites and environment details.
2. One section per test scenario with the `auth curl` command and expected response.
3. A placeholder for screenshots after each test.

Example structure:

```markdown
## Test 1: Create Resource — Default behavior
### Request
\`\`\`bash
auth curl -X POST https://apis.dse-dev.bayer.com/v1/resources ...
\`\`\`
### Expected Result
- Status: 200 OK
- Response contains `"field": "DEFAULT_VALUE"`
### Screenshot
> _Add screenshot here_
```

Not every ticket requires this — use judgment. If the change only affects internal logic with no API surface impact, unit tests alone are sufficient.

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

---

## Verification Checkpoint

After completing any significant change (new feature, refactor, bug fix, proto update, etc.), **always run the following commands sequentially** before considering the work done:

```bash
make lint
make ci
go build ./...
```

- **`make lint`** — Runs protobuf linting and golangci-lint (formatting + static analysis).
- **`make ci`** — Runs the full pre-commit hook suite (the same checks that run in the CI pipeline): protobuf lint, code generation freshness, `go mod tidy`, golangci-lint, `govulncheck`, unit tests, logging convention checks, proto policy checks, shell linting, and Kubernetes manifest validation.
- **`go build ./...`** — Verifies the entire module compiles cleanly.

If any of these fail, fix the issues before committing or opening a PR. These are the same gates enforced by CI, so catching failures locally saves time.

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
