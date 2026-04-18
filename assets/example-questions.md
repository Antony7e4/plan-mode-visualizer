# Example Scenarios

Three scenarios showing when `plan-mode-visualizer` should trigger, what the question set looks like, and what the generated HTML should contain. Use these as mental models when deciding how to structure the preview.

---

## Scenario 1: Building a new dashboard page (SHOULD trigger)

**Context**: User is in Plan mode, asked Claude Code to build a "数据总览页" for an analytics dashboard product.

**Planned questions** (6 total):
1. 页面布局：顶部导航 vs 侧边栏导航 **[visual → wireframe]**
2. 主要指标卡片的排列：2x2 网格 vs 单排横向 vs 垂直列表 **[visual → wireframe]**
3. 配色风格：简洁白 vs 深色专业 vs 品牌蓝 **[visual → hi-fi with live preview]**
4. 数据刷新策略：实时推送 vs 每分钟轮询 vs 手动刷新 **[technical, no visualization]**
5. 图表库选择：Recharts vs Chart.js vs D3 **[technical, no visualization]**
6. 空状态的视觉处理：插图 vs 纯文字引导 **[visual → hi-fi side-by-side]**

**Skill decision**: Generate HTML covering Q1, Q2, Q3, Q6. Q4 and Q5 asked in terminal only.

**HTML structure**:
- Section 1 (Q1): Two wireframe boxes side-by-side — top nav vs side nav
- Section 2 (Q2): Three wireframe boxes showing the three grid arrangements
- Section 3 (Q3): Three color-swatch buttons on left, a large live preview panel on right showing a sample dashboard card that recolors on click
- Section 6 (Q6): Two high-fidelity empty-state mockups side-by-side

Footer collects Q1/Q2/Q3/Q6 only; Q4/Q5 are answered in terminal.

---

## Scenario 2: Adding a file upload & parsing feature (SHOULD trigger, partial)

**Context**: User said "采访我几个问题再开始". Feature is "上传文档后自动提取字段并填充表单".

**Planned questions** (5 total):
1. 解析流程：同步阻塞 vs 异步后台 vs 流式增量 **[flow → Mermaid]**
2. 用户看到的上传反馈：进度条 vs 骨架屏 vs 模态加载 **[visual → hi-fi side-by-side]**
3. 解析失败时的降级策略：完全人工填写 vs AI 再试一次 vs 部分字段自动填 **[flow → Mermaid]**
4. 支持的文件格式：PDF only vs PDF+DOCX vs 全格式 **[technical list, no visualization]**
5. 最大文件大小：2MB vs 5MB vs 10MB **[technical, no visualization]**

**Skill decision**: Generate HTML covering Q1, Q2, Q3. Q4 and Q5 in terminal only.

**HTML structure**:
- Section 1: Three Mermaid diagrams side-by-side showing the three processing flows
- Section 2: Three high-fidelity mockups of the loading states (use realistic sample content to make it feel real)
- Section 3: Three Mermaid state diagrams showing the failure fallback flows

---

## Scenario 3: Choosing a caching strategy (SHOULD NOT trigger)

**Context**: User in Plan mode, asked Claude Code to "优化首屏加载速度".

**Planned questions** (4 total):
1. 缓存层：Redis vs Memcached vs in-memory LRU
2. TTL 设置：1 分钟 vs 5 分钟 vs 1 小时
3. 缓存键策略：URL-based vs user-scoped vs hash-based
4. 失效策略：主动推送 vs 写时失效 vs 定时重建

**Skill decision**: Do NOT trigger. All questions are pure technical choices with no spatial, visual, or flow component that benefits from visualization. Ask them in plain text.

**Why this matters**: Over-triggering creates noise and makes the skill annoying. If the decision is "which library / what number / what algorithm", text is faster than opening a browser.

---

## Edge case: Dependent questions (PARTIAL trigger)

**Context**: Building a settings page.

**Planned questions**:
1. 整体布局：卡片式 vs 列表式 **[visual — needs to be answered first]**
2. 每个设置项的控件密度：紧凑 vs 舒适 **[visual, but depends on Q1 — cards and lists have very different spacing]**

**Skill decision**: Generate HTML for Q1 only. After user answers, generate a NEW HTML for Q2 that uses the selected layout from Q1 as the base.

This is the dependent-questions exception from the SKILL.md. The second HTML replaces (or supplements) the first.

---

## Rule of thumb

> If you can imagine the user saying "wait, let me see what those look like" after hearing the question — visualize it.
> If the user can decide from the option name alone ("Redis" vs "Memcached") — don't.
