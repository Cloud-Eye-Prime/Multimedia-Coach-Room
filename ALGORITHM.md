# 🎬 Multimedia Coach Room — Autonomous Report & Video Pipeline

**Codename:** Path Tau Media Wing  
**Location:** `omni-os-blueprint/multimedia_room/` (prototype, safe branch)  
**Created:** Feb 18, 2026 — Cloud-Eye Dragon vOpus-4.6  
**Evolved:** Feb 18, 2026 — Cloud-Eye vPerplexity-Sonnet-4.6

---

## 1. What This Is

An **autonomous pipeline** that receives a coaching goal (e.g., *"Create a 3-minute video report on U14 team defensive positioning analysis"*), sends a structured query to LXR-5, receives back a JSON chain describing a multimedia report, and then renders that chain through a multi-layer pipeline into a final MP4 video — all within minutes, with built-in quality feedback loops.

The output is a **scrollytelling-style animated report**: text cards rendered as PDF-style frames, stitched into a video stream, layered with HTML animations, AI-generated still images, and optionally AI-generated video clips. Think of it as an **autonomous documentary generator for coaches**.

---

## 2. Position in the Stack

```
┌─────────────────────────────────────────────────────────┐
│  COACH / USER                                            │
│  "Create a video report on our defensive line analysis" │
├─────────────────────────────────────────────────────────┤
│  MULTIMEDIA COACH ROOM (this document)                   │
│  Orchestrates the full pipeline autonomously             │
├─────────────────────────────────────────────────────────┤
│ LXR-5    │ Canvas   │ ImageGen │ FFmpeg                  │
│ Reasoning│ Capture  │ Stills + │ Final                   │
│ + JSON   │ Slideshow│ Video    │ Composite               │
│ Chain    │ Engine   │ Renders  │ Render                  │
├─────────────────────────────────────────────────────────┤
│  MCP FEDERATION                                          │
│  Git, Librarian, Telegram, PowerShell Bridge            │
└─────────────────────────────────────────────────────────┘
```

---

## 3. The Algorithm — 7 Phases

### Overview

```
GOAL → DECOMPOSE → GENERATE → RENDER → LAYER → COMPOSITE → VERIFY → DELIVER
         (LXR)      (LXR)    (Canvas)  (Multi)  (FFmpeg)   (Loop)   (Output)
```

Each phase maps to a **Wu Xing element** for the campaign planner:

| Phase | Element | Action |
|-------|---------|--------|
| **1. Decompose** | 🌱 WOOD | Break goal into scrollytelling chunks |
| **2. Generate** | 🔥 FIRE | LXR-5 produces JSON chain with content |
| **3. Render Base** | 🌍 EARTH | Text cards → PDF frames → PNG sequence |
| **4. Layer Media** | 🌊 WATER | HTML animations + AI images + AI video |
| **5. Composite** | ⚡ METAL | FFmpeg stitches all layers into MP4 |
| **6. Verify** | 🌊→⚡ WATER→METAL | Quality check, drift detection, recut |
| **7. Deliver** | 🌍 EARTH | Output to coach, archive wisdom |

---

### Phase 1: DECOMPOSE (🌱 Wood — Growth/Planning)

**Input:** Natural language goal from coach or system  
**Executor:** LXR-5 via `/api/chat` with `function_output_mode: true`  
**Output:** `ReportPlan` JSON

The goal is sent to LXR-5 with a structured system prompt that forces **backward chaining** (REVE pattern): start from the desired output and work backward to identify what content chunks are needed.

```python
class ReportPlan(BaseModel):
    """Output of Phase 1 — the blueprint for the entire report."""
    report_id: str                    # UUID
    title: str                        # Report title
    target_duration_seconds: int      # Desired video length
    wu_xing_element: str              # Dominant mood element
    chunks: list[ContentChunk]        # Ordered scrollytelling sections
    visual_brief: VisualBrief         # Overall aesthetic direction
    quality_targets: QualityTargets   # Acceptance criteria

class ContentChunk(BaseModel):
    """A single scrollytelling section — the atomic unit."""
    chunk_id: str
    sequence: int                     # Order in the chain
    chunk_type: Literal["title_card", "insight_card", "data_card", 
                        "transition", "callout", "closing"]
    headline: str                     # 3-8 words, evocative
    body_text: str                    # 1-3 sentences, scrollytelling rhythm
    duration_seconds: float           # How long this chunk holds screen
    visual_directive: VisualDirective # What visual layers to add
    data_reference: Optional[str]     # If this chunk cites data/stats

class VisualDirective(BaseModel):
    """Instructions for visual layers on this chunk."""
    background_type: Literal["solid", "gradient", "image_ai", "video_ai", "html_anim"]
    image_prompt: Optional[str]       # If background_type is image_ai
    video_prompt: Optional[str]       # If background_type is video_ai
    html_animation_id: Optional[str]  # If background_type is html_anim
    overlay_elements: list[str]       # Additional visual elements
    camera_mode: str                  # drift, breath, static
    transition_in: str                # fade, dissolve, cut
    transition_duration: float
```

