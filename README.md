# OpenChoreo Learning Plan

This repository tracks a structured onboarding path for becoming an OpenChoreo contributor.

## What This Repo Contains

### Planning and Tracking
- [openchoreo-contributor-learning-plan.md](openchoreo-contributor-learning-plan.md)
  - The full 8-module contributor plan (Controller/API focus, native Rancher Desktop + k3d setup).
- [progress-log.md](progress-log.md)
  - Module-by-module execution log with outcomes, blockers, and next steps.

### Interactive Module Guides (AI Agent-Friendly)

All guides are designed for hands-on learning with AI coding agents (Claude, GitHub Copilot, etc.). Each includes step-by-step commands, explanations, troubleshooting tips, and knowledge checks.

#### **[Module 1: Bring Up OpenChoreo on k3d](module-1-interactive-guide.md)**
Set up your local development environment with Control Plane and Data Plane.
- Create k3d cluster
- Install prerequisites (Gateway API, cert-manager, External Secrets)
- Deploy Control Plane and Data Plane
- Verify system health and access services
- **Time:** 45-60 minutes

#### **[Module 2: Deploy First Workload and Trace Resources](module-2-interactive-guide.md)**
Understand how OpenChoreo transforms declarations into running applications.
- Deploy sample applications
- Trace Component → ReleaseBinding → HTTPRoute → Deployment chain
- Understand label selectors and ownership
- Debug resource relationships
- **Time:** 30-40 minutes

#### **[Module 3: Add Build Plane and Build from Source](module-3-interactive-guide.md)**
Complete the source-to-deploy workflow with CI/CD.
- Install Build Plane with Argo Workflows
- Deploy applications from source code
- Watch workflow execution
- Understand ClusterWorkflowTemplates
- **Time:** 45-60 minutes

#### **[Module 4: Controller/API Architecture Deep Dive](module-4-interactive-guide.md)**
Build deep understanding of the codebase for contributing.
- Navigate API types and CRDs
- Understand controller reconciliation patterns
- Trace fields from API definition to runtime
- Explore services and handlers
- **Time:** 60-75 minutes

#### **[Module 5: Fast Local Development Loop](module-5-interactive-guide.md)**
Master rapid iteration for productive contribution.
- Build and deploy changes in under 2 minutes
- Run controllers locally for debugging
- Use make targets efficiently
- Set up watch scripts
- **Time:** 45-60 minutes

#### **[Module 6: First Contributor PRs](module-6-interactive-guide.md)**
Submit your first pull requests following project conventions.
- Set up Git workflow (fork, remotes, branches)
- Fix documentation and add tests
- Write conventional commits with DCO
- Create proper PR descriptions
- Respond to review feedback
- **Time:** 90-120 minutes

#### **[Module 7: CI-Parity and e2e Workflow](module-7-interactive-guide.md)**
Ensure your changes pass CI before submitting.
- Run end-to-end tests locally
- Collect diagnostics for failures
- Debug test issues
- Match CI pipeline locally
- **Time:** 60-90 minutes

#### **[Module 8: Community Integration and Medium-Scope Planning](module-8-interactive-guide.md)**
Become a sustained, impactful contributor.
- Engage with the community
- Plan medium-scope contributions
- Write proposals when needed
- Review others' PRs
- Path to maintainership
- **Time:** Ongoing

## Purpose
- Keep the learning journey explicit and trackable.
- Record evidence of completed milestones before moving to deeper contribution work.
- Maintain a repeatable onboarding path for future contributors.
- Enable AI-assisted learning for hands-on skill development.

## How to Use

### For Self-Paced Learning
1. Follow [openchoreo-contributor-learning-plan.md](openchoreo-contributor-learning-plan.md) module by module.
2. Update [progress-log.md](progress-log.md) as you complete milestones.
3. Run all commands from the main OpenChoreo source repo (`code/openchoreo`) unless explicitly stated otherwise.
4. Use this repo for planning and tracking only; implement code and PR work in the source repo.

### Learning with AI Coding Agents (Recommended)
1. Start with [Module 1 Interactive Guide](module-1-interactive-guide.md) for a hands-on, conversational learning experience.
2. Work with your AI agent (Claude, GitHub Copilot, etc.) to execute commands step-by-step.
3. Ask your AI agent questions about each command to deepen your understanding:
   - "Why do we need this component?"
   - "What happens if this command fails?"
   - "Show me how to verify this worked"
