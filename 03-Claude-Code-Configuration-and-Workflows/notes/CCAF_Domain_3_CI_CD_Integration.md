# CCAF Domain 3 — CI/CD Integration

## What You Need to Know

CI/CD integration connects Claude Code workflows with the software delivery pipeline.

The core idea is to decide:

- **What work should happen automatically?**
- **What checks must pass before changes move forward?**
- **Where should human approval remain?**
- **How should failures be reported and recovered?**

Think of the workflow as:

```text
Code change
    ↓
CI pipeline
    ↓
Automated checks
    ↓
Build / test / validation
    ↓
Pass?
   /  Yes  No
  ↓    ↓
Continue  Report failure
  ↓
Deployment / release
```

---

# 1. Why CI/CD Integration Matters

Claude Code can assist with development tasks, but production software still needs reliable automated checks.

CI/CD provides a repeatable environment for:

- Building the application
- Running tests
- Running linting and static analysis
- Checking security requirements
- Validating configuration
- Packaging artifacts
- Deploying approved changes

The important principle is:

> **Do not rely on the agent's own claim that a change works. Use the CI pipeline to verify it.**

---

# 2. Agent + CI/CD Mental Model

A useful architecture is:

```text
Developer
    ↓
Claude Code
    ↓
Code changes
    ↓
Version control
    ↓
CI pipeline
    ↓
Tests / lint / security / build
    ↓
Result
    ↓
Claude or developer responds to failures
```

Claude can help create or modify code.

CI provides an independent verification step.

---

# 3. Automated Verification

After Claude makes a change, the pipeline can automatically run:

```text
Unit tests
Integration tests
Lint
Type checking
Security scans
Build
```

Example:

```text
Claude changes auth.ts
        ↓
Commit / PR
        ↓
CI starts
        ↓
npm test
npm run lint
npm run typecheck
npm run build
        ↓
Results
```

If everything passes:

```text
CI = PASS
```

If something fails:

```text
CI = FAIL
```

The failure becomes feedback for the next development iteration.

---

# 4. CI/CD as a Feedback Loop

CI/CD can provide the evaluation signal used by iterative refinement.

```text
Claude
  ↓
Implement
  ↓
CI
  ↓
Tests fail
  ↓
Analyze failure
  ↓
Refine code
  ↓
CI again
```

This creates:

> **Implement → Verify → Refine → Verify**

The important part is that the next iteration is driven by actual CI feedback.

---

# 5. Do Not Trust "It Works"

A common mistake is accepting an agent's statement:

> "The implementation is complete and all tests should pass."

That is not verification.

A stronger workflow is:

```text
Agent says:
"Implementation complete."

        ↓

Run actual tests

        ↓

CI verifies

        ↓

Pass / Fail
```

### Memory aid

> **Agent reasoning is not a substitute for automated verification.**

---

# 6. Pull Request Integration

A common CI/CD workflow is to run checks when a pull request is created or updated.

```text
Claude makes changes
        ↓
Pull Request
        ↓
CI starts
        ↓
Automated checks
        ↓
Review
        ↓
Merge
```

Typical checks include:

- Unit tests
- Integration tests
- Type checking
- Linting
- Security checks
- Build validation

Only after the required checks pass should the change move toward deployment.

---

# 7. Human Approval

Not every deployment should be completely autonomous.

For high-risk operations, keep a human approval gate.

Example:

```text
Code
 ↓
CI
 ↓
Tests
 ↓
Security checks
 ↓
Build
 ↓
Human approval
 ↓
Production deployment
```

This is especially appropriate for:

- Production deployments
- Database migrations
- Financial changes
- Security-sensitive changes
- Infrastructure changes
- High-impact releases

### Key principle

> **Automate verification where possible; keep human approval where the consequence of failure justifies it.**

---

# 8. CI Failure Handling

When CI fails, the agent should not blindly repeat the same action.

Instead:

```text
CI failure
    ↓
Read failure details
    ↓
Identify root cause
    ↓
Make targeted correction
    ↓
Run relevant checks
    ↓
Run CI again
```

Example:

```text
TypeScript compilation failed
        ↓
Read compiler error
        ↓
Locate incorrect type
        ↓
Fix type
        ↓
Run typecheck
        ↓
CI again
```

---

# 9. Targeted Recovery

If one test fails, do not automatically rewrite the entire feature.

Example:

```text
100 tests
99 pass
1 fails
```

A good recovery strategy is:

```text
Inspect failing test
       ↓
Understand failure
       ↓
Identify affected code
       ↓
Make targeted fix
       ↓
Run failing test
       ↓
Run full suite
```

This connects CI/CD integration with iterative refinement.

---

# 10. CI/CD and Safety

CI/CD should act as a **control boundary** around automated changes.

For example:

```text
Claude
  ↓
Change
  ↓
CI validation
  ↓
Required checks
  ↓
Approval gate
  ↓
Deployment
```

The agent should not be treated as the only control protecting production.

Automated checks provide deterministic gates for requirements that must not be skipped.

---

# 11. Required vs Optional Checks

Not every CI check has the same importance.

### Required checks

These must pass before merging or deploying.

Examples:

```text
Unit tests
Type checking
Security checks
Build
```

### Informational checks

These provide useful feedback but may not block delivery.

Example:

```text
Code coverage report
Performance report
Style suggestions
```

### Mental model

```text
Required
→ blocks progression when failed

Informational
→ provides feedback without necessarily blocking
```

---

# 12. Environment Separation

CI/CD workflows commonly separate environments:

```text
Development
    ↓
Staging
    ↓
Production
```

A typical flow:

```text
Code change
   ↓
CI tests
   ↓
Build artifact
   ↓
Staging
   ↓
Validation
   ↓
Approval
   ↓
Production
```

This reduces the risk of sending unverified changes directly to production.

---

# 13. Secrets and Credentials

Do not hardcode credentials in:

- Source code
- `CLAUDE.md`
- CI configuration committed to the repository
- MCP configuration with literal secrets

Use the CI platform's secret-management mechanism.

Conceptually:

```text
Repository
    ↓
References secret
    ↓
CI secret store
    ↓
Runtime credential
```

The secret value should not be committed to version control.

---

# 14. Claude Code in CI

Claude Code can be used in automated development workflows, but the permissions and scope should be carefully controlled.

For automated CI execution:

- Give the agent only the capabilities it needs.
- Avoid unnecessary write or deployment permissions.
- Keep production credentials protected.
- Require deterministic checks before deployment.
- Preserve logs and failure information.

### Principle

> **Automation should be powerful enough to do its job, but not more powerful than necessary.**

---

# 15. CI/CD vs Hooks

These mechanisms operate at different points.

### Hooks

Hooks can enforce behavior around tool execution.

```text
Before tool
    ↓
PreToolUse
    ↓
Tool
    ↓
PostToolUse
```

They are useful for deterministic local policy enforcement.

### CI/CD

CI/CD verifies the resulting change at the pipeline level.

```text
Code change
    ↓
CI
    ↓
Tests / build / security
    ↓
Deployment gate
```

### Memory aid

> **Hooks control tool execution.**

> **CI/CD validates the delivered change.**

Both can be used together.

---

# 16. Example: Safe Deployment Workflow

A production-oriented workflow could look like:

```text
1. Developer asks Claude to implement a feature.
2. Claude modifies the code.
3. Local tests run.
4. Changes are submitted as a PR.
5. CI runs the full test suite.
6. CI performs type checking and security checks.
7. CI builds the application.
8. If checks fail, the failure is investigated and corrected.
9. If checks pass, the PR is reviewed.
10. Human approval is required for production.
11. Deployment proceeds.
```

The key separation is:

```text
Agent
→ creates / modifies

CI
→ verifies

Human
→ approves high-impact changes

Deployment system
→ releases
```

---

# 17. Common Exam Traps

## Trap 1 — Deploying directly after the agent says the code is complete

Agent output is not sufficient verification.

Use CI checks before deployment.

---

## Trap 2 — Skipping tests because Claude already ran them

Local testing is useful, but CI provides an independent, repeatable verification environment.

---

## Trap 3 — Automatically retrying the entire workflow after a CI failure

Read the failure and perform targeted recovery.

Do not blindly repeat the same change.

---

## Trap 4 — Giving the CI agent unrestricted production access

Follow least privilege.

Give automated workflows only the permissions required for their role.

---

## Trap 5 — Hardcoding production credentials

Secrets should be stored in the CI/CD secret-management system rather than committed to source control.

---

## Trap 6 — Removing human approval from high-risk deployments

Automation does not mean every action should be autonomous.

Keep approval gates where the consequences justify them.

---

# 18. Practice Scenario

### Scenario

Claude Code implements a feature and reports that the implementation is complete. The team wants the change to reach production automatically.

What is the safer architecture?

### Recommended flow

```text
Claude implementation
        ↓
Pull request
        ↓
CI
        ↓
Tests
        ↓
Type checking
        ↓
Security checks
        ↓
Build
        ↓
All required checks pass?
       /      No   Yes
     ↓      ↓
Fix      Approval
           ↓
       Production
```

### Why?

The agent should not be the only verification mechanism.

CI provides an independent and repeatable quality gate, while human approval can remain for high-impact production changes.

---

# 19. Quick Reference

| Requirement | Recommended mechanism |
|---|---|
| Verify code changes | CI tests |
| Verify build | CI build |
| Check style | Linting |
| Check types | Type checker |
| Check security | Security scan |
| Recover from failure | Analyze feedback and refine |
| Protect production | Approval/deployment gates |
| Protect credentials | CI secret management |
| Restrict agent capabilities | Least privilege |
| Enforce tool-call policy | Hooks |
| Validate delivered changes | CI/CD |

---

# 20. CCAF Checklist

- [ ] Understand how Claude Code can participate in CI/CD.
- [ ] Know CI provides independent verification.
- [ ] Know tests, builds, linting, and security checks can be automated.
- [ ] Understand CI as a feedback loop for iterative refinement.
- [ ] Know not to trust an agent's claim that code works without verification.
- [ ] Understand targeted recovery after CI failures.
- [ ] Understand required vs informational checks.
- [ ] Understand human approval gates for high-risk operations.
- [ ] Know to protect CI/CD secrets.
- [ ] Understand least privilege for automated agents.
- [ ] Distinguish hooks from CI/CD.
- [ ] Know the typical PR → CI → approval → deployment flow.

---

# One-Line Memory Aids

> **Agent creates; CI verifies; approval controls high-impact release; deployment delivers.**

> **CI/CD = independent verification and delivery pipeline.**

> **CI failure = feedback for targeted refinement.**

> **Never treat "the agent says it works" as proof.**

> **High-risk production changes may require human approval.**

> **Secrets belong in secret management, not source control.**

> **Least privilege applies to CI agents too.**

> **Hooks control tool execution; CI/CD validates the resulting change.**
