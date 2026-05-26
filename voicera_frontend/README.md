# VoicERA frontend

Next.js **dashboard** for operators: agents (assistants), phone numbers, integrations, meetings, knowledge base, and campaigns.

## When you need it

Required for all deployments that use the web UI (port **3000** in Docker, or `make start-frontend` locally).

## Run

### Docker (recommended)

Started with `make start-all-services` as the `frontend` service.

### Local development

```bash
cd voicera_frontend
cp .env.example .env.local   # if present
npm install
npm run dev
```

Open http://localhost:3000

## Environment

| Variable | Purpose |
|----------|---------|
| `NEXT_PUBLIC_API_URL` | Backend API for browser |
| `API_URL` | Server-side backend URL |
| `VOICE_SERVER_URL` | Voice server (server-side) |
| `NEXT_PUBLIC_JOHNAIC_SERVER_URL` | Public HTTPS base for telephony answer URLs — [docs](../docs/deployment/public-voice-urls.md) |

## Depends on

- **Backend** (`:8000`) for data and auth
- **Voice server** (`:7860`) for Test on Browser and call orchestration

## Operator documentation

- [Dashboard walkthrough](../docs/guide/dashboard.md)
- [Integrations](../docs/services/integrations.md) — **Vobiz credentials go here**
- [Frontend service doc](../docs/services/frontend.md)
