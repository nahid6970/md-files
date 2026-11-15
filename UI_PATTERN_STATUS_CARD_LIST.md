# UI Pattern: Status Card List

A reusable dark-theme card list for showing items with categorised status badges, file lists, and action buttons. Used in the Rclone Projects modal.

---

## What it looks like

Each card shows:
- **Header row**: item name (bold) + last-checked time (muted, small) on the left; action buttons on the right
- **Sub-labels**: remote path (muted, truncated) + local path (grey or red if missing)
- **Status area** (after a check runs):
  - Coloured pill badges for each issue category (only non-zero ones shown)
  - A green "✓ N matching" label inline with the pills
  - A bullet list of affected file paths under the pills (up to 8, then "+N more")
  - "In sync" state: single green checkmark line
  - Checking/Queued state: pulsing blue dot + label
  - Error state: red text

Card border and background gradient shift based on state:
- **Default**: subtle border, flat card background
- **Has issues**: amber border, warm dark gradient + faint amber glow
- **Error**: red border, dark red gradient + faint red glow

---

## CSS variables expected

```css
:root {
  --bg-card: #161c26;
  --bg-card-hover: #1e2634;
  --border-color: #232c3d;
  --text-primary: #f1f5f9;
  --text-secondary: #64748b;
  --text-muted: #475569;
}
```

Also requires a `pulse` keyframe animation (add once in your global styles):

```css
@keyframes pulse {
  0%, 100% { opacity: 1; transform: scale(1); }
  50%       { opacity: 0.4; transform: scale(0.85); }
}
```

---

## Card HTML structure

Each card is a `<div>` with two children: a flex content block and a flex button group.

```html
<div style="
  display:flex; align-items:flex-start; gap:12px;
  padding:11px 12px; border-radius:9px;
  /* state-dependent border + background — see Card state styles below */
">

  <!-- LEFT: content -->
  <div style="min-width:0; flex:1;">

    <!-- Name + last-checked time -->
    <div style="display:flex; align-items:center; gap:6px; flex-wrap:wrap;">
      <span style="font-size:.83rem; font-weight:600; color:var(--text-primary);">Delta-Main</span>
      <span style="font-size:.6rem; color:var(--text-muted);">· 6:33 PM</span>
    </div>

    <!-- Remote / secondary label (truncated) -->
    <div style="font-size:.66rem; color:var(--text-muted); overflow:hidden; text-overflow:ellipsis; white-space:nowrap; margin-top:3px;">
      gu:/delta_main/
    </div>

    <!-- Local path (grey) or warning if missing (red) -->
    <div style="font-size:.64rem; color:#94a3b8; margin-top:2px;">
      C:/Users/nahid/delta
    </div>

    <!-- Status area — see states below -->
    <!-- INSERT STATUS HTML HERE -->

  </div>

  <!-- RIGHT: action buttons -->
  <div style="display:flex; gap:5px; flex-shrink:0;">
    <button style="padding:6px 9px; border:1px solid rgba(196,181,253,.45); border-radius:6px; background:rgba(196,181,253,.08); color:#c4b5fd; font-size:.68rem; font-weight:600; cursor:pointer;">Select</button>
    <button style="padding:6px 9px; border:1px solid rgba(96,165,250,.45);  border-radius:6px; background:rgba(96,165,250,.08);  color:#60a5fa; font-size:.68rem; font-weight:600; cursor:pointer;">Check</button>
    <button style="padding:6px 9px; border:1px solid rgba(34,197,94,.45);   border-radius:6px; background:rgba(34,197,94,.08);   color:#4ade80; font-size:.68rem; font-weight:600; cursor:pointer;">SRC → DST</button>
  </div>

</div>
```

---

## Card state styles

Replace the comment in the outer `<div>` style with one of these:

### Default (no check run yet, or clean)
```
border:1px solid var(--border-color);
background:var(--bg-card);
```

### Has differences / issues
```
border:1px solid rgba(251,191,36,.5);
background:linear-gradient(135deg, rgba(120,83,12,.2), var(--bg-card));
box-shadow:0 0 14px rgba(251,191,36,.06);
```

### Error
```
border:1px solid rgba(248,113,113,.55);
background:linear-gradient(135deg, rgba(127,29,29,.18), var(--bg-card));
box-shadow:0 0 14px rgba(248,113,113,.07);
```

---

## Status area HTML snippets

### Checking / Queued
```html
<div style="font-size:.7rem; color:var(--text-muted); margin-top:5px; display:flex; align-items:center; gap:5px;">
  <span style="width:8px; height:8px; border-radius:50%; background:#60a5fa; animation:pulse 1.2s infinite; display:inline-block;"></span>
  Checking…
</div>
```

