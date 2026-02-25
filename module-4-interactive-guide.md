# Module 4: Controller/API Architecture Deep Dive - Interactive Guide

**Learn by Doing with Your AI Coding Agent**

This guide takes you through OpenChoreo's codebase to understand how controllers work, how APIs are structured, and where business logic lives. This is essential knowledge for contributing code.

## Prerequisites

Before starting Module 4:
- Modules 1-3 complete (system running, apps deployed)
- Familiarity with Go programming language
- Understanding of Kubernetes controllers and CRDs
- Git and your favorite code editor

## Learning Objectives

By the end of this module, you will:
1. Navigate the OpenChoreo codebase confidently
2. Understand the controller reconciliation pattern
3. Trace a CRD field from API type → controller logic → runtime effect
4. Understand the relationship between controllers, API server, and services
5. Know where to look when debugging or adding features

## Time Estimate: 60-75 minutes

---

## Phase 1: Codebase Overview (10 minutes)

### Working Directory

```bash
cd /Users/lahirus/work/pet-projects/open-choreo/openchoreo-learning/code/openchoreo
```

---

### Command 1: Explore top-level structure

**Ask your AI agent:**
> "Show me the top-level directory structure of the OpenChoreo repository"

**Command:**
```bash
ls -la
```

**Key directories:**
- `api/` - CRD definitions (OpenChoreo's custom resource types)
- `internal/controller/` - Controller reconciliation logic
- `internal/openchoreo-api/` - REST API and services
- `cmd/` - Entry points (main.go files)
- `pkg/` - Shared libraries and utilities
- `install/` - Helm charts and installation configs
- `samples/` - Example resources
- `docs/` - Documentation

**Learning moment**: This is a standard Kubebuilder project structure. Most Kubernetes operators follow this pattern.

---

### Command 2: Check the Go module

**Ask your AI agent:**
> "Show me the Go module definition and major dependencies"

**Command:**
```bash
cat go.mod | head -30
```

**What to look for:**
- Module name: `github.com/openchoreo/openchoreo`
- Go version
- Key dependencies:
  - `sigs.k8s.io/controller-runtime` - Controller framework
  - `k8s.io/api` and `k8s.io/apimachinery` - Kubernetes types
  - `sigs.k8s.io/gateway-api` - Gateway API types

**Learning moment**: OpenChoreo is built on controller-runtime, which handles most of the Kubernetes integration plumbing.

---

### Command 3: Find the main entry point

**Ask your AI agent:**
> "Show me the main entry point for the OpenChoreo controller manager"

**Command:**
```bash
ls -la cmd/
cat cmd/main.go | head -50
```

**What to look for:**
- Manager setup
- Controller registration
- Webhook setup
- Signal handling

**Learning moment**: `cmd/main.go` is where everything starts. It bootstraps the manager and registers all controllers.

---

## Phase 2: API Types (CRDs) (15 minutes)

### Command 4: Explore API versions

**Ask your AI agent:**
> "List all API versions and their types"

**Command:**
```bash
ls -la api/
ls -la api/v1alpha1/
```

**What you'll find:**
- `api/v1alpha1/` - Current API version
- Multiple `*_types.go` files, one per CRD

**Learning moment**: `v1alpha1` means "version 1, alpha stability". As the API matures, it would become `v1beta1`, then `v1`.

---

### Command 5: Examine the Component CRD

**Ask your AI agent:**
> "Show me the Component API type definition"

**Command:**
```bash
cat api/v1alpha1/component_types.go
```

**What to look for:**
- `type Component struct` - The root type
- `type ComponentSpec struct` - User-provided configuration
- `type ComponentStatus struct` - Controller-managed status
- `+kubebuilder:` comments - Generate CRD YAML
- JSON/YAML tags on fields

**Key patterns:**
```go
// Spec = desired state (what user wants)
type ComponentSpec struct {
    Image      string              `json:"image,omitempty"`
    Source     *SourceConfig       `json:"source,omitempty"`
    Port       int32               `json:"port"`
    Bindings   []BindingReference  `json:"bindings"`
}

// Status = observed state (what controller sees)
type ComponentStatus struct {
    Conditions []metav1.Condition `json:"conditions,omitempty"`
}
```

**Learning moment**: Spec vs Status is a fundamental Kubernetes pattern. Spec is input (declared by user), Status is output (reported by controller).

---

### Command 6: Find all CRD types

**Ask your AI agent:**
> "List all the OpenChoreo CRD types"

**Command:**
```bash
ls api/v1alpha1/ | grep '_types.go' | sed 's/_types.go//' | sort
```

**Expected output (sample):**
- component
- releasebinding
- environment
- dataplane
- buildplane
- observabilityplane
- workload
- workflowrun
- etc.

**Learning moment**: Each CRD has its own controller. Understanding one CRD+controller pair helps you understand them all.

---

### Command 7: Trace a specific field

**Let's trace the `Component.Spec.Image` field from API to controller.**

**Ask your AI agent:**
> "Show me where Component.Spec.Image is defined in the API"

**Command:**
```bash
grep -n "Image" api/v1alpha1/component_types.go | head -10
```

**Now find where controllers use it:**
```bash
grep -r "\.Spec\.Image" internal/controller/ | head -10
```

**Learning moment**: Fields in API types are accessed by controllers using Go struct field access. Changes to API types require controller updates.

---

## Phase 3: Controllers and Reconciliation (20 minutes)

### Command 8: List all controllers

**Ask your AI agent:**
> "Show me all the controllers in the codebase"

**Command:**
```bash
ls -la internal/controller/
```

**What you'll find:**
- Directories or files for each controller
- Each controller watches one primary CRD type
- May also watch related resources

---

### Command 9: Examine a controller structure

**Let's look at the Component controller.**

**Ask your AI agent:**
> "Show me the Component controller implementation"

**Command:**
```bash
find internal/controller -name '*component*' -type f | head -5
```

**Look at the main file:**
```bash
# The exact path may vary, adjust based on your findings
cat internal/controller/component_controller.go | head -100
```

**What to look for:**
- `type ComponentReconciler struct` - The controller struct
- `Reconcile(ctx context.Context, req reconcile.Request)` - The main reconciliation function
- `SetupWithManager(mgr ctrl.Manager)` - Registers the controller

**Key pattern:**
```go
type ComponentReconciler struct {
    client.Client
    Scheme *runtime.Scheme
    // Other dependencies (loggers, clients, etc.)
}

func (r *ComponentReconciler) Reconcile(ctx context.Context, req reconcile.Request) (reconcile.Result, error) {
    // 1. Fetch the resource
    // 2. Check if it's being deleted
    // 3. Validate spec
    // 4. Create/update child resources
    // 5. Update status
    // 6. Return result
}
```

---

### Command 10: Trace reconciliation logic

**Ask your AI agent:**
> "Walk me through the Reconcile function step by step"

**Command:**
```bash
# Find the Reconcile function
grep -n "func.*Reconcile" internal/controller/component_controller.go
```

**Then read that function:**
```bash
# Adjust line numbers based on your grep results
sed -n '<start-line>,<end-line>p' internal/controller/component_controller.go
```

**What to trace:**
1. **Fetch resource**: `r.Get(ctx, req.NamespacedName, &component)`
2. **Check deletion**: `if !component.DeletionTimestamp.IsZero()`
3. **Business logic**: Creating ReleaseBindings, WorkflowRuns, etc.
4. **Update status**: `r.Status().Update(ctx, &component)`
5. **Return**: `return reconcile.Result{}, nil` or with error

**Learning moment**: This is the "reconciliation loop". It runs whenever the resource or its children change.

---

### Command 11: Find what triggers reconciliation

**Ask your AI agent:**
> "Show me what events cause the Component controller to reconcile"

**Command:**
```bash
grep -A 20 "SetupWithManager" internal/controller/component_controller.go
```

**What to look for:**
- `Owns(&v1alpha1.ReleaseBinding{})` - Reconcile when owned resources change
- `Watches(&source.Kind{Type: &v1alpha1.Environment{}})` - Reconcile when watched resources change
- Predicates for filtering events

**Learning moment**: Controllers don't poll. They react to events. The manager handles event filtering and queuing.

---

### Command 12: Understanding finalizers

**Ask your AI agent:**
> "Search for finalizer logic in the Component controller"

**Command:**
```bash
grep -n "finalizer" internal/controller/component_controller.go -i
```

**What to look for:**
- Adding finalizers: `controllerutil.AddFinalizer()`
- Removing finalizers: `controllerutil.RemoveFinalizer()`
- Cleanup logic before deletion

**Learning moment**: Finalizers prevent deletion until cleanup is done. Critical for resources with external dependencies (like cloud resources).

---

### Command 13: Trace status updates

**Ask your AI agent:**
> "Show me how controllers update resource status"

**Command:**
```bash
grep -n "Status().Update" internal/controller/component_controller.go
```

**Typical pattern:**
```go
// Update conditions
meta.SetStatusCondition(&component.Status.Conditions, metav1.Condition{
    Type:   "Ready",
    Status: metav1.ConditionTrue,
    Reason: "Reconciled",
})

// Persist to API server
if err := r.Status().Update(ctx, &component); err != nil {
    return reconcile.Result{}, err
}
```

**Learning moment**: Status updates are separate from spec updates. Use `r.Status().Update()`, not `r.Update()`.

---

## Phase 4: API Server and Services (15 minutes)

### Command 14: Explore the API server structure

**Ask your AI agent:**
> "Show me the REST API structure in openchoreo-api"

**Command:**
```bash
ls -la internal/openchoreo-api/
```

**What you'll find:**
- `services/` - Business logic layer
- `handlers/` or `api/` - HTTP handlers
- `models/` - API request/response models
- May integrate with Backstage plugins

---

### Command 15: Examine a service

**Ask your AI agent:**
> "Show me an example service implementation"

**Command:**
```bash
ls internal/openchoreo-api/services/
# Pick a service and examine it
cat internal/openchoreo-api/services/<service-name>.go | head -100
```

**What to look for:**
- Service struct with dependencies (K8s client, logger)
- Methods implementing business logic
- Calls to K8s API (Create, Get, Update, Delete)
- Error handling

**Typical pattern:**
```go
type ComponentService struct {
    client client.Client
}

func (s *ComponentService) CreateComponent(ctx context.Context, req *CreateComponentRequest) error {
    component := &v1alpha1.Component{
        ObjectMeta: metav1.ObjectMeta{
            Name: req.Name,
            Namespace: req.Namespace,
        },
        Spec: v1alpha1.ComponentSpec{
            Image: req.Image,
            Port: req.Port,
        },
    }
    return s.client.Create(ctx, component)
}
```

**Learning moment**: Services provide a layer between HTTP handlers and Kubernetes API. They encapsulate business logic.

---

### Command 16: Find handler/API routes

**Ask your AI agent:**
> "Show me how HTTP routes are defined"

**Command:**
```bash
find internal/openchoreo-api -name '*handler*' -o -name '*route*' -o -name '*api*' | grep -v '_test.go'
```

**Examine one:**
```bash
cat <path-to-handler-file> | head -80
```

**What to look for:**
- Route definitions (GET /api/components, POST /api/components, etc.)
- Handler functions calling services
- Request validation
- Response serialization

---

### Command 17: Trace a request flow

**Let's trace: API request → Handler → Service → Controller**

**Ask your AI agent:**
> "Help me trace what happens when a user creates a Component via the API"

**Flow:**
1. **HTTP POST** `/api/components` → Handler
2. **Handler** validates request, calls Service
3. **Service** creates K8s resource via client
4. **K8s API** persists resource, triggers event
5. **Controller** reconciles, creates child resources

**Commands to explore:**
```bash
# Find the create component handler
grep -r "POST.*components" internal/openchoreo-api/

# Find service call
grep -r "CreateComponent" internal/openchoreo-api/services/

# Find controller reconcile
grep -r "Reconcile" internal/controller/component_controller.go
```

**Learning moment**: The API server is optional. You can also create resources directly with `kubectl apply`. The controller doesn't care how resources were created.

---

## Phase 5: Putting It All Together (10 minutes)

### Command 18: Trace a field end-to-end

**Let's trace `Component.Spec.Replicas` from API to running pods.**

**Ask your AI agent:**
> "Help me trace the Component.Spec.Replicas field from API definition to Deployment"

**Steps:**

1. **API definition:**
```bash
grep -n "Replicas" api/v1alpha1/component_types.go
```

2. **Controller reads it:**
```bash
grep -rn "\.Spec\.Replicas" internal/controller/
```

3. **Controller applies it to child resource:**
```bash
# Find where Deployment is created
grep -rn "Replicas.*=.*component" internal/controller/
```

4. **Verify in cluster:**
```bash
# Get a component's replicas
kubectl get component react-starter -n default -o jsonpath='{.spec.replicas}'

# Check the deployment
kubectl get deployment -A -l openchoreo.dev/component=react-starter -o jsonpath='{.items[0].spec.replicas}'
```

**They should match!**

**Learning moment**: This is the power of declarative systems. Change the Component, controller updates Deployment, Kubernetes ensures pods match.

---

### Command 19: Find validation logic

**Ask your AI agent:**
> "Show me where API validation happens"

**Command:**
```bash
# Look for webhook validation
find . -name '*webhook*' | grep -v vendor

# Look for validation tags in API
grep -n "+kubebuilder:validation" api/v1alpha1/component_types.go
```

**What you'll find:**
- `+kubebuilder:validation:Required` - Field is required
- `+kubebuilder:validation:Minimum=1` - Numeric constraints
- `+kubebuilder:validation:Pattern=...` - Regex validation
- Validating webhooks for complex validation

**Learning moment**: Kubebuilder generates validation from comments. Complex validation requires webhooks.

---

### Command 20: Explore generated files

**Ask your AI agent:**
> "Show me what files are code-generated"

**Command:**
```bash
find . -name 'zz_generated*.go' | head -10
```

**What you'll find:**
- `zz_generated.deepcopy.go` - Deep copy methods for API types
- CRD YAML files in `config/crd/bases/`

**View one:**
```bash
cat api/v1alpha1/zz_generated.deepcopy.go | head -50
```

**Learning moment**: Never edit `zz_generated` files. They're regenerated by `make generate`.

---

## Phase 6: Development Workflow (10 minutes)

### Command 21: Understanding the Makefile

**Ask your AI agent:**
> "Show me the key make targets for development"

**Command:**
```bash
cat Makefile | grep '^[a-zA-Z]' | grep ':' | head -30
```

**Key targets:**
- `make generate` - Generate deepcopy and other code
- `make manifests` - Generate CRD YAMLs
- `make test` - Run tests
- `make build` - Compile binaries
- `make lint` - Run linters
- `make k3d.build.controller` - Build controller image for k3d

---

### Command 22: Run code generation

**Ask your AI agent:**
> "Run code generation to see what it produces"

**Commands:**
```bash
# See what changes (should be none if repo is clean)
git status

# Run generation
make generate

# See if anything changed
git status
git diff
```

**Expected output**: No changes (if repo is up-to-date)

**If there are changes**, it means the generated code was out of sync. You'd commit these changes.

**Learning moment**: Always run `make generate` and `make manifests` after modifying API types.

---

### Command 23: Examine test structure

**Ask your AI agent:**
> "Show me the test file structure"

**Command:**
```bash
find . -name '*_test.go' -not -path './vendor/*' | head -20
```

**Look at a controller test:**
```bash
ls internal/controller/*_test.go | head -1 | xargs cat | head -100
```

**What to look for:**
- `envtest` - Runs a real API server for integration tests
- `fake.NewClientBuilder()` - Fake Kubernetes client for unit tests
- Test tables for multiple scenarios

---

## Module 4 Exit Criteria

Before moving to Module 5, verify you can:

- [ ] Navigate the codebase structure confidently
- [ ] Find API type definitions in `api/v1alpha1/`
- [ ] Locate controller implementation for any CRD
- [ ] Explain the Reconcile function pattern
- [ ] Trace a field from API type → controller → runtime
- [ ] Understand Spec vs Status
- [ ] Explain how events trigger reconciliation
- [ ] Find service layer code in `internal/openchoreo-api/`
- [ ] Know when to run `make generate` and `make manifests`

### Knowledge Check

**Ask your AI agent these questions:**

1. "What's the difference between `r.Update()` and `r.Status().Update()`?"
2. "If I add a new field to ComponentSpec, what files need to be regenerated?"
3. "How does the controller know when a Component's ReleaseBinding changes?"
4. "Where would I add validation that requires cross-field checks?"
5. "What happens if Reconcile returns an error?"

**Discuss answers with your AI agent to cement understanding.**

---

## Advanced Exploration (Optional)

### Command 24: Set up your IDE

**Ask your AI agent:**
> "Help me configure my IDE for Go development in this project"

**For VS Code:**
```bash
# Install Go extension
code --install-extension golang.go

# Open workspace
code .
```

**Set breakpoints and debug:**
- Add breakpoints in controller code
- Run controller locally (Module 5)
- Step through reconciliation

---

### Command 25: Explore the CLI tool

**If OpenChoreo has a CLI:**

**Ask your AI agent:**
> "Show me the CLI tool structure"

**Command:**
```bash
ls -la pkg/cli/
```

**Learning moment**: CLIs often provide a friendlier interface to the same API the controllers use.

---

### Command 26: Read a proposal

**Ask your AI agent:**
> "Show me a design proposal to understand how features are planned"

**Command:**
```bash
ls docs/proposals/
cat docs/proposals/<pick-one>.md | head -100
```

**What to look for:**
- Problem statement
- Proposed solution
- API changes
- Implementation plan

**Learning moment**: Major features start as proposals. Read these to understand architectural decisions.

---

## Important File Locations Reference

**Ask your AI agent to save this:**

### Code Structure
```
api/v1alpha1/              - CRD type definitions
internal/controller/        - Controller reconciliation logic
internal/openchoreo-api/    - REST API and services
cmd/                        - Entry points (main.go)
pkg/                        - Shared libraries
config/                     - Kubernetes manifests (CRDs, RBAC, etc.)
```

### Key Files
```
cmd/main.go                       - Controller manager entry point
api/v1alpha1/component_types.go   - Component CRD definition
internal/controller/component_controller.go - Component reconciler
Makefile                          - Build and development targets
go.mod                            - Go dependencies
```

### Commands
```bash
# Generate code
make generate
make manifests

# Build
make build
make docker-build

# Test
make test
make lint

# For k3d development (Module 5)
make k3d.build.controller
make k3d.load.controller
make k3d.update.controller
```

---

## Troubleshooting Understanding

### Issue: Can't find where a field is used

**Solution:**
```bash
# Search recursively
grep -r "FieldName" .

# Exclude vendor and generated files
grep -r "FieldName" --exclude-dir=vendor --exclude='zz_generated*' .

# Search in specific directories
grep -r "FieldName" internal/controller/
```

---

### Issue: Don't understand what a controller does

**Solution:**
1. Read the CRD definition first (`api/v1alpha1/<type>_types.go`)
2. Find the controller (`internal/controller/<type>_controller.go`)
3. Read the Reconcile function
4. Look at test files (`*_test.go`) for examples
5. Search for related docs (`docs/` or `README.md`)

---

### Issue: Confused by code generation

**Solution:**
- **Never edit** `zz_generated*.go` files
- Only edit `*_types.go` and controller files
- Run `make generate manifests` after API changes
- Commit generated files with your changes

---

## Next Steps: Module 5

Once Module 4 is complete, you're ready for **Module 5: Fast Local Development Loop**.

In Module 5, you will:
- Make code changes to controllers
- Build and test locally
- Deploy changes to your k3d cluster
- Debug running controllers
- Use `make` targets for fast iteration

---

## Update Your Progress Log

**Ask your AI agent:**
> "Help me update the progress-log.md file with my Module 4 completion"

Add to [progress-log.md](progress-log.md):
- Date completed
- Key files explored
- Field trace you performed
- Controllers examined
- Understanding of Reconcile pattern
- Any architectural questions

---

## Recommended Reading

**Ask your AI agent to help you read these:**

1. **Kubebuilder Book**: https://book.kubebuilder.io/
   - Understand controller-runtime framework
   - Learn reconciliation patterns

2. **OpenChoreo Contributor Docs**:
```bash
cat docs/contributors/README.md
cat docs/contributors/development-process.md
```

3. **Controller Runtime Docs**: https://pkg.go.dev/sigs.k8s.io/controller-runtime

---

**Congratulations on completing Module 4!** You now understand the OpenChoreo codebase architecture and can navigate it confidently. You're ready to start making code changes in Module 5.

**Key Takeaway**: Everything in OpenChoreo follows the Kubernetes patterns: API types define structure, controllers implement behavior, and reconciliation ensures desired state matches actual state.
