# General UI Guidelines

Use these guidelines when designing or changing interfaces in any project. Treat them as defaults, then follow the product's existing design system and the user's specific direction.

## Design principles

- Make the primary content and next action obvious. Prefer simple, readable layouts over decorative complexity.
- Inspect the surrounding interface first. Reuse its typography, colors, spacing, components, and interaction patterns instead of introducing a competing style.
- Keep related information together and group unrelated information clearly. Use consistent alignment, spacing, and hierarchy.
- Make layouts responsive and resilient to long text, missing images, empty data, and loading or error states.
- Do not add unrequested features, controls, settings, or visual effects.

## Cards and lists

- Use cards when they help distinguish meaningful items or actions. Avoid nesting containers or adding a card around content that already reads clearly.
- For repeated items, use a consistent vertical list by default. Use a grid when the content benefits from side-by-side comparison or browsing and remains usable at narrow widths.
- Keep each item's title, supporting details, status, and relevant image together. Keep repeated rows visually distinct without making every row overly prominent.
- Use concise empty states and omit irrelevant zero-value details when they do not help the user.
- Make card contents adapt to different amounts of text and optional metadata without misalignment or clipping.

## Shape, color, and decoration

- Use restrained borders, surfaces, and shadows to establish hierarchy. Avoid stacking gradients, glows, blur, and heavy borders without a clear purpose.
- Prefer square or modestly rounded corners for panels, cards, and controls unless the product's visual language or user preference calls for another shape.
- Reserve strongly rounded or circular shapes for elements that benefit from them, such as avatars, media thumbnails, or intentionally pill-shaped status indicators.
- Use color consistently to communicate state and hierarchy. Maintain readable contrast for primary, secondary, disabled, and interactive text.
- Keep badges and pills for compact status labels; use plain text when a badge adds no useful meaning.
- On dark surfaces, check scrollable controls for browser-default white scrollbar corners or textarea resize grips. Style scrollbar tracks, thumbs, corners, and resize handles to match the surface; disable textarea resizing with `resize: none` when resizing is unnecessary. Check the bottom-right corner as well as the scrollbar itself.

## Interaction and accessibility

- Make buttons and links visibly interactive, with clear hover, focus-visible, active, and disabled states.
- Give icon-only controls a consistent, sufficiently large hit area. Center the icon so hover and focus backgrounds fill that area evenly.
- Do not rely on color or hover alone to explain a state or action. Provide labels or accessible names for icon-only controls.
- Use semantic elements, keyboard-accessible interactions, and useful alternative text. Mark decorative images appropriately.
- Make loading, success, empty, and error states clear. Keep persistent messages until dismissed when the user expects to refer back to them.
- Ensure dialogs, menus, notifications, and scrollable regions remain usable on small screens and do not obscure essential actions.

## Before implementing a UI change

1. Review nearby code and existing design patterns.
2. Identify the user's actual request and preserve the established page structure unless a change is needed.
3. Check the layout at narrow and wide sizes, with long content, missing media, and relevant interaction states.
4. Keep changes scoped, and report what changed and what was checked.
