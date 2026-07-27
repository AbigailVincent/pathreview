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
