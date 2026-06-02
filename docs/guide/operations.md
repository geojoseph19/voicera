# Operations guide

What operators can do from the dashboard, and what requires a hosting partner.

## Dashboard tasks (no SSH)

| Task | Where |
|------|--------|
| View call history | Meetings / calls |
| Create / edit agents | Assistants |
| Update telephony or AI credentials | Integrations |
| Link or unlink phone numbers | Phone numbers |
| Test without phone | Test on Browser |
| Team access | Members (if enabled) |
| Upload knowledge PDFs | Knowledge base |
| Outbound batches | Campaigns / batches (if enabled) |

There is **no built-in log viewer** in the dashboard today. Log access needs a hosting partner.

## Hosting partner tasks

| Task | Action |
|------|--------|
| Restart entire stack | `make stop-all-services` then `make start-all-services` |
| Restart one service | `docker compose restart backend` (or `voice_server`, `frontend`) |
| View logs | `docker compose logs -f <service>` |
| Check disk | `df -h`; monitor MinIO growth |
| Database backup | MongoDB backup per org policy |
| Renew TLS | Reverse proxy / certbot |
| Firewall | Allow 443 to frontend and voice proxies |

## Recordings

1. **Dashboard:** Meetings/calls — recording link if shown in UI.
2. **MinIO:** Partner may use console on port **9001** only on a secure internal network.
3. **API:** `GET /api/v1/call-recordings` — see [API reference](../api/endpoints.md).

## Troubleshooting failed calls (operators)

1. **Integrations** — Vobiz Auth ID and Token present?
2. **Phone numbers** — Number still linked to the correct agent?
3. **Test on Browser** — If this fails, fix voice/AI before telephony.
4. **Browser works, phone fails** — Webhook or public URL issue — [Telephony](../services/telephony.md), [Public URLs](../deployment/public-voice-urls.md).
5. **Call connects, agent silent** — AI or STT/TTS misconfiguration; partner checks `voice_server` logs.
6. **Call drops immediately** — Webhook reachability or WSS URL — partner checks voice server logs.

## Suggested maintenance

| Frequency | Action | Who |
|-----------|--------|-----|
| Daily | Glance at Meetings for failed/short calls | Operator |
| Weekly | Disk space / recording growth | Partner |
| Monthly | Integrations keys still valid | Operator + partner |
| Quarterly | Secret rotation review | Partner — [Security](../deployment/security-hardening.md) |
| As needed | OS/Docker image updates | Partner |

## Related

- [Verify it works](verification.md)
- [FAQ](faq.md)
- [Troubleshooting](../troubleshooting.md)
