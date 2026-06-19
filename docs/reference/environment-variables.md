---
description: Canonical reference for every environment variable used across Voicera services, grouped by service with defaults and requirement flags.
---

# Environment Variables

Every Voicera service is configured through environment variables, typically loaded from a per-service `.env` file. This page is the canonical list. For port-level defaults see [ports-and-defaults.md](ports-and-defaults.md); for credentials, see [../quickstart/default-credentials.md](../quickstart/default-credentials.md).

{% hint style="warning" %}
**Vobiz Auth ID / Token are not env vars in production.** They are stored per-organization in the database via **Dashboard -> Integrations** and consumed at call time by `fetch_integration_key(org_id, ...)`. The env entries below exist only for legacy single-tenant dev setups. See [../concepts/telephony-model.md](../concepts/telephony-model.md).
{% endhint %}

## Configuration files

| Service | File |
|---------|------|
| Backend | `voicera_backend/.env` (template: `voicera_backend/env.example`) |
| Voice server | `voice_2_voice_server/.env` |
| Frontend | `voicera_frontend/.env.local` |
| AI4Bharat STT (optional) | `ai4bharat_stt_server/.env` |
| AI4Bharat TTS (optional) | `ai4bharat_tts_server/.env` |

In Docker Compose deployments the same files are mounted via `env_file:` in `docker-compose.yml`. Service-name aliases (e.g. `mongodb`, `minio`, `backend`) resolve inside the `voicera_network` bridge.

---

## Backend (`voicera_backend/.env`)

### Database

| Name | Service | Default | Required | Description |
|------|---------|---------|----------|-------------|
| `MONGODB_HOST` | backend | `mongodb` (Docker) / `localhost` | yes | MongoDB hostname |
| `MONGODB_PORT` | backend | `27017` | yes | MongoDB port |
| `MONGODB_USER` | backend | `admin` | yes | Root username |
| `MONGODB_PASSWORD` | backend | `admin123` | yes | Root password — change in production |
| `MONGODB_DATABASE` | backend | `voicera` | yes | Database name |
| `MONGODB_AUTH_SOURCE` | backend | `admin` | no | Auth database |
| `MONGODB_MAX_POOL_SIZE` | backend | `50` | no | Connection pool size |
| `MONGODB_TIMEOUT_MS` | backend | `5000` | no | Connection timeout |

### Object storage (MinIO)

| Name | Service | Default | Required | Description |
|------|---------|---------|----------|-------------|
| `MINIO_ENDPOINT` | backend | `minio:9000` | yes | MinIO host:port |
| `MINIO_ACCESS_KEY` | backend | `minioadmin` | yes | Root access key |
| `MINIO_SECRET_KEY` | backend | `minioadmin` | yes | Root secret key — change in production |
| `MINIO_SECURE` | backend | `false` | no | Use HTTPS (`true` in production) |
| `MINIO_REGION` | backend | `us-east-1` | no | Region label |
| `MINIO_BUCKET_RECORDINGS` | backend | `recordings` | no | Recordings bucket |
| `MINIO_BUCKET_TRANSCRIPTS` | backend | `transcripts` | no | Transcripts bucket |

### Security and auth

| Name | Service | Default | Required | Description |
|------|---------|---------|----------|-------------|
| `SECRET_KEY` | backend | `secret_key` | yes | JWT signing secret — must be changed in production |
| `JWT_ALGORITHM` | backend | `HS256` | no | Signing algorithm |
| `JWT_EXPIRATION_HOURS` | backend | `24` | no | Access token TTL |
| `JWT_REFRESH_EXPIRATION_DAYS` | backend | `30` | no | Refresh token TTL |
| `INTERNAL_API_KEY` | backend | – | yes | Shared secret for voice-server-to-backend calls. Generate with `python -c "import secrets; print(secrets.token_urlsafe(32))"` |
| `CORS_ORIGINS` | backend | `["http://localhost:3000"]` | no | JSON list of allowed origins |
| `DEBUG` | backend | `False` | no | Enable verbose error responses |

### Email (Mailtrap)

| Name | Service | Default | Required | Description |
|------|---------|---------|----------|-------------|
| `MAILTRAP_API_TOKEN` | backend | – | no | Mailtrap API token for transactional email |
| `MAILTRAP_FROM_EMAIL` | backend | `noreply@voicera.com` | no | From address |
| `MAILTRAP_FROM_NAME` | backend | `Voicera` | no | From name |
| `FRONTEND_URL` | backend | `http://localhost:3000` | no | Used in password-reset links |

