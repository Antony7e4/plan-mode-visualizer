---
name: plan-mode-visualizer
description: Generate an interactive HTML preview alongside clarifying questions when Claude Code is in Plan mode or the user explicitly requests an interview/clarification session. Use this skill whenever the user enters Plan mode, says "采访我" / "interview me" / "ask me some questions" / "澄清需求" / "help me clarify requirements", AND at least one of the upcoming questions involves spatial/visual dimensions (layout, color, component placement, hierarchy, flow) OR has 2+ alternatives that would be easier to compare visually. The skill produces a single self-contained HTML file (Tailwind + Mermaid via CDN) that the user opens in a browser to preview options, toggle between them interactively, and click their choice — which auto-copies a formatted answer to clipboard for pasting back into the terminal. Do NOT trigger for pure technical choices (e.g., Postgres vs MySQL, timeout values) or when the user has asked for a quick answer.
---

# Plan Mode Visualizer

## Purpose

When Claude Code is planning and needs to ask the user clarifying questions, some of those questions are hard to answer from a text description alone — layout choices, color schemes, component arrangements, architecture diagrams, state flows. This skill augments the Q&A step by producing an interactive HTML preview so the user can **see** the alternatives before deciding.

## When to trigger

**Activate only when BOTH conditions are true:**

1. **Context condition** — either of:
   - Claude Code is in Plan mode AND about to ask the user clarifying questions
   - The user has explicitly asked to be interviewed (e.g., "采访我", "interview me", "ask me X questions", "澄清需求", "clarify requirements before you build")

2. **Content condition** — at least one of the upcoming questions meets ANY of these criteria:
   - **Spatial/visual dimension** — involves layout, component positioning, spacing, hierarchy, color, typography, or any "what does it look like" decision
   - **Multiple alternatives** — has 2+ options that would be meaningfully easier to compare side-by-side (e.g., "card layout vs list layout", "dark theme vs light theme")
   - **Flow/state** — involves user flow, state machine, data flow, or navigation paths
   - **Hierarchical structure** — involves file tree, component tree, nested data structure, or schema shape

**Do NOT trigger when:**
- All questions are pure technical choices (database engine, runtime version, timeout values, library versions, auth provider)
- The user has signaled they want a quick answer ("快速回答", "just pick one", "no need to visualize")
- The questions are all about parameters/thresholds/naming conventions with no visual component

## Generation timing

**Default: batch-then-generate.** Collect all planned questions, identify which ones need visualization, then produce ONE HTML file covering all of them in sections. This minimizes context-switching between terminal and browser.

**Exception — dependent questions:** If question B's visualization depends on the answer to question A (e.g., A: "card vs list layout", B: "information arrangement inside each card"), generate only A's visualization first. After the user answers A, generate B's visualization separately. Use judgment: if answering A meaningfully changes what B's options look like, they're dependent.

## File location

Decide based on the working context:

- **If there's a project root** (detect by presence of `.git`, `package.json`, `pyproject.toml`, etc.): create `.claude-preview/` in the project root. Add it to `.gitignore` if the user hasn't already. Name the file `plan-YYYYMMDD-HHMMSS.html`.
- **If no clear project root** (user is in `$HOME` or running ad-hoc): use `/tmp/claude-preview-YYYYMMDD-HHMMSS.html`.

**Always tell the user the exact absolute path** when presenting the questions. Example:
```
我为这次采访生成了一个可视化预览：
📍 /Users/alice/projects/myapp/.claude-preview/plan-20260418-143022.html

请在浏览器中打开查看，然后回答下面的问题。
```

## HTML construction

### Required structure

Every generated HTML file MUST contain, in order:

1. **Dependency manifest comment** at the very top — lists every CDN URL used, so the user can diagnose offline issues:
   ```html
   <!--
   Dependencies (CDN):
   - https://cdn.tailwindcss.com
   - https://cdn.jsdelivr.net/npm/mermaid@10/dist/mermaid.min.js  (only if diagrams used)
   - https://fonts.googleapis.com/css2?family=Inter  (only if used)
   -->
   ```

   **Important — avoid nested `<!-- -->` in the manifest comment.** HTML comments don't nest; `-->` inside a comment closes the outer block and leaks the rest onto the page. If you need to reference an HTML tag in a comment, write it as plain text (e.g., "section tags" not `<section>`).

   **Preserve the Tailwind warning suppression snippet** from `assets/template.html` (the small `<script>` block immediately after the `<title>` tag that overrides `console.warn`). Tailwind Play CDN logs a "not for production" warning on every page load. Since these previews are ephemeral local `file://` documents, the warning is noise — the suppression keeps the console clean for real errors.

2. **Header section** — title, timestamp, brief context of what's being planned, and a "How to use this page" instruction block.

