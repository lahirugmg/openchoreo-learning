# Module 8: Community Integration and Medium-Scope Planning - Interactive Guide

**Learn by Doing with Your AI Coding Agent**

This guide teaches you how to move beyond first contributions to sustained, impactful participation in the OpenChoreo community.

## Prerequisites

Before starting Module 8:
- Modules 1-7 complete (can develop, test, and contribute)
- At least 2 PRs submitted (Module 6)
- Comfortable with the codebase and workflows
- Ready to take on larger contributions

## Learning Objectives

By the end of this module, you will:
1. Engage with the OpenChoreo community effectively
2. Participate in issue triage and planning
3. Identify medium-scope contribution opportunities
4. Plan an implementation approach
5. Write a proposal (if needed)
6. Build relationships with maintainers and contributors
7. Become a sustained contributor

## Time Estimate: Ongoing (2-4 hours initial setup + ongoing participation)

---

## Phase 1: Understanding the Community (15 minutes)

### Command 1: Find community channels

**Ask your AI agent:**
> "Help me find all OpenChoreo community channels and communication methods"

**Research:**

1. **GitHub:**
   - https://github.com/openchoreo/openchoreo
   - Issues, discussions, pull requests

2. **Documentation site:**
```bash
cat README.md | grep -i "community\|chat\|slack\|discord"
```

3. **GOVERNANCE file:**
```bash
cat GOVERNANCE.md
```

4. **Community meetings:**
```bash
cat docs/community/meetings.md  # If exists
```

**Typical channels:**
- GitHub Issues
- GitHub Discussions
- Slack/Discord (check README)
- Community calls
- Mailing lists

---

### Command 2: Read the governance model

**Ask your AI agent:**
> "Show me OpenChoreo's governance structure"

**Command:**
```bash
cat GOVERNANCE.md
```

**What to understand:**
- Who are the maintainers?
- How are decisions made?
- How can you become a maintainer?
- What's the contribution ladder?

**Typical structure:**
- **Contributors** - Anyone who contributes
- **Reviewers** - Can review PRs
- **Approvers** - Can approve merges
- **Maintainers** - Project stewards

**Learning moment**: Every project has a path from contributor to maintainer. Understand it to set your goals.

---

### Command 3: Review the code of conduct

**Ask your AI agent:**
> "Show me the code of conduct"

**Command:**
```bash
cat CODE_OF_CONDUCT.md
```

**Key points:**
- Be respectful
- Be welcoming to newcomers
- Assume good intent
- Report inappropriate behavior

---

## Phase 2: Engaging with the Community (30 minutes)

### Command 4: Introduce yourself

**Where to introduce yourself:**

**GitHub Discussions (if available):**
1. Go to https://github.com/openchoreo/openchoreo/discussions
2. Create a new discussion in "Introductions"
3. Share:
   - Your background
   - What interests you about OpenChoreo
   - What you've worked on so far (Modules 1-7!)
   - Areas you'd like to contribute to

**Example:**
```markdown
## Introduction - [Your Name]

Hi everyone! 👋

I'm [Name], and I've been learning OpenChoreo through the contributor onboarding modules.
So far I've:
- Set up local development environment
- Deployed applications from source
- Submitted PRs for [doc fix] and [test improvement]

I'm particularly interested in [controller development / build plane / observability / etc.].
Looking forward to contributing more and learning from the community!

Happy to help other newcomers with the onboarding process.
```

---

### Command 5: Join community calls

**If the project has regular calls:**

**Ask your AI agent:**
> "Help me find and join OpenChoreo community calls"

**Look for:**
- Meeting schedule (weekly, biweekly?)
- Calendar invite or meeting link
- Agenda documents
- Past meeting notes

**First call tips:**
- Introduce yourself briefly
- Listen and observe
- Ask questions
- Volunteer for tasks

**Learning moment**: Community calls are where strategic decisions happen. Regular attendance builds relationships.

---

### Command 6: Watch repository for activity

**Ask your AI agent:**
> "Set up GitHub notifications for OpenChoreo"

**On GitHub:**
1. Go to https://github.com/openchoreo/openchoreo
2. Click "Watch" button
3. Choose notification level:
   - **All Activity** - For deep involvement
   - **Participating and @mentions** - For moderate involvement
   - **Custom** - Pick specific events

**Manage notifications:**
- Set up filters in email
- Check GitHub notifications daily
- Respond to @mentions promptly

---

### Command 7: Review and comment on issues