### Vobiz (legacy / dev only)

| Name | Service | Default | Required | Description |
|------|---------|---------|----------|-------------|
| `VOBIZ_API_BASE_URL` | backend | – | no | Vobiz API base URL for application CRUD |
| `VOBIZ_ACCOUNT_ID` | backend | – | no | Single-tenant dev fallback |
| `VOBIZ_AUTH_ID` | backend | – | no | Single-tenant dev fallback (prefer Integrations) |
| `VOBIZ_AUTH_TOKEN` | backend | – | no | Single-tenant dev fallback (prefer Integrations) |

### RAG / Knowledge base

| Name | Service | Default | Required | Description |
|------|---------|---------|----------|-------------|
| `CHROMA_BASE_DIR` | backend | `voicera_backend/rag_system/chroma_data` | no | Override Chroma persistence root |

### API server

| Name | Service | Default | Required | Description |
|------|---------|---------|----------|-------------|
| `HOST` | backend | `0.0.0.0` | no | Bind address |
| `PORT` | backend | `8000` | no | Listen port |
| `LOG_LEVEL` | backend | `INFO` | no | `DEBUG`, `INFO`, `WARNING`, `ERROR` |

---

## Voice server (`voice_2_voice_server/.env`)

### Public URLs

| Name | Service | Default | Required | Description |
|------|---------|---------|----------|-------------|
| `JOHNAIC_SERVER_URL` | voice server | – | yes (prod) | Public HTTPS base for webhooks (e.g. `https://voice.example.com`) |
| `JOHNAIC_WEBSOCKET_URL` | voice server | – | yes (prod) | Public WSS base for the audio stream |

`JOHNAIC_*` is the **public voice server URL**, not a third-party product. For local development use an ngrok tunnel; see [../guides/developer/local-setup.md](../guides/developer/local-setup.md).

### Telephony — Vobiz

| Name | Service | Default | Required | Description |
|------|---------|---------|----------|-------------|
| `VOBIZ_API_BASE` | voice server | `https://api.vobiz.in/v1` | yes | Vobiz API base URL |
| `VOBIZ_CALLER_ID` | voice server | – | no | Default outbound caller ID |

### Backend integration

| Name | Service | Default | Required | Description |
|------|---------|---------|----------|-------------|
| `VOICERA_BACKEND_URL` | voice server | `http://backend:8000` | yes | Backend API URL (Compose alias) |
| `INTERNAL_API_KEY` | voice server | – | yes | Must match the backend's `INTERNAL_API_KEY` |
| `BACKEND_API_TIMEOUT` | voice server | `30` | no | Request timeout in seconds |

### Object storage

| Name | Service | Default | Required | Description |
|------|---------|---------|----------|-------------|
| `MINIO_ENDPOINT` | voice server | `minio:9000` | yes | MinIO host:port |
| `MINIO_ACCESS_KEY` | voice server | `minioadmin` | yes | Access key |
| `MINIO_SECRET_KEY` | voice server | `minioadmin` | yes | Secret key |
| `MINIO_SECURE` | voice server | `false` | no | Use HTTPS |

### LLM providers

| Name | Service | Default | Required | Description |
|------|---------|---------|----------|-------------|
| `LLM_PROVIDER` | voice server | `openai` | yes | `openai`, `anthropic`, `local`, etc. |
| `OPENAI_API_KEY` | voice server | – | conditional | Required when `LLM_PROVIDER=openai` |
| `OPENAI_MODEL` | voice server | `gpt-4` | no | Model name |
| `OPENAI_TEMPERATURE` | voice server | `0.7` | no | Sampling temperature |
| `OPENAI_MAX_TOKENS` | voice server | `200` | no | Response cap |
| `ANTHROPIC_API_KEY` | voice server | – | conditional | Required when `LLM_PROVIDER=anthropic` |
| `ANTHROPIC_MODEL` | voice server | `claude-3-opus` | no | Model name |
| `LOCAL_LLM_API_BASE` | voice server | – | conditional | Base URL for self-hosted LLM |
| `LOCAL_LLM_MODEL` | voice server | – | conditional | Local model id |

