# Changelog

## [0.10.0] - 2026-05-28

### New Template: `templates/d3-graph.html`
- Nord preset (dark-first, Geist + Geist Mono); 16-node module dependency demo with 4 node groups
- D3 force simulation: charge, link distance, center, collision
- Drag nodes, wheel/pinch zoom, drag-to-pan via d3.zoom(); directed arrowheads via SVG marker
- Click node to highlight direct neighbors / dim the rest; second click resets
- Group color legend overlay; node+link count badge; Reset button
- ResizeObserver for responsive container; clean Slop Test lint pass

### Extended: `references/interactive-patterns.md` (Pattern 10)
- Runtime theme toggle: `<html data-theme>` attribute, CSS-only visual switch
- `localStorage` persistence; `prefers-color-scheme` fallback on first visit
- Sun/moon icon swap via CSS `content:` on `::before` — no JS for the icon
- Mermaid caveat: diagrams don't re-render on toggle (static SVG)

### Updated: `SKILL.md` (v0.10.0)
- Version bumped to `0.10.0`
- d3-graph.html added to template routing bullets and Dependency/Network Graphs section
- interactive-patterns.md bullet extended with "runtime theme toggle"

---

## [0.9.0] - 2026-05-26

### New Reference: `references/walkthrough-patterns.md`
- `initWalkthrough(sections, opts)` — step-by-step guided reveal mode
- Fixed `.walkthrough-bar` bottom control bar (Prev/Next/Close, step counter)
- `.walkthrough-mode` body class dims non-active sections (opacity + blur)
- Keyboard nav (ArrowLeft/Right, Escape); smooth scroll to active section
- `data-walkthrough-title` attribute for step labels (fallback: first h2/h3 text)
- Integration note: call startNarration() from v0.8.0 inside showStep() for auto-narrated walkthroughs
- Respects `prefers-reduced-motion` (disables blur/opacity transitions)

### New: `scripts/test.js` + `.github/workflows/lint.yml`
- `test.js`: zero-dependency Node runner that lints all HTML templates via `lint.js --json`; exits 0 on all-pass, 1 on any violation
- `lint.yml`: GitHub Actions workflow runs `test.js` on every push and PR to main; catches template slop regressions automatically
- No npm install required — pure Node, no dependencies

### Updated: `SKILL.md` (v0.9.0)
- Version bumped to `0.9.0`
- interactive-patterns.md bullet extended with "guided walkthrough mode"
- Quality Checks: CI regression guard bullet added

---

## [0.8.0] - 2026-05-22

### New: `scripts/lint.js`
- Node.js CLI that audits generated HTML against all 7 Slop Test checks from SKILL.md
- Zero dependencies — runs anywhere with `node scripts/lint.js <file>`
- `--json` flag for structured CI output; exits 0 on clean, 1 on violation, 2 on usage error
- Color checks scoped to CSS zones only (`<style>` blocks + inline `style=` attributes) so documentation text mentioning forbidden hex values doesn't produce false positives
- Checks: forbidden `--font-body` fonts, forbidden indigo/violet/fuchsia hexes, cyan+magenta+pink combo, gradient text on headings, animated glowing box-shadows, emoji section headers, three-dot window chrome
- SKILL.md Quality Checks section now references `node {{skill_dir}}/scripts/lint.js <file>` as a required pre-delivery step

### New: `references/theme-presets.md`
- Six complete drop-in theme presets, each with Google Fonts `<link>`, full `:root` block, `prefers-color-scheme` dark-mode override, and matched Mermaid `themeVariables`
- Identical variable surface across all presets (`--font-display`, `--font-body`, `--font-mono`, `--bg` through `--danger-dim`) — swapping presets is a single block replacement
- Presets: `paper-ink` (Instrument Serif + IBM Plex Sans + JetBrains Mono, cream/terracotta/sage), `blueprint` (IBM Plex Sans/Mono, teal/slate dark header, grid background), `editorial` (Fraunces + Source Sans 3 + Source Code Pro, deep navy/gold), `terminal` (JetBrains Mono throughout, green/amber on near-black, dark-first), `nord` (Geist + Geist Mono, authentic Nord palette), `dracula` (DM Sans + Fira Code, authentic Dracula palette, dark-first)
- SKILL.md Style section now references `./references/theme-presets.md` as the canonical preset catalog

