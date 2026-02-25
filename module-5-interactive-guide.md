# Module 5: Fast Local Development Loop - Interactive Guide

**Learn by Doing with Your AI Coding Agent**

This guide teaches you how to iterate quickly when developing OpenChoreo controllers - from making code changes to seeing them run in your local cluster in seconds.

## Prerequisites

Before starting Module 5:
- Modules 1-4 complete (system running, code understood)
- Local k3d cluster running
- Go development environment set up
- Comfortable editing Go code

## Learning Objectives

By the end of this module, you will:
1. Make changes to controller code
2. Build and load images into k3d
3. Update running controllers without cluster rebuild
4. View and debug controller logs
5. Run controllers locally outside the cluster
6. Test changes before committing
7. Achieve a fast edit-test-verify cycle

## Time Estimate: 45-60 minutes

---

## Phase 1: Understanding the Development Workflow (5 minutes)

### The Fast Loop

**Ask your AI agent:**
> "Explain the fast development loop for OpenChoreo controllers"

**The workflow:**
```
1. Edit code (controller, API, etc.)
2. Run make generate (if API changed)
3. Build controller image
4. Load image into k3d
5. Restart controller pods
6. Observe logs
7. Verify behavior
8. Repeat
```

**Traditional (slow) workflow:** Edit → rebuild cluster → reinstall → test (10+ minutes)

**Fast workflow:** Edit → rebuild controller → restart → test (1-2 minutes)

**Learning moment**: k3d allows loading images directly without pushing to a registry, making iteration much faster.

---

### Working Directory

```bash
cd /Users/lahirus/work/pet-projects/open-choreo/openchoreo-learning/code/openchoreo
```

---

## Phase 2: Make a Simple Code Change (10 minutes)

Let's add a debug log statement to see reconciliation in action.

### Command 1: View current controller code

**Ask your AI agent:**
> "Show me the Component controller's Reconcile function"

**Command:**
```bash
grep -n "func.*Reconcile" internal/controller/component_controller.go
```

**Read the function:**
```bash
# Adjust line numbers based on your environment
sed -n '<start>,<end>p' internal/controller/component_controller.go
```

---

### Command 2: Add a log statement

**Ask your AI agent:**
> "Help me add a log statement to the Component controller"

**Find the Reconcile function and add near the top:**

```go
func (r *ComponentReconciler) Reconcile(ctx context.Context, req ctrl.Request) (ctrl.Result, error) {
    log := log.FromContext(ctx)

    // ADD THIS LINE:
    log.Info("🚀 MODULE 5: Reconciling Component", "namespace", req.Namespace, "name", req.Name)

    // ... rest of the function
}
```

**Use your editor:**
```bash
# Open in your editor (VS Code example)
code internal/controller/component_controller.go

# Or use vim
vim internal/controller/component_controller.go
```

**Save the file.**

**Learning moment**: The rocket emoji helps us identify our log line among many others. You can use any marker.

---

### Command 3: Verify your change

**Ask your AI agent:**
> "Show me the change I just made"

**Command:**
```bash
git diff internal/controller/component_controller.go
```

**Expected output:** Your log line added

---

## Phase 3: Build and Deploy the Change (15 minutes)

### Command 4: Check if code generation is needed

**Ask your AI agent:**
> "Do I need to run code generation for my change?"

**Rule of thumb:**
- Changed `*_types.go` files? → Run `make generate manifests`
- Changed controller logic only? → No generation needed
- Changed webhooks? → Run `make generate`

**For our log change:** No generation needed.

**If you had changed API types:**
```bash
make generate
make manifests
```

---

### Command 5: Run tests

**Ask your AI agent:**
> "Run the test suite to ensure nothing broke"

**Command:**
```bash
make test
```

**This may take 1-2 minutes.**

**Expected output:** All tests pass

**If tests fail**, review your change and fix issues before continuing.

**Learning moment**: Always run tests before deploying. It's faster to catch issues here than in the cluster.

---

### Command 6: Build the controller image

**Ask your AI agent:**
> "Build the controller image for k3d"

**Command:**
```bash
make k3d.build.controller
```

**What this does:**
- Compiles the Go code
- Builds a Docker image
- Tags it appropriately for k3d

**Time:** ~30 seconds (subsequent builds are faster due to caching)

**Watch for:** `Successfully built` message

---

### Command 7: Load the image into k3d

**Ask your AI agent:**
> "Load the newly built controller image into the k3d cluster"

**Command:**
```bash
make k3d.load.controller
```

**What this does:**
- Copies the image from your local Docker to the k3d cluster
- No registry push needed!

**Time:** ~10 seconds

