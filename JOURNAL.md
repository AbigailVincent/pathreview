## Week 7 — Issue selection

**Issue link:** https://github.com/ascherj/pathreview/issues/155

**Issue title:** Health check references settings.redis_host, which does not exist on Settings

**Tier:** [x] Tier 1

**Problem summary:**
The `/health` endpoint's Redis check tries to connect using `settings.redis_host`
and `settings.redis_port`, but the `Settings` class doesn't define those
attributes — it only stores a combined `redis_url`. This causes an
`AttributeError` every time the health check runs, which gets silently
caught and reported as "redis: unhealthy," even when Redis is actually
running fine. This affects the `api` health check route (and touches
`core/config` where `Settings` is defined). A correct fix would update the
Redis connection code to use the actual config field(s) available on
`Settings`, so the health check accurately reflects Redis's real status.

**Branch name:** fix/155-health-check-redis-host

**Setup confirmation:** [x] App runs locally at localhost:5173

**Cohort ledger:** [x ] Issue added to cohort ledger
## Week 8 — Reproduction & solution planning

**Reproduction commit link:** https://github.com/AbigailVincent/pathreview/commit/0496aef

**Reproduction summary:**
Ran the app locally and hit `curl http://127.0.0.1:8000/health`. Server logs
confirmed the exact predicted error — `'Settings' object has no attribute
'redis_host'` — causing the endpoint to report Redis as unhealthy even
though `docker compose ps` showed the Redis container itself running and
healthy.

**PLAN.md link:** https://github.com/AbigailVincent/pathreview/blob/fix/155-health-check-redis-host/PLAN.md

**Walkthrough video (recommended):** N/A — did not record one this week.

**Blockers or open questions:**
The same `/health` response also shows `"postgres": "unhealthy"`, caused by
a separate, unrelated bug (raw SQL string needing `text()` wrapping under
SQLAlchemy 2.x — this matches issue #154, not mine). Not a blocker, just
noting it so it's not confused with my actual fix in Week 9. No open
questions on my own issue at this point — root cause and fix location are
both clear.
SQLAlchemy 2.x — this matches issue #154, not my assigned issue).
## Week 9 — Implementation (mid-week check-in)

Implemented the fix in `api/routes/health.py`: the Redis health check now
builds its client from `settings.redis_url` via `redis.Redis.from_url()`,
replacing the broken references to the nonexistent `settings.redis_host`
and `settings.redis_port`. Added a socket timeout (2s) so a slow/unreachable
Redis can't hang the health check indefinitely.

Verified manually in both directions: with Redis running, `/health` now
correctly reports `"redis": "healthy"`; with Redis stopped
(`docker compose stop redis`), it correctly reports `"redis": "unhealthy"`
with a real connection-timeout error in the logs, instead of the old
`AttributeError`.

Wrote `tests/unit/test_health.py` with 3 tests covering the healthy case,
the genuinely-unhealthy case, and a regression test asserting the fix uses
`settings.redis_url` specifically (guards against this exact bug recurring).
All 3 pass. Ran the full test suite to confirm no regressions — pre-existing
failures exist in the suite (tracked separately as issues #158/#159,
unrelated to this fix) but my change introduces zero new failures.

## Week 9 — Submission

**PR link:** https://github.com/ascherj/pathreview/pull/787

**Summary:** Fixed issue #155 — the `/health` endpoint's Redis check now
uses the correct `settings.redis_url` config field instead of nonexistent
`redis_host`/`redis_port` attributes, so it accurately reports Redis's real
status instead of always failing.