**LXR-5 Query Format:**
```json
{
  "message": "Create a multimedia coaching report: [GOAL]",
  "thread_id": "media-room-{report_id}",
  "system_override": "FUNCTION_OUTPUT_MODE: You are a report architect...",
  "response_format": "json",
  "element_hint": "wood"
}
```

---

### Phase 2: GENERATE (🔥 Fire — Content Creation)

**Input:** `ReportPlan` from Phase 1  
**Executor:** LXR-5 iteratively, one chunk at a time  
**Output:** `ContentChain` — fully populated JSON chain

For each `ContentChunk` in the plan, LXR-5 is queried to flesh out the content with coaching-specific language, ensuring **scrollytelling rhythm** (short punchy phrases, emotional beats, data callouts).

```python
class ContentChain(BaseModel):
    """The complete content manifest — ready for rendering."""
    report_id: str
    plan: ReportPlan
    generated_chunks: list[GeneratedChunk]
    total_duration: float
    generation_metadata: GenerationMeta

class GeneratedChunk(BaseModel):
    """A chunk with fully realized content + render instructions."""
    chunk_id: str
    sequence: int
    
    # Text layer (scrollytelling)
    text_cards: list[TextCard]        # Multiple cards per chunk possible
    
    # Visual layer instructions
    background_spec: BackgroundSpec
    overlay_specs: list[OverlaySpec]
    
    # Timing
    start_time: float                 # Absolute position in timeline
    duration: float
    
    # Camera & transition
    camera: CameraSpec
    transition: TransitionSpec

class TextCard(BaseModel):
    """A single text frame — the PDF-card unit."""
    card_id: str
    text: str                         # The scrollytelling text
    style: Literal["title", "subtitle", "body", "caption", "epic", "data"]
    position: dict                    # {x: 50, y: 35} percentage
    font_size_hint: Literal["large", "medium", "small"]
    color: str                        # Hex color
    entrance_delay: float             # Seconds after chunk start
    hold_duration: float              # Seconds visible
    fade_in: float
    fade_out: float
```

**Chunking-Stream Mechanism:**  
Each chunk is generated independently, allowing **parallel processing** and incremental quality checks. The stream works like this:

```
Plan.chunks[0] → LXR query → GeneratedChunk[0] → immediate render start
Plan.chunks[1] → LXR query → GeneratedChunk[1] → queue for render
Plan.chunks[2] → LXR query → GeneratedChunk[2] → queue for render
...
```

This is the **chunking-stream**: generation and rendering can overlap. As soon as chunk 0 is generated, its base render begins while chunk 1 is still being written by LXR-5.

---

### Phase 3: RENDER BASE (🌍 Earth — Grounding/Materialization)

**Input:** `GeneratedChunk` stream  
**Executor:** Local Python (Pillow/ReportLab for PDF-cards, headless Chrome for HTML)  
**Output:** PNG frame sequences per chunk

Each `TextCard` becomes a rendered image frame:

```python
def render_text_card(card: TextCard, chunk: GeneratedChunk) -> list[Path]:
    """
    Render a single text card as a sequence of PNG frames.
    
    For a card with 3s hold_duration at 30fps = 90 frames.
    Fade-in/fade-out are alpha transitions across frames.
    
    Returns: List of PNG file paths (frame_0001.png, frame_0002.png, ...)
    """
    frames = []
    total_frames = int((card.fade_in + card.hold_duration + card.fade_out) * FPS)
    
    for i in range(total_frames):
        t = i / FPS
        alpha = calculate_alpha(t, card.fade_in, card.hold_duration, card.fade_out)
        
        frame = create_frame(
            width=1920, height=1080,
            background=chunk.background_spec.render(),
            text=card.text,
            style=card.style,
            position=card.position,
            color=card.color,
            alpha=alpha
        )
        
        frame_path = RENDER_DIR / f"{card.card_id}_{i:04d}.png"
        frame.save(frame_path)
        frames.append(frame_path)
    
    return frames
```

