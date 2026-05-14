# Interactive Patterns

Client-side interactivity patterns for copy-to-clipboard, sortable tables, search/filter, print support, and Mermaid error handling. No build step, no external JS libraries beyond what's already in use.

## Copy-to-Clipboard Buttons

Add to code blocks in API docs, code tours, and implementation plans. Uses the Clipboard API with a brief success-state transition.

```css
.code-file {
  position: relative;
}

.copy-btn {
  position: absolute;
  top: 8px;
  right: 8px;
  background: var(--surface-elevated, var(--surface));
  border: 1px solid var(--border);
  border-radius: 6px;
  padding: 4px 10px;
  font-family: var(--font-mono);
  font-size: 10px;
  font-weight: 500;
  color: var(--text-dim);
  cursor: pointer;
  z-index: 5;
  transition: background 0.15s, color 0.15s, border-color 0.15s;
}

.copy-btn:hover {
  background: var(--border);
  color: var(--text);
}

.copy-btn.copied {
  color: var(--green, #059669);
  border-color: var(--green, #059669);
}
```

```html
<div class="code-file">
  <div class="code-file__header">
    <span>curl example</span>
    <button class="copy-btn" data-copy>Copy</button>
  </div>
  <pre class="code-file__body"><code>curl -X POST https://api.example.com/v1/generate \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"prompt": "hello world"}'</code></pre>
</div>
```

```javascript
// Wire all copy buttons at page load — works for any number of blocks
document.querySelectorAll('[data-copy]').forEach(btn => {
  btn.addEventListener('click', async () => {
    const block = btn.closest('.code-file, .code-block-wrap');
    const text = block?.querySelector('code, pre')?.textContent ?? '';
    try {
      await navigator.clipboard.writeText(text.trim());
      btn.textContent = 'Copied!';
      btn.classList.add('copied');
    } catch {
      btn.textContent = 'Failed';
    }
    setTimeout(() => {
      btn.textContent = 'Copy';
      btn.classList.remove('copied');
    }, 2000);
  });
});
```

**Placement:** Position `copy-btn` inside the `.code-file` header (alongside the filename label) rather than floating over the code body — avoids obscuring the first line of code.

**Fallback:** If `navigator.clipboard` is unavailable (HTTP, old browser), the catch block shows "Failed" briefly. No silent errors.

---

## Sortable Tables

Click column headers to sort ascending/descending. Works on any `.data-table`. No library required.

```css
/* Mark sortable columns */
.data-table th[data-sort] {
  cursor: pointer;
  user-select: none;
  padding-right: 28px;  /* room for the arrow indicator */
  position: relative;
}

/* Default: unsorted indicator */
.data-table th[data-sort]::after {
  content: '\21D5';  /* ⇕ up-down arrow */
  position: absolute;
  right: 10px;
  color: var(--border-bright);
  font-size: 11px;
}

/* Active sort state */
.data-table th[data-sort="asc"]::after  { content: '\2191'; color: var(--accent); }  /* ↑ */
.data-table th[data-sort="desc"]::after { content: '\2193'; color: var(--accent); }  /* ↓ */
```

```html
<!-- Add data-sort and data-col to any th you want sortable -->
<thead>
  <tr>
    <th data-sort data-col="0">Name</th>
    <th data-sort data-col="1" class="num">Requests</th>
    <th data-sort data-col="2" class="num">Latency</th>
    <th>Status</th>  <!-- no data-sort = not sortable -->
  </tr>
</thead>
```

