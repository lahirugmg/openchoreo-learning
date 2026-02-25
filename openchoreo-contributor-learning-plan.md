# 8-Module OpenChoreo Contributor Onboarding Plan

## Summary
This plan follows a native `Rancher Desktop + k3d` workflow (not quick-start container as the primary path) with an 8-module deep dive focused on `Controller/API` contributions.

## Fixed Defaults
- Focus area: `Controller/API code`
- Timeline: `8-module deep dive`
- Install path: `Native Rancher Desktop + k3d`
- Cluster model: start with single-cluster k3d, add optional planes progressively
- Known blockers to clear in Module 0:
  - `go` missing
  - `k3d` missing
  - `kubectl` and `helm` below contributor guide target versions

## Deliverables
- [ ] `open-choreo-learning-plan/openchoreo-contributor-learning-plan.md`
- [ ] `open-choreo-learning-plan/progress-log.md`

## Working Rules
- [ ] Record module evidence in `progress-log.md`
- [ ] Do not skip exit criteria before moving to the next module
- [ ] Keep commands reproducible from `code/openchoreo`
- [ ] Use fork + upstream Git workflow before first PR

---

## Module 0: Preflight and Environment Hardening
Goal: Make the local machine contributor-ready.

### Actions
- [ ] Configure Rancher Desktop to use Docker (`dockerd`/Moby)
- [ ] Install required tools (`go`, `k3d`, `kind`, `kubebuilder`, expected `kubectl`, expected `helm`)
- [ ] Validate toolchain and Git remotes
- [ ] Configure fork/upstream remotes for contribution workflow

### Commands
```bash
cd /Users/lahirus/work/pet-projects/open-choreo/code/openchoreo
docker version
docker info
k3d version
kubectl version --client
helm version --short
go version
./check-tools.sh
git remote -v
```

### Standards
- Go: `1.24.x`
- k3d: `>= 5.8`
- kubectl: `1.31.x` to `1.33.x`
- helm: `>= 3.16`
- plus `kind` and `kubebuilder` as required by `./check-tools.sh`

### Exit Criteria
- [ ] `./check-tools.sh` passes
- [ ] `origin` points to personal fork
- [ ] `upstream` points to `https://github.com/openchoreo/openchoreo.git`

---

## Module 1: Bring Up OpenChoreo on Native k3d (Single Cluster)
Goal: Run a working local control plane + data plane.

### Actions
- [ ] Follow single-cluster setup in `install/k3d/single-cluster/README.md`
- [ ] Create cluster and install prerequisites
- [ ] Install control plane and default resources
- [ ] Install and register data plane

### Primary Doc
- `code/openchoreo/install/k3d/single-cluster/README.md`

### Verify Commands
```bash
kubectl get pods -n openchoreo-control-plane
kubectl get pods -n openchoreo-data-plane
kubectl get dataplane -n default
```

### Exit Criteria
- [ ] Control plane pods healthy
- [ ] Data plane pods healthy
- [ ] DataPlane resource exists in `default`
- [ ] OpenChoreo console/API is reachable

---

## Module 2: Deploy First Workload and Trace Runtime Resources
Goal: Understand mapping from declarative model to runtime objects.

### Actions
- [ ] Deploy one from-image sample
- [ ] Trace object chain:
  - `Component -> ReleaseBinding -> HTTPRoute -> Deployment`
- [ ] Confirm endpoint reachability

### Sample Doc
- `code/openchoreo/samples/from-image/react-starter-web-app/README.md`

### Exit Criteria
- [ ] Sample app is reachable
- [ ] Can explain each resource in the chain and its role

---

## Module 3: Add Build Plane and Run Build-from-Source Flow
Goal: Complete source-to-deploy workflow.

### Actions
- [ ] Install build plane (single-cluster guide, build plane section)
- [ ] Apply workflow templates
- [ ] Deploy source sample: `go-docker-greeter`
- [ ] Watch workflow run to completion

### Sample Doc
- `code/openchoreo/samples/from-source/services/go-docker-greeter/README.md`

### Exit Criteria
- [ ] `WorkflowRun` succeeds
- [ ] Greeter endpoint returns expected response

