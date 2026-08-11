# Safety gates and evidence report

## Decision gates

### Historical diagnosis gate

Proceed only when the current bad revision and at least one candidate good revision are identifiable. Keep historical execution isolated from the user's working tree and production systems.

### Causality gate

Require the regression oracle to pass on the last-good revision and fail on the first-bad revision. If builds are unevaluable, mark them as skipped rather than guessing.

### Patch selection gate

For each candidate strategy, record:

| Strategy | Benefit | Compatibility risk | Modern behavior lost | Validation needed |
|---|---|---|---|---|
| Port minimal hunk | Smallest surface | Context may have changed | Usually none | Focused + adjacent tests |
| Partial revert | Direct causal undo | Later code may depend on it | Possible | Dependency/caller audit |
| Cherry-pick later fix | Preserves known fix | Commit may be broad | Possible | Inspect every path |
| Compatibility adapter | Preserves current design | Adds complexity | None | Contract + integration tests |
| Reimplement contract | Best current fit | Highest implementation risk | None | Full relevant suite |

Prefer the option with the smallest change surface that preserves every current required contract.

### Paid-canary gate

Before paid or externally mutating validation, establish:

- Why fixtures or sandbox probes are insufficient.
- Maximum number of calls or objects.
- Maximum total cost.
- Idempotency key or duplicate-prevention method.
- External identities and assets that must be preserved.
- Cleanup or observation plan.
- The user's existing authorization for this scope and budget.

Never place this canary inside a multi-commit automated bisect unless the entire worst-case cost and mutation count are explicitly authorized.

### Deployment gate

Require:

- Clean intended diff.
- Focused and adjacent tests passing.
- Build/type/schema/API checks passing.
- Current branch updated according to repository policy.
- Approved deployment path and environment.
- Canary and rollback/forward-fix observation plan.

## Evidence report template

### Regression

- Expected behavior:
- Current behavior:
- Scope and severity:
- First observed deployment/time:

### Proof

- Oracle:
- Current bad revision and result:
- Last-good revision and result:
- First-bad revision and result:
- Culprit or suspect range:

### Root cause

- Causal code path:
- Original change intent:
- Why the intent produced the regression:
- Configuration/data/provider factors ruled out:

### Repair

- Strategy selected:
- Historical source revision/hunks:
- Current files changed:
- New regression tests:
- Modern behavior deliberately preserved:

### Compatibility audit

- Callers and APIs:
- Schema and data:
- Security and permissions:
- Idempotency and concurrency:
- Paid work and rate limits:
- Compliance and validation:
- Runtime and dependencies:
- Observability and recovery:

### Validation

- Regression oracle:
- Focused tests:
- Adjacent tests:
- Build/type/lint:
- Integration/schema/API checks:
- Live canary and cost, if any:

### Outcome

- Confidence:
- Remaining risks:
- Commit/PR/deployment:
- Observation window:

