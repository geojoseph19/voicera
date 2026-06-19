---
description: Open-source voice AI platform for building telephony agents in Indian languages.
---

# Welcome to VoicEra

**VoicEra** is an open-source voice AI platform for building real-time conversational phone agents in Indian languages. It bundles speech-to-text, large language models, text-to-speech, and telephony into a single stack you can self-host.

Use VoicEra to run inbound helplines, outbound campaigns, IVR replacements, and citizen-services hotlines — without writing voice infrastructure from scratch.

{% hint style="info" %}
New here? Start with [What is VoicEra](introduction/what-is-voicera.md) for the plain-language overview, or jump straight to the [Quickstart](quickstart/install-and-run.md) if you have Docker ready.
{% endhint %}

## What you get

| Capability | What it does |
| --- | --- |
| **Real-time voice agents** | Sub-second STT → LLM → TTS loop powered by [Pipecat](concepts/voice-pipeline.md). |
| **Indian-language support** | Local STT/TTS via [AI4Bharat](services/ai4bharat-stt.md) servers or hosted providers. |
| **Telephony integration** | Inbound and outbound calls through [Vobiz](concepts/telephony-model.md) over WebSocket. |
| **Knowledge base (RAG)** | Ground answers in your own PDFs and documents. See [Knowledge base](concepts/knowledge-base-rag.md). |
| **Web dashboard** | Configure agents, link numbers, run campaigns, review transcripts and recordings. |
| **Self-hosted** | Docker Compose stack. Your data, your servers, your model keys. |

## Pick your path

{% tabs %}
{% tab title="I want to try it" %}
1. [Check prerequisites](quickstart/prerequisites.md)
2. [Install and run with Docker](quickstart/install-and-run.md)
3. [Make your first call](quickstart/first-call.md)
{% endtab %}

{% tab title="I'm an operator" %}
1. [Dashboard tour](guides/operator/dashboard-tour.md)
2. [Daily operations](guides/operator/operations.md)
3. [Operator FAQ](guides/operator/faq.md)
{% endtab %}

{% tab title="I'm a developer" %}
1. [Architecture](concepts/architecture.md)
2. [Local setup](guides/developer/local-setup.md)
3. [REST](reference/rest-api.md) and [WebSocket](reference/websocket-api.md) APIs
{% endtab %}

{% tab title="I'm deploying to prod" %}
1. [Production deployment](guides/deployment/production.md)
2. [Public voice URLs](guides/deployment/public-voice-urls.md)
3. [Security hardening](guides/deployment/security-hardening.md)
{% endtab %}
{% endtabs %}

## Architecture at a glance

```
┌──────────┐    ┌──────────┐    ┌──────────────┐
│ Frontend │◄──►│ Backend  │◄──►│ Voice Server │
│ Next.js  │    │ FastAPI  │    │ Pipecat      │
│ :3000    │    │ :8000    │    │ :7860        │
└──────────┘    └────┬─────┘    └──────┬───────┘
                     ▼                 ▼
                ┌─────────┐      ┌──────────┐
                │ MongoDB │      │  MinIO   │
                │ :27017  │      │ :9000/01 │
                └─────────┘      └──────────┘
```

Three services, two stores, optional local AI servers. Full diagram and call flow in [Architecture](concepts/architecture.md).

## Project links

* **Source** — [github.com/COSS-India/voicera\_mono\_repository](https://github.com/COSS-India/voicera_mono_repository)
* **License** — [MIT](legal/license.md)
* **Maintainer** — Centre for Open Source Software, India

## Need help?

* Browse the [Troubleshooting](troubleshooting/common-issues.md) section.
* Search the [Glossary](concepts/glossary.md) for unfamiliar terms.
* Open an issue on GitHub.
