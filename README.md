# Plan Mode Visualizer

> A Claude Code skill that turns Plan-mode clarifying questions into an interactive HTML preview, so you can *see* the alternatives (layouts, themes, flows, hierarchies) before you answer — and one click copies your picks back into the terminal.

English · [简体中文](README.zh-CN.md)

---

## What it does

When Claude Code enters **Plan mode** (or you say "interview me" / "采访我" / "ask me some clarifying questions before you build"), Claude normally fires a stream of text questions at you. For questions like *"card layout vs list layout"*, *"light theme vs dark theme"*, or *"sync vs async upload flow"*, text alternatives are slow to imagine and easy to misread.

This skill detects those moments and produces a **single self-contained HTML file** that:

- Shows each option as a **wireframe**, **high-fidelity mockup**, or **Mermaid diagram** — whichever fits the question type.
- Lets you **click to choose**, with a **live preview** where it makes sense (e.g. click "Dark" → the sample UI recolors instantly).
- Collects all your selections in a sticky footer and, on **"Copy answers"**, writes a formatted string like:
  ```
  Q1: A  (Card layout)
  Q2: B  (Dark theme)
  Q3: A  (Top navigation)
  ```
  …straight to your clipboard. You paste it back into the terminal as your reply.

One HTML file, opened with `file://`, no server, no build step. It uses Tailwind and Mermaid via CDN.

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

## Install

Drop the skill folder into your Claude Code user skills directory:

```bash
git clone https://github.com/Antony7e4/plan-mode-visualizer.git \
  ~/.claude/skills/plan-mode-visualizer
```

Or clone anywhere and symlink:

```bash
ln -s /path/to/plan-mode-visualizer ~/.claude/skills/plan-mode-visualizer
```

Claude Code discovers skills under `~/.claude/skills/` at startup. Restart Claude Code or open a new session, and the skill will appear in the available-skills list.

**Requirements**: Claude Code, and a browser to open the generated HTML. The HTML itself needs an internet connection the first time (CDN fetch for Tailwind + Mermaid + Inter font); cached afterwards.

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

## Customizing

- Want different fidelity rules (e.g. always hi-fi, never Mermaid)? Edit the **"Fidelity rules"** section of `SKILL.md`.
- Want a different clipboard output format? Edit the copy handler in `assets/template.html` and mirror the format description in `SKILL.md`'s "Answer collector" section.
- Want a different save location (default: `.claude-preview/` in the project root, else `/tmp/`)? Edit the **"File location"** section of `SKILL.md`.

`SKILL.md` is prose Claude reads — change the rules in plain language, no code changes needed.

## Why this exists

Text-only clarification rounds have two failure modes:
- **You pick the wrong thing** because "2×2 grid" and "single row" sound similar on paper but feel very different on screen.
- **You skip the question** with a shrug because reading five abstract options is tiring.

Visualizing the comparable options takes the cognitive load off you and puts it onto a 2-second glance. That's the whole idea.

## License

MIT — see [LICENSE](LICENSE).

## Credits

Built for use with [Claude Code](https://claude.com/claude-code). `SKILL.md` follows the Claude Code skill format.