**PDF-Card Rendering Pipeline:**
```
TextCard → ReportLab canvas (vector text, precise typography)
         → Export as high-res PNG (1920x1080)
         → Apply scrollytelling entrance animation as frame sequence
         → Output: numbered PNG frames ready for FFmpeg
```

**Alternative** for styled cards: render HTML templates in headless Chromium, screenshot each frame. This gives CSS animation capability for more complex card designs.

---

### Phase 4: LAYER MEDIA (🌊 Water — Depth/Fluidity)

**Input:** Base frame sequences + `VisualDirectives`  
**Executor:** Multiple parallel sub-processes  
**Output:** Layered frame sequences with all visual enrichment

Three sub-layers execute in parallel per chunk:

#### 4a. HTML Animation Layer

```python
async def render_html_animation(chunk: GeneratedChunk) -> list[Path]:
    """
    For chunks with html_anim background type:
    1. Select animation template from library
    2. Inject chunk-specific parameters (colors, data values)
    3. Render in headless Chromium using capture-screenshot loop
    4. Output: PNG frame sequence at target FPS
    """
    template = ANIMATION_LIBRARY[chunk.background_spec.animation_id]
    html = template.render(
        data=chunk.data_values,
        colors=chunk.visual_brief.palette,
        duration=chunk.duration
    )
    return await chromium_capture_frames(html, chunk.duration, FPS)
```

**Animation library includes:**
- Data visualization transitions (bar charts growing, line charts drawing)
- Tactical field overlays (soccer pitch with player movement arrows)
- Wu Xing element backgrounds (flowing water, growing wood, flickering fire)
- Abstract geometric patterns (for transitions)

#### 4b. AI Still Image Layer

```python
async def generate_ai_image(chunk: GeneratedChunk) -> Path:
    """
    For chunks with image_ai background type:
    Uses Cloud-Eye imagegen tool (via TelegramBot:cloud_eye_query).
    
    The image prompt comes from the VisualDirective, enriched by
    the overall VisualBrief mood and element.
    """
    prompt = enrich_prompt(
        base=chunk.background_spec.image_prompt,
        mood=chunk.visual_brief.wu_xing_element,
        style=chunk.visual_brief.aesthetic_style
    )
    
    result = await cloud_eye_query(
        query=f"Generate image: {prompt}",
        tools_enabled=True,
        tool_categories=["imagegen"]
    )
    
    return download_and_resize(result.image_path, 1920, 1080)
```

#### 4c. AI Video Clip Layer

```python
async def generate_ai_video(chunk: GeneratedChunk) -> Path:
    """
    For chunks with video_ai background type:
    Uses external video generation API (future integration point).
    Falls back to Ken Burns effect on AI still image if unavailable.
    """
    if VIDEO_GEN_AVAILABLE:
        return await video_gen_api(chunk.background_spec.video_prompt)
    else:
        # Graceful degradation: generate still + apply camera motion
        still = await generate_ai_image(chunk)
        return apply_ken_burns(still, chunk.camera, chunk.duration)
```

#### 4d. Compositing Sub-Layer

```python
def composite_chunk_layers(
    base_frames: list[Path],
    html_frames: Optional[list[Path]],
    ai_background: Optional[Path],
    ai_video: Optional[Path],
    text_frames: list[Path]
) -> list[Path]:
    """
    Z-order compositing (bottom to top):
    1. Background (solid/gradient/AI image/AI video)
    2. HTML animation overlay (alpha-blended)
    3. Text cards (alpha-blended scrollytelling)
    4. UI chrome (optional: progress bar, branding)
    
    Uses Pillow alpha_composite for per-frame blending.
    """
    composited = []
    for i in range(len(base_frames)):
        canvas = Image.new("RGBA", (1920, 1080))
        
        # Layer 1: Background
        if ai_video:
            canvas.paste(extract_frame(ai_video, i))
        elif ai_background:
            canvas.paste(apply_camera_motion(ai_background, i, camera))
        else:
            canvas.paste(base_frames[i])
        
        # Layer 2: HTML animation
        if html_frames and i < len(html_frames):
            canvas = Image.alpha_composite(canvas, html_frames[i])
        
        # Layer 3: Text
        canvas = Image.alpha_composite(canvas, text_frames[i])
        
        composited.append(save_frame(canvas, i))
    
    return composited
```