### In sync (clean)
```html
<div style="font-size:.7rem; color:#34d399; margin-top:5px; display:flex; align-items:center; gap:4px;">
  <svg width="12" height="12" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5" stroke-linecap="round" stroke-linejoin="round">
    <polyline points="20 6 9 17 4 12"/>
  </svg>
  In sync · 405 matching files
</div>
```

### Error (check failed)
```html
<div style="font-size:.7rem; color:#f87171; margin-top:5px;">
  rclone was not found on the server PATH.
</div>
```

### Has differences — pills + file list
```html
<!-- Pill row -->
<div style="display:flex; flex-wrap:wrap; gap:4px; align-items:center; margin-top:6px;">

  <!-- Pill: amber — changed files -->
  <span style="display:inline-flex; align-items:center; gap:3px; padding:2px 7px; border-radius:10px;
               background:rgba(251,191,36,.1); border:1px solid rgba(251,191,36,.4);
               color:#fbbf24; font-size:.62rem; font-weight:700;">
    changed <span>2</span>
  </span>

  <!-- Pill: orange — missing from dest -->
  <span style="display:inline-flex; align-items:center; gap:3px; padding:2px 7px; border-radius:10px;
               background:rgba(251,146,60,.1); border:1px solid rgba(251,146,60,.4);
               color:#fb923c; font-size:.62rem; font-weight:700;">
    missing <span>3</span>
  </span>

  <!-- Pill: red — errors -->
  <span style="display:inline-flex; align-items:center; gap:3px; padding:2px 7px; border-radius:10px;
               background:rgba(248,113,113,.1); border:1px solid rgba(248,113,113,.4);
               color:#f87171; font-size:.62rem; font-weight:700;">
    errors <span>3</span>
  </span>

  <!-- Pill: purple — symlinks -->
  <span style="display:inline-flex; align-items:center; gap:3px; padding:2px 7px; border-radius:10px;
               background:rgba(167,139,250,.1); border:1px solid rgba(167,139,250,.4);
               color:#a78bfa; font-size:.62rem; font-weight:700;">
    symlinks <span>2</span>
  </span>

  <!-- Matching count — no pill, just green text -->
  <span style="font-size:.62rem; color:#34d399; padding:2px 0;">✓ 405 matching</span>

</div>

<!-- File list for one category (repeat per category with its dot colour) -->
<div style="margin-top:5px; display:flex; flex-direction:column; gap:1px;">

  <div style="display:flex; align-items:center; gap:5px; padding:2px 0;">
    <span style="color:#fbbf24; font-size:.6rem; flex-shrink:0;">●</span>
    <code style="font-size:.64rem; color:#cbd5e1; font-family:'Fira Code',monospace;
                 overflow:hidden; text-overflow:ellipsis; white-space:nowrap; flex:1;">
      tools/terminal_tui/templates/index.html
    </code>
  </div>

  <div style="display:flex; align-items:center; gap:5px; padding:2px 0;">
    <span style="color:#fbbf24; font-size:.6rem; flex-shrink:0;">●</span>
    <code style="font-size:.64rem; color:#cbd5e1; font-family:'Fira Code',monospace;
                 overflow:hidden; text-overflow:ellipsis; white-space:nowrap; flex:1;">
      job-search/all-job-listing-tracker.md
    </code>
  </div>

  <!-- "+N more" when list is capped -->
  <div style="font-size:.6rem; color:var(--text-muted); padding:2px 0 0 14px;">+3 more</div>

</div>
```

---

## Dot colours per category

| Category | Dot / pill colour | Hex |
|---|---|---|
| Changed (size differ) | Amber | `#fbbf24` |
| Missing (not in dest) | Orange | `#fb923c` |
| Errors | Red | `#f87171` |
| Symlinks | Purple | `#a78bfa` |
| In sync / matching | Green | `#34d399` |

---

## Modal shell (outer wrapper)

The list sits inside a modal overlay. Minimal structure:

```html
<!-- Overlay backdrop -->
<div id="my-modal" class="modal-overlay" role="dialog" aria-modal="true"
     onclick="if(event.target===this) closeModal()"
     style="z-index:1250;">

  <!-- Dialog box -->
  <div style="width:820px; max-width:96vw; max-height:82vh;
              display:flex; flex-direction:column;
              padding:0; overflow:hidden;
              background:var(--bg-card); border:1px solid var(--border-color);
              border-radius:12px;">

    <!-- Header -->
    <div style="display:flex; align-items:center; justify-content:space-between; gap:12px;
                padding:16px 20px; border-bottom:1px solid var(--border-color);">
      <div>
        <h2 style="margin:0; font-size:1rem;">My Projects</h2>
        <div style="font-size:.72rem; color:var(--text-muted); margin-top:4px;">
          Subtitle / description line.
        </div>
      </div>
      <div style="display:flex; align-items:center; gap:8px; flex-shrink:0;">
        <!-- Primary action button -->
        <button onclick="checkAll()"
                style="padding:6px 11px; border:1px solid rgba(96,165,250,.55); border-radius:6px;
                       background:rgba(96,165,250,.1); color:#60a5fa;
                       font-size:.72rem; font-weight:600; cursor:pointer;">
          Check all
        </button>
        <!-- Close -->
        <button onclick="closeModal()" aria-label="Close"
                style="background:none; border:0; color:var(--text-secondary);
                       font-size:1.4rem; cursor:pointer;">×</button>
      </div>
    </div>

    <!-- Scrollable card list -->
    <div id="my-modal-list"
         style="overflow:auto; padding:10px 14px;
                display:flex; flex-direction:column; gap:8px; min-height:120px;">
      <!-- Cards injected here by renderModal() -->
    </div>

    <!-- Status bar (shows during / after check-all) -->
    <div id="my-modal-message" role="status"
         style="display:none; padding:8px 18px;
                border-top:1px solid var(--border-color);
                font-size:.75rem; color:var(--text-secondary);">
    </div>

  </div>
</div>
```

Required CSS for `.modal-overlay`:

```css
.modal-overlay {
  display: none;
  position: fixed;
  inset: 0;
  background: rgba(0,0,0,.55);
  backdrop-filter: blur(3px);
  align-items: center;
  justify-content: center;
  z-index: 900;
}
.modal-overlay.show {
  display: flex;
}
```

---

## JavaScript render pattern

