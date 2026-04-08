# Django Performance and Scalability

This reference covers profiling, query optimization, caching, and deployment strategies to keep Django fast and scalable.

## Measure before optimizing

- Start with measurement, not guesses:
  - Use Django debug toolbar locally to inspect SQL queries and timings.
  - Add timing logs around slow views or use APM tools (New Relic, Sentry Performance, etc.) in staging/production.
- Identify:
  - Slow endpoints (p95/p99 latency).
  - N+1 query patterns and heavy joins.
  - Hot code paths in Python and templates.

## Database query optimization

- Avoid N+1 queries:
  - Apply `select_related` for foreign keys and one-to-one relationships.
  - Apply `prefetch_related` for many-to-many and one-to-many.
- Use appropriate indexes:
  - Add indexes for frequently filtered fields and unique constraints.
  - Use Django’s `indexes` and `index_together`/`constraints` in `Meta`.
- Keep transactions short and targeted; avoid doing heavy CPU or I/O inside DB transactions.

## Caching strategies

Django’s caching framework provides several levels of caching for performance.

- Configure a real cache backend (e.g. Redis or Memcached) for production:
  - Use `CACHES` setting with a backend like `django_redis.cache.RedisCache` or Memcached.
- Levels of caching:
  - **Per-view caching** with `cache_page`.
  - **Template fragment caching** with `{% cache %}` tags.
  - **Low-level caching** using `django.core.cache` API for expensive computations or object lookups.
- Use caching selectively:
  - Prioritize expensive or frequently requested views.
  - Be clear about invalidation strategies (on writes or via timeouts).

## Application-level performance

- Reduce Python overhead:
  - Avoid heavy work in views that could be moved to background jobs (emails, report generation).
  - Use `cached_property` for expensive per-request attributes when appropriate.
- Optimize templates:
  - Avoid deep template inheritance with heavy loops where possible.
  - Consider the cached template loader (enabled in production by default in newer Django versions) and ensure templates are not recompiled unnecessarily.
- Async where it helps:
  - For I/O-bound tasks, consider async views under ASGI (e.g. for high-concurrency APIs).
  - Keep DB access in sync functions unless using a fully async DB stack; mixing patterns incorrectly can hurt performance.

## Deployment considerations

- Use a proper WSGI/ASGI server in production:
  - `gunicorn` or `uwsgi` for WSGI; `uvicorn`/`hypercorn` for ASGI.
- Tune worker count and timeouts based on CPU cores and workload characteristics.
- Put a reverse proxy (nginx, Caddy, etc.) in front for TLS termination, compression, and static/media serving where appropriate.
- Use separate services for:
  - Application (Django).
  - Database (managed Postgres/MySQL).
  - Cache (Redis/Memcached).
  - Task queue (Celery + broker).

## Scaling patterns

- Scale **vertically** first (more CPU/RAM) until returns diminish.
- Scale **horizontally**:
  - Multiple app instances behind a load balancer.
  - Shared DB/cache and stateless application nodes.
  - Use sticky sessions only when absolutely necessary; prefer server-side sessions with a shared cache or DB.

Plan any major performance changes with monitoring and rollback in mind so regressions are visible and fixable.