**Start with low-barrier participation:**

**Ask your AI agent:**
> "Help me find issues I can comment on helpfully"

**Strategies:**

1. **Help newcomers:**
```
Find: issues labeled "question" or "help wanted"
Comment: Share your experience from Modules 1-7
```

2. **Reproduce bugs:**
```
Find: issues labeled "bug" without "confirmed"
Comment: "I can reproduce this on [version]. Steps: ..."
```

3. **Clarify requirements:**
```
Find: feature requests with unclear requirements
Comment: "Could you clarify...?" or "Would this cover...?"
```

4. **Add context:**
```
Find: issues related to your expertise
Comment: "This is related to..."
```

**Example comment:**
```markdown
I encountered this issue during Module 2 of the onboarding.

The problem occurs when [specific scenario].

Workaround: [if you found one]

I'd be interested in working on a fix if this is confirmed as a bug.
```

---

## Phase 3: Finding Medium-Scope Work (30 minutes)

### Command 8: Identify medium-scope opportunities

**Ask your AI agent:**
> "Help me find medium-scope issues to work on"

**Command:**
```bash
# Browse issues on GitHub
open https://github.com/openchoreo/openchoreo/issues
```

**Filters:**
- Not labeled "good first issue" (too small)
- Not labeled "epic" or "large" (too big)
- Labeled "help wanted" or "enhancement"
- Clear requirements
- Matches your interests

**Medium scope means:**
- 1-2 weeks of work
- Affects multiple files/components
- Requires design thinking
- May need proposal discussion

**Examples:**
- Add a new controller feature
- Improve error handling across a component
- Add observability to a subsystem
- Performance optimization
- New API field with validation

---

### Command 9: Assess complexity

**Before committing, assess the task:**

**Ask your AI agent:**
> "Help me assess if this issue is appropriate for my skill level"

**Questions to ask yourself:**

1. **Do I understand the problem?**
   - Read the issue thoroughly
   - Check linked issues/PRs
   - Ask questions if unclear

2. **Do I know the affected code?**
```bash
# Find related files
grep -r "KeywordFromIssue" internal/ pkg/

# Check how complex they are
wc -l <relevant-files>
```

3. **Can I test it?**
   - Will existing tests cover it?
   - Can I write new tests?
   - How will I verify it works?

4. **Is anyone already working on it?**
   - Check issue comments
   - Look for open PRs
   - Ask: "Is anyone working on this?"

**If yes to all:** It's a good fit!

---

### Command 10: Claim the issue

**Ask your AI agent:**
> "Help me properly claim an issue I want to work on"

**Comment on the issue:**
```markdown
I'd like to work on this issue.

**My understanding:**
[Briefly summarize the problem and proposed solution]

**My approach:**
1. [Step 1]
2. [Step 2]
3. [Step 3]

**Questions:**
- [Any clarifications needed]

**Timeline:**
I can start immediately and aim to have a PR ready within [timeframe].

cc @maintainer-name (if you know who to tag)
```

**Wait for:**
- Maintainer approval
- Feedback on approach
- Assignment of issue to you

**Learning moment**: Always communicate intent before starting large work. Avoids duplicate effort and ensures alignment.

---

## Phase 4: Planning Your Implementation (45 minutes)

### Command 11: Research existing patterns

**Ask your AI agent:**
> "Help me find similar existing implementations to learn from"

**Commands:**
```bash
# Find similar features
grep -r "SimilarFeature" internal/controller/

# Find similar API types
grep -r "type.*Spec struct" api/v1alpha1/

# Find similar tests
find . -name '*_test.go' | xargs grep -l "SimilarTest"
```

**Look for:**
- Code patterns to follow
- Conventions (naming, structure)
- Test patterns
- Error handling approaches

---

### Command 12: Design your approach

**Ask your AI agent:**
> "Help me design the implementation for my medium-scope task"

**Create a design document (save as `design-notes.md`):**

```markdown
# Implementation Plan: [Feature Name]

## Problem Statement
[What issue are you solving?]

## Proposed Solution
[High-level approach]

## API Changes
[If any - show before/after]

### Before
```yaml
apiVersion: openchoreo.dev/v1alpha1
kind: Component
spec:
  image: foo
```

### After
```yaml
apiVersion: openchoreo.dev/v1alpha1
kind: Component
spec:
  image: foo
  newField: bar  # NEW