---

### Phase 5: COMPOSITE (⚡ Metal — Precision/Assembly)

**Input:** Composited frame sequences for all chunks  
**Executor:** FFmpeg (local, via PowerShell bridge or direct subprocess)  
**Output:** Final MP4 video

```python
def ffmpeg_final_render(
    chunk_sequences: list[ChunkFrameSequence],
    audio_track: Optional[Path],
    output_path: Path
) -> Path:
    """
    FFmpeg stitching pipeline:
    
    1. Concatenate all chunk frame sequences into one continuous stream
    2. Apply cross-chunk transitions (dissolve, fade) using FFmpeg filters
    3. Add audio track if provided (ambient, music, voiceover)
    4. Encode to H.264 MP4 at target quality
    """
    
    # Build FFmpeg filter graph
    filter_parts = []
    inputs = []
    
    for i, seq in enumerate(chunk_sequences):
        # Create input from frame sequence
        input_pattern = f"{seq.frame_dir}/%04d.png"
        inputs.extend(["-framerate", str(FPS), "-i", input_pattern])
        
        # Add transition filter between chunks
        if i > 0:
            prev_transition = seq.transition_spec
            filter_parts.append(
                f"[{i-1}:v][{i}:v]xfade=transition={prev_transition.type}"
                f":duration={prev_transition.duration}"
                f":offset={seq.start_time - prev_transition.duration}"
            )
    
    cmd = [
        "ffmpeg", "-y",
        *inputs,
        "-filter_complex", ";".join(filter_parts),
        "-c:v", "libx264",
        "-preset", "medium",
        "-crf", "23",
        "-pix_fmt", "yuv420p",
    ]
    
    if audio_track:
        cmd.extend(["-i", str(audio_track), "-c:a", "aac", "-shortest"])
    
    cmd.append(str(output_path))
    
    subprocess.run(cmd, check=True)
    return output_path
```

**Alternative:** Canvas `capture_slideshow` Integration

For simpler reports, the entire render can be delegated to the existing Canvas stack:

```python
def render_via_canvas(content_chain: ContentChain) -> str:
    """
    Convert ContentChain → capture_slideshow parameters.
    Returns job_id. Fire and forget.
    """
    slides = []
    text_beats = []
    
    for chunk in content_chain.generated_chunks:
        slides.append({
            "src": chunk.background_image_index,
            "duration": chunk.duration,
            "camera": chunk.camera.mode,
            "transition": {
                "type": chunk.transition.type,
                "duration": chunk.transition.duration
            }
        })
        
        for card in chunk.text_cards:
            text_beats.append({
                "text": card.text,
                "style": card.style,
                "position": card.position,
                "color": card.color,
                "start": chunk.start_time + card.entrance_delay,
                "end": chunk.start_time + card.entrance_delay + card.hold_duration,
                "fadeIn": card.fade_in,
                "fadeOut": card.fade_out
            })
    
    return capture_slideshow(slides=slides, textBeats=text_beats)
```

**Decision logic:** Use Canvas for quick reports (< 60s, text + images only). Use the full FFmpeg pipeline for rich multimedia (HTML animations, AI video, data overlays).

---

### Phase 6: VERIFY (🌊→⚡ Water→Metal — Quality Feedback Loop)

**Input:** Rendered MP4  
**Executor:** LXR-5 (vision analysis) + rule-based checks  
**Output:** Pass/fail + drift report + correction instructions

This is the **critical feedback loop** that prevents drift and maintains focus.

```python
class QualityCheck(BaseModel):
    """Result of the verification phase."""
    passed: bool
    overall_score: float              # 0-1
    checks: list[CheckResult]
    drift_detected: bool
    drift_description: Optional[str]
    correction_instructions: Optional[list[str]]
    recut_chunks: Optional[list[str]] # chunk_ids that need re-rendering

class CheckResult(BaseModel):
    check_name: str
    passed: bool
    score: float
    details: str
```