### Enhanced: `references/interactive-patterns.md`
Four new patterns appended after the existing five:
- **Export to PNG / SVG**: SVG export via Blob → `URL.createObjectURL` → `<a download>` wired inside `initDiagram(shell)`; PNG export via `html-to-image` CDN (lazy-loaded, `@2x` pixel ratio)
- **Voice narration**: `speechSynthesis.speak()`, play/pause/stop button cluster, `.is-narrating` highlight on active section, `data-narrate` attribute for explicit override text, silent graceful degradation on unsupported browsers
- **Lazy-load CDN libraries**: Promise-wrapped `<script>` injection with `Map`-based deduplication; `IntersectionObserver` trigger with `rootMargin: '200px'`; ESM note (use `import()` for Mermaid/D3, not `loadLibrary`)
- **In-page search overlay**: Cmd/Ctrl+K or `/` trigger (no Ctrl+F conflict); `TreeWalker` text-node traversal; `<mark class="search-match">` injection with `clearMarks()` DOM restore; arrow-key + Enter navigation between matches

### Enhanced: `references/libraries.md`
New **Chart.js Plugins** section with three additional chart types:
- **Heatmap** (`chartjs-chart-matrix`): `type: 'matrix'`, `{x, y, v}` data shape, dynamic `width`/`height` callbacks, `cellColor()` helper, dual-axis category labels — use for time-by-day grids, correlation matrices, severity heatmaps
- **Sankey** (`chartjs-plugin-sankey`): `{from, to, flow}` data shape, `colorMode: 'gradient'` for source-to-destination fade, nodes inferred automatically, limit <20 nodes/<30 links
- **Treemap** (`chartjs-chart-treemap`): `groups: ['group', 'label']` hierarchy, `item.g`/`item.l`/`item.v` accessors, `labels.overflow: 'hidden'` for small cells — use for codebase size breakdowns, market-share, time allocation

### Updated: `SKILL.md` (v0.8.0)
- Version bumped to `0.8.0`
- Style section: new **Named theme presets** paragraph pointing to `./references/theme-presets.md`
- Structure section: interactive-patterns.md bullet expanded to list all 9 patterns including the 4 new ones
- Rendering approach table: three new rows for Heatmap, Sankey diagram, and Treemap pointing to Chart.js Plugins in libraries.md
- Dashboard / Metrics Overview section: note added directing agents to the Chart.js Plugins section for heatmaps, Sankey, and treemaps
- Quality Checks: **Slop Test (automated)** bullet added pointing to `scripts/lint.js`

---

## [0.7.0] - 2026-05-14

### New Commands
- `/code-tour` — generates a visual guided tour of any codebase: project overview hero, directory tree, Mermaid module dependency graph (with D3.js fallback for dense graphs), entry-point flow diagrams, data model tables, config reference, test coverage map, and onboarding checklist
- `/api-docs` — generates polished API documentation pages: auto-detects REST / GraphQL / CLI / gRPC / OpenAPI, extracts endpoints and auth from actual code, renders collapsible endpoint groups with copy-to-clipboard curl examples, request/response tables, and a Mermaid auth flow sequence diagram
- `/data-viz` — converts CSV / JSON / TSV input into a self-contained dashboard page: auto-detects data shape and picks the right Chart.js chart type, embeds data inline, produces KPI summary cards, a sortable + searchable data table, and per-chart interpretive captions

### New Templates
- `templates/timeline.html` — Warm Signal aesthetic (Instrument Serif + terracotta/sage/amber); central vertical spine with CSS Grid alternating left/right card layout; three card states (`--past`, `--current` with pulsing dot animation, `--future` with dashed border); IntersectionObserver scroll-triggered entrance animations; mobile-responsive collapse; print-safe
- `templates/dashboard.html` — Blueprint aesthetic (IBM Plex Sans/Mono + teal/slate); dark header bar; 4 KPI cards with colored top-border accent stripe; Chart.js bar + line chart grid; inline SVG sparklines; method and status badges; mobile-responsive; print styles included