**Expected output:** Image loaded into cluster

**Learning moment**: This is the magic of k3d. Images are loaded directly, avoiding slow registry pushes.

---

### Command 8: Update the running controller

**Ask your AI agent:**
> "Restart the controller pods with the new image"

**Command:**
```bash
make k3d.update.controller
```

**What this does:**
- Deletes the current controller pods
- Kubernetes automatically recreates them with the new image
- Uses `imagePullPolicy: Never` to use the loaded image

**Time:** ~5 seconds

**Verify new pods are running:**
```bash
kubectl get pods -n openchoreo-control-plane -l control-plane=controller-manager
```

**Expected output:** New pods in "Running" state

---

### Command 9: View the logs with your change

**Ask your AI agent:**
> "Show me the controller logs to see my debug statement"

**Command:**
```bash
make k3d.logs.controller
```

**Or manually:**
```bash
kubectl logs -n openchoreo-control-plane -l control-plane=controller-manager --tail=50 -f
```

**What to look for:**
- Your rocket emoji: `🚀 MODULE 5: Reconciling Component`
- It should appear whenever a Component is reconciled

**Press Ctrl+C to stop following logs**

---

### Command 10: Trigger a reconciliation

**Ask your AI agent:**
> "Trigger a Component reconciliation to see my log statement"

**Command:**
```bash
# Add an annotation to trigger reconciliation
kubectl annotate component react-starter -n default test-annotation="$(date +%s)" --overwrite
```

**Watch the logs:**
```bash
kubectl logs -n openchoreo-control-plane -l control-plane=controller-manager --tail=20 -f
```

**You should see your log line!**

**Success!** You've completed your first development cycle.

---

## Phase 4: Running Controllers Locally (15 minutes)

Sometimes it's faster to run the controller on your laptop instead of in the cluster.

### Command 11: Scale down the in-cluster controller

**Why?** Two controller instances would conflict, both trying to reconcile the same resources.

**Ask your AI agent:**
> "Scale down the in-cluster controller so I can run it locally"

**Command:**
```bash
kubectl scale deployment controller-manager -n openchoreo-control-plane --replicas=0
```

**Verify:**
```bash
kubectl get pods -n openchoreo-control-plane -l control-plane=controller-manager
```

**Expected output:** No pods

---

### Command 12: Export kubeconfig

**The controller needs to talk to the cluster.**

**Ask your AI agent:**
> "Ensure my kubeconfig points to the k3d cluster"

**Command:**
```bash
kubectl cluster-info
```

**Expected output:** Points to `k3d-openchoreo`

**If not:**
```bash
k3d kubeconfig get openchoreo > ~/.kube/config-openchoreo
export KUBECONFIG=~/.kube/config-openchoreo
```

---

### Command 13: Run the controller locally

**Ask your AI agent:**
> "Run the OpenChoreo controller manager locally on my machine"

**Command:**
```bash
make run
```

**What happens:**
- Compiles and runs `cmd/main.go`
- Connects to your k3d cluster
- Starts all controllers
- Logs appear in your terminal

**You'll see:**
- Startup logs
- Controller registration
- Your rocket emoji logs!

**Learning moment**: Running locally gives you immediate feedback and allows using debuggers.

---

### Command 14: Test while running locally

**In another terminal, trigger a reconciliation:**

**Ask your AI agent:**
> "Trigger a Component reconciliation while the controller runs locally"

**Command (new terminal):**
```bash
cd /Users/lahirus/work/pet-projects/open-choreo/openchoreo-learning/code/openchoreo
kubectl annotate component react-starter -n default local-test="$(date +%s)" --overwrite
```

**Watch the first terminal:**
- You should see reconciliation logs immediately
- Much faster feedback than reading from cluster

**Press Ctrl+C in the first terminal to stop the local controller**

---

### Command 15: Scale the controller back up

**Ask your AI agent:**
> "Restore the in-cluster controller"

**Command:**
```bash
kubectl scale deployment controller-manager -n openchoreo-control-plane --replicas=1
```

**Verify:**
```bash
kubectl get pods -n openchoreo-control-plane -l control-plane=controller-manager
```

**Expected output:** One pod running

**Learning moment**: Remember to scale back up! Otherwise, no reconciliation happens when you're not running locally.

---

## Phase 5: Debugging Techniques (10 minutes)

### Command 16: View detailed logs

**Ask your AI agent:**
> "Show me how to get detailed logs for debugging"

