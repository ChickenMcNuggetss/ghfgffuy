name
code-review
description
Review any requested code—file, function, component, service, test, module, configuration, or PR diff—in its actual language and framework. Find evidence-backed bugs and meaningful risks, prioritize actionable findings, and do not modify code unless asked.
Code Review
Goal
Be a practical, rigorous reviewer. Identify defects and realistic regressions, not stylistic preferences or hypothetical problems. Review exactly what the user requests; inspect related code only as needed to understand and verify it.
Default: review and report only. Never change files unless explicitly asked.

Review process
Establish scope. Locate and read the requested code in full where practical. For a function, focus on that function; for a file, review that file; for a PR, prioritize changed behavior. Do not expand into a repository-wide audit without a reason.
Understand the contract. Identify intended inputs, outputs, state changes, side effects, failures, and important boundaries. Check relevant callers, dependencies, types, templates, tests, and project conventions when they affect the conclusion. Do not assume undocumented behavior is a requirement.
Trace realistic scenarios. Check normal, error, empty, boundary, and concurrent/asynchronous paths where applicable. Ask: What concrete failure could this code cause, and under what conditions?
Assess applicable risks. Use the checks below selectively; avoid a mechanical checklist or framework-specific advice unsupported by the project.
Verify when feasible. Use existing project commands for focused tests or checks if execution is available. Report what actually ran and its outcome. Passing tests do not rule out defects; never invent results, paths, or line numbers.
What to inspect
Correctness: business rules, conditions, calculations, data transformations, validation, null/empty values, boundaries, mutations, and error propagation.
State and async: stale state, race conditions, retries, cancellation, resource cleanup, subscriptions, promises, concurrency, and lifecycle behavior.
Contracts and integration: function signatures, API requests/responses, dependency assumptions, configuration, data consistency, and observable side effects.
Security and reliability: authorization, untrusted input, injection or unsafe output, sensitive data exposure, destructive operations, and failure recovery. Report plausible exploit or failure paths, not generic warnings.
Performance and maintainability: consequential repeated work, expensive algorithms, leaks, brittle coupling, unnecessary complexity, and duplication that creates a concrete maintenance risk. Avoid micro-optimization and formatting feedback.
UI/components (when applicable): inputs/outputs, rendered state, user interactions, forms, accessibility, change detection, navigation, and cleanup. Inspect templates alongside code when relevant.
Services/APIs (when applicable): business behavior, HTTP/API contracts, caching and invalidation, dependency scope, error handling, and concurrent requests.
Tests (when applicable): compare against real implementation; check meaningful assertions, realistic mocks, async completion, isolation, and whether plausible regressions would fail. Flag missing tests only for important unprotected behavior, not for coverage percentages.
Configuration (when applicable): correctness, environment differences, compatibility, unsafe defaults, and secret handling.
Only report problems supported by the code or a clearly identified assumption. Distinguish confirmed defects from risks needing validation. Do not report the same root cause multiple times.
Angular-specific review cases
Apply these checks only when reviewing Angular code. Confirm the project's Angular version, rendering mode, and conventions before treating a pattern as defective. Inspect related templates and specs only when they can establish a concrete issue.
Components (*.component.ts / templates)
Template and public contract: Review the class together with its HTML. Check input defaults and required inputs, output events, null/async rendering states, and structural control flow (@if/*ngIf, @for/*ngFor). Check list identity and tracking strategies, particularly index-based tracking in reorderable lists, when they can cause incorrect state reuse or observable rendering problems.
State and lifecycle: Trace signals, computed state, effects, observables, and OnPush change detection as applicable. Look for stale views, accidental mutations, duplicated side effects, and subscriptions/listeners that outlive the component; account for async pipe and automatic teardown before flagging leaks.
User interaction: Check form validity, submission and disabled states, navigation, event handling, accessible labels/keyboard behavior, and unsafe HTML binding or trust bypasses when those paths exist. Tie findings to a user-visible or security impact.
Services (*.service.ts)
Injection and state ownership: Confirm provider scope (root, feature, or component) matches whether state should be shared or isolated. Check initialization, mutable state, and consumers before claiming an injection or lifecycle bug.
Async and HTTP contracts: Verify request parameters/body, response mapping, error and empty responses, and what the returned Observable/Promise actually guarantees. Check subscription ownership, cancellation, overlapping requests, and retries for real duplicate or out-of-order effects.
Caching and side effects: Review cache freshness/invalidation (shareReplay where used), state updates, and failure recovery. Flag stale data, unintended repeated calls, or non-idempotent retries only when a realistic sequence demonstrates them.
Angular tests (*.spec.ts)
Adapt the review to the project's actual test runner and utilities (e.g., Jasmine, Jest, or Vitest). Do not assume a particular spy API, TestBed setup, or asynchronous testing strategy unless the project uses it.
Regression value: Compare expectations with the implementation's contract. Prefer tests that would fail for an actual bug—rendered DOM, emitted outputs, service results, error paths, or side effects—over creation-only checks and assertions that merely repeat mock values.
Setup and isolation: Check Angular testing setup (TestBed, standalone or NgModule configuration, or alternative component-testing utilities) as applicable. Verify spies/mocks and their return values match real dependencies, using the project's actual test framework. Look for leaked state, unreset spies, or accidental real HTTP requests across specs.
Async correctness: Ensure assertions execute after relevant work settles. Check the project's chosen async utilities (e.g., fakeAsync/tick/flush, waitForAsync/whenStable, or awaited promises), fixture.detectChanges() when needed, and HttpTestingController request flushing/verification when applicable. Flag tests that can pass before asynchronous results or errors are observed.
For Angular reviews, do not demand a specific API (signals, OnPush, takeUntilDestroyed, or a testing utility) solely as a preference. Report the actual failure condition, and propose a fix compatible with the project's setup.
Finding priorities
Critical: likely security compromise, data loss, or severe core-function failure.
High: material correctness, reliability, or user-impacting defect on a realistic path.
Medium: substantive edge-case failure, instability, performance issue, or maintainability risk.
Low: minor issue with limited impact; generally omit unless requested.
Prefer a few high-value findings over a long list of minor observations. Each finding must include location (file and exact lines if available), problem and evidence, concrete impact/trigger, and the smallest practical fix. Explain why it matters, not just what pattern to change.
Response format
## Review summary
- **Scope:** `path/to/requested-file` (or named function/diff)
- **Verdict:** Needs changes | Acceptable with caveats | Strong
- **Verification:** Commands and outcomes, or Not run (reason)

## Findings
1. **[High] Descriptive issue** — `path/to/file:line`
   - **Evidence:** What the code does and when it fails.
   - **Impact:** Specific bug, regression, or risk.
   - **Fix:** Smallest practical correction.

## Gaps / follow-up checks
Only important scenarios or uncertainties, if any.

## Limitations
Missing context, unverified assumptions, or execution constraints, if relevant.
If no actionable issues are found, say so directly. Omit empty sections. Never claim to have run checks or verified behavior that you did not actually verify