### New Reference: `interactive-patterns.md`
- Copy-to-clipboard button pattern (`.copy-btn`, `navigator.clipboard.writeText()`, `.copied` state with 2 s reset)
- Sortable table pattern (`th[data-sort]`, `initSortableTable(table)`, numeric vs string detection, `↕/↑/↓` arrow indicators)
- Table search / filter pattern (`.table-filter__input`, `data-filter-target`, live row count)
- Print styles (`@media print` hide list, expanded `<details>`, `thead { display: table-header-group }`, `break-inside: avoid`)
- Mermaid loading skeleton (shimmer animation while diagram renders)
- Mermaid error fallback (`.diagram-error` with source display; does not rethrow so sibling diagrams continue)

### Enhanced `references/libraries.md`
- **D3.js section**: decision guide (Chart.js vs Mermaid vs D3), CDN snippet, full force-directed network graph example with `d3.forceSimulation`, drag handlers, zoom/pan (`scaleExtent [0.3, 4]`), node group colors, and `isDark` branching
- **Prism.js section**: core + autoloader CDN pattern, conditional dark/light theme loading (`prism-tomorrow.css` vs `prism-solarizedlight.css`), `language-*` class usage, CSS override to remove Prism's hardcoded background

### Updated `SKILL.md` (v0.7.0)
- Commands table: added `/code-tour`, `/api-docs`, `/data-viz` rows
- Structure section: added `timeline.html` and `dashboard.html` template routing bullets; added `interactive-patterns.md` to reference list
- Rendering approach table: new row `| Dependency/network graph (many nodes) | D3.js |`
- New "Dependency / Network Graphs" section under Diagram Types
- Timeline and Dashboard sections updated to reference their canonical template files
- Quality Checks: added Mermaid error-handling bullet pointing to `interactive-patterns.md`
- **New Accessibility section**: semantic HTML elements, ARIA for custom widgets (toolbar role, `aria-sort` on sortable headers, `aria-live` on zoom label), keyboard navigation, color-independent status communication, alt text for diagrams, `prefers-reduced-motion` CSS rule

---

## [0.6.3] - 2026-03-09

### Documentation
- Added explicit warning against using bare `<pre class="mermaid">` tags — they render but produce tiny unusable diagrams without zoom/pan controls. Updated SKILL.md to point to the full `diagram-shell` pattern from `templates/mermaid-flowchart.html`.

## [0.6.2] - 2026-03-08

### Bug Fixes
- Fixed fullscreen diagram export using wrong background color — now uses the same dark/light mode that was used to render the Mermaid theme

## [0.6.1] - 2026-03-08

### Pi Install Script
- New `install-pi.sh` for one-command installation
- Automatically patches `{{skill_dir}}` to actual install path
- Usage: `curl -fsSL https://raw.githubusercontent.com/nicobailon/visual-explainer/main/install-pi.sh | bash`

## [0.6.0] - 2026-03-08

Based on PR #25 by [@peak-flow](https://github.com/peak-flow), with additional multi-diagram architecture and bug fixes.

### Multi-Diagram Support
- New vector-based zoom/pan engine replacing CSS `zoom` with direct SVG sizing
- Closure-based `initDiagram(shell)` pattern — per-diagram state in closures, shared drag listeners at module scope
- Unlimited diagrams per page with no ID collisions (each diagram gets a unique generated ID)
- New HTML structure: `.diagram-shell` > `.mermaid-wrap` > `.mermaid-viewport` > `.mermaid-canvas`
- Source Mermaid code lives in `<script type="text/plain" class="diagram-source">` to avoid parsing issues
- Adaptive viewport height based on diagram aspect ratio
- Smart fit algorithm with readability floor (prevents tiny unreadable diagrams)
- New zoom controls: 1:1 button, zoom percentage label
- Touch pinch-to-zoom support with proper pan transition
- Double-click to fit diagram

### Bug Fixes
- Fixed touch pinch→pan transition (reset start coords after pinch ends)
- Removed dead `fitZoom` variable from previous implementation
- Removed 12 lines of dead scrollbar CSS (no longer needed with new viewport approach)

### Documentation
- Updated `css-patterns.md` with new multi-diagram structure and JavaScript pattern
- Simplified Mermaid section to reference `mermaid-flowchart.html` as canonical source

## [0.5.1] - 2026-03-05

### Claude Code Marketplace Structure
- Restructured repo to follow Claude Code's official plugin marketplace spec
- Moved all skill files into `plugins/visual-explainer/` subdirectory
- Added `.claude-plugin/marketplace.json` catalog for marketplace discovery
- Plugin manifest now at `plugins/visual-explainer/.claude-plugin/plugin.json`
- Install via marketplace: `/plugin marketplace add nicobailon/visual-explainer` then `/plugin install visual-explainer@visual-explainer-marketplace`

### Pi Manual Install
- Replaced `pi install` one-liner with manual installation instructions
- Pi users now clone repo and copy skill + prompts to `~/.pi/agent/skills/` and `~/.pi/agent/prompts/`
- Removed stale `pi` field from `package.json` (was pointing to non-existent root paths)

### OpenAI Codex Install Fix
- Fixed prompts path: `~/.codex/prompts/` (was incorrectly `~/.agents/commands/`)
- Prompts are optional (deprecated feature) — skill works without them via `$visual-explainer`
- With prompts installed, invoke as `/prompts:diff-review`, `/prompts:plan-review`, etc.

### Breaking Changes
- Direct Claude Code plugin install (`/plugin install https://...`) no longer works — use marketplace flow instead
- `pi install https://github.com/nicobailon/visual-explainer` no longer works — use manual install

## [0.5.0] - 2026-03-04

### Class Diagram and C4 Architecture Support
- Added `classDiagram` guidance for OOP design and domain modeling
- Documented relationships: association, composition, aggregation, inheritance
- Added C4 architecture support using `graph TD` + `subgraph` (not native `C4Context` which ignores themes)
- Added `.dir-tree` CSS pattern for file structures with tree connectors
- Added quick-reference table for choosing Mermaid diagram types

### Claude Code Plugin Support
- Added `.claude-plugin/plugin.json` manifest for Claude Code plugin installation
- Renamed `prompts/` to `commands/` (compatible with both pi and Claude Code)
- Claude Code: `claude /plugin install https://github.com/nicobailon/visual-explainer`
- Note: Claude Code namespaces commands as `/visual-explainer:command-name`

### OpenAI Codex Support
- Added install instructions for OpenAI Codex to README
- Uses `~/.agents/skills` and `~/.agents/commands` paths

### Share Command
- New `/share` prompt and `scripts/share.sh` for instant sharing of visual explainer pages
- Uses vercel-deploy skill — no account or authentication required
- Zero-friction: just `bash scripts/share.sh mypage.html` → live URL in seconds
- Returns claimable deployment (can transfer to your Vercel account later)
- JSON output for programmatic use

### Bug Fixes
- Fixed pi skill loading: scoped `pi.skills` to `./SKILL.md` instead of `./` (was trying to load README.md and CHANGELOG.md as skills)
- Fixed Mermaid line breaks: use `<br/>` instead of `\n` in flowchart labels (renders as literal text otherwise)
- Fixed `mermaid-flowchart.html` to match documented pattern: moved flex centering from `.mermaid-wrap .mermaid` to `.mermaid-wrap`, added `min-height: 400px`
- Fixed `share.sh` to properly capture and display deployment errors (was silently exiting due to `set -e`)

## [0.4.5] - 2026-03-04

### Click-to-Expand Mermaid Diagrams
- Clicking anywhere on a Mermaid diagram (without dragging) opens it full-size in a new browser tab
- Added expand button (⛶) to zoom controls for discoverability
- New `openMermaidInNewTab()` and `openDiagramFullscreen()` functions in the Mermaid JavaScript pattern
- Click detection distinguishes quick clicks from drag-to-pan (5px movement threshold, 300ms time threshold)
- Full-size view preserves the page's background color for visual consistency
- Updated all templates (`mermaid-flowchart.html`, `slide-deck.html`) with new pattern
- Updated `css-patterns.md` and `slide-patterns.md` documentation