```js
const statusByName = new Map();

function renderModal() {
  const list = document.getElementById('my-modal-list');
  const items = getItems(); // your data array

  if (!items.length) {
    list.innerHTML = '<div style="padding:30px 12px; text-align:center; color:var(--text-muted); font-size:.82rem;">No items found.</div>';
    return;
  }

  list.innerHTML = items.map(item => {
    const status   = statusByName.get(item.name);
    const hasError = !!status?.error;
    const hasDiff  = !hasError && someConditionForDiff(status);

    const cardStyle = hasError
      ? 'border:1px solid rgba(248,113,113,.55);background:linear-gradient(135deg,rgba(127,29,29,.18),var(--bg-card));box-shadow:0 0 14px rgba(248,113,113,.07);'
      : hasDiff
        ? 'border:1px solid rgba(251,191,36,.5);background:linear-gradient(135deg,rgba(120,83,12,.2),var(--bg-card));box-shadow:0 0 14px rgba(251,191,36,.06);'
        : 'border:1px solid var(--border-color);background:var(--bg-card);';

    const pill = (label, count, color, bg) => count > 0
      ? `<span style="display:inline-flex;align-items:center;gap:3px;padding:2px 7px;border-radius:10px;background:${bg};border:1px solid ${color}66;color:${color};font-size:.62rem;font-weight:700;">${label} <span>${count}</span></span>`
      : '';

    const fileList = (files, color) => files.length
      ? `<div style="margin-top:5px;display:flex;flex-direction:column;gap:1px;">
           ${files.slice(0,8).map(f => `
             <div style="display:flex;align-items:center;gap:5px;padding:2px 0;">
               <span style="color:${color};font-size:.6rem;flex-shrink:0;">●</span>
               <code style="font-size:.64rem;color:#cbd5e1;font-family:'Fira Code',monospace;overflow:hidden;text-overflow:ellipsis;white-space:nowrap;flex:1;">${escapeHtml(f)}</code>
             </div>`).join('')}
           ${files.length > 8 ? `<div style="font-size:.6rem;color:var(--text-muted);padding:2px 0 0 14px;">+${files.length - 8} more</div>` : ''}
         </div>`
      : '';

    const checkedLabel = status?.checkedAt
      ? `<span style="font-size:.6rem;color:var(--text-muted);margin-left:4px;">· ${new Intl.DateTimeFormat(undefined,{timeStyle:'short'}).format(status.checkedAt)}</span>`
      : '';

    // Build statusHtml based on current state
    let statusHtml = '';
    if (status) {
      if (status.summary === 'Checking…' || status.summary === 'Queued…') {
        statusHtml = `<div style="font-size:.7rem;color:var(--text-muted);margin-top:5px;display:flex;align-items:center;gap:5px;">
          <span style="width:8px;height:8px;border-radius:50%;background:#60a5fa;animation:pulse 1.2s infinite;display:inline-block;"></span>
          ${escapeHtml(status.summary)}
        </div>`;
      } else if (hasError) {
        statusHtml = `<div style="font-size:.7rem;color:#f87171;margin-top:5px;">${escapeHtml(status.error)}</div>`;
      } else if (hasDiff) {
        statusHtml = `
          <div style="display:flex;flex-wrap:wrap;gap:4px;align-items:center;margin-top:6px;">
            ${pill('changed',  status.changedCount,  '#fbbf24', 'rgba(251,191,36,.1)')}
            ${pill('missing',  status.missingCount,  '#fb923c', 'rgba(251,146,60,.1)')}
            ${pill('errors',   status.errorCount,    '#f87171', 'rgba(248,113,113,.1)')}
            ${pill('symlinks', status.symlinkCount,  '#a78bfa', 'rgba(167,139,250,.1)')}
            ${status.matchingCount != null ? `<span style="font-size:.62rem;color:#34d399;padding:2px 0;">✓ ${status.matchingCount.toLocaleString()} matching</span>` : ''}
          </div>
          ${fileList(status.changedFiles  || [], '#fbbf24')}
          ${fileList(status.missingFiles  || [], '#fb923c')}
          ${fileList(status.symlinkFiles  || [], '#a78bfa')}`;
      } else if (status.matchingCount != null) {
        statusHtml = `<div style="font-size:.7rem;color:#34d399;margin-top:5px;display:flex;align-items:center;gap:4px;">
          <svg width="12" height="12" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5" stroke-linecap="round" stroke-linejoin="round"><polyline points="20 6 9 17 4 12"/></svg>
          In sync · ${status.matchingCount.toLocaleString()} matching
        </div>`;
      }
    }

    return `<div style="display:flex;align-items:flex-start;gap:12px;padding:11px 12px;${cardStyle}border-radius:9px;">
      <div style="min-width:0;flex:1;">
        <div style="display:flex;align-items:center;gap:6px;flex-wrap:wrap;">
          <span style="font-size:.83rem;font-weight:600;color:var(--text-primary);">${escapeHtml(item.name)}</span>
          ${checkedLabel}
        </div>
        <div style="font-size:.66rem;color:var(--text-muted);overflow:hidden;text-overflow:ellipsis;white-space:nowrap;margin-top:3px;">${escapeHtml(item.subtitle)}</div>
        <div style="font-size:.64rem;color:#94a3b8;margin-top:2px;">${escapeHtml(item.path)}</div>
        ${statusHtml}
      </div>
      <div style="display:flex;gap:5px;flex-shrink:0;">
        <button class="card-action-select" data-name="${escapeHtml(item.name)}"
                style="padding:6px 9px;border:1px solid rgba(196,181,253,.45);border-radius:6px;background:rgba(196,181,253,.08);color:#c4b5fd;font-size:.68rem;font-weight:600;cursor:pointer;">
          Select
        </button>
        <button class="card-action-check" data-name="${escapeHtml(item.name)}"
                style="padding:6px 9px;border:1px solid rgba(96,165,250,.45);border-radius:6px;background:rgba(96,165,250,.08);color:#60a5fa;font-size:.68rem;font-weight:600;cursor:pointer;">
          Check
        </button>
      </div>
    </div>`;
  }).join('');

  list.querySelectorAll('.card-action-check').forEach(btn =>
    btn.addEventListener('click', () => runCheck(btn.dataset.name)));
  list.querySelectorAll('.card-action-select').forEach(btn =>
    btn.addEventListener('click', () => selectItem(btn.dataset.name)));
}

function escapeHtml(str) {
  return String(str ?? '')
    .replace(/&/g,'&amp;').replace(/</g,'&lt;').replace(/>/g,'&gt;')
    .replace(/"/g,'&quot;').replace(/'/g,'&#039;');
}
```

---

## Action button colour palette

Adapt these for other actions by swapping the RGB values:

| Colour | Use case | Border / bg / text |
|---|---|---|
| Purple `#c4b5fd` | Select / navigate | `rgba(196,181,253,.45)` / `.08` |
| Blue `#60a5fa` | Check / refresh | `rgba(96,165,250,.45)` / `.08` |
| Green `#4ade80` | Sync / confirm | `rgba(34,197,94,.45)` / `.08` |
| Amber `#fbbf24` | Clone / restore | `rgba(251,191,36,.45)` / `.08` |
| Red `#f87171` | Delete / danger | `rgba(248,113,113,.45)` / `.08` |

Pattern: `border:1px solid rgba(R,G,B,.45); background:rgba(R,G,B,.08); color:#HEX;`

---

## Adapting for other use cases

The pattern works for any list of items that can be in one of these states: unchecked, running, clean, degraded, failed. Swap out the pill labels and file list categories to match your domain:

- **Git repos** → pills: `uncommitted`, `ahead`, `behind`; file list: changed files
- **Docker services** → pills: `stopped`, `unhealthy`; file list: container names
- **API endpoints** → pills: `degraded`, `down`; file list: failing routes
- **Build targets** → pills: `failed`, `warnings`; file list: error file paths
