# ADR-002: A 24-hour window instead of a "seen before" memory

- **Date:** 2026-11-24
- **Status:** accepted
- **Context:** The digest should show each job once. Adzuna's `max_days_old=1` returns jobs from roughly the last one to two days, so consecutive runs overlap. The first design used n8n's Remove Duplicates node in "remove items repeated in past executions" mode. On n8n 2.49.5 it stored nothing: the `processed_data` table stayed empty after both command-line and scheduled runs, so every run let all jobs through again.
- **Options considered:**
  - **A. Debug n8n's built-in memory.** Pros: the intended feature, robust to late listings. Cons: unclear why it fails; time-boxed debugging didn't find it.
  - **B. Own store** (an n8n Data Table or a file of seen job IDs). Pros: robust, under our control. Cons: more nodes, state to clean up, more to explain in v0.
  - **C. Keep only jobs created in the last 24 hours.** The workflow runs once a day, so each job falls into exactly one window. Pros: stateless, one small Code node, easy to reason about. Cons: see below.
- **Decision:** C for v0.
- **Consequences:**
  - Easier: no state, nothing to reset, manual test runs don't affect tomorrow's digest.
  - Harder:
    - A job Adzuna indexes after the 07:00 run but dated before it is never shown.
    - If a run is skipped (Mac asleep, Docker stopped), that day's jobs are lost.
    - Running twice on the same day sends the same jobs twice.
  - Revisit with option B if skipped runs or late listings show up in practice.