```

## Implementation Steps

1. **API Changes** (if needed)
   - Add fields to `api/v1alpha1/component_types.go`
   - Run `make generate manifests`
   - Add validation

2. **Controller Changes**
   - Update `internal/controller/component_controller.go`
   - Add reconciliation logic
   - Handle new field

3. **Tests**
   - Unit tests for controller logic
   - e2e test for end-to-end flow
   - Update existing tests if needed

4. **Documentation**
   - Update CRD reference
   - Add examples
   - Update user guide

## Testing Plan
- [ ] Unit tests for X
- [ ] e2e test for Y
- [ ] Manual testing: Z

## Open Questions
- Q1: ...
- Q2: ...

## Timeline
- Week 1: API and controller changes
- Week 2: Tests and documentation
```

---

### Command 13: Determine if proposal is needed

**Ask your AI agent:**
> "Help me determine if I need to write a formal proposal"

**Proposal needed if:**
- Changes API in breaking way
- Adds new CRD
- Changes architecture significantly
- Affects multiple components
- Has security implications

**Proposal NOT needed if:**
- Internal refactoring
- Bug fix
- Adding optional feature
- Documentation
- Tests

**Check project guidelines:**
```bash
cat docs/proposals/README.md
```

---

### Command 14: Write a proposal (if needed)

**Ask your AI agent:**
> "Help me write a proposal following the project template"

**Command:**
```bash
# Copy template
cp docs/proposals/xxxx-proposal-template.md docs/proposals/0XXX-my-feature.md

# Edit it
code docs/proposals/0XXX-my-feature.md
```

**Template sections:**
- Title
- Authors
- Status (Draft, Accepted, Implemented)
- Summary
- Motivation
- Proposal
- Design Details
- Implementation Plan
- Alternatives Considered
- References

**Learning moment**: Proposals ensure everyone understands and agrees with the approach before code is written. Saves time in review.

---

## Phase 5: Executing and Collaborating (Ongoing)

### Command 15: Create implementation branch

**Ask your AI agent:**
> "Create a feature branch for my medium-scope implementation"

**Commands:**
```bash
git checkout main
git pull upstream main
git checkout -b feat/my-medium-scope-feature
```

---

### Command 16: Work in incremental PRs

**Don't submit one huge PR. Break it up:**

**Ask your AI agent:**
> "Help me break my work into smaller PRs"

**Example breakdown:**

**PR 1: API Changes**
- Add new fields to CRD
- Run code generation
- Add validation
- No controller logic yet

**PR 2: Controller Logic**
- Implement reconciliation for new field
- Add unit tests

**PR 3: e2e Tests**
- Add end-to-end test
- Update test documentation

**PR 4: Documentation**
- User guide updates
- API reference updates
- Examples

**Benefits:**
- Easier to review
- Faster feedback
- Can merge parts incrementally
- Reduces risk

---

### Command 17: Share progress updates

**Ask your AI agent:**
> "Help me write progress updates for my issue"

**Comment on the issue regularly:**

```markdown
## Progress Update - Week 1

**Completed:**
- [x] API changes (PR #123)
- [x] Basic controller logic

**In Progress:**
- [ ] Unit tests

**Blocked:**
- Waiting for feedback on API design in PR #123

**Next Steps:**
- Complete unit tests
- Start e2e test

**ETA:**
- PR ready for review by end of week
```

---

### Command 18: Seek feedback early

**Ask your AI agent:**
> "Help me request early feedback on my work"

**Strategies:**

1. **Draft PRs:**
   - Open PR as "Draft"
   - Ask for early feedback
   - Mark "Ready for review" when done

2. **RFC (Request for Comments):**
   - Share design doc in issue
   - Ask specific questions
   - Tag relevant maintainers

3. **Show progress in calls:**
   - Demo in community call
   - Get live feedback

**Example:**
```markdown
Opening this as a draft PR to get early feedback on the approach.

**Feedback requested on:**
1. API field naming - does `retryPolicy` sound right?
2. Controller structure - am I missing any edge cases?
3. Test coverage - is this sufficient?

@maintainer cc for review when you have time
```

---

## Phase 6: Becoming a Sustained Contributor (Ongoing)

### Command 19: Build your expertise area

**Pick an area to specialize in:**

**Ask your AI agent:**
> "Help me identify an expertise area within OpenChoreo"

**Options:**
- Build plane and workflows
- Controller development
- Gateway and routing
- Observability integration
- CLI tools
- Documentation
- Testing infrastructure

