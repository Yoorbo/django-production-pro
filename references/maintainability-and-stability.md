# Django Maintainability and Stability

This reference covers code organization, project structure, operations, and long-term stability for Django projects.

## Project structure and modularity

- Follow Django’s philosophy of modular apps:
  - Each app has a clear responsibility (e.g. `users`, `billing`, `orders`).
  - Avoid “god” apps doing everything.
- Keep a clean project layout:
  - Dedicated folders for apps, logs, media, static, scripts.
  - Avoid deep, overly nested packages that make navigation difficult.

## Coding style and quality

- Follow Python’s PEP8 and Django’s coding style guidelines:
  - Consistent formatting with tools like `black` and `isort`.
  - Use `flake8` or similar linters to catch common mistakes.
- Keep views, models, and forms lean:
  - Avoid fat views; push business logic into services, model methods, or domain layers.
  - Prefer class-based views where they improve reuse and clarity.
- Write meaningful docstrings and comments where behavior is non-obvious.

## Settings and environment management

- Split settings by environment:
  - Common/base settings plus separate `development`, `staging`, `production` modules or equivalent.
  - Load secrets and environment-specific values from environment variables or env files.
- Avoid environment-specific hacks scattered through the code; centralize environment differences in settings and deployment config.

## Testing and regression safety

- Combine this skill with `django-testing-pro` for a comprehensive testing strategy.
- Ensure:
  - Key business flows have tests.
  - Security-sensitive areas (auth, permissions, payments) have regression tests.
- Run tests in CI on every merge/push and before deploys.

## Logging, monitoring, and alerting

- Logging:
  - Use Django’s logging configuration to route logs appropriately (console, file, external log sinks).
  - Include context such as request IDs, user IDs (where appropriate), and correlation IDs for tracing incidents.
- Monitoring:
  - Integrate metrics and/or APM (request throughput, latency, error rates).
  - Setup dashboards and alerts for key SLIs (availability, latency, error rate).
- Error tracking:
  - Use Sentry or similar tools to capture unhandled exceptions and track trends over time.

## Upgrades and dependency management

- Lock dependency versions and maintain a clear `requirements.txt` or `pyproject.toml`.
- Regularly schedule upgrade work:
  - Apply Django security patches promptly.
  - Test minor and major upgrades in staging before production.
  - Watch for deprecation warnings and fix them early.
- Keep migrations organized:
  - Run migrations as part of deployments.
  - Avoid large, dangerous schema changes without downtime planning or migration strategies.

## Operational stability

- Deployments:
  - Use repeatable, automated deployments (CI/CD) with clear rollbacks.
  - Run pre-deploy checks (tests, `check --deploy`) and migrations before switching traffic.
- Health checks:
  - Provide simple health endpoints (e.g. `/healthz`) that check DB connectivity and basic app readiness.
- Disaster readiness:
  - Ensure regular DB backups and restore drills.
  - Document incident response and on-call runbooks for critical flows.

These practices make Django projects easier to evolve, safer to operate, and more resilient to failures.