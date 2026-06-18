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

## Custom LLM integrations

Custom LLMs let each organisation connect **multiple** OpenAI Chat Completions API v1-compatible endpoints (for example NVIDIA NIM, vLLM, or other hosted models). Configuration is stored in the `CustomLLMIntegrations` MongoDB collection — not the flat `Integrations` table.

```
Dashboard → Integrations → Custom LLM
    │  POST /api/v1/custom-llm-integrations  (JWT)
    ▼
Backend → MongoDB CustomLLMIntegrations (per org_id, multiple rows)

Assistant create/edit
    │  llm_model.name = "Custom LLM"
    │  llm_model.custom_llm_id = "<mongo_id>"
    ▼
Voice Server
    │  POST /api/v1/custom-llm-integrations/bot/get-config  (INTERNAL_API_KEY)
    ▼
Pipecat OpenAILLMService → POST {base_url}/chat/completions
```

Each custom LLM record stores:

| Field | Description |
|-------|-------------|
| `name` | Display label in the dashboard |
| `base_url` | Normalised OpenAI base URL (ends with `/v1`) |
| `api_key` | Bearer token for the endpoint |
| `model` | Model id sent in the chat completion request body |

**Agent config example:**

```json
{
  "llm_model": {
    "name": "Custom LLM",
    "custom_llm_id": "507f1f77bcf86cd799439011",
    "model": "google/gemma-4-26B-A4B-it"
  }
}
```

**Custom LLM API endpoints:**

| Method | Path | Auth |
|--------|------|------|
| `GET` | `/api/v1/custom-llm-integrations` | JWT |
| `POST` | `/api/v1/custom-llm-integrations` | JWT |
| `PUT` | `/api/v1/custom-llm-integrations/{id}` | JWT |
| `DELETE` | `/api/v1/custom-llm-integrations/{id}` | JWT |
| `POST` | `/api/v1/custom-llm-integrations/bot/get-config` | `X-API-Key` |

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
| Custom LLM | _(see Custom LLM section)_ | User-provided OpenAI-compatible endpoints |

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