**Commands:**
```bash
# Get all controller logs
kubectl logs -n openchoreo-control-plane -l control-plane=controller-manager --all-containers=true --tail=100

# Follow logs with timestamps
kubectl logs -n openchoreo-control-plane -l control-plane=controller-manager -f --timestamps

# Filter for specific resource
kubectl logs -n openchoreo-control-plane -l control-plane=controller-manager --tail=200 | grep react-starter

# Get previous logs (if pod crashed)
kubectl logs -n openchoreo-control-plane -l control-plane=controller-manager --previous
```

---

### Command 17: Debug resource status

**Ask your AI agent:**
> "Show me how to inspect resource conditions for debugging"

**Commands:**
```bash
# Check component status
kubectl get component react-starter -n default -o jsonpath='{.status.conditions}' | jq .

# Check all related resources
kubectl get component,releasebinding,httproute,deployment -A -l openchoreo.dev/component=react-starter

# Describe for events
kubectl describe component react-starter -n default
```

---

### Command 18: Use Kubernetes events

**Ask your AI agent:**
> "Show me recent events related to my component"

**Command:**
```bash
kubectl get events -n default --sort-by='.lastTimestamp' | grep react-starter
```

**Learning moment**: Events are created by controllers for important state changes. Great for debugging.

---

## Phase 6: Advanced Development Tips (5 minutes)

### Command 19: Use delve for debugging

**If you need breakpoints:**

**Ask your AI agent:**
> "Help me set up delve for debugging OpenChoreo"

**Install delve:**
```bash
go install github.com/go-delve/delve/cmd/dlv@latest
```

**Run with debugger:**
```bash
dlv debug ./cmd/main.go
```

**In delve:**
```
(dlv) break internal/controller/component_controller.go:123
(dlv) continue
```

**Learning moment**: Debuggers are powerful for complex logic. Use logs for quick iterations.

---

### Command 20: Set up watch for auto-rebuild

**For even faster iteration:**

**Ask your AI agent:**
> "Set up a watch script that rebuilds on file changes"

**Simple watch script (save as `watch-build.sh`):**
```bash
#!/bin/bash
while true; do
  inotifywait -r -e modify internal/ api/ cmd/
  echo "🔄 Files changed, rebuilding..."
  make k3d.build.controller && make k3d.load.controller && make k3d.update.controller
done
```

**Run it:**
```bash
chmod +x watch-build.sh
./watch-build.sh
```

**Now just edit and save - builds happen automatically!**

---

### Command 21: View resource finalizers

**If resources won't delete:**

**Ask your AI agent:**
> "Show me how to debug stuck deletions"

**Command:**
```bash
kubectl get component react-starter -n default -o jsonpath='{.metadata.finalizers}'
```

**If stuck with finalizers and controller is failing:**
```bash
# CAUTION: Only do this if you know what you're doing
kubectl patch component react-starter -n default -p '{"metadata":{"finalizers":null}}' --type=merge
```

---

## Module 5 Exit Criteria

Before moving to Module 6, verify you can:

- [ ] Make a code change to a controller
- [ ] Build controller image with `make k3d.build.controller`
- [ ] Load image with `make k3d.load.controller`
- [ ] Update running controller with `make k3d.update.controller`
- [ ] View logs with `make k3d.logs.controller`
- [ ] Run tests with `make test`
- [ ] Run controller locally with `make run`
- [ ] Complete the full cycle in under 2 minutes
- [ ] Debug issues using logs and resource status

### Knowledge Check

**Ask your AI agent these questions:**

1. "Why do we need to scale down the in-cluster controller before running locally?"
2. "What's the difference between `make k3d.build.controller` and `make k3d.update.controller`?"
3. "When do I need to run `make generate`?"
4. "How can I see logs from a crashed controller pod?"
5. "What's faster: running locally or deploying to cluster? When would I use each?"

---

## Practice Exercise: Make a Real Change

### Exercise: Add a status field

**Ask your AI agent:**
> "Help me add a new status field to track the last reconciliation time"

**Steps:**

1. **Edit API type** (`api/v1alpha1/component_types.go`):
```go
type ComponentStatus struct {
    Conditions []metav1.Condition `json:"conditions,omitempty"`

    // ADD THIS:
    LastReconcileTime *metav1.Time `json:"lastReconcileTime,omitempty"`
}
```

2. **Run code generation:**
```bash
make generate
make manifests
```

3. **Update controller** to set the field:
```go
func (r *ComponentReconciler) Reconcile(ctx context.Context, req ctrl.Request) (ctrl.Result, error) {
    // ... fetch component ...

    // ADD THIS:
    now := metav1.Now()
    component.Status.LastReconcileTime = &now

    // ... update status ...
}
```

4. **Build and deploy:**
```bash
make test
make k3d.build.controller
make k3d.load.controller
make k3d.update.controller
```

