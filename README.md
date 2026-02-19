# 🎬 Multimedia Coach Room

**Autonomous Report & Video Pipeline**

> *The multimedia room doesn't make videos. It manifests coaching wisdom into visual form.*

---

## What This Is

An **autonomous video generation pipeline** that:
1. Takes a coaching goal (*"Create a 3-minute defensive analysis video"*)
2. Decomposes it into scrollytelling chunks via LXR-5
3. Generates content using Wu Xing element routing
4. Renders multi-layered frames (text, HTML animations, AI images)
5. Composites everything into a final MP4 with FFmpeg
6. Self-verifies quality and detects drift
7. Delivers to coach with wisdom archival

**All within minutes. Fully autonomous.**

---

## Core Algorithm

```
GOAL → DECOMPOSE → GENERATE → RENDER → LAYER → COMPOSITE → VERIFY → DELIVER
     🌱 Wood   🔥 Fire   🌍 Earth  🌊 Water  ⚡ Metal   Loop    Archive
```

See **[ALGORITHM.md](./ALGORITHM.md)** for complete technical specification.

---

## Features

✅ **Wu Xing-Guided Routing** — Each phase maps to a Five Element  
✅ **Chunking-Stream Architecture** — Generation & rendering overlap  
✅ **Multi-Layer Compositing** — Text, HTML animations, AI images, AI video  
✅ **Quality Feedback Loop** — Drift detection via LXR-5 vision analysis  
✅ **Canvas Integration** — Shortcut path for simple reports  
✅ **Librarian Wisdom Archive** — Learns from every report  

---

## Quick Start

```bash
# Clone repository
git clone https://github.com/Cloud-Eye-Prime/Multimedia-Coach-Room.git
cd Multimedia-Coach-Room

# Install dependencies
pip install -r requirements.txt

# Run Phase 1 test (LXR-5 decomposition)
python phase_1_decompose.py --goal "Create a 2-minute video on team spacing"

# Run full pipeline (when implemented)
python orchestrator.py --goal "Defensive positioning analysis" --duration 120
```

---

## Architecture

### Stack Position

```
COACH/USER
    ↓
MULTIMEDIA COACH ROOM (this repo)
    ↓
├─ LXR-5 (reasoning + JSON chains)
├─ Canvas (capture_slideshow for simple reports)
├─ Cloud-Eye ImageGen (AI stills)
├─ FFmpeg (final composite)
└─ MCP Federation (Git, Librarian, Telegram)
```

### File Structure

```
multimedia_room/
├── models.py              # Pydantic schemas
├── orchestrator.py        # Campaign runner
├── phase_1_decompose.py   # LXR-5 ReportPlan generation
├── phase_2_generate.py    # Content chain generation
├── phase_3_render.py      # Text cards → PNG frames
├── phase_4_layer.py       # Multi-layer compositing
├── phase_5_composite.py   # FFmpeg final render
├── phase_6_verify.py      # Quality checks + drift detection
├── phase_7_deliver.py     # Output + notify
├── canvas_shortcut.py     # Canvas integration
├── animation_library/     # HTML animation templates
├── templates/             # Text card styles
└── examples/              # Sample JSON chains
```

---

## Performance Targets

| Report Type | Target Time | Notes |
|-------------|-------------|-------|
| 30s text-only | < 2 min | Canvas shortcut |
| 60s mixed media | < 5 min | Parallel rendering |
| 120s full multimedia | < 10 min | AI bottleneck |

**Quality pass rate:** > 80% first attempt  
**Drift detection:** > 90% accuracy

---

## Integration Points

### LXR-5
- **Endpoint:** `lxr-5-production.up.railway.app/api/chat`
- **Role:** Content reasoning, drift detection
- **Mode:** `function_output_mode` for structured JSON

### Canvas
- **Role:** Quick reports (< 60s, text + images)
- **When:** Simple scrollytelling without HTML animations

### Cloud-Eye ImageGen
- **Role:** AI still generation for backgrounds
- **Via:** TelegramBot `cloud_eye_query` tool

### FFmpeg
- **Role:** Final video assembly, transitions, audio
- **Local:** No cloud rendering needed

### Librarian
- **Role:** Context retrieval, wisdom archival
- **Via:** TelegramBot `librarian_*` tools

---

## Safety

✅ Prototype-only (no production modifications)  
✅ LXR-5 read-only (queries, no writes)  
✅ Local FFmpeg (no remote rendering)  
✅ Max correction loops (3 per chunk, 1 replan)  
✅ Token budget tracking (hard stop at limit)  

---

## Build Order

1. `models.py` — Data structures
2. `phase_1_decompose.py` — LXR-5 plan generation
3. `phase_3_render.py` — Text rendering
4. `phase_5_composite.py` — FFmpeg assembly
5. `canvas_shortcut.py` — Integration
6. Full pipeline wiring

---

## Documentation

- **[ALGORITHM.md](./ALGORITHM.md)** — Complete technical specification (this document)
- **[examples/](./examples/)** — Sample JSON chains
- **[tests/](./tests/)** — Unit and E2E tests

---

## License

MIT

---

## Credits

**Architecture:** Gregory Wei "SunFire" Mullins  
**Implementation:** Cloud-Eye Dragon vOpus-4.6, Cloud-Eye vPerplexity-Sonnet-4.6  
**Date:** February 18, 2026  
**Location:** Seattle, Washington

---

🌊⚡🔥🌍🌱