```javascript
function initSortableTable(table) {
  const headers = table.querySelectorAll('th[data-sort]');
  let lastCol = -1;
  let ascending = true;

  headers.forEach(th => {
    th.addEventListener('click', () => {
      const col = parseInt(th.dataset.col, 10);
      ascending = (col === lastCol) ? !ascending : true;
      lastCol = col;

      // Update header indicators
      headers.forEach(h => delete h.dataset.sort);
      th.dataset.sort = ascending ? 'asc' : 'desc';

      // Sort rows
      const tbody = table.querySelector('tbody');
      const rows = Array.from(tbody.querySelectorAll('tr'));
      rows.sort((a, b) => {
        const av = a.cells[col]?.textContent.trim() ?? '';
        const bv = b.cells[col]?.textContent.trim() ?? '';
        // Detect numeric values (strip commas, %, units)
        const an = parseFloat(av.replace(/[^0-9.-]/g, ''));
        const bn = parseFloat(bv.replace(/[^0-9.-]/g, ''));
        const cmp = (!isNaN(an) && !isNaN(bn))
          ? an - bn
          : av.localeCompare(bv, undefined, { sensitivity: 'base' });
        return ascending ? cmp : -cmp;
      });
      rows.forEach(r => tbody.appendChild(r));
    });
  });
}

document.querySelectorAll('.data-table').forEach(initSortableTable);
```

**Note:** The `data-sort` attribute on `<th>` serves double duty — it marks the column as sortable AND carries the current sort state (`asc`/`desc`/unset). Starting with no value means unsorted (shows ⇕). Clicking once sets it to `asc`, clicking again flips to `desc`.

---

## Table Search / Filter

A search input that hides rows not matching the query. Substring match across all visible cell text.

```css
.table-filter {
  display: flex;
  align-items: center;
  gap: 10px;
  margin-bottom: 10px;
}

.table-filter__input {
  padding: 7px 12px;
  max-width: 280px;
  width: 100%;
  background: var(--surface);
  border: 1px solid var(--border);
  border-radius: 6px;
  font-family: var(--font-mono);
  font-size: 12px;
  color: var(--text);
  outline: none;
  transition: border-color 0.15s;
}

.table-filter__input:focus {
  border-color: var(--accent);
}

.table-filter__count {
  font-family: var(--font-mono);
  font-size: 11px;
  color: var(--text-dim);
  white-space: nowrap;
}
```

```html
<div class="table-filter">
  <input
    class="table-filter__input"
    type="search"
    placeholder="Filter rows…"
    aria-label="Filter table rows"
    data-filter-target="endpoints-table"
  >
  <span class="table-filter__count" id="endpoints-count"></span>
</div>

<table id="endpoints-table" class="data-table">...</table>
```

```javascript
function initTableFilter(input) {
  const tableId = input.dataset.filterTarget;
  const table = document.getElementById(tableId);
  if (!table) return;

  const rows = Array.from(table.querySelectorAll('tbody tr'));
  const countEl = document.querySelector(
    `[aria-label="Filter table rows"] ~ .table-filter__count, #${tableId}-count`
  );

  function updateCount(visible) {
    if (!countEl) return;
    const q = input.value.trim();
    countEl.textContent = q ? `${visible} of ${rows.length}` : `${rows.length} rows`;
  }

  input.addEventListener('input', () => {
    const q = input.value.toLowerCase().trim();
    let visible = 0;
    rows.forEach(row => {
      const match = !q || row.textContent.toLowerCase().includes(q);
      row.style.display = match ? '' : 'none';
      if (match) visible++;
    });
    updateCount(visible);
  });

  updateCount(rows.length);
}

