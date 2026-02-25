# Module 7: CI-Parity and e2e Workflow - Interactive Guide

**Learn by Doing with Your AI Coding Agent**

This guide teaches you how to run end-to-end tests locally, matching the CI/CD pipeline, so you can catch issues before submitting PRs.

## Prerequisites

Before starting Module 7:
- Modules 1-6 complete (can develop, test, and submit PRs)
- Local k3d cluster available
- Comfortable with test concepts
- Understanding of CI/CD concepts

## Learning Objectives

By the end of this module, you will:
1. Understand the OpenChoreo CI/CD pipeline
2. Run e2e tests locally
3. Interpret test results and logs
4. Diagnose and fix test failures
5. Collect diagnostics for debugging
6. Ensure CI will pass before pushing
7. Debug flaky tests

## Time Estimate: 60-90 minutes

---

## Phase 1: Understanding the CI Pipeline (10 minutes)

### Command 1: Explore CI configuration

**Ask your AI agent:**
> "Show me the CI/CD configuration for OpenChoreo"

**Command:**
```bash
cd /Users/lahirus/work/pet-projects/open-choreo/openchoreo-learning/code/openchoreo

# GitHub Actions workflows
ls -la .github/workflows/

# View main CI workflow
cat .github/workflows/ci.yml | head -100
```

**What to look for:**
- Jobs: `lint`, `test`, `build`, `e2e`
- Triggers: `push`, `pull_request`
- Steps in each job

**Learning moment**: CI runs the same checks you should run locally. Running them locally first saves time and CI resources.

---

### Command 2: Check the Makefile e2e targets

**Ask your AI agent:**
> "Show me the e2e test targets in the Makefile"

**Command:**
```bash
grep -A 5 "^e2e\." Makefile
```

**Key targets:**
- `e2e.setup` - Set up test environment
- `e2e.test` - Run e2e tests
- `e2e.status` - Check test status
- `e2e.diagnostics` - Collect logs and diagnostics
- `e2e.down` - Tear down test environment

**Learning moment**: The Makefile provides the same commands CI uses. Use them!

---

## Phase 2: Running e2e Tests (30 minutes)

### Command 3: Clean up existing environment

**Before running e2e, clean up:**

**Ask your AI agent:**
> "Help me clean up my current k3d cluster to prepare for e2e tests"

**Commands:**
```bash
# List clusters
k3d cluster list

# Delete if it exists (optional - e2e may create its own)
k3d cluster delete openchoreo

# Verify
k3d cluster list
```

**Learning moment**: e2e tests often create their own isolated environment to ensure clean state.

---

### Command 4: Run e2e setup

**Ask your AI agent:**
> "Run the e2e setup to create the test environment"

**Command:**
```bash
make e2e.setup
```

**What this does:**
- Creates a fresh k3d cluster (or uses existing)
- Builds images
- Loads them into cluster
- Installs OpenChoreo components
- Prepares test data

**Time:** 5-10 minutes (first run)

**Watch for:**
- Cluster creation success
- Image builds complete
- Helm installs succeed

**If it fails:**
- Check logs
- Ensure Docker is running
- Verify no port conflicts (8080, 19080, etc.)

---

### Command 5: Run e2e tests

**Ask your AI agent:**
> "Run the e2e test suite"

**Command:**
```bash
make e2e.test
```

**What happens:**
- Runs all end-to-end test scenarios
- Tests real workflows (deploy, build, update, etc.)
- Verifies system behavior

**Time:** 10-20 minutes

**Output:**
- Test progress
- Pass/fail status
- Logs from failing tests

**Expected outcome:** All tests pass ✅

---

### Command 6: Check test status

**Ask your AI agent:**
> "Check the status of the e2e test run"

**Command:**
```bash
make e2e.status
```

**What this shows:**
- Number of tests run
- Passed/failed/skipped
- Test durations
- Summary

**Learning moment**: Use this to quickly see if tests passed without reading all output.

---

### Command 7: View test logs

**If tests failed:**

**Ask your AI agent:**
> "Show me detailed logs from the failed tests"

**Commands:**
```bash
# View all e2e logs
make e2e.logs

# Or specific component logs
kubectl logs -n openchoreo-control-plane -l app=controller-manager --tail=200

# Check test namespace
kubectl get pods -n <test-namespace>
kubectl logs -n <test-namespace> <pod-name>
```

