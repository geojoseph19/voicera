# Frequently asked questions

## Telephony and Integrations

### Do I put Vobiz credentials in a .env file?

No. For normal operation, enter **Vobiz Auth ID** and **Vobiz Auth Token** in **Dashboard → Integrations**. The voice server loads them from the database per organization. See [Integrations](../services/integrations.md).

### What is JOHNAIC in configuration?

Legacy name for your **public voice server URL** (HTTPS and WSS). Use your own domain. See [Public voice server URLs](../deployment/public-voice-urls.md).

### Test on Browser works but phone calls do not — why?

Browser test only needs the voice server and AI keys. Phone calls also need correct telephony webhooks, a public URL reachable from Vobiz, and the number linked to the agent. See [Telephony](../services/telephony.md).

### Call connects but the agent does not speak — why?

Often missing or invalid AI keys in **Integrations**, or wrong STT/TTS provider on the agent. Your hosting partner should check `voice_server` logs at the time of the call.

## Deployment

### Do we need a GPU?

Only if you run optional local **AI4Bharat** STT/TTS servers. Cloud-only speech can run without a GPU on the voice host (still size for concurrent calls).

### Does the system fall back from cloud to local speech automatically?

**No.** Each agent uses the STT/TTS providers configured on that agent.

### What are the default MongoDB and MinIO passwords?

Development defaults are in the technical README — **change them before production**. See [Security hardening](../deployment/security-hardening.md).

### Port already in use?

Partner can run `make stop-all-ports` or stop services on ports 3000, 8000, 7860, 27017, 9000, 9001.

## Dashboard

### What is an "agent"?

A configured virtual voice assistant — not a human team member. See [Overview](overview.md).

### How do I test without phone minutes?

**Test on Browser** on the agent card in Assistants.

### How do I make an outbound call?

Use campaigns/batches in the dashboard if enabled, or `POST /outbound/call/` on the voice server.

## Documentation and license

### Where is the full API list?

Backend: `http://<your-backend>:8000/docs`. Voice server: `http://<your-voice-host>:7860/docs`. See [Endpoint reference](../api/endpoints.md).

### Is the software MIT or proprietary?

**MIT License** — Copyright (c) 2026 COSS India. See [License](../legal/license.md).

## Getting help

### What should I send when reporting a problem?

Time of incident, agent name, phone number (if relevant), whether Test on Browser worked, any error messages shown, organization name.

### Who fixes server errors?

**Hosting partner** uses Docker logs; **operators** check dashboard configuration.
