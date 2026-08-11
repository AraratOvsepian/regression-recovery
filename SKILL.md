---
name: regression-recovery
description: Diagnose and safely repair a code regression by proving the failure, locating the last known-good and first bad revisions, identifying the causal change and its intent, and porting the smallest compatible known-good behavior into the current code. Use when a feature used to work, a recent deployment introduced a defect, a user asks to trace or roll back a regression, or old working behavior must be restored without losing newer features, schema changes, safeguards, or unrelated work.
---

# Regression Recovery

Restore behavior, not an old repository snapshot. Prove causality, preserve current contracts, and port only the minimum compatible repair.

## Operating contract

- Treat diagnosis as read-only until a causal boundary is proven.
- Preserve unrelated modified and untracked files.
- Use an isolated worktree for historical builds, bisecting, and candidate repair work.
- Never use `git reset --hard`, destructive checkout/restore, force push, or history rewriting.
- Never assume a successful old implementation is compatible with the current schema, API, runtime, security, or compliance rules.
- Never weaken a current safeguard merely because the historical code predates it.
- Prefer zero-cost historical probes. Allow bounded paid or externally mutating canaries on the final candidate when they are necessary and within the user's authorized scope and budget.
- Do not run paid or externally mutating actions inside an automated bisect unless the user explicitly authorizes the total worst-case cost.
- Do not commit, push, deploy, migrate data, or mutate live systems unless the request authorizes that action.

Read [references/safety-and-evidence.md](references/safety-and-evidence.md) before applying a historical patch, cherry-pick, schema change, paid canary, or deployment.

## Workflow

### 1. Define the regression precisely

Record:

- Expected behavior.
- Current behavior.
- Affected feature boundary and users/environments.
- First observed time or deployment.
- Candidate good tag, commit, deployment, or date.
- Safety, cost, identity, and data constraints.

Separate a code regression from configuration drift, permission changes, provider behavior, stale data, race conditions, capacity limits, or a previously hidden defect.

### 2. Build a regression oracle

Create or identify a deterministic probe that returns:

- `0` when the feature works.
- `1` when the regression is present.
- `125` when a historical revision cannot be evaluated during `git bisect run`.

Prefer, in order:

1. Existing focused test.
2. New focused regression test.
3. Recorded request/response fixture or replay.
4. Read-only database or log assertion.
5. Sandboxed UI or API check.
6. Bounded live canary on the final candidate only.

Verify the oracle fails at the current bad revision and passes at a known-good revision. If it cannot distinguish both states, improve it before continuing.

### 3. Protect the current workspace

Inspect the branch, HEAD, status, remotes, submodules, worktrees, and applicable repository instructions.

If the current worktree is dirty, leave it untouched. Create a sibling temporary worktree from the current commit for diagnosis. Record the original HEAD and every temporary worktree path.

Fetch only when needed and authorized. Do not merge, rebase, or pull during diagnosis.

### 4. Locate last good and first bad

Use this evidence order:

1. Deployment metadata and CI artifacts.
2. Feature-specific tests, logs, and release history.
3. Pull requests and commit messages explaining intent.
4. `git log -- <paths>`, `git log -S`, `git log -G`, and `git blame`.
5. `git bisect run <oracle>` in the isolated worktree.
6. Manual comparison when historical dependencies cannot build.

Do not stop at correlation. Report:

- Last revision where the oracle passes.
- First revision where the oracle fails.
- Culprit commit or smallest unresolved range.
- Files and contracts changed.
- Original intent of the change.
- Evidence connecting it to the observed failure.

### 5. Choose the smallest repair strategy

Evaluate in this order:

1. Port the specific known-good function, condition, or hunk into current architecture.
2. Revert only the causal portion of the bad change.
3. Cherry-pick an existing later fix.
4. Add a compatibility adapter around the current implementation.
5. Reimplement the old behavioral contract when direct porting is incompatible.

Do not blindly cherry-pick a last-good commit: a known-good state is a tree state, while cherry-pick replays a commit's complete change. If cherry-pick is appropriate, apply with `--no-commit`, inspect every changed path, adapt it, and retain source-commit provenance in the final report.

### 6. Audit forward compatibility

Before accepting the candidate, compare it with current:

- Callers and downstream consumers.
- Database schema, migrations, and data shape.
- API and event contracts.
- Authentication, authorization, secrets, and environment routing.
- Idempotency, retries, concurrency, and external identity handling.
- Paid-work and rate-limit controls.
- Compliance, validation, and safety policies.
- Runtime, dependencies, feature flags, and configuration.
- Observability, recovery, and rollback behavior.
- Features added after the known-good revision.

Use `git diff`, `git show`, and `git range-diff` as appropriate. Check that no current behavior disappears unintentionally.

### 7. Implement with regression protection

- Add a focused test that reproduces the original regression.
- Make the smallest coherent change.
- Preserve the current public and data contracts unless an intentional migration is authorized.
- Record the last-good commit, first-bad commit, culprit, and ported source in comments only when that history is operationally useful; otherwise keep it in the report and commit message.
- Avoid broad formatting or refactoring in the repair commit.

### 8. Validate in layers

Run:

1. Regression oracle.
2. Focused tests for the repaired feature.
3. Tests for adjacent current behavior.
4. Type checking, linting, and build.
5. Relevant integration and schema/API contract checks.
6. Diff audit for unexpected deletions or restored obsolete code.
7. Full relevant suite proportional to risk.

For live validation, dry-run first, back up scoped data, revalidate the affected set, use idempotent operations, and advance 1–3 canaries before broader release. Enforce the user's cost ceiling and preserve existing external identities and successful paid assets.

### 9. Deliver an evidence report

Use the report structure in [references/safety-and-evidence.md](references/safety-and-evidence.md). State uncertainty explicitly. Do not call the regression repaired until the current candidate passes the oracle and the compatibility gates.

## Stop conditions

Stop and request direction when:

- No reliable good/bad oracle can be established.
- The last-good state requires removing a current safety or compliance control.
- Historical tests require unauthorized paid work or production mutation.
- The repair requires destructive data/schema rollback.
- Multiple plausible culprit ranges remain after safe diagnostics.
- The candidate affects environments, repositories, or users outside the authorized scope.

