# AdForge — AI-Powered Video Advertising Platform

> Production SaaS for fully automated long-form video ad generation — from product URL to a complete 60–90 second advertisement, with no human intervention.

---

## Overview

AdForge is a full-stack AI platform that automates the creation of video advertisements for e-commerce brands. Given a product URL or description, AdForge autonomously:

1. Researches the product and competitive landscape
2. Writes a multi-scene creative strategy and script
3. Breaks the script into 6–10 individual scenes with distinct visual types
4. Generates an AI avatar character and animates each scene independently and in parallel
5. Synthesizes a natural voiceover matched to each scene's pacing
6. Concatenates all scenes into a single cohesive video — typically **60 to 90 seconds** — with synchronized audio
7. Publishes the final video ready for Meta/TikTok ads

The backend is a **150,000+ line production Python codebase**, deployed on AWS and serving real customers.

---

## The Core Innovation — Parallel Multi-Scene Pipeline

Most AI video tools generate a single short clip (5–15 seconds). AdForge solves the **long-form coherence problem**: generating multiple scenes that look and feel like one continuous shoot.

```
Product Brief
      │
      ▼
┌─────────────────────────────────────────────────┐
│         Multi-Agent Scene Breakdown              │
│                                                 │
│  Claude + GPT-4o + Gemini vote on:              │
│  • Number of scenes (6–10)                      │
│  • Visual type per scene                        │
│    (avatar_visible / face_closeup /             │
│     lifestyle_ambient / pov_hands /             │
│     product_only)                               │
│  • Duration per scene (2–9 seconds)             │
│  • Dialogue + narrative beat per scene          │
└──────────────────┬──────────────────────────────┘
                   │
                   ▼
┌─────────────────────────────────────────────────┐
│         Parallel Scene Generation               │
│                                                 │
│  Scene 1 ──▶ [Avatar Gen → Image-to-Video]     │
│  Scene 2 ──▶ [Avatar Gen → Image-to-Video]     │  ← All run
│  Scene 3 ──▶ [Avatar Gen → Image-to-Video]     │    in parallel
│  Scene 4 ──▶ [Avatar Gen → Image-to-Video]     │    via asyncio
│  ...                                           │
│  Scene N ──▶ [Avatar Gen → Image-to-Video]     │
└──────────────────┬──────────────────────────────┘
                   │
                   ▼
┌─────────────────────────────────────────────────┐
│     Seedance Chain Mode (visual continuity)     │
│                                                 │
│  last_frame(scene N) → first_frame(scene N+1)  │
│  Creates seamless transitions between clips     │
└──────────────────┬──────────────────────────────┘
                   │
                   ▼
┌─────────────────────────────────────────────────┐
│         FFmpeg Concatenation + Audio Sync       │
│                                                 │
│  • Merge N video clips into one timeline        │
│  • Layer ElevenLabs voiceover (scene-matched)   │
│  • Normalize audio levels                       │
│  • Output: 60–90 second final video             │
└─────────────────────────────────────────────────┘
                   │
                   ▼
            Final Video (MongoDB + S3)
```

---

## AI Providers Integrated (10+)

| Provider | Role in Pipeline |
|---|---|
| **Claude (Anthropic)** | Scene breakdown, creative strategy, script generation — primary reasoning engine |
| **GPT-4o (OpenAI)** | Scene breakdown consensus vote, image generation (gpt-image-2) |
| **Gemini / Nano Banana** | Avatar character image generation (6–10 candidates per job) |
| **Kling** | Image-to-video synthesis — animates avatar reference images |
| **Seedance** | Image-to-video with first+last frame interpolation — chain mode for continuity |
| **ElevenLabs** | Voice cloning and TTS — natural voiceover synchronized to scene duration |
| **HeyGen** | Alternative avatar video generation |
| **Sora (OpenAI)** | Cinematic product video generation (non-avatar style) |
| **KIE** | Text-to-image fallback |
| **Pinecone** | Vector search for creative intelligence and angle matching |

---

## Key Technical Components

### Multi-Agent LLM Orchestration
- **Consensus architecture**: Claude + GPT-4o + Gemini independently generate scene breakdowns; majority vote selects the best output
- All AI decisions are structured JSON — no keyword matching, no hardcoded rules
- Three-layer enforcement: strategic planner prompt → post-generation auto-fix → orchestrator re-validation
- Scene types strictly enforced per video style (e.g. `perfect_ugc_hybrid` allows only 3 of 5 visual types)

### Parallel Scene Generation
- All N scenes generated concurrently via `asyncio.gather()`
- Each scene: reference image generation (Nano Banana) → image-to-video (Kling or Seedance) → polling until complete
- Seedance **chain mode**: `last_frame` of scene N becomes `first_frame` of scene N+1 — creates visual continuity across the full video
- Duration mapper computes exact clip length per scene from word count and speaking pace (2.6 words/second)

### Async Architecture at Scale
- Every I/O operation non-blocking via `asyncio.to_thread()`
- MongoDB connection pool — round-robin across 4 connections
- ThreadPoolExecutor(100 workers) pre-warmed at startup
- AWS SQS job queues (main pipeline + scraping) with real-time progress updates
- Double-submit guard: active job check before any new job creation

### Full UGC Pipeline (Product URL → 60–90s Video)
1. **Product scraping** (Playwright) → extract product data, images, dominant colors
2. **Deep research** → competitor analysis, target personas, strategic creative angles
3. **Multi-agent scene breakdown** → 6–10 scenes with narrative arc (hook → problem → solution → CTA)
4. **Avatar character design** (Nano Banana) → generate and select best avatar image
5. **Parallel image-to-video** (Kling/Seedance) → animate each scene with motion
6. **Chain-mode continuity** → last frame of each scene anchors the next
7. **Voice synthesis** (ElevenLabs) → natural voiceover, paced to scene duration
8. **FFmpeg concat** → assemble all clips + audio into final long-form video
9. **Publish** → stored in MongoDB + S3, delivered to frontend

---

## Scale

- **150,000+ lines** of production Python
- **80+ service modules**
- **38 API route files**
- **10+ AI providers** coordinated in a single pipeline
- Billing system: credit deduction, refund-on-failure, subscription tiers
- Dual schema migration (in-production data layer upgrade without downtime)

---

## Tech Stack

| Layer | Technology |
|---|---|
| **Backend** | Python 3.11, FastAPI, asyncio |
| **Queue** | AWS SQS (main + scraping queues) |
| **Database** | MongoDB |
| **Storage** | AWS S3 |
| **Scraping** | Playwright (async, headless) |
| **Video processing** | FFmpeg, Kling API, Seedance API |
| **Voice** | ElevenLabs API |
| **AI/LLM** | Claude (Anthropic), GPT-4o, Gemini, Sora, HeyGen |
| **Vector search** | Pinecone |
| **Infrastructure** | AWS ECS, Docker |

---

## Note on Code Availability

The full codebase is proprietary and not publicly available. This repository provides an architectural overview for research and portfolio purposes.

For technical questions or research collaboration inquiries: [a.deghbouche@esi-sba.dz](mailto:a.deghbouche@esi-sba.dz)