### Bug Fixes
- Removed unused `text` variable in `openMermaidInNewTab()` function
- Removed unused `e` parameter in mouseup handlers
- Fixed inconsistent zoom range limits (standardized to 0.5x–5x across all files)
- Removed dead CSS `.mermaid-wrap.is-zoomed` rule (class was never applied by JavaScript)
- Removed dead CSS `transition: transform` on `.mermaid` (zoom property is not animatable)
- Added missing `cursor: grab` to base `.mermaid-wrap` selector in templates
- Added missing flex centering (`display: flex; justify-content: center; align-items: center`) to `.mermaid-wrap` in `slide-deck.html`
- Updated `SKILL.md` to explicitly mention the click-to-expand feature and expand button so agents include it when generating pages
- Updated all prompt templates (`diff-review.md`, `plan-review.md`, `project-recap.md`, `generate-visual-plan.md`) to specify the expand button and click-to-expand functionality for Mermaid diagrams

## [0.4.3] - 2026-03-01

### Mermaid Zoom and Positioning Fixes
- **Fixed zoom clipping**: Replaced `transform: scale()` with CSS `zoom` property. Transform only changes visual appearance — content expanding upward/leftward goes into negative space which can't be scrolled to. Zoom changes actual layout size, so overflow scrolls normally in all directions.
- **Fixed vertical centering**: Changed `align-items: flex-start` to `align-items: center` so diagrams are centered both horizontally and vertically in their container.
- **Added initial zoom**: Complex diagrams can start at zoom > 1 (e.g., 1.4x) for better readability while keeping zoom controls functional.
- **Added min-height**: Containers now have `min-height: 400px` to prevent vertical flowcharts from compressing into unreadable thumbnails.
- Removed unnecessary `.mermaid-inner` wrapper — no longer needed with zoom-based approach.
- Updated JavaScript to use `INITIAL_ZOOM` constant for consistent reset behavior.
- Updated "Scaling Small Diagrams" section to use `zoom` instead of `transform: scale()` for consistency.

## [0.4.4] - 2026-03-02

### Hybrid Architecture Pattern
- New pattern for complex architectures (15+ elements): simple Mermaid overview (5-8 nodes) + CSS Grid cards for details
- Updated "Architecture / System Diagrams" section in SKILL.md with three-tier approach based on complexity
- Reduced max Mermaid node count from 15-20 to 10-12 in `libraries.md`
- Updated Mermaid scaling guidance to recommend hybrid pattern over scaling tricks for complex diagrams

## [0.4.2] - 2026-03-01

### Link Styling
- New "Link Styling" section in `css-patterns.md` — never rely on browser default link colors; use accent colors with sufficient contrast

## [0.4.1] - 2026-03-01

### Mermaid Layout Direction
- New "Layout Direction: TD vs LR" section in `libraries.md`
- Prefer `flowchart TD` (top-down) over `flowchart LR` (left-to-right) for complex diagrams
- LR spreads horizontally and makes labels unreadable with many nodes
- Rule: use TD for 5+ nodes or any branching; LR only for simple 3-4 node linear flows

### Documentation
- Simplified README: trimmed Usage section, consolidated Install, added Slide Deck Mode section
- Added `/generate-visual-plan` to command table

## [0.4.0] - 2026-02-28

### New Prompt Template
- `/generate-visual-plan` — generate visual implementation plans for features and extensions. Produces editorial/blueprint-style HTML pages with problem comparison panels, state machine diagrams, code snippets, edge case tables, and implementation notes. Designed for documenting feature specs before implementation.

### Prose Accent Patterns
Added patterns for use as accent elements within visual pages.

**`css-patterns.md`** — New "Prose Page Elements" section:
- Body text settings (font-size, line-height, max-width for comfortable reading)
- Lead paragraph patterns (larger size, drop cap variants)
- Pull quotes (border-left, centered with quotation mark)
- Section dividers (horizontal rule, ornamental)
- Article hero patterns (centered, editorial)
- Author byline pattern
- Prose-specific anti-patterns