document.querySelectorAll('[data-filter-target]').forEach(initTableFilter);
```

**Combining sort + filter:** Both work independently on the same table. Sort reorders DOM rows; filter hides them. When the user clears the filter, hidden rows reappear in whatever sort order is currently active.

---

## Print Styles

Support Ctrl+P / browser print for saving diagrams as PDF. Include in any page that might be shared in document form.

```css
@media print {
  /* Hide interactive chrome */
  .zoom-controls,
  .copy-btn,
  .theme-toggle,
  nav,
  .nav-sidebar,
  .table-filter { display: none !important; }

  /* Expand all collapsible sections */
  details { display: block !important; }
  details > summary + * { display: block !important; }

  /* Page margins */
  @page { margin: 1.5cm; }

  /* Force white background for ink savings */
  body { background: white !important; color: black !important; font-size: 11pt; }

  /* Keep cards and table rows together */
  .ve-card, .timeline-entry, tr { break-inside: avoid; }

  /* Repeat table header on each page */
  thead { display: table-header-group; }

  /* Show link URLs inline */
  a[href]::after    { content: ' (' attr(href) ')'; font-size: 9pt; color: #555; }
  a[href^="#"]::after { content: none; }  /* skip internal anchors */
}
```

**Usage:** No JavaScript required. Users Ctrl+P → Save as PDF. The `@media print` block handles hiding controls and fixing backgrounds automatically.

**Mermaid diagrams in print:** The SVGs render as vectors and print cleanly. The zoom/pan controls are hidden by the rule above. If a diagram is very wide, it will scale to fit the page width via the browser's print engine.

---

## Mermaid Loading State and Error Fallback

Two problems to solve: (1) a brief blank flash while Mermaid initializes, (2) a broken diagram leaving a blank space with a console error the user never sees.

### Loading Skeleton

Show a shimmer placeholder while the async render runs. Hide it as soon as the SVG is ready.

```css
.diagram-skeleton {
  width: 100%;
  height: 300px;
  border-radius: 8px;
  background: linear-gradient(
    90deg,
    var(--border)          0%,
    var(--surface-elevated, var(--surface)) 40%,
    var(--border)          80%
  );
  background-size: 200% 100%;
  animation: shimmer 1.5s linear infinite;
}

@keyframes shimmer {
  from { background-position: 200% 0; }
  to   { background-position: -200% 0; }
}

@media (prefers-reduced-motion: reduce) {
  .diagram-skeleton { animation: none; }
}
```

```html
<section class="diagram-shell">
  <!-- Shown while rendering, removed on success -->
  <div class="diagram-skeleton"></div>

  <!-- Hidden until render succeeds -->
  <div class="mermaid-wrap" style="display:none;">
    <div class="zoom-controls"><!-- … --></div>
    <div class="mermaid-viewport">
      <div class="mermaid mermaid-canvas"></div>
    </div>
  </div>

  <script type="text/plain" class="diagram-source">
    graph TD
      A --> B
  </script>
</section>
```

In the `render()` function inside `initDiagram(shell)` (see `css-patterns.md` full pattern):

```javascript
async function render() {
  const skeleton = shell.querySelector('.diagram-skeleton');
  try {
    const { svg } = await mermaid.render(id, code);
    canvas.innerHTML = svg;
    skeleton?.remove();
    wrap.style.display = '';
    // … readSvgNaturalSize, fitDiagram, wire controls
  } catch (err) {
    showError(err);
  }
}
```

### Error Fallback

When Mermaid fails to parse, show the source code as a formatted code block. A syntax error in one diagram must not blank out the rest of the page.

```css
.diagram-error {
  border: 1px solid var(--red-dim, rgba(239,68,68,0.2));
  border-radius: 10px;
  overflow: hidden;
}

.diagram-error__header {
  padding: 9px 16px;
  background: var(--red-dim, rgba(239,68,68,0.08));
  border-bottom: 1px solid var(--red-dim, rgba(239,68,68,0.2));
  font-family: var(--font-mono);
  font-size: 11px;
  color: var(--red, #ef4444);
}

.diagram-error__source {
  font-family: var(--font-mono);
  font-size: 12px;
  line-height: 1.55;
  padding: 14px 16px;
  white-space: pre-wrap;
  color: var(--text-dim);
  max-height: 280px;
  overflow-y: auto;
}
```

```javascript
function showError(err) {
  const skeleton = shell.querySelector('.diagram-skeleton');
  skeleton?.remove();

  const fallback = document.createElement('div');
  fallback.className = 'diagram-error';
  fallback.innerHTML = `
    <div class="diagram-error__header">
      Diagram failed to render: ${(err.message ?? 'Syntax error').replace(/</g, '&lt;')}
    </div>
    <pre class="diagram-error__source">${code.replace(/</g, '&lt;')}</pre>
  `;
  // Insert before the (still-hidden) wrap so layout stays intact
  shell.insertBefore(fallback, wrap);
  // Don't rethrow — let the rest of the page render normally
}
```

**Why not throw?** If `initDiagram` lets the error propagate, an unhandled rejection in one diagram can silence the other `initDiagram` calls that haven't run yet. Catching and displaying locally keeps every other diagram functional.