**Become the go-to person for that area:**
- Review PRs in that area
- Answer questions
- Write documentation
- Propose improvements

---

### Command 20: Review others' PRs

**Ask your AI agent:**
> "Help me start reviewing pull requests"

**Find PRs to review:**
```
https://github.com/openchoreo/openchoreo/pulls
```

**Review checklist:**
- Does it solve the problem?
- Are tests included?
- Is code clear and maintainable?
- Does it follow project conventions?
- Are docs updated?

**Leave helpful comments:**
```markdown
Thanks for this PR! A few thoughts:

**Positive:**
- Great test coverage!
- Clear commit messages

**Suggestions:**
- Could we extract this logic into a helper function for reusability?
- Consider adding a comment explaining why we need this check

**Questions:**
- How does this handle the case where...?

Not a maintainer, but this looks good to me overall. cc @maintainer for approval
```

**Learning moment**: Reviewing others' code is one of the best ways to learn and build credibility.

---

### Command 21: Mentor newcomers

**Pay it forward:**

**Ask your AI agent:**
> "Help me create resources to help other newcomers"

**Ideas:**
- Answer "good first issue" questions
- Improve onboarding docs
- Create tutorials/blog posts
- Help in Slack/Discord
- Review first-time contributor PRs gently

**Example:**
```markdown
Welcome to OpenChoreo! @new-contributor

I went through this same learning curve a few months ago. Here are some tips:

1. Start with Modules 1-3 to understand the system
2. The [specific doc] helped me understand [concept]
3. Feel free to ask questions - maintainers are helpful!

Happy to help if you get stuck.
```

---

## Module 8 Exit Criteria

Module 8 is ongoing, but initial completion means:

- [ ] Introduced yourself in community channels
- [ ] Attended at least one community call (if available)
- [ ] Set up GitHub notifications
- [ ] Commented helpfully on issues
- [ ] Identified a medium-scope task
- [ ] Planned implementation approach
- [ ] Started work on medium-scope contribution
- [ ] Submitted at least one PR for the medium task
- [ ] Reviewed at least one other contributor's PR
- [ ] Built relationships with maintainers

### Knowledge Check

**Ask your AI agent these questions:**

1. "What's the difference between a contributor and a maintainer?"
2. "When should I write a formal proposal?"
3. "How do I break large work into smaller PRs?"
4. "What makes a helpful code review comment?"
5. "How can I become a maintainer someday?"

---

## Long-Term Contribution Strategies

### Strategy 1: Consistent presence

**Show up regularly:**
- Weekly PR reviews
- Attend calls
- Respond to issues
- Contribute code monthly

**Learning moment**: Consistency builds trust and visibility. Maintainers remember regular contributors.

---

### Strategy 2: Quality over quantity

**Focus on impact:**
- One great PR > ten mediocre ones
- Thoughtful reviews > quick "LGTM"
- Well-documented code > rushed features

---

### Strategy 3: Listen and learn

**Be receptive to feedback:**
- Don't argue about style
- Ask "why" to understand reasoning
- Adapt to project norms
- Learn from more experienced contributors

---

### Strategy 4: Communicate proactively

**Keep everyone informed:**
- Update issues with progress
- Explain delays
- Ask for help when stuck
- Share learnings

---

## Path to Maintainership

**Ask your AI agent:**
> "Show me the typical path from contributor to maintainer"

**Progression:**

1. **Contributor** (You are here!)
   - Submit PRs
   - Fix bugs
   - Improve docs

2. **Regular Contributor**
   - Consistent participation
   - Multiple merged PRs
   - Known in community

3. **Reviewer**
   - Start reviewing PRs
   - Provide helpful feedback
   - Catch issues

4. **Trusted Contributor**
   - Major features implemented
   - Deep expertise in area
   - Mentors others

5. **Maintainer**
   - Nominated by existing maintainers
   - Given commit access
   - Stewards project direction

**Timeline:** 6 months to 2+ years depending on activity level

---

## Celebrating Milestones

### Your accomplishments so far:

**By completing Modules 1-8, you have:**
- ✅ Set up a complete development environment
- ✅ Deployed applications from image and source
- ✅ Understood the codebase architecture
- ✅ Developed a fast iteration workflow
- ✅ Submitted and merged PRs
- ✅ Passed e2e tests
- ✅ Engaged with the community
- ✅ Planned a medium-scope contribution

**You are now a capable OpenChoreo contributor!** 🎉

---

## Maintaining Momentum