**`libraries.md`** — New "Typography by Content Voice" section:
- Font recommendations by content type (literary, technical, bold, minimal)
- Special mention of Literata for screen reading

**`SKILL.md`** — New sections:
- "Prose Accent Elements" — when to use lead paragraphs, pull quotes, callouts
- "Documentation" — content-to-visual mapping (features→cards, steps→flows, APIs→tables)

### Overflow Fix: List Markers in Bordered Containers
- `css-patterns.md`: New section "List markers overlapping container borders" with three solutions
- Rule of thumb: use `list-style-position: inside` or `padding-left: 2em` for lists in bordered containers

### Mermaid Fixes
- Centering: narrow vertical flowcharts must be centered, not left-aligned
- Scaling: complex diagrams with 10+ nodes render too small — increase fontSize to 18-20px or use CSS scale transform
- Special characters: node labels starting with `/`, `\`, `(`, `{` must be quoted to avoid shape syntax conflicts
- New "Scaling Small Diagrams" section in css-patterns.md
- New "Node Label Special Characters" section in libraries.md

### Code Block Patterns
- `css-patterns.md`: New "Code Blocks" section with:
  - Basic pattern with `white-space: pre-wrap` (critical for preserving line breaks)
  - File header pattern for displaying code with filename
  - Implementation plan guidance: don't dump full files, show structure instead
- `SKILL.md`: New "Implementation Plans" section with structure guidance

## [0.3.0] - 2026-02-26

### Anti-Slop Guardrails
- Added explicit "Anti-Patterns (AI Slop)" section to SKILL.md with forbidden patterns
- Removed "Neon dashboard" and "Gradient mesh" from allowed aesthetics — they always produce generic output
- Categorized aesthetics as "Constrained" (safer) vs "Flexible" (use with caution)
- Explicit forbidden fonts: Inter, Roboto, Arial, Helvetica, system-ui as primary
- Explicit forbidden colors: indigo/violet range (`#8b5cf6`, `#7c3aed`, `#a78bfa`), cyan-magenta-pink combination
- Explicit forbidden effects: gradient text on headings, animated glowing box-shadows, emoji section headers
- Added "The Slop Test" — 7-point checklist to catch AI-generated patterns before delivery
- Strengthened typography guidance with 5 explicit good pairings to use
- Strengthened color guidance with 5 explicit good palettes to use
- Referenced `websocket-implementation-plan.html` as positive example of Blueprint aesthetic

### Template Fixes
- Replaced violet secondary colors in `mermaid-flowchart.html` with sky blue to match anti-slop guidelines
- Updated Mermaid themeVariables example in `libraries.md` to use teal/slate palette instead of violet

## [0.2.0] - 2026-02-25

### Slide Deck Mode
- New output format: magazine-quality slide deck presentations as self-contained HTML files
- 10 slide types: Title, Section Divider, Content, Split, Diagram, Dashboard, Table, Code, Quote, Full-Bleed
- SlideEngine JS: keyboard/touch/wheel navigation, progress bar, nav dots, slide counter, keyboard hints with auto-fade
- Cinematic transitions: fade + translateY + scale on slide entrance, staggered child reveals via IntersectionObserver
- 4 curated presets: Midnight Editorial, Warm Signal, Terminal Mono, Swiss Clean (each with full light/dark palette)
- Event delegation: Mermaid zoom, scrollable code/tables don't trigger slide navigation
- Responsive height breakpoints (700px, 600px, 500px) for projection and small viewports
- Typography scale 2–3× larger than scrollable pages (80–120px display, 28–48px headings, 16–24px body)
- Per-slide background variation, SVG decorative accents, compositional variety rules
- Proactive imagery: surf-cli integration for title/full-bleed backgrounds, inline sparklines, mini-charts
- New `/generate-slides` prompt template; existing prompts support `--slides` flag via SKILL.md workflow
- Unified `autoFit()` post-render function: auto-scales Mermaid SVGs, KPI values, and long blockquotes to fit their containers
- Fix Mermaid diagrams rendering tiny in slide containers (flex shrink-wrap + inline max-width)
- Fix KPI card overflow for text values (white-space + transform scale)
- Fix quote slides with long text (proportional font-size reduction)

### Files
- `references/slide-patterns.md` — slide engine CSS, all 10 type layouts, transitions, nav chrome, content density limits, presets
- `templates/slide-deck.html` — reference template demonstrating all 10 types in Midnight Editorial preset
- `prompts/generate-slides.md` — slash command for generating slide decks
- `SKILL.md` — new "Slide Deck Mode" section with slide routing, `--slides` flag detection, visual richness guidance

## [0.1.4] - 2026-02-24

- Removed Mermaid `handDrawn` mode — Rough.js hachure fills are hardcoded and render unreadable diagonal scribbles inside nodes with no user-facing override. All diagrams now use `look: 'classic'` with custom `themeVariables` for visual distinction.
- Added `package.json` for `pi install` support — installs the skill and all slash commands in one step instead of `git clone` + manual `cp`

## [0.1.3] - 2026-02-24

- Extended `classDef` color warning to also cover per-node `style` directives — both hardcode text color that breaks in the opposite color scheme
- Renamed `.node` card classes to `.ve-card` to fix CSS collision with Mermaid's internal `.node` class that broke diagram layout (PR #7)

## [0.1.2] - 2026-02-19

- Added sequence diagram syntax guidance to "Writing Valid Mermaid" — curly braces, brackets, and ampersands in message labels silently break rendering

## [0.1.1] - 2026-02-19

- Prompts no longer require the `pi-prompt-template-model` extension — each prompt now explicitly loads the skill itself
- Added "Writing Valid Mermaid" section to `libraries.md` (quoting special chars, simple IDs, max node count, arrow styles, pipe escaping)
- Fixed mobile scroll offset in `responsive-nav.md` — section headings now clear the sticky nav bar via `scroll-margin-top`
- Added video preview to README

## [0.1.0] - 2026-02-16

Initial release.

### Skill
- Core workflow: Think (pick aesthetic) → Structure (read template) → Style (apply design) → Deliver (write + open)
- 11 diagram types with rendering approach routing (Mermaid, CSS Grid, HTML tables, Chart.js)
- 9 aesthetic directions (monochrome terminal, editorial, blueprint, neon, paper/ink, sketch, IDE-inspired, data-dense, gradient mesh)
- Mermaid deep theming with `theme: 'base'` + `themeVariables`, hand-drawn mode, ELK layout
- Zoom controls (buttons, scroll-to-zoom, drag-to-pan) required on all Mermaid containers
- Proactive table rendering — agent generates HTML instead of ASCII for complex tables
- Optional AI-generated illustrations via surf-cli + Gemini Nano Banana Pro
- Both light and dark themes via CSS custom properties and `prefers-color-scheme`
- Quality checks: squint test, swap test, overflow protection, zoom controls verification

### References
- `css-patterns.md` — theme setup, depth tiers, node cards, grid layouts, data tables, status badges, KPI cards, before/after panels, connectors, animations (fadeUp, fadeScale, drawIn, countUp), collapsible sections, overflow protection, generated image containers
- `libraries.md` — Mermaid (CDN, ELK, deep theming, hand-drawn mode, CSS overrides, diagram examples), Chart.js, anime.js, Google Fonts with 13 font pairings
- `responsive-nav.md` — sticky sidebar TOC on desktop, horizontal scrollable bar on mobile, scroll spy

### Templates
- `architecture.html` — CSS Grid card layout, terracotta/sage palette, depth tiers, flow arrows, pipeline with parallel branches
- `mermaid-flowchart.html` — Mermaid flowchart with ELK + handDrawn mode, teal/cyan palette, zoom controls
- `data-table.html` — HTML table with KPI cards, status badges, collapsible details, rose/cranberry palette

### Prompt Templates
- `/generate-web-diagram` — generate a diagram for any topic
- `/diff-review` — visual diff review with architecture comparison, KPI dashboard, code review, decision log
- `/plan-review` — plan vs. codebase with current/planned architecture, risk assessment, understanding gaps
- `/project-recap` — project mental model snapshot for context-switching
- `/fact-check` — verify factual accuracy of review pages and plan docs against actual code
