## Week 7 — Issue selection

**Issue link:** [(https://github.com/ascherj/pathreview/issues/155)]

**Issue title:** [Health check references settings.redis_host, which does not exist on Settings
 #155]

**Tier:**  Tier 1 

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
**Branch name:**fix/155-health-check-redis-host

**Setup confirmation:** [ ] App runs locally at localhost:5173

**Cohort ledger:** [ ] Issue added to cohort ledger