---

## Phase 3: Diagnosing Test Failures (20 minutes)

### Command 8: Collect diagnostics

**When tests fail, collect diagnostics:**

**Ask your AI agent:**
> "Collect comprehensive diagnostics from the test environment"

**Command:**
```bash
make e2e.diagnostics
```

**What this collects:**
- All pod logs
- Resource states (Components, Deployments, etc.)
- Events
- Controller logs
- Saves to `diagnostics/` directory

**Review diagnostics:**
```bash
ls -la diagnostics/
```

---

### Command 9: Analyze a specific failure

**Example scenario: Deployment test fails**

**Ask your AI agent:**
> "Help me debug why the deployment e2e test failed"

**Steps:**

1. **Check test output:**
```bash
grep -A 20 "FAIL.*deployment" test-output.log
```

2. **Check component status:**
```bash
kubectl get component <test-component> -n <test-namespace> -o yaml
```

3. **Check controller logs:**
```bash
kubectl logs -n openchoreo-control-plane -l app=controller-manager | grep <test-component>
```

4. **Check pods:**
```bash
kubectl get pods -A -l openchoreo.dev/component=<test-component>
```

5. **Check events:**
```bash
kubectl get events -n <test-namespace> --sort-by='.lastTimestamp'
```

**Learning moment**: Debug tests the same way you'd debug production issues - trace from test → resource → controller → logs.

---

### Command 10: Re-run a specific test

**Ask your AI agent:**
> "Help me re-run just one specific e2e test"

**Command:**
```bash
# If using Go test framework
go test ./test/e2e -run TestDeployComponent -v

# Or with make (if supported)
make e2e.test TEST_FILTER=TestDeployComponent
```

**Adjust based on project's test framework.**

---

### Command 11: Debug with verbose output

**Ask your AI agent:**
> "Run tests with maximum verbosity for debugging"

**Command:**
```bash
make e2e.test VERBOSE=true
# or
go test ./test/e2e -v -count=1
```

**Flags:**
- `-v` - Verbose output
- `-count=1` - Disable test caching
- `-timeout 30m` - Increase timeout for slow tests

---

## Phase 4: Local Pre-CI Checks (15 minutes)

**Run these before every PR to match CI:**

### Command 12: Full quality gate

**Ask your AI agent:**
> "Run all pre-CI quality checks locally"

**Commands:**
```bash
# 1. Code formatting
gofmt -s -w .
git diff --exit-code

# 2. Linter
make lint

# 3. Code generation
make generate
make manifests
git diff --exit-code

# 4. Unit tests
make test

# 5. e2e tests
make e2e.setup
make e2e.test
```

**If all pass:** You're ready to push! ✅

**Time:** 15-30 minutes total

---

### Command 13: Check for race conditions

**Ask your AI agent:**
> "Run tests with race detector enabled"

**Command:**
```bash
go test ./... -race
```

**What this detects:**
- Data races (concurrent access issues)
- Common in controller code

**If races found:** Fix them! They're bugs.

---

### Command 14: Test with different Kubernetes versions

**If project supports multiple K8s versions:**

**Ask your AI agent:**
> "Test against different Kubernetes versions"

**Commands:**
```bash
# Use different k3s image
K3S_IMAGE=rancher/k3s:v1.31.0-k3s1 make e2e.setup
make e2e.test

# Clean up
make e2e.down

# Test another version
K3S_IMAGE=rancher/k3s:v1.30.0-k3s1 make e2e.setup
make e2e.test
```

---

## Phase 5: Cleanup and Teardown (5 minutes)

### Command 15: Clean up e2e environment

**Ask your AI agent:**
> "Tear down the e2e test environment"

**Command:**
```bash
make e2e.down
```

**What this does:**
- Deletes test cluster
- Removes test data
- Cleans up resources

**Verify:**
```bash
k3d cluster list  # Should not show e2e cluster
```

---

### Command 16: Clean up test artifacts

**Ask your AI agent:**
> "Remove test output and diagnostic files"

**Commands:**
```bash
rm -rf diagnostics/
rm -f test-output.log
rm -rf coverage.out
```

