# Web SVG Icon Editor Guide

Use this guide to add editable SVG icons with a live preview, base-color replacement, and hover-color overrides to a browser-based app.

## Feature overview

An SVG icon editor has four pieces:

1. A multiline input that accepts the complete SVG document.
2. A live preview rendered from the current source.
3. Base-color swatches that replace matching colors in the source.
4. Hover-color swatches that map each base color to a different color while the icon is hovered.

Keep the original SVG text as the saved value. Generate a separate render copy for the preview and dropdown. Do not serialize a limited or simplified render copy back into the saved field; that can discard paths, styles, gradients, and definitions.

## Editor markup

Use a textarea, preview surface, and two color rows. The rows can be built dynamically from colors found in the SVG.

```html
<textarea id="svg-code" spellcheck="false"></textarea>
<div id="svg-preview" aria-label="SVG preview"></div>
<div>
  <h3>Base colors · click to replace</h3>
  <div id="svg-base-colors"></div>
</div>
<div>
  <h3>Hover colors · click to set overrides</h3>
  <div id="svg-hover-colors"></div>
</div>
```

For a modal editor, initialize the textarea and hover map from the selected record when opening. On Apply, copy both values back to the form. On Cancel, discard the modal's working values.

## Render a complete SVG in an image

Parse the input to check that it is a complete SVG document, then serialize the root into an SVG data URL for an `<img>`. This keeps the SVG's multiple paths, definitions, styles, gradients, and other supported SVG features intact without inserting the user's SVG elements into the page DOM.

```js
const SVG_NAMESPACE = 'http://www.w3.org/2000/svg';

function makeSvgImage(svgCode, size = 32, colorOverrides = {}) {
  let source = String(svgCode || '').trim();
  if (!source) return null;

  // Replace only complete hex color tokens, not prefixes of longer colors.
  for (const [baseColor, hoverColor] of Object.entries(colorOverrides)) {
    if (!/^#[0-9a-f]{3}(?:[0-9a-f]{3})?$/i.test(baseColor) ||
        !/^#[0-9a-f]{6}$/i.test(hoverColor)) continue;
    const escaped = baseColor.replace(/[.*+?^${}()|[\]\\]/g, '\\$&');
    source = source.replace(new RegExp(escaped + '(?![0-9a-f])', 'gi'), hoverColor);
  }

  const parsed = new DOMParser().parseFromString(source, 'image/svg+xml');
  const root = parsed.documentElement;
  if (!root || root.localName.toLowerCase() !== 'svg' ||
      parsed.querySelector('parsererror') ||
      !root.querySelector('path,circle,rect,line,polyline,polygon,ellipse,use,text')) {
    return null;
  }

  if (!root.getAttribute('xmlns')) root.setAttribute('xmlns', SVG_NAMESPACE);

  // Set an independent currentColor fallback. An explicit artwork color wins;
  // currentColor without a concrete root color uses the app's theme color.
  const rootStyle = root.getAttribute('style') || '';
  const styleColor = (rootStyle.match(/(?:^|;)\s*color\s*:\s*([^;]+)/i) || [])[1]?.trim() || '';
  const rootColor = styleColor || root.getAttribute('color') || '';
  if (!rootColor || rootColor.toLowerCase() === 'currentcolor') {
    const themeColor = getComputedStyle(document.documentElement)
      .getPropertyValue('--text-primary').trim() || '#d1d5db';
    if (styleColor) root.setAttribute('style', rootStyle.replace(
      /((?:^|;)\s*color\s*:\s*)currentcolor/i, `$1${themeColor}`
    ));
    else root.setAttribute('color', themeColor);
  }

  const image = document.createElement('img');
  image.alt = '';
  image.draggable = false;
  image.src = `data:image/svg+xml;charset=utf-8,${encodeURIComponent(
    new XMLSerializer().serializeToString(root)
  )}`;
  image.style.cssText = `display:block;width:${size}px;height:${size}px;object-fit:contain;`;
  return image;
}
```

Use the image returned by `makeSvgImage` for the preview and saved icon display. Replace the preview's children with the new image when the user edits code or colors:

```js
function renderPreview(svgCode, hoverMap = {}, showHover = false) {
  const preview = document.getElementById('svg-preview');
  preview.replaceChildren();
  const icon = makeSvgImage(svgCode, 48, showHover ? hoverMap : {});
  if (icon) preview.appendChild(icon);
  else preview.textContent = 'Preview unavailable for this SVG';
}
```

## Extract and replace base colors

This pattern finds unique 3-digit and 6-digit hex colors. Expand 3-digit values before assigning them to an HTML color input.

