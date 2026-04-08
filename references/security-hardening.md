# Django Security Hardening

This reference covers the core steps for shipping a secure Django application to production, building on Django’s deployment checklist and security guidance.

## Baseline: deployment checklist and checks

- Always run `python manage.py check --deploy` before deploying; fix all reported issues.
- Ensure:
  - `DEBUG = False` in production.
  - `ALLOWED_HOSTS` is set to explicit domains, not `['*']`.
  - `SECRET_KEY` is long, random, and loaded from environment variables; never committed to source control.

## HTTPS, cookies, and security headers

- Enforce HTTPS:
  - Use `SECURE_SSL_REDIRECT = True` behind a proper TLS-terminating proxy.
  - Set `SESSION_COOKIE_SECURE = True` and `CSRF_COOKIE_SECURE = True` to prevent cookies over plain HTTP.
  - Configure `SECURE_HSTS_SECONDS`, `SECURE_HSTS_INCLUDE_SUBDOMAINS`, and `SECURE_HSTS_PRELOAD` for HTTP Strict Transport Security when ready.
- Use `SecurityMiddleware` and configure:
  - `SECURE_CONTENT_TYPE_NOSNIFF = True`
  - `SECURE_BROWSER_XSS_FILTER = True` (for older browsers where relevant)
  - Content-Security-Policy headers via middleware or reverse proxy where appropriate.

## CSRF, XSS, and input validation

- Keep CSRF protection enabled by default:
  - Only use `@csrf_exempt` for very carefully justified endpoints.
  - Ensure templates include `{% csrf_token %}` in forms.
- Prevent XSS:
  - Use Django’s auto-escaping; avoid marking arbitrary user input `|safe`.
  - Validate and sanitize user input server‑side; never trust client‑side validation only.
- For Django REST framework:
  - Use serializers for validation and output normalization.
  - Avoid leaking internal fields (e.g., internal IDs, debug info) in serialized responses.

## Authentication, authorization, and permissions

- Use Django’s auth system and permission framework:
  - Enforce login where needed via `LoginRequiredMixin` or decorators.
  - Use per-object permissions where necessary; apply least privilege.
- For APIs:
  - Use DRF authentication (Token/JWT/session as appropriate) and permission classes.
  - Configure throttling and CORS explicitly; avoid `CORS_ALLOW_ALL_ORIGINS = True` in production.
- Ensure password storage uses Django’s recommended password hashers and that passwords are never logged or emailed in plain text.

## Secrets and environment management

- Keep secrets out of settings:
  - Use environment variables or a secrets manager (env files + `django-environ`, Vault, AWS/GCP secret managers).
- Lock down access:
  - Restrict who can read production env files and environment variables.
  - Use separate credentials for dev, staging, and production DBs and services.

## Dependencies, updates, and security scans

- Keep Django and major libraries up to date; subscribe to Django security announcements.
- Use dependency scanning (e.g. `pip-audit`, Snyk) in CI to catch vulnerable packages.
- Regularly run tests against new minor/patch versions in a staging environment before production.

## Logging, auditing, and monitoring

- Log security-relevant events:
  - Failed logins, permission denials, suspicious requests, admin actions.
- Avoid logging sensitive data:
  - Passwords, tokens, personal info, payment details.
- Integrate error tracking and monitoring (Sentry, etc.):
  - Capture exceptions and performance issues.
  - Configure alerts for abnormal error rates or suspicious patterns.

Use these checks to build a repeatable “security review before deployment” process.