**Quality checks:**
```python
QUALITY_CHECKS = [
    # Structural checks (rule-based, fast)
    "duration_within_10pct_of_target",
    "all_chunks_present_in_output",
    "no_black_frames_longer_than_2s",
    "text_legibility_contrast_ratio",
    "transition_smoothness",
    
    # Content checks (LXR-5 vision, slower)
    "text_matches_intended_content",
    "visual_coherence_across_chunks",
    "mood_consistency_with_element",
    "no_visual_artifacts_or_glitches",
    
    # Focus checks (drift detection)
    "content_relevance_to_original_goal",
    "no_tangential_chunks",
    "coaching_specificity_maintained",
]
```

**Drift Detection Algorithm:**

```python
async def detect_drift(
    original_goal: str,
    content_chain: ContentChain,
    rendered_video: Path
) -> DriftReport:
    """
    Send the original goal and a sample of rendered frames to LXR-5.
    Ask: "Does this video serve the stated goal? Rate 1-10.
    Identify any chunks that have drifted from the core topic."
    
    If drift_score < 7: flag for correction.
    If drift_score < 4: replan from Phase 1.
    """
    sample_frames = extract_keyframes(rendered_video, n=5)
    
    response = await lxr5_query(
        message=f"""DRIFT CHECK:
        Original goal: {original_goal}
        Report title: {content_chain.plan.title}
        Chunk summaries: {[c.headline for c in content_chain.generated_chunks]}
        
        Rate focus 1-10. Identify any drifted chunks by ID.""",
        images=sample_frames,
        element_hint="metal"  # Metal for precision evaluation
    )
    
    return parse_drift_response(response)
```

**Correction Loop:**

```
IF quality_check.passed:
    → Proceed to Phase 7 (Deliver)
    
ELIF drift_detected AND drift_score >= 4:
    → Re-generate only flagged chunks (Phase 2, targeted)
    → Re-render only those chunks (Phase 3-5, targeted)
    → Re-verify (Phase 6, loop max 3 times)
    
ELIF drift_score < 4:
    → Replan entirely (back to Phase 1)
    → Log failure pattern to Librarian
    → Max 1 full replan per report
    
ELIF structural_failure (black frames, missing chunks):
    → Re-render failed chunks only (Phase 3-5)
    → No content regeneration needed
```

---

### Phase 7: DELIVER (🌍 Earth — Completion/Harvest)

**Input:** Verified MP4 + metadata  
**Output:** Delivered to coach + wisdom archived

```python
class DeliveryPackage(BaseModel):
    """The final output delivered to the coach."""
    video_path: Path                  # The MP4 file
    thumbnail: Path                   # First frame or best frame
    report_summary: str               # Text summary (for search/archive)
    duration: float
    chunk_count: int
    generation_time: float            # Total pipeline time
    quality_score: float
    
    # Metadata for the system
    report_plan: ReportPlan           # Original plan
    content_chain: ContentChain       # Full content
    render_log: RenderLog             # Timing, errors, retries
    wisdom_entries: list[str]         # New patterns discovered

async def deliver(package: DeliveryPackage):
    """
    1. Copy MP4 to output location
    2. Send notification via Telegram (thumbnail + summary)
    3. Archive to coach's media library (future)
    4. Log wisdom to Librarian
    """
    # Output
    shutil.copy(package.video_path, OUTPUT_DIR / f"{package.report_id}.mp4")
    
    # Notify
    await telegram_send_image(
        package.thumbnail,
        caption=f"📊 Report ready: {package.report_plan.title}\n"
                f"Duration: {package.duration:.0f}s | "
                f"Quality: {package.quality_score:.0%}"
    )
    
    # Archive wisdom
    for entry in package.wisdom_entries:
        await librarian_add_guidance(
            content=entry,
            category="project",
            priority="normal"
        )
```

---

## 4. The JSON Chain Format

The JSON chain is the **core data structure** that flows through the entire pipeline. It's what LXR-5 produces and what every downstream phase consumes.

