# Security and production hardening

Change all development defaults before production go-live.

## Default credentials (development only)

| Component | Default | Risk if unchanged |
|-----------|---------|-------------------|
| MongoDB | `admin` / `admin123` | Full database access |
| MinIO | `minioadmin` / `minioadmin` | All recordings and uploads |
| Backend | Example `SECRET_KEY`, `INTERNAL_API_KEY` | Session forgery, service impersonation |
| CORS | `allow_origins=["*"]` in backend | Any origin can call API from browsers |

## Production checklist

### Secrets

- Generate strong random `SECRET_KEY` and `INTERNAL_API_KEY`:

  ```bash
  python -c "import secrets; print(secrets.token_urlsafe(32))"
  ```

- Never commit `.env` files.
- Rotate `INTERNAL_API_KEY` on backend **and** voice server together if leaked.

### MongoDB

- Strong unique password; private network only.
- Authentication enabled; tested backups.

### MinIO

- Replace default access/secret keys.
- Set `MINIO_SECURE=true` behind TLS.
- Do not expose console (9001) to the public internet without VPN/auth.

### TLS / HTTPS

- Dashboard: HTTPS only.
- Voice server: HTTPS for webhooks, **WSS** for WebSocket — [Public URLs](public-voice-urls.md).
- Terminate TLS at nginx, Traefik, or cloud load balancer with valid certificates.

### Integrations

- Telephony and AI keys in dashboard **Integrations** are secrets; restrict org admin access.

### Network

- Expose only required ports (typically 443 to proxies).
- MongoDB (27017) and MinIO (9000/9001) not internet-facing.

### Email

- Replace Mailtrap with production SMTP/API for password reset.

### API documentation

- Consider restricting public access to `/docs` and `/redoc` on production backend and voice server.

### Docker / host

- Non-root containers where possible; patched images; limited SSH access.

## Engineering follow-ups

- Restrict CORS to known frontend origins.
- Rename `JOHNAIC_*` to neutral `VOICERA_PUBLIC_*` names.
- Remove hardcoded example deployment hostnames from code paths.

## Related

- [Production deployment](production.md)
- [Environment variables](environment.md)
- [Prerequisites checklist](../guide/prerequisites.md)