```js
function findSvgColors(svgCode) {
  return [...new Set(String(svgCode || '').match(/#[0-9a-f]{6}\b|#[0-9a-f]{3}\b/gi) || [])];
}

function expandHex(color) {
  const value = color.toLowerCase();
  return /^#[0-9a-f]{3}$/.test(value)
    ? '#' + [...value.slice(1)].map(char => char + char).join('')
    : value;
}

function replaceSvgColor(svgCode, oldColor, newColor) {
  const escaped = oldColor.replace(/[.*+?^${}()|[\]\\]/g, '\\$&');
  return svgCode.replace(new RegExp(escaped + '(?![0-9a-f])', 'gi'), newColor);
}
```

For every color returned by `findSvgColors`, add a color input to the Base colors row. When its `change` event fires, replace that color in the textarea and refresh the preview and both color rows. Using `change` avoids rebuilding the row while the native color picker is still open.

```js
const codeField = document.getElementById('svg-code');
const baseRow = document.getElementById('svg-base-colors');
const hoverRow = document.getElementById('svg-hover-colors');
const hoverMap = {};
const preview = document.getElementById('svg-preview');

function makeColorInput(container, color, title, onChange) {
  const input = document.createElement('input');
  input.type = 'color';
  input.value = expandHex(color);
  input.title = title;
  input.setAttribute('aria-label', title);
  input.addEventListener('change', () => onChange(input.value.toUpperCase()));
  container.appendChild(input);
}

function refreshEditor() {
  const colors = findSvgColors(codeField.value);
  baseRow.replaceChildren();
  hoverRow.replaceChildren();

  for (const color of colors) {
    makeColorInput(baseRow, color, `Replace ${color}`, replacement => {
      codeField.value = replaceSvgColor(codeField.value, color, replacement);
      const oldHoverKey = Object.keys(hoverMap).find(key => key.toLowerCase() === color.toLowerCase());
      if (oldHoverKey) {
        hoverMap[replacement.toUpperCase()] = hoverMap[oldHoverKey];
        delete hoverMap[oldHoverKey];
      }
      refreshEditor();
    });
    const hoverKey = Object.keys(hoverMap).find(key => key.toLowerCase() === color.toLowerCase());
    makeColorInput(hoverRow, (hoverKey && hoverMap[hoverKey]) || color, `Hover color for ${color}`, replacement => {
      hoverMap[color.toUpperCase()] = replacement;
      renderPreview(codeField.value, hoverMap, preview.matches(':hover'));
    });
  }

  renderPreview(codeField.value, hoverMap, preview.matches(':hover'));
}

codeField.addEventListener('input', refreshEditor);
preview.addEventListener('mouseenter', () => renderPreview(codeField.value, hoverMap, true));
preview.addEventListener('mouseleave', () => renderPreview(codeField.value, hoverMap, false));
refreshEditor();
```

## Add hover-color overrides

Keep hover colors in a map keyed by the exact base color found in the current SVG source:

```js
const hoverMap = {
  '#3B82F6': '#22C55E'
};
```

Add a second color input beside every base-color swatch. Its value is `hoverMap[baseColor]` when set, or the base color otherwise. When changed, store the selected hex value in the map and refresh the hover preview. When the pointer enters or leaves the preview, render with the hover map or the original colors:

```js
preview.addEventListener('mouseenter', () => renderPreview(codeField.value, hoverMap, true));
preview.addEventListener('mouseleave', () => renderPreview(codeField.value, hoverMap, false));
```

When replacing a base color, move any corresponding hover-map entry to the new base-color key.

## Save and validate

Save the raw SVG source and its hover map separately. Example record:

```json
{
  "iconSvg": "<svg viewBox=\"0 0 24 24\">…</svg>",
  "iconHoverMap": {
    "#3B82F6": "#22C55E"
  }
}
```

Validate the source length and validate the hover map on both client and server. A practical limit is 20,000 characters for the source and 64 color pairs; accept `#RGB` or `#RRGGBB` base keys and `#RRGGBB` replacement values. Re-run the SVG parser before display so malformed markup produces a clear validation message.

## Security and compatibility

- Do not place user-provided SVG source in `innerHTML` or append its parsed elements directly to the main page DOM.
- Rendering SVG through an `<img>` data URL keeps its document separate from the page DOM. For untrusted uploads, still apply a security policy appropriate to the app: reject scripts, `foreignObject`, event-handler attributes, and external resource references, and consider a vetted SVG sanitizer.
- External images, fonts, linked stylesheets, and relative URLs may not work inside a data URL. Embed required resources or use a controlled asset URL.
- SVGs using `currentColor` need a color defined inside the SVG because an `<img>` does not inherit CSS color from the page. Set a theme fallback on the SVG root, as in the example.
- Test with icons that contain multiple `<path>` elements, `<defs>`, gradients, class-based styles, and `currentColor`.
- If an old saved icon looks incomplete, clear that code and paste the complete source from the SVG source page again. A previously simplified save cannot restore removed markup.
