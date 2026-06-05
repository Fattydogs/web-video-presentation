# Web Video Presentation Skill

**A method-driven agent skill for turning scripts, articles, and PDFs into click-driven 16:9 web presentations that can be screen-recorded as cinematic videos.**

[中文文档](./README.zh-CN.md)

---

## ⚠️ Fork & Attribution

This repository is a **fork / derivative work** of the original
[`web-video-presentation`](https://github.com/ConardLi/garden-skills/tree/main/skills/web-video-presentation)
skill maintained by **[@ConardLi](https://github.com/ConardLi)** in
[`garden-skills`](https://github.com/ConardLi/garden-skills/).

> **All credit for the original design, methodology, scaffold, themes, and
> documentation belongs to ConardLi.** This fork adds exactly **one feature
> on top**: PDF input bridging (see "What Changed" below).

| | Upstream | This fork |
|---|---|---|
| **Repo** | [`ConardLi/garden-skills`](https://github.com/ConardLi/garden-skills/) | `<your-github-username>/<your-repo>` |
| **Maintainer** | [@ConardLi](https://github.com/ConardLi) | `<your-github-username>` |
| **Skill version** | v1.2.1 | v1.2.1 + PDF bridging |

If you only need the original skill without PDF input, please use the
upstream — this fork is only useful if you want to feed it PDFs.

---

## What Changed

Exactly **one additive change** from the upstream v1.2.1:

### ➕ PDF input bridging

The upstream skill only accepted a `article.md` (markdown text) as its
content source. This fork adds a documented bridge that lets you feed a
**PDF file** (academic paper, technical report, white paper, eBook) as
the input. The bridge is implemented as documentation + workflow
guidance — it does not modify the upstream scaffold, themes, or audio
pipeline.

**Files added / modified in this fork:**

| File | Change | Why |
|---|---|---|
| `references/PDF-INPUT.md` | **new** (176 lines) | 4-step PDF → `article.md` bridge + 5 boundary cases (images, formulas, scanned PDFs, encrypted, super-long) + self-check list |
| `SKILL.md` | edited in 3 places | (1) New row in Phase 1.1 "user input" table for PDF; (2) PDF bridging note in the reading-guide table; (3) New entry in the "related resources" table |
| `README.md` / `README.zh-CN.md` | **rewritten** | This file. Adds fork attribution + new "PDF input bridging" chapter |
| All other files | **unchanged** | Scaffold, themes, audio pipeline, TTS providers, recording workflow — all preserved byte-for-byte from upstream |

> **No code was removed, no themes were modified, no scaffold was touched.**
> The only behavioral change is that an agent following this skill will
> now recognize a PDF input and route it through `PDF-INPUT.md` before
> continuing the normal Phase 1.1 → 1.2 → ... flow.

---

## What Is This?

`web-video-presentation` helps an agent build a Vite + React + TypeScript presentation that behaves like a video production surface rather than a slide deck. Each click advances one narration beat, each step owns the whole 1920×1080 stage, and the progress UI stays hidden unless hovered so the output is clean for screen recording.

It is designed for:

- Turning a written article into a Bilibili / YouTube / video-channel narration script
- **Turning a PDF paper / report into a narrated web video** (this fork's addition)
- Turning an existing voiceover script into a cinematic web presentation
- Building product demos, tutorials, keynote-style explainers, and visual talks
- Creating "dynamic PPT, but not PPT" experiences with strong motion and pacing
- Optionally synthesizing narration audio after the visual outline is approved

The skill is primarily a **methodology and collaboration workflow**. The scaffold supplies reusable tokens, stage primitives, themes, and examples, but each project should still choose a visual language that fits the topic.

---

## Core Ideas

- **Fixed 16:9 stage** — content is authored in a stable 1920×1080 coordinate system and scaled to the viewport.
- **One global step cursor** — click or keyboard advances `(chapter, step)`, with the cursor persisted locally.
- **One step, one idea** — every beat gets a focused full-screen scene instead of accumulating slide bullets.
- **Script beats drive structure** — narration rhythm maps directly to visual steps.
- **Hidden chrome** — progress controls are hover-only, keeping recordings clean.
- **Motion first** — each scene needs a moving visual anchor; static paragraphs are treated as a smell.
- **Theme tokens** — visual decisions flow through semantic tokens so themes can change the whole feel.
- **Pluggable TTS** — provider-agnostic audio runner ships **two built-in providers** (MiniMax `mmx-cli` and OpenAI TTS via curl); swap to ElevenLabs / edge-tts / Azure / Google Cloud / macOS `say` / any self-hosted TTS by dropping a single shell file into `tts-providers/`.
- **Hard checkpoints** — the agent pauses after script/theme alignment, after outline approval, and before optional audio synthesis.

---

## Workflow

```text
Phase 1.1  Identify input
   ├── article.md (markdown) ─────────────────┐
   ├── PDF (this fork) ──► PDF-INPUT.md bridge ┤
   └── existing voiceover script ──────────────┤
                                               ▼
Phase 1.2  Article -> narration script
   |
Checkpoint A1  Script, theme, and rough asset plan
   |
Phase 1.3  Script + article -> outline.md
   |
Checkpoint A2  Outline approval + development mode
   |
Phase 2    Build the Vite / React / TS presentation
   |
Checkpoint B   Ask whether to synthesize audio
   |
Phase 3    Optional audio synthesis
Phase 4    Recording and post-production
```

The checkpoints are part of the skill contract: the agent should not silently rush from raw article to finished code. Theme choice influences motion design, and outline approval keeps chapter pacing from drifting.

---

## PDF Input Bridging (this fork)

If your source material is a PDF, follow the bridge defined in
[`references/PDF-INPUT.md`](./references/PDF-INPUT.md). Summary:

### The 4-step bridge

```text
PDF file
  ↓ 1. Read tool reads it (auto page-by-page text extraction)
  ↓ 2. Clean: strip headers/footers, repair hyphenated breaks,
            restore tables, keep formulas as $...$ LaTeX
  ↓ 3. Write the cleaned text to article.md (with a meta-info header)
  ↓ 4. Continue with the normal Phase 1.1 → ... flow
```

### Quick example (agent-side)

```python
# Step 1: read the PDF (≤20 pages in one go; >20 pages must split)
Read(path="paper.pdf")                       # ≤ 20 pages
Read(path="paper.pdf", pages="1-20")         # > 20 pages, batched
Read(path="paper.pdf", pages="21-45")

# Step 2+3: clean and write article.md
Write(path="article.md", content="""\
# <Paper title>

> Source: paper.pdf (<N> pages / <authors> / <venue>)
> Extraction: Read tool, page-by-page, then hand-cleaned
> Date: <YYYY-MM-DD>

<body text...>
""")

# Step 4: the skill's normal flow continues from here.
# SKILL.md Phase 1.2 sees article.md like any other input.
```

### Boundary cases the agent must warn about

| Case | What to do |
|---|---|
| **Images / figures / screenshots** | Read can't read images. User must export figures to `assets/figures/fig-N.png` separately; agent inserts `![desc](assets/figures/fig-N.png)` placeholders in `article.md` |
| **Formulas** | Kept as LaTeX inline (`$...$`) / display (`$$...$$`); OCR-style errors must be checked back against the source PDF |
| **Scanned PDF (no text layer)** | Read returns nothing. User must find an HTML version (e.g. arXiv) or run OCR externally |
| **Encrypted / DRM PDF** | Cannot bypass. User must decrypt with legal authorization first |
| **Super-long PDF (>50 pages)** | Don't read everything. Ask the user to scope the video to a specific section / page range, or split into multiple videos |

For the full self-check list, see
[`references/PDF-INPUT.md`](./references/PDF-INPUT.md#自检清单-pdf-桥接完成时过一遍).

---

## What It Ships

```text
skills/web-video-presentation/
├── SKILL.md
├── README.md / README.zh-CN.md
├── references/
│   ├── PDF-INPUT.md            ← new in this fork
│   ├── CHAPTER-CRAFT.md
│   ├── OUTLINE-FORMAT.md
│   ├── SCRIPT-STYLE.md
│   ├── THEMES.md
│   ├── AUDIO.md
│   └── RECORDING.md
├── scripts/
│   └── scaffold.sh
├── templates/
│   ├── index.html
│   ├── vite.config.ts
│   ├── scripts/
│   │   ├── extract-narrations.ts
│   │   ├── synthesize-audio.sh       # provider-agnostic runner
│   │   └── tts-providers/            # 1 file = 1 TTS backend
│   │       ├── README.md             # contract + ready-to-paste ElevenLabs / edge-tts / Azure / Google / say snippets
│   │       ├── minimax.sh            # default — uses mmx-cli
│   │       └── openai.sh             # built-in — uses OPENAI_API_KEY via curl
│   └── src/
└── themes/                    # 23 themes, each with its own signature
    ├── midnight-press/
    ├── warm-keynote/
    ├── newsroom/
    ├── bauhaus-bold/
    └── ...                     # full list in references/THEMES.md
```

---

## Quick Start

Copy the skill into the directory your agent scans, then ask it to turn a script, article, or **PDF** into a web-video presentation.

To scaffold manually from inside a project:

```bash
bash skills/web-video-presentation/scripts/scaffold.sh ./presentation --theme=paper-press
```

List available themes:

```bash
bash skills/web-video-presentation/scripts/scaffold.sh --list-themes
```

The generated `presentation/` project is a normal Vite + React + TypeScript app. Run it like any other Vite project, then record the 16:9 stage with your screen recorder.

---

## Built-In Theme Directions

The skill ships **23 themes**, each with its own design DNA — not a simple color swap. Browse the two groups below by canvas tone, pick one that fits, or use any of them as a starting point for a derived theme.

### Dark (8 themes)

- `midnight-press` — cinematic editorial dark, warm espresso + hot orange
- `chalk-garden` — slate chalkboard, handwritten Patrick Hand + chalk-yellow
- `terminal-green` — 80s phosphor CRT, mono-only + scanlines
- `blueprint` — drafting board, deep navy + cyan + 60px grid
- `dark-botanical` — premium editorial dark, terracotta / blush / gold glow
- `neon-cyber` — cyberpunk future, cyan + magenta double-neon
- `bold-signal` — hero pitch deck, dark gradient + orange focal card
- `creative-voltage` — saturated electric blue + neon yellow halftone

### Light (15 themes)

- `paper-press` — editorial paper, warm cream + hot orange
- `warm-keynote` — modern SaaS keynote, glass slab + teal + warm grid
- `newsroom` — NYT broadsheet, newsprint cream + banner red
- `bauhaus-bold` — manifesto modernist, 0 radius + 4px thick frame
- `sunset-zine` — risograph zine, peach + magenta + dashed cut lines
- `monochrome-print` — refined Monocle / Wallpaper print restraint
- `vintage-editorial` — witty Fraunces + geometric overlay (circle / line / dot)
- `pastel-dream` — soft pastel + sage + right-edge pill ribbon
- `split-canvas` — dual-tone, peach left + lavender right
- `electric-studio` — corporate clarity, crisp white + electric-blue base bar
- `indigo-porcelain` — indigo IS the ink (not just an accent) + porcelain white
- `forest-ink` — forest green IS the ink + ivory (vintage National Geographic)
- `kraft-paper` — deep brown IS the ink + kraft beige + copper accent
- `dune` — charcoal + sand, near-zero accent (architecture brochure)
- `swiss-ikb` — extra-light 200 weight Helvetica + IKB + 1px hairline grid

See [THEMES.md](./references/THEMES.md) for the full token contract, signature for each theme, and how to derive new themes from existing ones (including Swiss yellow / green / orange variants).

---

## Reference Map

- **[PDF-INPUT.md](./references/PDF-INPUT.md)** — **new in this fork** · PDF → `article.md` bridging workflow + boundary cases
- [CHAPTER-CRAFT.md](./references/CHAPTER-CRAFT.md) — chapter implementation rules and visual checklist
- [OUTLINE-FORMAT.md](./references/OUTLINE-FORMAT.md) — required outline structure
- [SCRIPT-STYLE.md](./references/SCRIPT-STYLE.md) — article-to-narration rewrite guidance
- [THEMES.md](./references/THEMES.md) — theme token contract + how to derive new themes
- [AUDIO.md](./references/AUDIO.md) — optional narration synthesis workflow (provider-agnostic)
- [tts-providers/README.md](./templates/scripts/tts-providers/README.md) — TTS provider contract + 2 built-ins (minimax / openai) + ready-to-paste snippets for ElevenLabs / edge-tts / Azure / Google Cloud / macOS say
- [RECORDING.md](./references/RECORDING.md) — screen recording and post-production notes

---

## Credits

### Original skill

- **Author**: [Conard Li (@ConardLi)](https://github.com/ConardLi)
- **Source repository**: [`ConardLi/garden-skills`](https://github.com/ConardLi/garden-skills/)
- **Original skill path**: [`skills/web-video-presentation`](https://github.com/ConardLi/garden-skills/tree/main/skills/web-video-presentation)
- **Upstream version**: v1.2.1
- **License**: inherited from upstream (see upstream repo)

> All methodology, scaffold, themes, audio pipeline, TTS provider
> contract, and original documentation are the work of ConardLi. This
> fork is a derivative work that adds the PDF input bridging feature
> documented in `references/PDF-INPUT.md` and `SKILL.md`.

### This fork

- **Maintainer**: `<your-github-username>`
- **Repository**: `<your-github-username>/<your-repo>`
- **Fork version**: v1.2.1 + PDF bridging
- **Sole addition**: `references/PDF-INPUT.md` (new) + 3 small edits in `SKILL.md` referencing it
- **All other code**: unchanged from upstream

### How to update from upstream

```bash
git remote add upstream https://github.com/ConardLi/garden-skills.git
git fetch upstream
git merge upstream/main --no-ff -m "merge upstream vX.Y.Z"
# If conflicts appear in SKILL.md, prefer the upstream version + re-apply
# the 3 PDF-INPUT.md references from this fork's commit history.
```

---

## License

Inherited from the upstream `garden-skills` repository. Please check
[`ConardLi/garden-skills`](https://github.com/ConardLi/garden-skills/)
for the license file before redistributing.
