# Problem statement: JobPilot

**PM:** Kapil · **Sprint:** 2

## User
A product manager looking for a job in Germany.

## Pain
- Checking job boards every day takes about 1 hour.
- Good roles still get missed or are found too late.

## Why AI
An LLM can read a job ad against a CV and judge fit (seniority, domain, skills, language) the way a person would, which keyword filters can't do.

## What v0 does
A daily Telegram digest of new PM jobs from the Adzuna API (Germany), each scored 0–100 for fit against the CV by Gemini.

## Success metrics
| Metric | Target | How measured |
|---|---|---|
| Reliability | Digest arrives every day | Telegram message history |
| Relevance | ≥ 70% of top-5 jobs judged relevant | Our own review of each digest |
| Cost | €0 | Free tiers only (n8n self-hosted, Adzuna, Gemini, Telegram) |

## Out of scope (v0)
- Job sources other than Adzuna
- Applying to jobs or writing cover letters
- Multiple users or CVs
