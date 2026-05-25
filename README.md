# AdForge — AI-Powered Video Advertising Platform

> Production SaaS for automated video ad generation — from product URL to published video ad.

---

## Overview

AdForge is a full-stack AI platform that automates the creation of video advertisements for e-commerce brands. Given a product URL, AdForge generates a complete video ad — including creative strategy, script, avatar video, voiceover, and final edited video — without human intervention.

The backend is a **150,000+ line production Python codebase**, deployed on AWS and serving real customers.

---

## Architecture

```
Product URL / Description
        │
        ▼
┌─────────────────────────────────────────────────────┐
│              AdForge Backend (FastAPI)               │
│                                                     │
│  ┌─────────────┐    ┌──────────────────────────┐   │
│  │  API Routes  │───▶│   AWS SQS Job Queues      │   │
│  │  (38 routes) │    │   (main + scraping)       │   │
│  └─────────────┘    └──────────┬───────────────┘   │
│                                │                    │
│                    ┌───────────▼──────────────┐     │
│                    │    Async Workers          │     │
│                    │  AdGenerationWorker       │     │
│                    │  ScrapingWorker           │     │
│                    └───────────┬──────────────┘     │
│                                │                    │
│            ┌───────────────────▼──────────────┐     │
│            │         AI Pipeline               │     │
│            │                                  │     │
│            │  Research → Creative → Scenes    │     │
│            │  → Images → Video → Voice        │     │
│            │  → Concat → Upload               │     │
│            └──────────────────────────────────┘     │
└─────────────────────────────────────────────────────┘
        │
        ▼
  Firebase Storage + Firestore
```

---

## AI Providers Integrated (10+)

| Provider | Usage |
|---|---|
| **Claude (Anthropic)** | Scene breakdown, creative strategy, script generation, multi-model voting |
| **GPT-4o (OpenAI)** | Scene breakdown consensus, image generation (gpt-image-2) |
| **Gemini / Nano Banana** | Avatar image generation, scene soft-correction |
| **Kling** | Image-to-video synthesis (avatar scenes) |
| **Seedance** | Image-to-video with first+last frame interpolation |
| **ElevenLabs** | Voice cloning and TTS for UGC voiceovers |
| **HeyGen** | Avatar video generation |
| **Sora (OpenAI)** | Cinematic video generation |
| **KIE** | Text-to-image fallback |
| **Pinecone** | Vector search for creative intelligence |

---

## Key Technical Components

### Multi-Agent LLM Orchestration
- Claude + GPT-4o + Gemini vote on scene breakdown decisions (consensus architecture)
- Structured JSON output from all AI calls — no keyword matching, no hardcoded rules
- Scene breakdown service: strategic planner → script split → batch scene generation → enforcement gates

### UGC Video Pipeline
Full pipeline from brief to finished video:
1. **Product scraping** (Playwright) → extract product data, images, colors
2. **Deep research** → competitor analysis, personas, strategic angles
3. **Scene breakdown** → narrative beats, visual types, duration mapping
4. **Avatar generation** (Nano Banana) → character design from description
5. **Image-to-video** (Kling/Seedance) → animate each scene with first+last frame
6. **Voice synthesis** (ElevenLabs) → generate voiceover per script
7. **FFmpeg concat** → assemble final video with audio sync
8. **Upload** → Firebase Storage → notify frontend

### Async Architecture
- All I/O non-blocking via `asyncio.to_thread()`
- Firestore connection pool (4 gRPC channels, round-robin)
- ThreadPoolExecutor(100 workers) pre-warmed at startup
- Real-time progress updates via Firestore (frontend polls every 2–5s)

### Scale
- 150,000+ lines of production Python
- 80+ service modules
- 38 API route files
- Dual Firestore schema migration (flat + legacy nested)
- Credit/billing system with refund-on-failure

---

## Tech Stack

| Layer | Technology |
|---|---|
| **Backend** | Python 3.11, FastAPI, asyncio |
| **Queue** | AWS SQS (main + scraping queues) |
| **Database** | Google Firestore (Firebase Admin) |
| **Storage** | Firebase Storage |
| **Scraping** | Playwright (async) |
| **Video** | FFmpeg, Kling API, Seedance API |
| **Voice** | ElevenLabs API |
| **AI** | Claude, GPT-4o, Gemini, Sora, HeyGen |
| **Infrastructure** | AWS ECS, Docker |

---

## Note on Code Availability

The full codebase is proprietary and not publicly available. This repository provides an architectural overview for research and portfolio purposes.

For technical questions or research collaboration inquiries: [a.deghbouche@esi-sba.dz](mailto:a.deghbouche@esi-sba.dz)