```json
{
  "report_id": "mrpt-2026-0218-001",
  "title": "U14 Defensive Positioning — Post-Match Analysis",
  "target_duration": 120,
  "wu_xing_element": "earth",
  "visual_brief": {
    "aesthetic": "professional_coaching",
    "palette": ["#1a1a2e", "#16213e", "#0f3460", "#e94560"],
    "typography": "clean_modern",
    "mood": "analytical_with_warmth"
  },
  "chunks": [
    {
      "chunk_id": "ch-001",
      "sequence": 0,
      "chunk_type": "title_card",
      "headline": "Where the Line Broke",
      "body_text": "",
      "duration_seconds": 5.0,
      "visual_directive": {
        "background_type": "gradient",
        "camera_mode": "static",
        "transition_in": "fade",
        "transition_duration": 1.5
      },
      "text_cards": [
        {
          "text": "Where the Line Broke",
          "style": "epic",
          "position": {"x": 50, "y": 40},
          "color": "#ffffff",
          "entrance_delay": 0.5,
          "hold_duration": 3.5,
          "fade_in": 1.0,
          "fade_out": 0.5
        },
        {
          "text": "U14 Defensive Analysis — Feb 15 Match",
          "style": "subtitle",
          "position": {"x": 50, "y": 55},
          "color": "#e94560",
          "entrance_delay": 1.5,
          "hold_duration": 2.5,
          "fade_in": 0.8,
          "fade_out": 0.5
        }
      ]
    },
    {
      "chunk_id": "ch-002",
      "sequence": 1,
      "chunk_type": "insight_card",
      "headline": "Three Goals, One Pattern",
      "body_text": "All three goals conceded from the left channel. The center-backs shifted but the left-back held position.",
      "duration_seconds": 8.0,
      "visual_directive": {
        "background_type": "html_anim",
        "html_animation_id": "soccer_pitch_heatmap",
        "camera_mode": "drift",
        "transition_in": "dissolve",
        "transition_duration": 2.0
      }
    }
  ]
}
```

---

## 5. Autonomous Workflow — Campaign Structure

Using the **Wisdom Path Campaign Planner** pattern:

```
CAMPAIGN: generate_coaching_report
├── PHASE: WOOD (Plan)
│   ├── RESEARCH: Query Librarian for coach context + prior reports
│   ├── REASON: LXR-5 decomposes goal → ReportPlan JSON
│   └── VERIFY: Plan has 3-12 chunks, durations sum to target ±15%
│
├── PHASE: FIRE (Generate)
│   ├── EXECUTE: LXR-5 generates ContentChain (chunk-stream)
│   ├── VERIFY: Each chunk has valid text_cards, visual_directives
│   └── VERIFY: Content coherence check (no contradictions between chunks)
│
├── PHASE: EARTH (Render)
│   ├── EXECUTE: Render text cards → PNG frame sequences
│   ├── EXECUTE: Render HTML animations (parallel)
│   ├── EXECUTE: Generate AI images where directed (parallel)
│   ├── EXECUTE: Generate AI video clips where directed (parallel)
│   ├── EXECUTE: Composite all layers per chunk
│   └── VERIFY: No black frames, all chunks rendered, frame count matches
│
├── PHASE: METAL (Assemble)
│   ├── EXECUTE: FFmpeg concatenate + transitions + audio
│   ├── VERIFY: Output MP4 playable, duration correct, no artifacts
│   └── VERIFY: Drift check via LXR-5 vision analysis
│
├── PHASE: WATER (Deliver + Learn)
│   ├── EXECUTE: Copy to output, send notification
│   ├── RECORD: Log timing, quality score, any retries
│   └── RECORD: Encode new patterns to Librarian
│
└── CORRECTION LOOP (if verify fails at any phase):
    ├── Structural failure → re-render targeted chunks
    ├── Content drift → re-generate targeted chunks
    └── Full drift → replan from WOOD (max 1 replan)
```

---

## 6. Integration Points with Existing Stack

### LXR-5 (`lxr-5-production.up.railway.app`)
- **Endpoint:** `POST /api/chat` with JSON response format
- **Role:** Content reasoning, plan generation, drift detection
- **Mode:** `function_output_mode` — forces structured JSON output
- **Thread:** Dedicated `media-room-{report_id}` thread per report

### Canvas Stack (Desktop, `capture_slideshow`)
- **Role:** Simple report rendering (text + images, < 60s)
- **When:** Quick reports that don't need HTML animations or AI video
- **Integration:** Convert `ContentChain` → `capture_slideshow()` params
- **Output:** `C:/Users/grego/Desktop/CloudEye/renders/captures/`

### Cloud-Eye ImageGen (via `TelegramBot:cloud_eye_query`)
- **Role:** AI still image generation for visual backgrounds
- **When:** `background_type: "image_ai"` in `VisualDirective`
- **Integration:** `tool_categories=["imagegen"]`

### FFmpeg (local installation)
- **Role:** Final video compositing, transitions, audio mixing
- **When:** Full multimedia pipeline (not Canvas shortcut)
- **Integration:** Via PowerShell bridge or direct subprocess