### STT providers

| Name | Service | Default | Required | Description |
|------|---------|---------|----------|-------------|
| `STT_PROVIDER` | voice server | `deepgram` | yes | `deepgram`, `google`, `ai4bharat` |
| `DEEPGRAM_API_KEY` | voice server | – | conditional | Required for Deepgram |
| `DEEPGRAM_MODEL` | voice server | `nova-2` | no | Model name |
| `DEEPGRAM_LANGUAGE` | voice server | `en` | no | Language code |
| `GOOGLE_CLOUD_STT_CREDENTIALS` | voice server | – | conditional | Path to service-account JSON |
| `STT_SERVICE_URL` | voice server | `http://ai4bharat_stt_server:8001` | conditional | AI4Bharat STT URL |
| `STT_LANGUAGE` | voice server | `hi` | no | AI4Bharat language code |
| `STT_SAMPLE_RATE` | voice server | `16000` | no | Sample rate in Hz |

### TTS providers

| Name | Service | Default | Required | Description |
|------|---------|---------|----------|-------------|
| `TTS_PROVIDER` | voice server | `cartesia` | yes | `cartesia`, `google`, `ai4bharat`, `elevenlabs` |
| `CARTESIA_API_KEY` | voice server | – | conditional | Required for Cartesia |
| `CARTESIA_VOICE` | voice server | `english_male` | no | Voice id |
| `CARTESIA_LANGUAGE` | voice server | `en` | no | Language code |
| `CARTESIA_SAMPLE_RATE` | voice server | `16000` | no | Output sample rate |
| `GOOGLE_CLOUD_TTS_CREDENTIALS` | voice server | – | conditional | Path to service-account JSON |
| `TTS_SERVICE_URL` | voice server | `http://ai4bharat_tts_server:8002` | conditional | AI4Bharat TTS URL |
| `TTS_LANGUAGE` | voice server | `hi` | no | AI4Bharat language code |
| `TTS_SPEAKER` | voice server | `female` | no | AI4Bharat speaker |

### Server, audio, logging

| Name | Service | Default | Required | Description |
|------|---------|---------|----------|-------------|
| `HOST` | voice server | `0.0.0.0` | no | Bind address |
| `PORT` | voice server | `7860` | no | Listen port |
| `AUDIO_SAMPLE_RATE` | voice server | `16000` | no | Pipeline sample rate (Hz) |
| `AUDIO_CHANNELS` | voice server | `1` | no | Mono |
| `AUDIO_FORMAT` | voice server | `pcm` | no | Codec |
| `SESSION_TIMEOUT_MINUTES` | voice server | `30` | no | Max call duration |
| `MAX_CONCURRENT_CALLS` | voice server | `100` | no | Concurrency cap |
| `ENABLE_CALL_RECORDING` | voice server | `true` | no | Persist WAV to MinIO |
| `LOG_LEVEL` | voice server | `INFO` | no | Logging level |
| `DEBUG_MODE` | voice server | `false` | no | Verbose pipeline logs |
| `ENABLE_AUDIO_LOGGING` | voice server | `false` | no | Log raw audio (CPU intensive) |

---

## Frontend (`voicera_frontend/.env.local`)

All public frontend env vars must be prefixed `NEXT_PUBLIC_` to be exposed to the browser bundle.

| Name | Service | Default | Required | Description |
|------|---------|---------|----------|-------------|
| `NEXT_PUBLIC_API_BASE_URL` | frontend | `http://localhost:8000` | yes | Backend REST base URL |
| `NEXT_PUBLIC_API_TIMEOUT` | frontend | `30000` | no | Request timeout in ms |
| `NEXT_PUBLIC_VOICE_SERVER_URL` | frontend | `http://localhost:7860` | yes | Voice server base URL |
| `NEXT_PUBLIC_WS_URL` | frontend | `ws://localhost:7860` | yes | Voice server WebSocket URL |
| `NEXT_PUBLIC_JOHNAIC_SERVER_URL` | frontend | – | yes (prod) | Public voice server URL used when creating agents |
| `NEXT_PUBLIC_AUTH_ENABLED` | frontend | `true` | no | Enable login UI |
| `NEXT_PUBLIC_JWT_STORAGE_KEY` | frontend | `voicera_token` | no | localStorage key |
| `NEXT_PUBLIC_APP_NAME` | frontend | `VoicEra` | no | Display name |
| `NEXT_PUBLIC_APP_VERSION` | frontend | `1.0.0` | no | Display version |
| `NEXT_PUBLIC_LOG_LEVEL` | frontend | `info` | no | `debug`, `info`, `warn`, `error` |
| `NEXT_PUBLIC_FEATURE_VOICE_CALLS` | frontend | `true` | no | Toggle browser test |
| `NEXT_PUBLIC_FEATURE_ANALYTICS` | frontend | `true` | no | Toggle analytics tab |
| `NEXT_PUBLIC_GA_TRACKING_ID` | frontend | – | no | Google Analytics id |
| `NEXT_PUBLIC_SENTRY_DSN` | frontend | – | no | Sentry DSN |