---

## Phase 6: Understanding Flaky Tests (10 minutes)

### Command 17: Identify flaky tests

**Flaky test = sometimes passes, sometimes fails**

**Ask your AI agent:**
> "Help me identify and debug a flaky test"

**Run test multiple times:**
```bash
for i in {1..10}; do
  echo "Run $i"
  go test ./test/e2e -run TestProbablyFlaky -v
done
```

**If it fails once out of 10:** It's flaky!

**Common causes:**
- Timing issues (race conditions)
- Not waiting for async operations
- Assuming order of operations
- Test pollution (state from previous test)

---

### Command 18: Fix timing issues

**Ask your AI agent:**
> "Show me patterns for fixing timing-related test failures"

**Bad (flaky):**
```go
// Create resource
client.Create(ctx, component)
time.Sleep(5 * time.Second)  // ❌ Arbitrary wait
// Assert it's ready
```

**Good (reliable):**
```go
// Create resource
client.Create(ctx, component)

// Wait for condition
err := wait.PollImmediate(1*time.Second, 30*time.Second, func() (bool, error) {
    c := &v1alpha1.Component{}
    if err := client.Get(ctx, key, c); err != nil {
        return false, nil
    }
    return meta.IsStatusConditionTrue(c.Status.Conditions, "Ready"), nil
})
require.NoError(t, err)
```

**Learning moment**: Always wait for conditions, never use arbitrary sleeps.

---

## Module 7 Exit Criteria

Before moving to Module 8, verify:

- [ ] Understand the CI pipeline structure
- [ ] Can run `make e2e.setup` successfully
- [ ] Can run `make e2e.test` successfully
- [ ] Can collect diagnostics with `make e2e.diagnostics`
- [ ] Can debug test failures
- [ ] Know how to run pre-CI checks locally
- [ ] Can clean up with `make e2e.down`
- [ ] Understand how to fix flaky tests
- [ ] e2e tests pass at least once completely

### Knowledge Check

**Ask your AI agent these questions:**

1. "What's the difference between unit tests and e2e tests?"
2. "Why should I run e2e tests locally before pushing?"
3. "What diagnostics should I collect when a test fails?"
4. "How do I make tests more reliable (less flaky)?"
5. "What CI checks must pass before a PR can be merged?"

---

## e2e Testing Cheat Sheet

**Ask your AI agent to save this:**

### Setup and Run
```bash
# Full workflow
make e2e.setup      # Create environment
make e2e.test       # Run tests
make e2e.status     # Check results
make e2e.diagnostics # Collect logs (if failed)
make e2e.down       # Clean up

# Quick retry (if test failed)
make e2e.test       # Environment already exists
```

### Debugging
```bash
# View logs
kubectl logs -n openchoreo-control-plane -l app=controller-manager --tail=100

# Check test resources
kubectl get all -A | grep test

# Get events
kubectl get events -A --sort-by='.lastTimestamp' | tail -50

# Describe failing resource
kubectl describe <resource-type> <name> -n <namespace>
```

### Common Issues
```bash
# Port conflicts
lsof -i :8080      # Check what's using port
kill <PID>         # Kill if needed

# Stale cluster
make e2e.down      # Full cleanup
k3d cluster delete openchoreo  # Nuclear option

# Image not loaded
make k3d.load.controller  # Reload images

# Test hanging
Ctrl+C             # Kill test
make e2e.down      # Clean up
```

---

## Advanced e2e Techniques

### Command 19: Run tests in parallel

**If supported:**

**Ask your AI agent:**
> "Run e2e tests in parallel for faster execution"

**Command:**
```bash
go test ./test/e2e -v -parallel 4
```

**Caution:** Ensure tests don't interfere with each other!

---

### Command 20: Profile test performance

**Ask your AI agent:**
> "Profile e2e tests to find slow tests"

**Commands:**
```bash
# CPU profiling
go test ./test/e2e -cpuprofile=cpu.prof

# Memory profiling
go test ./test/e2e -memprofile=mem.prof

# View profile
go tool pprof cpu.prof
```

---

### Command 21: Run tests with different configurations

**Ask your AI agent:**
> "Test with different feature flags or configurations"