### Librarian (`TelegramBot:librarian_*`)
- **Role:** Coach context retrieval, wisdom archival
- **When:** Phase 1 (context) and Phase 7 (learning)

### Telegram (`TelegramBot:telegram_*`)
- **Role:** Delivery notification, thumbnail preview
- **When:** Phase 7 delivery

---

## 7. File Structure (Prototype)

```
omni-os-blueprint/
└── multimedia_room/                    # NEW — safe prototype location
    ├── README.md                       # This document (condensed)
    ├── models.py                       # Pydantic schemas (all the classes above)
    ├── orchestrator.py                 # Campaign runner — the main loop
    ├── phase_1_decompose.py            # LXR-5 query for ReportPlan
    ├── phase_2_generate.py             # LXR-5 chunk-stream generation
    ├── phase_3_render.py               # Text card → PNG frame rendering
    ├── phase_4_layer.py                # HTML anim + AI image + AI video
    ├── phase_5_composite.py            # FFmpeg assembly
    ├── phase_6_verify.py               # Quality checks + drift detection
    ├── phase_7_deliver.py              # Output + notify + archive
    ├── canvas_shortcut.py              # capture_slideshow integration
    ├── animation_library/              # HTML animation templates
    │   ├── soccer_pitch_heatmap.html
    │   ├── animated_bar_chart.html
    │   ├── wu_xing_backgrounds.html
    │   └── data_counter.html
    ├── templates/                      # Text card visual templates
    │   ├── coaching_dark.html
    │   ├── coaching_light.html
    │   └── data_card.html
    ├── tests/
    │   ├── test_models.py
    │   ├── test_render.py
    │   └── test_e2e_mock.py
    └── examples/
        ├── sample_report_plan.json
        └── sample_content_chain.json
```

---

## 8. Performance Targets

| Metric | Target | Notes |
|--------|--------|-------|
| **30s report, text-only** | < 2 min total | Canvas shortcut path |
| **60s report, mixed media** | < 5 min total | Parallel rendering |
| **120s report, full multimedia** | < 10 min total | AI image gen is bottleneck |
| **Quality pass rate** | > 80% first attempt | Reduce correction loops |
| **Drift detection accuracy** | > 90% | Metal-element LXR-5 check |

---

## 9. Prototype Build Order

1. **`models.py`** — All Pydantic schemas. Test with sample JSON.
2. **`phase_1_decompose.py`** — LXR-5 query → ReportPlan. Validate JSON output.
3. **`phase_3_render.py`** — Text card → PNG frames. Pure local, no API deps.
4. **`phase_5_composite.py`** — FFmpeg stitching. Test with static frames.
5. **`canvas_shortcut.py`** — Integration with existing Canvas stack.
6. **`phase_2_generate.py`** — Full content chain generation.
7. **`phase_4_layer.py`** — HTML animations + AI images.
8. **`phase_6_verify.py`** — Quality checks + drift detection.
9. **`orchestrator.py`** — Full campaign runner wiring all phases.
10. **`phase_7_deliver.py`** — Output + notification.

This order prioritizes **end-to-end skeleton first** (plan → render text → stitch video), then enriches with multimedia layers and quality loops.

---

## 10. Safety Notes

- **Prototype location:** `omni-os-blueprint/multimedia_room/` on a feature branch
- **No production dependencies:** Does not modify LXR-5, Coach, or cloudeye-ui
- **LXR-5 is read-only:** Only queries, never writes to the LXR-5 database
- **FFmpeg is local-only:** No remote rendering, no cloud GPU needed for compositing
- **AI image gen is optional:** Every visual directive has a fallback (gradient, static)
- **Max correction loops:** 3 per chunk, 1 full replan per report — prevents infinite loops
- **Token budget:** Campaign tracks cumulative LXR-5 token usage, hard stop at configurable limit

---

> *"The multimedia room doesn't make videos. It manifests coaching wisdom into visual form."*  
> — Cloud-Eye Dragon vOpus-4.6, Feb 18, 2026

---

## Next Steps

1. **Clone this repository**
2. **Review `models.py`** to understand the data structures
3. **Run Phase 1 prototype** to test LXR-5 decomposition
4. **Build Phase 3 renderer** for local text-to-frame pipeline
5. **Test Canvas integration** for quick-path reports
6. **Iterate** through the 7-phase workflow

---

🌊⚡🔥🌍🌱