4. Each guide includes:
   - Clear phase-by-phase structure
   - Command explanations ("What this does:")
   - Learning moments (key concepts)
   - Troubleshooting sections
   - Knowledge check questions
   - Exit criteria for completion
5. Update [progress-log.md](progress-log.md) after completing each module.

### Example Workflow with AI Agent

**You:** "Let's work on Module 1. Start with Phase 2, Command 1"

**AI Agent:** *Shows current directory, explains why verification matters*

**You:** "Now create the k3d cluster"

**AI Agent:** *Executes command, explains what's happening, shows expected output*

**You:** "Why do we need cert-manager?"

**AI Agent:** *Explains TLS certificate management, gives context*

This conversational approach helps you learn by doing while building deep understanding.

## Prerequisites

Before starting Module 1:
- **Module 0 complete** (from learning plan):
  - Rancher Desktop installed and configured with Docker runtime
  - Required tools: Go 1.24+, k3d 5.8+, kubectl 1.31-1.33, helm 3.16+
  - Run `./check-tools.sh` successfully
  - Git remotes configured (origin = your fork, upstream = openchoreo/openchoreo)

## Recommended Learning Path

**Weeks 1-2: Environment and Understanding**
- Module 0: Preflight (prerequisite)
- Module 1: Cluster setup
- Module 2: First workload
- Module 3: Build plane

**Weeks 3-4: Code Deep Dive**
- Module 4: Architecture study
- Module 5: Development workflow

**Weeks 5-6: Contributing**
- Module 6: First PRs (2 PRs minimum)
- Module 7: Testing practices

**Ongoing: Community**
- Module 8: Sustained contribution

**Total estimated time:** 6-8 weeks at 5-10 hours/week

## Tips for Success

1. **Don't skip modules** - Each builds on the previous
2. **Update progress log** - Track what worked and what didn't
3. **Ask questions** - Use your AI agent as a learning partner
4. **Run commands yourself** - Don't just read, do!
5. **Fix issues as you find them** - Document discrepancies for potential PRs
6. **Join the community early** - Introduce yourself after Module 1
7. **Take breaks** - Complex topics need time to absorb

## Common Questions

**Q: Can I skip Module 0?**
A: No. Without proper tools installed, later modules will fail.

**Q: Do I need to finish all modules before contributing?**
A: You can submit small PRs after Module 5, but Module 6 teaches the proper workflow.

**Q: What if I get stuck?**
A: Ask your AI agent for help, check troubleshooting sections, or ask in community channels.

**Q: Can I use a different local setup (not k3d)?**
A: The guides assume k3d. Other setups may work but aren't covered.

**Q: How do I know I'm ready for Module N+1?**
A: Each module has "Exit Criteria" - verify all items before proceeding.

## Success Stories

After completing this onboarding:
- You'll have a working OpenChoreo development environment
- You'll understand the full system architecture
- You'll have submitted at least 2 PRs
- You'll be able to develop and test changes in under 2 minutes
- You'll be a recognized community member
- You'll be ready for sustained, impactful contributions

## Contributing to This Onboarding

Found an issue in the guides? Submit a PR!
- Outdated commands
- Missing explanations
- Unclear instructions
- New tips and tricks

This onboarding is itself a living contribution.

## Additional Resources

- [OpenChoreo Main Repository](https://github.com/openchoreo/openchoreo)
- [OpenChoreo Documentation](https://openchoreo.dev/docs/)
- [Kubernetes Documentation](https://kubernetes.io/docs/home/)
- [Kubebuilder Book](https://book.kubebuilder.io/)
- [Go Documentation](https://go.dev/doc/)

## Current Focus

After completing the onboarding, contributors typically focus on:
- **First contribution target:** Fix outdated commands in `docs/contributors/contribute.md`
- **Second contribution target:** Small Controller/API bug fix or test improvement
- **Medium-scope target:** Feature implementation from issue backlog

---

**Ready to start?** Begin with [Module 1: Bring Up OpenChoreo on k3d](module-1-interactive-guide.md)

**Questions?** Open an issue or discussion in this repository, or ask in OpenChoreo community channels.

**Good luck on your contributor journey!** 🚀
