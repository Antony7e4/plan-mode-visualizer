# Plan Mode Visualizer

> 一个 Claude Code skill：在 Plan 模式需要反问你澄清问题时，生成一个交互式 HTML 预览，让你在点选前**看到**备选方案（布局、配色、流程、层级），一键把选择复制回终端。

[English](README.md) · 简体中文

---

## 它做什么

当 Claude Code 进入 **Plan 模式**（或你说「采访我」/「先问我几个问题再开始」）时，Claude 通常会抛出一串文字问题。但对于类似「卡片布局还是列表布局」「浅色还是深色主题」「同步上传还是异步上传」这类问题，光靠文字想象既慢又容易误读。

这个 skill 会在这些时刻介入，生成**一个独立的 HTML 文件**：

- 把每个备选方案用合适的保真度呈现——**线框图**（布局类）、**高保真 mockup**（视觉风格类）、**Mermaid 流程图**（流程 / 状态机类）。
- 让你**点击选择**，并在合适的场景提供**实时预览**（比如点「深色」→ 示例 UI 立刻换色）。
- 把所有选择汇总在底部常驻栏，点击「**复制答案**」就把格式化好的字符串写进剪贴板：
  ```
  Q1: A  (卡片布局)
  Q2: B  (深色主题)
  Q3: A  (顶部导航)
  ```
  你粘贴回终端作为回答即可。

只有一个 HTML 文件，用 `file://` 打开，不需要服务器、不需要构建。Tailwind 和 Mermaid 通过 CDN 加载。

## 触发条件（以及不触发的情况）

**同时**满足以下两条才会触发：

1. **上下文条件** —— Claude Code 处于 Plan 模式，或者你明确要求进入澄清 / 采访流程。
2. **内容条件** —— 至少有一个待问的问题带有明显的**视觉 / 空间 / 流程 / 层级**维度。会触发的例子：
   - 布局类：「顶部导航 vs 侧边导航」「2×2 网格 vs 单排横向」
   - 视觉风格类：「浅色 vs 深色」「圆角 vs 直角按钮」
   - 流程 / 状态类：「同步处理 vs 异步队列」
   - 层级结构类：「组件树形状」「数据 schema 嵌套」

**不会触发**的场景：纯技术选型（Postgres vs MySQL、超时值、库版本等），或者你已经要求快速回答。过度触发会制造噪音，这是明确的反模式。

三个实际场景（Dashboard 页面、文件上传、缓存策略）见 [`assets/example-questions.md`](assets/example-questions.md)，演示了应触发和不应触发的判定过程。

## 在 Plan 模式里的位置

```
┌───────────────────────────┐
│  你：帮我做一个 Dashboard │
│      （Plan 模式）        │
└─────────────┬─────────────┘
              │
              ▼
      Claude 起草澄清问题
              │
      有视觉维度？── 否 ──▶ 仅在终端提问
              │
             是
              │
              ▼
   Skill 生成 plan-<时间戳>.html
              │
              ▼
   Claude 在终端输出路径 + 问题清单
              │
              ▼
   你打开 HTML → 点选 → 复制
              │
              ▼
   在终端粘贴「Q1: A  Q2: B …」
              │
              ▼
        Claude 继续开始实施
```

## 安装

把 skill 文件夹放进 Claude Code 的用户 skills 目录：

```bash
git clone https://github.com/Antony7e4/plan-mode-visualizer.git \
  ~/.claude/skills/plan-mode-visualizer
```

或者 clone 到任意位置后软链过去：

```bash
ln -s /path/to/plan-mode-visualizer ~/.claude/skills/plan-mode-visualizer
```

Claude Code 启动时会扫描 `~/.claude/skills/` 下的所有 skill。重启或开一个新会话后，它就会出现在可用 skill 列表里。

**前置条件**：装了 Claude Code，有浏览器可以打开生成的 HTML。HTML 首次打开需联网（从 CDN 拉 Tailwind / Mermaid / Inter 字体），之后走浏览器缓存。

## 目录结构

```
plan-mode-visualizer/
├── SKILL.md                     ← skill 定义，Claude Code 启动时加载
├── assets/
│   ├── template.html            ← Claude 每次会话基于它填充的 HTML 模板
│   └── example-questions.md     ← 三个判定示例（何时触发、何时不触发）
└── README.md / README.zh-CN.md
```

- **`SKILL.md`** —— 权威规范。列出触发规则、保真度规则（布局用低保真、风格用高保真、流程用 Mermaid、层级用树形）、HTML 必备结构、质量标准。
- **`assets/template.html`** —— 一份可运行的起点，含 header、三种示例 section（线框图 / 高保真带实时预览 / Mermaid）、常驻底栏答案汇总、复制到剪贴板、键盘快捷键 1/2/3。约 320 行，单文件自包含。
- **`assets/example-questions.md`** —— 让 Claude 做模式匹配的参考场景。

## 自定义

- 想改保真度规则（比如全部用高保真、不用 Mermaid）？编辑 `SKILL.md` 的 **"Fidelity rules"** 段。
- 想改剪贴板里的输出格式？改 `assets/template.html` 的 copy handler，并同步更新 `SKILL.md` 的 "Answer collector" 段描述。
- 想改保存路径（默认：项目根下的 `.claude-preview/`，否则 `/tmp/`）？编辑 `SKILL.md` 的 **"File location"** 段。

`SKILL.md` 本身是 Claude 会读的说明文档 —— 用自然语言改规则即可，不需要改代码。

## 为什么需要这个

纯文字的澄清问答有两个典型失败模式：
- **选错**：「2×2 网格」和「单排横向」在纸面上听着差不多，但在屏幕上体验差很远。
- **糊弄过去**：一口气读五个抽象选项，人会累，干脆随便选一个或者说「你看着办」。

把可比较的选项视觉化，把认知负担从"想象文字"变成"扫一眼图"，大致就是这么个事。

## 许可协议

MIT —— 见 [LICENSE](LICENSE)。

## 致谢

基于 [Claude Code](https://claude.com/claude-code) 使用。`SKILL.md` 遵循 Claude Code skill 格式规范。
