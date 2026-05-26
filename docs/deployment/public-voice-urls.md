# Public voice server URLs

Environment variables named `JOHNAIC_*` are **not** a third-party product. They mean the **public base URL of your deployed voice server** — the address on the internet used for:

- Telephony webhooks (Vobiz)
- WebSocket live audio
- Frontend answer URLs and **Test on Browser**

Early deployments used hostnames like `johnaic.com`; new documentation uses **public voice server URL**. Future releases may rename variables to `VOICERA_PUBLIC_URL` / `VOICERA_PUBLIC_WS_URL`.

## Variables

| Variable | Where | Meaning |
|----------|-------|---------|
| `JOHNAIC_SERVER_URL` | voice server | Public **HTTPS** base, e.g. `https://voice.example.gov.in` |
| `JOHNAIC_WEBSOCKET_URL` | voice server | Public **WSS** base, e.g. `wss://voice.example.gov.in` |
| `NEXT_PUBLIC_JOHNAIC_SERVER_URL` | frontend | Same HTTPS base; builds `/answer?agent_id=...` |
| `NEXT_PUBLIC_JOHNAIC_WEBSOCKET_URL` | frontend (optional) | Explicit WSS for Test on Browser |

## URLs built from these values

| Use case | Example |
|----------|---------|
| Vobiz answer webhook | `{JOHNAIC_SERVER_URL}/answer?agent_id={uuid}` |
| Audio WebSocket | `{JOHNAIC_WEBSOCKET_URL}/agent/{agent_id}` |
| Browser test | `wss://.../agent/{agent_id}` |

**Code references:**

- `voice_2_voice_server/api/server.py`
- `voicera_frontend/app/(dashboard)/assistants/page.tsx`
- `voicera_frontend/components/assistants/test-browser-dialog.tsx`

## Requirements

- Reachable from the **public internet** (telephony callbacks).
- Use **WSS** when the dashboard is served over HTTPS.
- DNS and TLS certificate must match the reverse proxy in front of `voice_server`.

## Local development

Use ngrok or similar and set both HTTPS and WSS bases. Prefer `wss://` for `JOHNAIC_WEBSOCKET_URL` when the tunnel provides secure WebSockets.

!!! warning
    Do **not** instruct customers to use example hostnames from early deployments (e.g. `vobiz.johnaic.com`). Use **your** domain.

## Related

- [Telephony](../services/telephony.md)
- [Configuration](../getting-started/configuration.md)
- [FAQ](../guide/faq.md)