---

## AI4Bharat STT (`ai4bharat_stt_server/.env`)

| Name | Service | Default | Required | Description |
|------|---------|---------|----------|-------------|
| `HOST` | ai4bharat-stt | `0.0.0.0` | no | Bind address |
| `PORT` | ai4bharat-stt | `8001` | no | Listen port |
| `WORKERS` | ai4bharat-stt | `4` | no | Worker processes |
| `MODEL_NAME` | ai4bharat-stt | `indic-conformer-hi` | yes | Model id |
| `MODEL_PATH` | ai4bharat-stt | `/models` | yes | Path to downloaded model |
| `DEVICE` | ai4bharat-stt | `cuda` | no | `cuda` or `cpu` |
| `BATCH_SIZE` | ai4bharat-stt | `32` | no | Inference batch size |
| `SAMPLE_RATE` | ai4bharat-stt | `16000` | no | Expected sample rate |
| `LOG_LEVEL` | ai4bharat-stt | `INFO` | no | Logging level |

## AI4Bharat TTS (`ai4bharat_tts_server/.env`)

| Name | Service | Default | Required | Description |
|------|---------|---------|----------|-------------|
| `HOST` | ai4bharat-tts | `0.0.0.0` | no | Bind address |
| `PORT` | ai4bharat-tts | `8002` | no | Listen port |
| `WORKERS` | ai4bharat-tts | `2` | no | Worker processes |
| `MODEL_NAME` | ai4bharat-tts | `indic-parler-hi` | yes | Model id |
| `MODEL_PATH` | ai4bharat-tts | `/models` | yes | Path to downloaded model |
| `DEVICE` | ai4bharat-tts | `cuda` | no | `cuda` or `cpu` |
| `SAMPLE_RATE` | ai4bharat-tts | `16000` | no | Output sample rate |
| `SPEED` | ai4bharat-tts | `1.0` | no | Synthesis speed multiplier |
| `ENABLE_CACHING` | ai4bharat-tts | `true` | no | Cache synthesised clips |
| `CACHE_SIZE_MB` | ai4bharat-tts | `500` | no | Cache budget |
| `CACHE_TTL_HOURS` | ai4bharat-tts | `24` | no | Cache TTL |
| `LOG_LEVEL` | ai4bharat-tts | `INFO` | no | Logging level |

---

## Generating secrets

```bash
# JWT / INTERNAL_API_KEY
python -c "import secrets; print(secrets.token_urlsafe(32))"
openssl rand -hex 32

# MinIO credentials
openssl rand -base64 32
```

## Validation checklist

- All `Required: yes` variables are set.
- `INTERNAL_API_KEY` is identical between backend and voice server.
- `MONGODB_PASSWORD`, `MINIO_SECRET_KEY`, `SECRET_KEY` are changed from defaults before any exposure to a network.
- `CORS_ORIGINS` restricts to your real frontend origin in production.
- `JOHNAIC_*` URLs are HTTPS / WSS in production.
- API provider keys (OpenAI, Deepgram, Cartesia) belong to non-trial accounts in production.

For hardening guidance, see [../guides/deployment/security-hardening.md](../guides/deployment/security-hardening.md).

## Next steps

- [ports-and-defaults.md](ports-and-defaults.md) — Port map and default URLs
- [../quickstart/default-credentials.md](../quickstart/default-credentials.md) — Out-of-the-box passwords
- [../guides/deployment/docker-compose.md](../guides/deployment/docker-compose.md) — How env vars flow into Compose
- [../guides/deployment/production.md](../guides/deployment/production.md) — Production checklist
