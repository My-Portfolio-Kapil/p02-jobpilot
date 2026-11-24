# ADR-001: Adzuna API as the job source

- **Date:** 2026-11-24
- **Status:** accepted
- **Context:** v0 needs every new PM job in Germany once a day, with enough of the description for Gemini to judge fit. Budget is €0. The plan was the Bundesagentur für Arbeit (BA) job search API. Building it (P2-03) showed:
  - The BA list endpoint (`pc/v6/jobs`) works and returns ~80 new PM jobs a day for "Product Manager", "Produktmanager" and "Product Owner". The older `v4` list endpoint now returns 403.
  - The list has no description. Each job needs a second call to the `v4/jobdetails` endpoint.
  - Fetching ~80 details in a burst got the IP throttled: 403s, then `ECONNREFUSED` and `ECONNRESET` for several minutes. Even at 1 request per second, 2 of 20 calls failed. n8n's node-level retry re-runs all items, so one failure re-triggered the burst.
- **Options considered:**

  | | BA API | Arbeitnow API | Adzuna API | LinkedIn scraping |
  |---|---|---|---|---|
  | New PM jobs in Germany per day | ~80 | 1–3 | ~200 unique (485 results) | many |
  | Description | full, but 1 extra call per job | full, in the list | first ~500 characters, in the list | full |
  | Calls per day | ~85 | 1 | ~15 | n/a |
  | Reliability | throttled on detail calls | good | good | blocked often |
  | Key needed | no | no | free key | account |
  | Allowed | yes | yes | yes | no: breaks LinkedIn's User Agreement |

  - **A. BA, detail call per job with pacing and per-item retries.** Pros: official source, full descriptions. Cons: ~85 calls a day against an unofficial, throttled API. A fix (one job at a time in a loop) was built but not proven.
  - **B. Arbeitnow.** Pros: one call, full descriptions, no key. Cons: of 300 jobs in a day, 9 were PM roles and only 1 was in Germany. Too few to stop good roles being missed.
  - **C. Adzuna.** Pros: aggregates many boards, most volume, no detail calls, ~15 calls a day. Cons: needs a free key; descriptions are cut to ~500 characters; the same job often appears once per city.
  - **D. LinkedIn scraping.** Rejected outright: against LinkedIn's terms, risks the account, and looks bad in a public portfolio. Job-alert emails are a compliant route for later (issue #8).
- **Decision:** Use Adzuna for v0: 3 search phrases × up to 5 pages of 50, last 24 hours, Germany only. Duplicates are merged on normalised title + company (ignoring case and gender tags like "(m/w/d)").
- **Consequences:**
  - Easier: one source, no throttling, ~2.5× the volume of BA, simple workflow.
  - Harder: Gemini sees only ~500 characters per job, so language requirements and seniority are sometimes guessed. A job can still be missed if Adzuna doesn't list it. Re-evaluate if the relevance metric (top-5 ≥ 70%) misses because of short descriptions.