**Ask your AI agent:**
> "Help me create a sustainable contribution schedule"

**Example schedule:**

**Weekly:**
- 30 min: Review new issues
- 1 hour: Code contribution
- 30 min: Review 1-2 PRs
- 1 hour: Community call (if weekly)

**Monthly:**
- 1 medium-scope feature
- 5+ PR reviews
- 1 blog post or doc improvement

**Quarterly:**
- Attend contributor summit (if exists)
- Propose a significant feature
- Help onboard new contributors

---

## Final Thoughts and Next Steps

### You've completed the onboarding!

**What now?**

1. **Keep contributing:** Work on your medium-scope task
2. **Stay engaged:** Regular participation in community
3. **Deepen expertise:** Specialize in an area
4. **Help others:** Mentor new contributors
5. **Think bigger:** Propose significant improvements

### Resources for continued learning

**OpenChoreo specific:**
```bash
# Re-read with deeper understanding
cat docs/contributors/development-process.md
cat docs/proposals/README.md

# Explore advanced topics
ls docs/architecture/
```

**Kubernetes and Cloud Native:**
- CNCF landscape: https://landscape.cncf.io/
- Kubebuilder book: https://book.kubebuilder.io/
- Kubernetes docs: https://kubernetes.io/docs/home/

**Go programming:**
- Effective Go: https://go.dev/doc/effective_go
- Go blog: https://go.dev/blog/

---

## Update Your Progress Log - Final Entry

**Ask your AI agent:**
> "Help me write the final completion entry in progress-log.md"

Add to [progress-log.md](progress-log.md):

```markdown
## Completion Summary

**Completed on:** [Date]

**Main Outcomes:**
- Completed all 8 modules of OpenChoreo contributor onboarding
- Submitted [N] PRs ([N] merged, [N] in review)
- Set up efficient development workflow
- Joined community and attended calls
- Started medium-scope contribution: [Task name]

**Key Lessons:**
1. [Biggest technical learning]
2. [Most important workflow insight]
3. [Key community practice]

**What I'm most proud of:**
[Your accomplishment]

**Follow-up Plan:**
- Complete medium-scope feature by [date]
- Review 2+ PRs per week
- Attend community calls regularly
- Build expertise in [area]
- Goal: Become a trusted contributor within [timeframe]

**Message to future contributors:**
[Your advice for others following this path]
```

---

## Share Your Journey

**Help others by sharing:**

**Ask your AI agent:**
> "Help me write a blog post about my OpenChoreo contributor journey"

**Blog post outline:**
- Why I chose OpenChoreo
- What I learned in each module
- Challenges I overcame
- Tips for new contributors
- What's next for me

**Share on:**
- Personal blog
- Dev.to
- Medium
- LinkedIn
- Twitter/X
- OpenChoreo community channels

---

## Thank You!

**You've completed the OpenChoreo contributor onboarding journey!**

Your contributions make OpenChoreo better for everyone. The project needs contributors like you who:
- Take time to learn deeply
- Follow best practices
- Help others
- Think long-term

**Welcome to the OpenChoreo contributor community!** 🚀

---

## Appendix: Quick Reference

### Communication

**Before starting work:**
- Comment on issue: "I'd like to work on this"
- Wait for assignment

**While working:**
- Open draft PR early
- Share progress updates
- Ask questions proactively

**When blocked:**
- Document what you tried
- Ask specific questions
- Suggest alternatives

**After completion:**
- Celebrate milestone
- Thank reviewers
- Update issue

### Contribution Sizes

**Small (1-2 days):**
- Doc fixes
- Simple tests
- Minor refactors
- Good first issues

**Medium (1-2 weeks):**
- New controller features
- API enhancements
- Significant refactors
- e2e test additions

**Large (1+ months):**
- New CRDs
- Major features
- Architecture changes
- Requires proposal

### When to Write Proposals

**Always:**
- New APIs
- Breaking changes
- Architecture changes

**Sometimes:**
- Significant features
- Cross-component changes
- When asked by maintainers

**Never:**
- Bug fixes
- Internal refactors
- Documentation
- Tests

### Key Make Targets

```bash
make generate              # After API changes
make manifests            # After API changes
make test                 # Before every commit
make lint                 # Before every PR
make e2e.test            # Before significant PRs
make k3d.build.controller # During development
make k3d.update.controller # Deploy changes
```

---

**End of Module 8 and Contributor Onboarding**

Thank you for taking this journey. Now go build something amazing! 🎉
