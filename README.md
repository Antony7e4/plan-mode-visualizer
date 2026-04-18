# Plan Mode Visualizer

> A Claude Code skill that turns Plan-mode clarifying questions into an interactive HTML preview, so you can *see* the alternatives (layouts, themes, flows, hierarchies) before you answer — and one click copies your picks back into the terminal.

English · [简体中文](README.zh-CN.md)

---

## Install

One-line install into Claude Code's user skills directory:

```bash
git clone https://github.com/Antony7e4/plan-mode-visualizer.git ~/.claude/skills/plan-mode-visualizer
```

Restart Claude Code or start a new session — the skill is picked up automatically.

## What it does

When Claude Code enters **Plan mode** (or you say "interview me" / "ask me some clarifying questions before you build"), Claude normally fires a stream of text questions at you. But for questions about design style, UI layout, component arrangement — text alternatives are hard to picture clearly.

This skill steps in at those moments and produces a **single self-contained HTML file**:

- Each option rendered at the right fidelity — **wireframes** (layout), **high-fidelity mockups** (visual style), **Mermaid diagrams** (flow / state).
- **Click to choose**, with a **live preview** where it makes sense (click "Dark" → the sample UI recolors instantly).
- A sticky footer collects all selections; **"Copy answers"** writes a formatted string to your clipboard:
  ```
  Q1: A  (Card layout)
  Q2: B  (Dark theme)
  Q3: A  (Top navigation)
  ```
  Paste it back into the terminal as your reply.

One HTML file, opened with `file://`, no server, no build step. Tailwind and Mermaid are loaded via CDN.

## When the skill triggers (and when it doesn't)

Activates only when **both** conditions hold:

1. **Context** — Claude Code is in Plan mode, OR you explicitly ask for an interview / clarification round.
2. **Content** — at least one upcoming question has a meaningful **visual, spatial, flow, or hierarchy** dimension. Examples that trigger:
   - Layout: "top nav vs side nav", "2×2 grid vs single row"
   - Visual style: "light vs dark", "rounded vs sharp buttons"
   - Flow / state: "sync vs async processing pipeline"
   - Hierarchy: "component tree shape", "schema nesting"

Does **not** trigger for pure technical choices (Postgres vs MySQL, timeout values, library versions) or when you've asked for a quick answer. Over-triggering is explicitly treated as a failure mode.

See [`assets/example-questions.md`](assets/example-questions.md) for three worked scenarios (dashboard page, upload feature, caching strategy) showing when it should and shouldn't fire.

## How it fits into Plan mode

```
┌─────────────────────────────┐
│  You: build me a dashboard  │
│      (Plan mode)            │
└─────────────┬───────────────┘
              │
              ▼
      Claude drafts questions
              │
       some visual? ──── no ──▶ ask in terminal only
              │
             yes
              │
              ▼
   Skill generates plan-<ts>.html
              │
              ▼
  Claude prints path + question list
              │
              ▼
   You open HTML → click → copy
              │
              ▼
   Paste "Q1: A  Q2: B …" in terminal
              │
              ▼
     Claude proceeds with build
```

## Repository contents

```
plan-mode-visualizer/
├── SKILL.md                     ← skill definition Claude Code loads
├── assets/
│   ├── template.html            ← base HTML Claude fills in per session
│   └── example-questions.md     ← three worked scenarios (trigger / no-trigger)
└── README.md / README.zh-CN.md
```

- **`SKILL.md`** — the authoritative spec. Lists trigger rules, fidelity rules (low-fi for layout, hi-fi for style, Mermaid for flow, tree for hierarchy), required HTML structure, and the quality bar.
- **`assets/template.html`** — a working starting point with header, three sample question sections (wireframe / hi-fi with live preview / Mermaid), sticky answer-collector footer, copy-to-clipboard, keyboard shortcuts (1/2/3). ~320 lines, self-contained.
- **`assets/example-questions.md`** — scenarios Claude can pattern-match against.

## License

MIT — see [LICENSE](LICENSE).
