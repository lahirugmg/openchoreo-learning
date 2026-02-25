# Module 6: First Contributor PRs - Interactive Guide

**Learn by Doing with Your AI Coding Agent**

This guide walks you through submitting your first pull requests to OpenChoreo, following all project conventions and best practices.

## Prerequisites

Before starting Module 6:
- Modules 1-5 complete (can develop and test locally)
- GitHub account configured
- Git installed and configured
- Fork of openchoreo/openchoreo repository
- Understanding of Git workflows

## Learning Objectives

By the end of this module, you will:
1. Set up your fork and remotes correctly
2. Create feature branches following conventions
3. Make changes following project standards
4. Write conventional commit messages
5. Sign commits with DCO
6. Run pre-PR quality checks
7. Submit your first two PRs
8. Respond to review feedback

## Time Estimate: 90-120 minutes (including research and writing)

---

## Phase 1: Git Workflow Setup (15 minutes)

### Command 1: Verify your fork

**Ask your AI agent:**
> "Help me verify I have a fork of openchoreo/openchoreo"

**Check on GitHub:**
1. Go to https://github.com/openchoreo/openchoreo
2. Click "Fork" (if you haven't already)
3. Wait for fork creation
4. Your fork URL: `https://github.com/YOUR_USERNAME/openchoreo`

---

### Command 2: Check remote configuration

**Ask your AI agent:**
> "Show me my current Git remotes"

**Command:**
```bash
cd /Users/lahirus/work/pet-projects/open-choreo/openchoreo-learning/code/openchoreo
git remote -v
```

**Expected output:**
```
origin    https://github.com/YOUR_USERNAME/openchoreo.git (fetch)
origin    https://github.com/YOUR_USERNAME/openchoreo.git (push)
upstream  https://github.com/openchoreo/openchoreo.git (fetch)
upstream  https://github.com/openchoreo/openchoreo.git (push)
```

**If `upstream` is missing:**
```bash
git remote add upstream https://github.com/openchoreo/openchoreo.git
git remote -v
```

**Learning moment**: `origin` = your fork, `upstream` = main project. You push to origin, create PRs from origin to upstream.

---

### Command 3: Sync with upstream

**Ask your AI agent:**
> "Sync my main branch with upstream to get latest changes"

**Commands:**
```bash
git checkout main
git fetch upstream
git merge upstream/main
git push origin main
```

**What this does:**
- Fetches latest changes from openchoreo/openchoreo
- Merges them into your local main
- Pushes to your fork

**Always do this before starting new work!**

---

### Command 4: Configure commit signing

**Ask your AI agent:**
> "Help me set up DCO (Developer Certificate of Origin) signing"

**Check current config:**
```bash
git config user.name
git config user.email
```

**If not set:**
```bash
git config user.name "Your Name"
git config user.email "your.email@example.com"
```

**DCO means:** You certify you have the right to contribute this code.

**How to sign:** Add `-s` flag to commits:
```bash
git commit -s -m "Your message"
```

**Learning moment**: Most open source projects require sign-off. It's a legal protection for the project.

---

## Phase 2: PR #1 - Documentation Fix (30 minutes)

Let's fix outdated commands in the contributor documentation (from the learning plan).

### Command 5: Create a feature branch

**Ask your AI agent:**
> "Create a feature branch for fixing documentation"

**Commands:**
```bash
git checkout main
git pull upstream main
git checkout -b docs/fix-contributor-guide-commands
```

**Branch naming convention:**
- `docs/` - Documentation changes
- `feat/` - New features
- `fix/` - Bug fixes
- `refactor/` - Code refactoring
- `test/` - Test improvements

---

### Command 6: Identify outdated commands

**Ask your AI agent:**
> "Help me find outdated commands in docs/contributors/contribute.md"

**Command:**
```bash
cat docs/contributors/contribute.md
```

**Look for:**
- Commands that don't work
- Old version numbers
- Missing prerequisites
- Incorrect paths

**Common issues:**
- `make` targets that have been renamed
- Tool versions that are outdated
- Steps that are no longer needed

---

### Command 7: Research correct commands

**Ask your AI agent:**
> "Help me verify the correct commands to use"

**Commands to verify:**
```bash
# Check available make targets
make help

# Try suggested commands
make <command>

# Check tool versions
kubectl version --client
helm version --short
go version
```

**Compare with:**
- Current installation guides in `install/k3d/`
- Makefile targets
- Your own experience from Modules 1-5

---

### Command 8: Make the changes

**Ask your AI agent:**
> "Help me edit the documentation with corrections"

**Command:**
```bash
# Open in your editor
code docs/contributors/contribute.md
# or
vim docs/contributors/contribute.md
```

**Example changes:**
```diff
- make install-tools
+ ./check-tools.sh

- kubectl version 1.28.x
+ kubectl version 1.31.x to 1.33.x

- make deploy
+ make k3d.build.controller && make k3d.load.controller && make k3d.update.controller
```

**Tips:**
- Be specific about changes
- Verify each command works
- Update version numbers accurately
- Fix formatting issues

---

### Command 9: Verify your changes

**Ask your AI agent:**
> "Help me verify my documentation changes are correct"

**Commands:**
```bash
# View your changes
git diff docs/contributors/contribute.md

# Try running the commands you documented (if applicable)
<test commands>
```

**Checklist:**
- [ ] All commands are correct
- [ ] Version numbers are current
- [ ] Formatting is consistent
- [ ] No typos

---

### Command 10: Commit with DCO

**Ask your AI agent:**
> "Help me create a proper commit following conventional commit format"

**Command:**
```bash
git add docs/contributors/contribute.md
git commit -s -m "docs: update outdated commands in contributor guide

- Update kubectl version requirements (1.28.x -> 1.31.x-1.33.x)
- Fix make targets (install-tools -> check-tools.sh)
- Update deployment commands to use k3d workflow
- Remove deprecated steps

Fixes commands that no longer work in current setup."
```

**Conventional commit format:**
```
<type>: <subject>

<body>

<footer>
```

**Types:**
- `docs:` - Documentation changes
- `feat:` - New feature
- `fix:` - Bug fix
- `refactor:` - Code refactoring
- `test:` - Test changes
- `chore:` - Build/tool changes

**Learning moment**: Conventional commits make history readable and enable automated changelog generation.

---

### Command 11: Push to your fork

**Ask your AI agent:**
> "Push my documentation fix branch to my fork"

**Command:**
```bash
git push origin docs/fix-contributor-guide-commands
```

**Expected output:** Branch pushed successfully

---

### Command 12: Create the pull request

**Ask your AI agent:**
> "Help me create a pull request on GitHub"

**Steps:**
1. Go to https://github.com/YOUR_USERNAME/openchoreo
2. GitHub shows "Compare & pull request" button
3. Click it
4. **Title:** `docs: update outdated commands in contributor guide`
5. **Description:**
```markdown
## Summary
Updates outdated commands and version requirements in the contributor guide to match current setup.

## Changes
- Updated kubectl version requirements (1.28.x -> 1.31.x-1.33.x)
- Fixed make targets (install-tools -> check-tools.sh)
- Updated deployment commands to use k3d workflow
- Removed deprecated installation steps

## Testing
- Verified all commands work in current environment (Modules 1-5 setup)
- Confirmed version numbers match current toolchain

## Related Issue
Addresses contributor onboarding improvements (if there's an issue, link it)
```

6. **Select base:** `main` (from `openchoreo/openchoreo`)
7. **Reviewers:** Leave blank (maintainers will assign)
8. Click "Create pull request"

**PR URL:** Save this for your progress log!

---

## Phase 3: PR #2 - Small Code Improvement (45 minutes)

Now let's make a small controller or test improvement.

### Command 13: Find a good first issue

**Ask your AI agent:**
> "Help me find a small issue to work on"

**Options:**

**Option A: Browse issues**
1. Go to https://github.com/openchoreo/openchoreo/issues
2. Filter: `is:open is:issue label:"good first issue"`
3. Pick one that interests you

**Option B: Fix a TODO in code**
```bash
grep -r "TODO" internal/controller/ pkg/ | head -20
```

**Option C: Add a missing test**
```bash
# Find controllers with low test coverage
find internal/controller -name '*_test.go' -o -name '*_controller.go' | sort
```

**Option D: Improve error messages**
```bash
grep -r 'errors.New("' internal/ | head -10
```

**For this guide, let's assume you're adding a test case.**

---

### Command 14: Create a feature branch

**Ask your AI agent:**
> "Create a feature branch for my test improvement"

**Commands:**
```bash
git checkout main
git pull upstream main
git checkout -b test/add-component-validation-test
```

---

### Command 15: Write the test

**Ask your AI agent:**
> "Help me add a test case for Component validation"

**Example: Test that Component rejects invalid port numbers**

**File:** `internal/controller/component_controller_test.go` (or similar)

```go
func TestComponentValidation_InvalidPort(t *testing.T) {
    ctx := context.Background()

    component := &v1alpha1.Component{
        ObjectMeta: metav1.ObjectMeta{
            Name:      "test-component",
            Namespace: "default",
        },
        Spec: v1alpha1.ComponentSpec{
            Image: "test-image:latest",
            Port:  -1, // Invalid port
        },
    }

    // Create fake client
    client := fake.NewClientBuilder().
        WithScheme(scheme).
        Build()

    // Attempt to create
    err := client.Create(ctx, component)

    // Should be rejected by validation
    assert.Error(t, err)
    assert.Contains(t, err.Error(), "port")
}
```

**Adjust based on actual project structure.**

---

### Command 16: Run the test

**Ask your AI agent:**
> "Run my new test to verify it works"

**Commands:**
```bash
# Run all tests
make test

# Run specific test
go test ./internal/controller -run TestComponentValidation_InvalidPort -v
```

**Expected output:** Test passes

---

### Command 17: Run quality checks

**Ask your AI agent:**
> "Run all pre-PR quality checks"

**Commands:**
```bash
# Run linter
make lint

# Check code generation is up to date
make generate
make manifests
git diff  # Should show no changes

# Run all tests
make test

# Check code formatting
gofmt -s -w .
git diff  # Should show no changes
```

**Fix any issues before committing.**

---

### Command 18: Commit your test

**Ask your AI agent:**
> "Create a proper commit for my test improvement"

**Command:**
```bash
git add internal/controller/component_controller_test.go
git commit -s -m "test: add validation test for invalid component port

Add test case to verify that Components with invalid port numbers
(negative or > 65535) are properly rejected by validation.

Improves test coverage for Component CRD validation."
```

---

### Command 19: Push and create PR

**Ask your AI agent:**
> "Push my test improvement and create a PR"

**Commands:**
```bash
git push origin test/add-component-validation-test
```

**Create PR on GitHub (same process as PR #1):**
- **Title:** `test: add validation test for invalid component port`
- **Description:**
```markdown
## Summary
Adds test coverage for Component port validation to ensure invalid ports are rejected.

## Changes
- Added `TestComponentValidation_InvalidPort` test case
- Verifies negative port numbers are rejected
- Improves validation test coverage

## Testing
```bash
make test
go test ./internal/controller -run TestComponentValidation_InvalidPort -v
```

## Checklist
- [x] Tests pass locally
- [x] Linter passes
- [x] Code generation up to date
- [x] DCO sign-off included
```

**Submit the PR!**

---

## Phase 4: PR Best Practices (15 minutes)

### Command 20: Monitor your PRs

**Ask your AI agent:**
> "Help me monitor my pull requests and respond to feedback"

**Check regularly:**
1. GitHub notifications
2. PR comments
3. CI/CD status checks

**If CI fails:**
```bash
# Check what failed (visible on PR page)
# Fix locally
git commit -s -m "fix: address CI feedback"
git push origin <branch-name>
```

**Responding to review feedback:**
1. Read comments carefully
2. Ask questions if unclear
3. Make requested changes
4. Push changes (automatically updates PR)
5. Reply to comments when addressed

---

### Command 21: Squash commits if requested

**Ask your AI agent:**
> "Help me squash my commits if reviewers request it"

**If you have multiple commits and need to squash:**
```bash
# Interactive rebase
git rebase -i HEAD~3  # Adjust number based on commit count

# In the editor:
# Change 'pick' to 'squash' for commits you want to combine
# Save and exit

# Force push (PR branch only!)
git push origin <branch-name> --force
```

**Learning moment**: Some projects prefer single commits per PR. Check project guidelines.

---

### Command 22: Keep your PR up to date

**If upstream main has advanced:**

**Ask your AI agent:**
> "Help me rebase my PR on the latest main"

**Commands:**
```bash
git checkout <your-branch>
git fetch upstream
git rebase upstream/main

# If conflicts, resolve them
git add <resolved-files>
git rebase --continue

# Force push
git push origin <your-branch> --force
```

---

## Phase 5: After Merge (5 minutes)

### Command 23: Clean up after merge

**Once your PR is merged:**

**Ask your AI agent:**
> "Help me clean up after my PR was merged"

**Commands:**
```bash
# Switch to main
git checkout main

# Update from upstream
git pull upstream main
git push origin main

# Delete local branch
git branch -d docs/fix-contributor-guide-commands

# Delete remote branch
git push origin --delete docs/fix-contributor-guide-commands
```

---

### Command 24: Celebrate!

**Ask your AI agent:**
> "Log my contributor achievement"

**You're now an OpenChoreo contributor!** 🎉

---

## Module 6 Exit Criteria

Before moving to Module 7, verify:

- [ ] Two PRs submitted to OpenChoreo
- [ ] PR #1: Documentation fix
- [ ] PR #2: Code/test improvement
- [ ] All commits include DCO sign-off
- [ ] Followed conventional commit format
- [ ] Pre-PR checks passed (lint, test, code-gen)
- [ ] Proper PR descriptions written
- [ ] Know how to respond to review feedback
- [ ] Understand the fork/branch/PR workflow

### Knowledge Check

**Ask your AI agent these questions:**

1. "What's the difference between origin and upstream remotes?"
2. "Why do we need DCO sign-off?"
3. "What's the conventional commit format for a bug fix?"
4. "How do I update my PR after reviewer feedback?"
5. "When should I squash my commits?"

---

## PR Checklist Template

**Ask your AI agent to save this for future PRs:**

```markdown
## Before Creating PR

- [ ] Synced with upstream main
- [ ] Feature branch created with proper naming
- [ ] Changes made and tested locally
- [ ] All tests pass (`make test`)
- [ ] Linter passes (`make lint`)
- [ ] Code generation up to date (`make generate manifests`)
- [ ] Commits follow conventional format
- [ ] All commits signed with DCO (`-s`)
- [ ] Changes pushed to fork

## PR Description Must Include

- [ ] Summary of changes
- [ ] What problem this solves
- [ ] How to test
- [ ] Related issue (if any)
- [ ] Breaking changes (if any)

## After Creating PR

- [ ] Monitor CI status
- [ ] Respond to reviewer comments
- [ ] Update PR if requested
- [ ] Keep PR up to date with main (rebase if needed)
```

---

## Common PR Mistakes and Fixes

### Mistake: Forgot DCO sign-off

**Symptom:** CI fails with "DCO check failed"

**Fix:**
```bash
# Amend last commit
git commit --amend -s --no-edit
git push origin <branch> --force

# For older commits
git rebase -i HEAD~<N>
# Mark commits as 'edit', then:
git commit --amend -s --no-edit
git rebase --continue
git push origin <branch> --force
```

---

### Mistake: Wrong commit message format

**Symptom:** Reviewer asks for format change

**Fix:**
```bash
git commit --amend
# Edit the message in your editor
git push origin <branch> --force
```

---

### Mistake: Committed to main instead of branch

**Symptom:** Changes on main branch

**Fix:**
```bash
# Create branch from main
git checkout -b fix/my-changes

# Reset main
git checkout main
git reset --hard upstream/main

# Continue on feature branch
git checkout fix/my-changes
```

---

### Mistake: Forgot to run code generation

**Symptom:** CI fails "generated files out of sync"

**Fix:**
```bash
make generate
make manifests
git add -A
git commit -s -m "chore: update generated files"
git push origin <branch>
```

---

## Finding Good Issues to Work On

**Ask your AI agent:**
> "Help me find issues I can contribute to"

**Strategies:**

1. **Good First Issue label:**
```
https://github.com/openchoreo/openchoreo/issues?q=is%3Aissue+is%3Aopen+label%3A%22good+first+issue%22
```

2. **Help Wanted label:**
```
https://github.com/openchoreo/openchoreo/issues?q=is%3Aissue+is%3Aopen+label%3A%22help+wanted%22
```

3. **Documentation issues:**
```
https://github.com/openchoreo/openchoreo/issues?q=is%3Aissue+is%3Aopen+label%3Adocumentation
```

4. **Ask in community:**
- Slack/Discord (if available)
- Issue comments: "I'd like to work on this"
- Community meetings

5. **Create your own issue:**
- Found a bug? Report it
- Have an idea? Propose it
- Ask maintainers if it's welcome before large PRs

---

## Responding to Review Feedback

**Common scenarios:**

### Reviewer: "Can you add tests?"

**Response:**
```
Good point! I'll add test coverage for:
- Case A
- Case B

Will update shortly.
```

**Then add tests and push.**

---

### Reviewer: "This could be simplified"

**Response:**
```
Thanks for the suggestion! You're right, that's cleaner.
Updated to use your approach.
```

**Don't take it personally - reviews improve code quality.**

---

### Reviewer: "Please rebase on main"

**Response:**
```
Done! Rebased on latest main (commit abc123).
```

**Commands:**
```bash
git fetch upstream
git rebase upstream/main
git push origin <branch> --force
```

---

## Next Steps: Module 7

Once Module 6 is complete, you're ready for **Module 7: CI-Parity and e2e Workflow**.

In Module 7, you will:
- Understand the CI/CD pipeline
- Run e2e tests locally
- Interpret test results
- Debug failures
- Ensure your changes pass CI before submission

---

## Update Your Progress Log

**Ask your AI agent:**
> "Help me update the progress-log.md file with my Module 6 completion"

Add to [progress-log.md](progress-log.md):
- Date completed
- PR #1 URL and status
- PR #2 URL and status
- What you learned
- Review feedback received
- Any challenges overcome

---

## Contributor Recognition

**Add yourself to CONTRIBUTORS (if project has one):**
```bash
# Check if file exists
cat CONTRIBUTORS.md

# Add your name
echo "- Your Name (@yourusername)" >> CONTRIBUTORS.md
git commit -s -m "docs: add myself to contributors"
```

**Update your GitHub profile:**
- Pin the OpenChoreo repo
- Add "OpenChoreo Contributor" to bio
- Share your PRs on social media!

---

**Congratulations on completing Module 6!** You're now an active OpenChoreo contributor. Your PRs help improve the project for everyone. Keep contributing!

**Key Takeaway**: Quality contributions follow conventions, include tests, have clear commit messages, and respond professionally to feedback. This applies to all open source projects.
