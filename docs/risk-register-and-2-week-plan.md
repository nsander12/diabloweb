# DiabloWeb Risk Register and 1–2 Week Execution Plan

This document translates the recent maintainability and security/runtime review into an actionable register and short execution plan.

## Scoring model

- **Severity (Impact):** 1 (Low) to 5 (Critical)
- **Likelihood:** 1 (Unlikely) to 5 (Highly likely)
- **Exposure score:** Severity × Likelihood
- **Priority bands:**
  - 16–25: P0
  - 9–15: P1
  - 4–8: P2
  - 1–3: P3

---

## Risk register

| ID | Risk | Category | Evidence | Severity | Likelihood | Exposure | Priority | Mitigation | Owner | Status |
|---|---|---|---|---:|---:|---:|---|---|---|---|
| R-001 | Synchronous XHR for remote MPQ chunk fetch blocks worker thread and is fragile under network latency/failure. | Runtime / Reliability | `RemoteFile` uses `XMLHttpRequest` with sync mode (`open(..., false)`). | 4 | 4 | 16 | P0 | Replace with async `fetch` + range requests; add timeout/retry and explicit error paths. | Web runtime | Open |
| R-002 | Hardcoded websocket endpoint reduces deployment flexibility and trust boundaries are implicit. | Security / Operability | URL fixed as `wss://diablo.rivsoft.net/websocket`. | 4 | 3 | 12 | P1 | Make endpoint configurable via env/runtime config; add connection health logging and fallback policy. | Networking | Open |
| R-003 | Virtual FS writes are broad; commented allowlist indicates prior intent to restrict writable filenames. | Security / Data Integrity | `put_file_contents` accepts arbitrary lowercased paths; allowlist is commented out. | 4 | 3 | 12 | P1 | Reinstate strict allowlist (`spawn*.sv`, `single_*.sv`, `config.ini`) and reject others with telemetry. | Save/FS | Open |
| R-004 | Input/event lifecycle listeners are added during game start but teardown is not centralized, increasing leak/duplication risk. | Maintainability / Reliability | Multiple `addEventListener` calls in `start()`, no single teardown method. | 3 | 4 | 12 | P1 | Implement centralized `attachInputListeners()` / `detachInputListeners()` and call on exit/error/unmount. | Frontend | Open |
| R-005 | Repeating timers (`setInterval`) in loader/worker have no clear cancellation strategy. | Runtime / Maintainability | Render loop and packet flush use long-lived intervals. | 3 | 3 | 9 | P1 | Track interval IDs and clear on exit, failure, and worker termination. | Web runtime | Open |
| R-006 | Large monolithic `App` component raises change risk and review/test burden. | Maintainability | `App.js` owns many responsibilities (input, saves, error UI, startup). | 3 | 3 | 9 | P1 | Extract modules/hooks: input adapter, save manager, game-session orchestration, error/reporting presenter. | Frontend | Open |
| R-007 | Legacy dependency/toolchain baseline slows upgrades and can increase supply-chain/security risk. | Security / Maintainability | Older React/Webpack/Jest ecosystem versions. | 3 | 3 | 9 | P1 | Create phased upgrade roadmap and run `npm audit` + dependency policy gate in CI. | Platform | Open |

---

## 1–2 week execution plan

## Week 1 (Hardening + lifecycle hygiene)

### Day 1: Design + acceptance criteria
- Finalize technical design for replacing sync XHR with async range-loading in worker.
- Define env/config approach for websocket URL.
- Define FS allowlist behavior and user-facing error semantics.

**Deliverables**
- Mini design note in PR description.
- Test checklist and acceptance criteria.

### Days 2–3: Runtime/security fixes (highest risk)
- Implement async remote file chunk loading in `RemoteFile` path.
- Make websocket URL configurable (`process.env` or runtime config object).
- Restore FS writable-file allowlist with explicit validation and error reporting.

**Acceptance criteria**
- No sync XHR in worker code path.
- Websocket URL no longer hardcoded.
- Invalid file paths are rejected and logged.

### Days 4–5: Lifecycle cleanup
- Add listener lifecycle helpers and use them consistently for start/exit/error/unmount paths.
- Track + clear interval IDs in loader and worker termination paths.

**Acceptance criteria**
- No duplicate listeners after restart flow.
- Intervals are cleaned up on shutdown/error.

## Week 2 (Refactor + modernization kickoff)

### Days 6–7: `App` decomposition (non-functional changes)
- Split `App` responsibilities into focused modules:
  - Input/controller adapter
  - Save management facade
  - Error/reporting UI helper
- Keep behavior unchanged.

**Acceptance criteria**
- Same user-visible behavior in smoke test.
- Reduced file size/complexity in main component.

### Days 8–9: Regression checks + test scaffolding
- Add lightweight tests for:
  - FS path allowlist
  - listener attach/detach idempotency
  - interval cleanup behavior

**Acceptance criteria**
- Tests pass locally/CI.
- New guardrails prevent regression of the highest-risk fixes.

### Day 10: Toolchain modernization kickoff
- Create dependency baseline report (`outdated`, `audit`, upgrade candidates).
- Propose phased upgrade sequence (lint/test infra first, then bundler/runtime).

**Acceptance criteria**
- Approved roadmap issue or milestone.
- Clear next sprint backlog items.

---

## Definition of done (for this plan window)

- P0 risk (R-001) mitigated and merged.
- At least two P1 risks (R-002, R-003, R-004, R-005) mitigated and merged.
- No new critical runtime regressions in manual smoke pass.
- Follow-up tickets created for remaining risks with owners and target milestones.

## Suggested issue breakdown

1. Replace sync XHR remote loader with async range fetch.
2. Externalize websocket endpoint and add status telemetry.
3. Reinstate FS filename allowlist.
4. Centralize input listener lifecycle.
5. Interval lifecycle management.
6. App decomposition phase 1.
7. Upgrade roadmap + dependency baseline.