5. **Verify:**
```bash
kubectl get component react-starter -n default -o jsonpath='{.status.lastReconcileTime}'
```

**You should see a timestamp!**

---

## Important Make Targets Reference

**Ask your AI agent to save these:**

### Building and Testing
```bash
make generate              # Generate deepcopy and other code
make manifests             # Generate CRD YAMLs
make test                  # Run unit tests
make lint                  # Run linters
make build                 # Build binary locally
```

### k3d Development
```bash
make k3d.build.controller  # Build controller Docker image
make k3d.load.controller   # Load image into k3d cluster
make k3d.update.controller # Restart controller pods
make k3d.logs.controller   # View controller logs
```

### Local Development
```bash
make run                   # Run controller locally
make debug                 # Run with debugger (if defined)
```

### Complete Workflow
```bash
# Fast iteration (no API changes)
make test && make k3d.build.controller && make k3d.load.controller && make k3d.update.controller

# With API changes
make generate && make manifests && make test && make k3d.build.controller && make k3d.load.controller && make k3d.update.controller
```

---

## Troubleshooting

### Issue: Image not updating

**Symptoms:** Code changes not reflected in cluster

**Diagnosis:**
```bash
# Check if image was loaded
k3d image list openchoreo

# Check imagePullPolicy
kubectl get deployment controller-manager -n openchoreo-control-plane -o jsonpath='{.spec.template.spec.containers[0].imagePullPolicy}'
```

**Solution:**
- Ensure `imagePullPolicy: Never` or `IfNotPresent`
- Re-run `make k3d.load.controller`
- Delete pods manually: `kubectl delete pods -n openchoreo-control-plane -l control-plane=controller-manager`

---

### Issue: Tests failing

**Symptoms:** `make test` fails

**Diagnosis:**
```bash
# Run with verbose output
make test ARGS="-v"

# Run specific test
go test ./internal/controller -run TestComponentController
```

**Common causes:**
- API changes not reflected in test expectations
- Missing mock setup
- Incorrect assertions

---

### Issue: Controller crashes locally

**Symptoms:** `make run` exits with error

**Diagnosis:**
```bash
# Check KUBECONFIG
echo $KUBECONFIG
kubectl cluster-info

# Check if required CRDs exist
kubectl get crd | grep openchoreo
```

**Common causes:**
- Can't connect to cluster
- Missing CRDs (install them)
- Missing RBAC permissions (run with cluster-admin context)

---

### Issue: Logs not showing my changes

**Symptoms:** Don't see new log statements

**Diagnosis:**
```bash
# Check pod age (ensure it's recent)
kubectl get pods -n openchoreo-control-plane -l control-plane=controller-manager

# Force pod restart
kubectl delete pods -n openchoreo-control-plane -l control-plane=controller-manager

# Check image
kubectl get pods -n openchoreo-control-plane -l control-plane=controller-manager -o jsonpath='{.items[0].spec.containers[0].image}'
```

---

## Next Steps: Module 6

Once Module 5 is complete, you're ready for **Module 6: First Contributor PRs**.

In Module 6, you will:
- Prepare your first pull request
- Fix a documentation issue
- Make a small controller/API improvement
- Follow the project's PR workflow
- Get your code reviewed and merged

---

## Update Your Progress Log

**Ask your AI agent:**
> "Help me update the progress-log.md file with my Module 5 completion"

Add to [progress-log.md](progress-log.md):
- Date completed
- Changes made (log statement, status field, etc.)
- Development cycle time achieved
- Any issues encountered
- Key learnings about the development workflow

---

## Time-Saving Aliases

**Ask your AI agent:**
> "Help me set up bash aliases for common commands"

**Add to `~/.bashrc` or `~/.zshrc`:**
```bash
# OpenChoreo development aliases
alias occ-build='make k3d.build.controller'
alias occ-load='make k3d.load.controller'
alias occ-update='make k3d.update.controller'
alias occ-logs='make k3d.logs.controller'
alias occ-test='make test'
alias occ-deploy='make test && make k3d.build.controller && make k3d.load.controller && make k3d.update.controller'
alias occ-run='make run'
```

**Reload:**
```bash
source ~/.bashrc  # or ~/.zshrc
```

**Now use:**
```bash
occ-deploy  # Runs full test → build → load → update
occ-logs    # View logs quickly
```

---

**Congratulations on completing Module 5!** You now have a fast, efficient development workflow. You can iterate on controller code in under 2 minutes, which is essential for productive contribution work.

**Key Takeaway**: The faster your feedback loop, the more productive you'll be. Invest time in setting up your development environment to minimize iteration time.