---

## Module 4: Controller/API Architecture Deep Dive
Goal: Build a clear mental model of API types, reconcilers, and service layer.

### Read Order
- [ ] `code/openchoreo/api/v1alpha1`
- [ ] `code/openchoreo/internal/controller`
- [ ] `code/openchoreo/internal/openchoreo-api/services`
- [ ] `code/openchoreo/cmd/main.go`

### Exit Criteria
- [ ] Can trace one CRD field from API type to reconciler behavior and runtime effect
- [ ] Can explain where API/service/controller boundaries are in this repo

---

## Module 5: Fast Local Development Loop (Controller/API)
Goal: Shorten edit-test-deploy cycle.

### Actions
- [ ] Practice local component rebuild/load/update loop
- [ ] Practice controller logs inspection
- [ ] Run tests after code changes
- [ ] Practice local manager run after scaling down in-cluster manager

### Targets
```bash
make k3d.build.controller
make k3d.load.controller
make k3d.update.controller
make k3d.logs.controller
make test
```

### Exit Criteria
- [ ] Complete one end-to-end loop:
  - change -> deploy -> observe -> verify -> cleanup

---

## Module 6: First Contributor PRs (2 PR Milestone)
Goal: Contribute safely and match repository workflow.

### PR Targets
- [ ] PR A: docs fix for stale commands in `docs/contributors/contribute.md`
- [ ] PR B: one small Controller/API bug fix or test improvement from active issues/board

### Required Local Checks Before PR
```bash
make lint
make code.gen-check
make test
git commit -s -m "<conventional-commit-title>"
```

### Exit Criteria
- [ ] Two PRs opened
- [ ] DCO sign-off included
- [ ] PR titles follow conventional commit format

---

## Module 7: CI-Parity and e2e Workflow
Goal: Validate changes using project-aligned e2e workflow.

### Actions
- [ ] Run e2e setup/test/status/diagnostics/down sequence
- [ ] Capture diagnostics if failures occur
- [ ] Record root cause notes

### Targets
```bash
make e2e.setup
make e2e.test
make e2e.status
make e2e.diagnostics
make e2e.down
```

### Exit Criteria
- [ ] At least one clean e2e pass, or
- [ ] One documented failure analysis with diagnostics and root cause

---

## Module 8: Community Integration and Medium-Scope Planning
Goal: Move from first PRs to sustained contribution.

### Actions
- [ ] Follow regular triage/planning process
- [ ] Pick one medium-scope task
- [ ] Open proposal discussion only if the change is CRD/architecture-impacting
- [ ] Create implementation branch for approved task

### Process Docs
- `code/openchoreo/docs/contributors/development-process.md`
- `code/openchoreo/docs/proposals/README.md`

### Exit Criteria
- [ ] One accepted medium-scope issue plan
- [ ] Implementation branch started

---

## Plan Validation Checklist
- [ ] Toolchain gate passes (`./check-tools.sh`)
- [ ] Native k3d cluster starts and required namespaces/pods are healthy
- [ ] Control plane and data plane resources are registered and visible
- [ ] One from-image sample is reachable through gateway URL
- [ ] One from-source sample builds and deploys successfully
- [ ] Controller update loop works without cluster rebuild
- [ ] Local quality gates pass (`make lint`, `make code.gen-check`, `make test`)
- [ ] e2e lifecycle run is completed and interpreted
- [ ] PR compliance passes (DCO + conventional commit title)
- [ ] Clear understanding of proposal workflow vs regular issue/PR

---

## References
1. https://openchoreo.dev/docs/getting-started/quick-start-guide/
2. https://openchoreo.dev/docs/getting-started/deploy-first-component/
3. https://docs.rancherdesktop.io/ui/preferences/application/general/
4. `code/openchoreo/docs/contributors/contribute.md`
5. `code/openchoreo/docs/contributors/github_workflow.md`
6. `code/openchoreo/install/k3d/single-cluster/README.md`
7. `code/openchoreo/make/k3d.mk`
8. `code/openchoreo/make/e2e.mk`
