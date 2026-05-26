# Integrations

Integrations let you securely store API keys for **AI providers** (LLMs, STT, TTS) and **telephony** (Vobiz) at the organisation level. The voice server fetches these at call time via internal APIs, so you do not put per-org provider keys in the voice server `.env` for normal multi-tenant operation.

## Telephony credentials (Vobiz)

!!! important
    **Vobiz Auth ID** and **Vobiz Auth Token** belong in **Dashboard → Integrations**, not in `voice_2_voice_server/.env` for production. The voice server reads them from MongoDB per `org_id` when placing outbound calls and when the backend manages Vobiz applications.

| Integration key | Used for |
|-----------------|----------|
| Vobiz Auth ID | Vobiz API account id |
| Vobiz Auth Token | Vobiz API token |

See [Telephony (Vobiz)](telephony.md) for call flow and webhook setup.

## How AI integrations work

```
Dashboard user
    │  POST /api/v1/integrations  (JWT)
    ▼
Backend → MongoDB Integrations collection (per org_id)

Voice Server
    │  POST /api/v1/integrations/bot/get-api-key  (INTERNAL_API_KEY)
    ▼
Backend → returns stored API key for requested model
```

When an agent uses OpenAI (LLM or Knowledge Base embeddings), the backend looks up the org's OpenAI integration key automatically — no environment variable needed on the voice server.

---

## Managing Integrations from the Dashboard

Navigate to **Integrations** in the left sidebar. You can:

- **Add** a new integration by selecting the provider and pasting your API key
- **View** existing integrations (keys are not shown in full after saving)
- **Delete** an integration

---

## API Reference

### List integrations

```
GET /api/v1/integrations
```

**Auth:** JWT

Returns all integrations for the current org.

---

### Get integration by model

```
GET /api/v1/integrations/{model}
```

**Auth:** JWT

Fetch a specific integration. The `model` parameter is the provider model name (e.g. `openai`, `deepgram`).

---

### Create integration

```
POST /api/v1/integrations
```

**Auth:** JWT

**Request body:**

```json
{
  "model": "openai",
  "api_key": "sk-...",
  "org_id": "your-org-id"
}
```

---

### Delete integration

```
DELETE /api/v1/integrations/{model}
```

**Auth:** JWT

---

### Get integration key (voice server)

```
POST /api/v1/integrations/bot/get-api-key
```

**Auth:** API Key (`X-API-Key`)

Used internally by the voice server to retrieve an org's provider key at call time.

**Request body:**

```json
{
  "org_id": "your-org-id",
  "model": "openai"
}
```

**Response:**

```json
{
  "api_key": "sk-..."
}
```

---

## Supported Providers

The following provider API keys can be stored as integrations:

### LLM Providers

| Provider | `model` value | Used for |
|----------|--------------|---------|
| OpenAI | `openai` | GPT models as LLM + Knowledge Base embeddings |
| Anthropic | `anthropic` | Claude models as LLM |
| Grok / xAI | `grok` | Grok models as LLM |

### STT Providers

| Provider | `model` value |
|----------|--------------|
| Deepgram | `deepgram` |
| ElevenLabs | `elevenlabs` |
| Sarvam | `sarvam` |
| Bhashini | `bhashini` |

### TTS Providers

| Provider | `model` value |
|----------|--------------|
| Cartesia | `cartesia` |
| Deepgram | `deepgram` |
| ElevenLabs | `elevenlabs` |
| Sarvam | `sarvam` |

---

## Integration vs. Environment Variable

Both approaches work. The integration database takes precedence for OpenAI when the agent config specifies an org_id.

| Approach | When to use |
|----------|------------|
| **Integration (database)** | Multi-tenant: different orgs can have different API keys; keys managed from the dashboard |
| **Environment variable** | Single-tenant or development: one global key for all orgs on this instance |

!!! note
    For Knowledge Base (RAG) embeddings, the org's **OpenAI integration must be configured** — the global `OPENAI_API_KEY` environment variable is not used for KB ingest or retrieval.

---

## Security

- API keys are stored in MongoDB in the `Integrations` collection, scoped by `org_id`.
- Keys are returned only via the `bot/get-api-key` endpoint which requires the `INTERNAL_API_KEY` header, restricting access to trusted services.
- In production, ensure `INTERNAL_API_KEY` is a long random string and is not exposed publicly.