**Example:**
```bash
# Test with feature X enabled
ENABLE_FEATURE_X=true make e2e.test

# Test with different replica counts
DEFAULT_REPLICAS=3 make e2e.test
```

---

## Writing Your Own e2e Tests

**If you add a feature, add an e2e test:**

### Command 22: Create a new e2e test

**Ask your AI agent:**
> "Help me write an e2e test for my new feature"

**Structure:**
```go
func TestMyNewFeature(t *testing.T) {
    ctx := context.Background()

    // 1. Setup
    client := getTestClient()
    namespace := createTestNamespace(t)
    defer deleteTestNamespace(t, namespace)

    // 2. Create resources
    component := &v1alpha1.Component{
        // ... your test component
    }
    require.NoError(t, client.Create(ctx, component))

    // 3. Wait for expected state
    err := wait.Poll(interval, timeout, func() (bool, error) {
        // Check condition
        return true, nil
    })
    require.NoError(t, err)

    // 4. Assert behavior
    assert.Equal(t, expected, actual)

    // 5. Cleanup (defer handles this)
}
```

**Learning moment**: Good e2e tests are isolated, deterministic, and clean up after themselves.

---

## Troubleshooting CI Failures

### Issue: CI passes locally but fails in GitHub Actions

**Possible causes:**
- Different environment (OS, arch)
- Different Kubernetes version
- Timing differences (slower CI)
- State from previous tests

**Solutions:**
```bash
# Test with CI's K8s version
K3S_IMAGE=<ci-k3s-version> make e2e.setup

# Increase timeouts
TEST_TIMEOUT=60m make e2e.test

# Run tests in order (not parallel)
go test ./test/e2e -parallel 1
```

---

### Issue: e2e tests hang indefinitely

**Diagnosis:**
```bash
# In another terminal, check what's waiting
kubectl get pods -A

# Check controller logs
kubectl logs -n openchoreo-control-plane -l app=controller-manager --tail=100

# Check if waiting for resources
kubectl get all -A
```

**Solution:**
- Press Ctrl+C
- Run `make e2e.diagnostics`
- Review logs for stuck operations
- Fix the hang (usually waiting for nonexistent resource)

---

### Issue: Tests pass individually but fail together

**Cause:** Test pollution (state from one test affects another)

**Solution:**
```go
// Ensure proper cleanup in every test
defer func() {
    client.Delete(ctx, component)
    // Wait for deletion
    wait.Poll(interval, timeout, func() (bool, error) {
        err := client.Get(ctx, key, &v1alpha1.Component{})
        return errors.IsNotFound(err), nil
    })
}()

// Or use unique namespaces per test
namespace := fmt.Sprintf("test-%s", randomString(8))
```

---

## Next Steps: Module 8

Once Module 7 is complete, you're ready for **Module 8: Community Integration and Medium-Scope Planning**.

In Module 8, you will:
- Engage with the OpenChoreo community
- Participate in triage and planning
- Pick a medium-scope contribution
- Plan an implementation
- Potentially write a proposal

---

## Update Your Progress Log

**Ask your AI agent:**
> "Help me update the progress-log.md file with my Module 7 completion"

Add to [progress-log.md](progress-log.md):
- Date completed
- e2e test results (pass/fail)
- Any failures encountered and how you fixed them
- Diagnostics collected
- Key learnings about testing
- CI readiness confidence

---

## Recommended Testing Reading

1. **Go Testing Best Practices:**
   - https://go.dev/doc/tutorial/add-a-test
   - https://go.dev/blog/subtests

2. **Kubernetes Testing:**
   - https://kubernetes.io/blog/2019/03/22/kubernetes-end-to-end-testing-for-everyone/

3. **Controller-Runtime Testing:**
   - https://book.kubebuilder.io/reference/testing/envtest.html

4. **OpenChoreo Test Docs:**
```bash
cat docs/contributors/testing.md  # If exists
```

---

**Congratulations on completing Module 7!** You now understand how to ensure your changes will pass CI before submitting them. This is a critical skill for productive contribution - catching issues locally is much faster than waiting for CI feedback.

**Key Takeaway**: Running e2e tests locally before pushing saves time, reduces CI load, and increases confidence in your changes. Always run `make e2e.test` before submitting significant PRs.