3. **Question sections** — one `<section>` per question, each containing:
   - Question number and text (matching what Claude will ask in terminal)
   - The visualization (wireframe OR high-fidelity, see Fidelity rules below)
   - Interactive option selectors (radio cards or toggle buttons)
   - A "live preview" area if applicable (e.g., theme toggle updates the preview in real time)

4. **Answer collector** — a pinned footer or sidebar that:
   - Shows the user's current selections across all questions
   - Has a "Copy answers to clipboard" button that copies a formatted string like:
     ```
     Q1: A  (Card layout)
     Q2: B  (Dark theme)
     Q3: A  (Top navigation)
     ```
   - The user pastes this single string back into the terminal as their reply.

### Fidelity rules (mixed mode)

Apply per-question based on what the question is asking:

- **Layout questions** (component placement, page structure, navigation position, grid vs list) → **Low-fidelity wireframes**. Grayscale boxes with labels. Don't distract with colors/shadows/fonts — the decision is about structure.

- **Visual/style questions** (color scheme, theme, typography, button styles, spacing density) → **High-fidelity mockups**. Real colors, real fonts (via Google Fonts CDN), real shadows, realistic sample content. The decision IS about how it looks.

- **Flow/state questions** (user flow, state machine, data flow) → **Mermaid diagrams**. Use `flowchart`, `stateDiagram-v2`, or `sequenceDiagram` as appropriate.

- **Hierarchy questions** (component tree, file tree, schema shape) → **Indented tree with monospace font** OR Mermaid `flowchart LR`.

- **Mixed within one question** (e.g., "choose between these two entire landing pages") → high-fidelity for both, side-by-side.

### Interaction requirements

**Every question must be interactive, not static.** Specifically:

- **Radio-card selection**: each option rendered as a clickable card. Clicking highlights it with a ring/border and records the selection in a JS state object.
- **Live preview (when applicable)**: If the question is about a toggleable property that affects a preview (theme, density, color), clicking the option updates a preview panel in real time. Example: clicking "Dark theme" option swaps the sample UI's colors immediately.
- **Keyboard navigation**: support number keys (1, 2, 3) to select options within the currently-focused question.
- **Copy to clipboard**: single button at the bottom that assembles all selections into the format shown above and writes to `navigator.clipboard`. Show a toast/flash confirmation on success.

### Tech stack

- **Tailwind CSS** via `<script src="https://cdn.tailwindcss.com"></script>` — primary styling.
- **Mermaid** via CDN — only include if the HTML actually has a diagram (avoid unnecessary dependencies).
- **Google Fonts** (Inter, JetBrains Mono) — only include if high-fidelity sections exist.
- **No frameworks** — vanilla JS for state and interactions. Keep it under ~400 lines of JS.
- **Single file only** — all CSS, JS, SVG, assets inline or via CDN. No external local files.

### Self-containment check

Before finalizing, verify:
- File opens correctly with `file://` protocol (no CORS-dependent fetches)
- No references to local files the user doesn't have
- All fonts either have system fallbacks or are CDN-loaded
- Mermaid diagrams render (call `mermaid.initialize({startOnLoad: true})`)

## Output template

For a full working HTML template with all the patterns above pre-wired, see `assets/template.html`. Copy it, fill in the question sections, and adjust.

## What to say in the terminal

When the skill fires, Claude Code's terminal output should look like:

```
我需要先确认几个关键决策。其中涉及视觉和结构选择的部分，我生成了一个交互式预览帮你决策：

📍 预览路径: <absolute path>
   在浏览器中打开，点击你倾向的方案，最后点击"复制答案"按钮。

以下是完整的问题清单（共 N 个，其中 M 个在预览中有可视化对比）：

1. [问题 1 文本]
   [选项 A / B / C — 如在预览中有可视化则标注 "见预览 Q1"]

2. [问题 2 文本]
   ...

回答方式：
- 如果问题在预览中有可视化，你可以直接从浏览器复制答案字符串粘贴回来
- 否则用文字回答
```

## Quality bar

A good generated HTML should:
- Open and render correctly within 2 seconds on a normal connection
- Make each decision feel **faster** than reading text alternatives — if the user ends up reading more than looking, the visualization failed
- Never require the user to scroll more than twice to see all questions (use accordion or tabs if needed)
- Have the "copy answers" button always visible (sticky footer)

A bad generated HTML:
- Uses high-fidelity mockups for layout questions (clutter obscures structure)
- Uses wireframes for color questions (defeats the purpose)
- Has non-interactive options (user can't click, has to squint at labels)
- Forgets the copy-to-clipboard button (user has to manually type answers)
- Renders differently on mobile — not a hard requirement but avoid fixed widths

## Examples

See `assets/example-questions.md` for three example Q&A scenarios showing when this skill should trigger, what the question set looks like, and what the generated HTML should contain.
