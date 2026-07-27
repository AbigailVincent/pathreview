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
## Week 8 — Reproduction

**Reproduction steps:**
Started the app locally with `make run` (backend on port 8000, frontend on 5173,
with Postgres/Redis/ChromaDB running via `docker compose up -d`). Hit the health
check endpoint directly:

curl http://127.0.0.1:8000/health

**Observed output:**
{"detail":{"status":"unhealthy","dependencies":{"postgres":"unhealthy","redis":"unhealthy","vector_db":"healthy"},"safety_events_last_hour":0,"timestamp":"2026-07-27T21:30:10.544903"}}

**Server logs confirmed the exact cause:**
redis_health_check_failed      error="'Settings' object has no attribute 'redis_host'"

This confirms the bug described in issue #155: the Redis health check in
`api/routes/health.py` references `settings.redis_host` and `settings.redis_port`,
neither of which exist on the `Settings` class (only `redis_url` is defined in
`.env`/config). This raises an `AttributeError` every time the health check runs,
which is silently caught and reported as `"redis": "unhealthy"` — even though
Docker confirms the Redis container itself is healthy
(`docker compose ps` shows `pathreview-redis-1 ... (healthy)`).

Note: the same response also showed `"postgres": "unhealthy"`, caused by a
separate, unrelated bug (raw SQL string needing `text()` wrapping under
SQLAlchemy 2.x — this matches issue #154, not my assigned issue).
