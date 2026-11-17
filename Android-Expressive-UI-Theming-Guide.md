# Expressive UI and theming guide

A reusable visual-system guide derived from [samuelbeckman/expressive-clock](https://github.com/samuelbeckman/expressive-clock), reviewed 2026-11-15. The source project is an Android app built with Kotlin and Jetpack Compose. This document describes transferable design patterns and omits its product-specific features and screens.

## Visual character

- **Material 3 Expressive foundation:** familiar Material roles and components, with more motion, adaptable layouts, and stronger emphasis on a small number of key elements.
- **Calm canvas, expressive focus:** give one important element or action visual emphasis. Keep supporting rows and labels restrained.
- **Soft layered surfaces:** distinguish content using Material surface-container roles and tonal contrast rather than drawing borders around every element.
- **Rounded, tactile controls:** use pills, circular controls, rounded sheets, and floating navigation where appropriate. Keep shapes consistent, then give special actions slightly more visual weight.
- **Ambient polish:** translucent treatments, subtle colored glow, and custom visual details can add character. Use them as accents; preserve legibility and avoid making every surface luminous.

## Color system

Treat colors as semantic roles, not scattered hex literals. The implementation uses Material 3 `ColorScheme` roles: `primary`, `onPrimary`, `primaryContainer`, secondary and tertiary pairs, `background`/`surface`, `onSurface`, `surfaceVariant`, `outline`, `error`, and the surface-container ladder. It provides both light and dark schemes, offers Android wallpaper-derived dynamic color on Android 12+, and falls back to selectable fixed palettes.

The source’s **Concept** palette is teal-forward. Representative role colors:

| Role | Light | Dark |
|---|---|---|
| Primary | `#006874` | `#4FD8EB` |
| Primary container | `#97F0FF` | `#004F58` |
| Background / surface | `#F5FAFB` | `#0E1415` |
| Main foreground | `#171D1E` | `#E0E7E8` |

These are examples from the source palette, not universal defaults. For a new product, choose its own accent hue and generate coherent role pairs with accessible contrast. Keep containers tinted and nearby in hue; use `on*` roles for readable foregrounds.

Dark mode follows the system by default, with explicit light/dark options. An optional AMOLED mode sets the base background and lowest surface to pure black while retaining a small number of raised dark containers so content groups remain distinguishable. This is an optional treatment for OLED-heavy, low-light apps, not the default dark palette.

## Type

The source bundles **Google Sans Flex** for its Material expressive type scale. It uses a single variable font with optical size adjusted to the text size, plus regular/medium/bold weights. For another app, use a licensed, available variable sans font, then create a role-based scale rather than styling every label independently.

Approximate scale anchors from the source:

- Display: 57 / 45 / 36 sp
- Headline: 32 / 28 / 24 sp
- Title: 22 / 16 / 14 sp
- Body: 16 / 14 / 12 sp
- Labels: 14 / 12 / 11 sp

Pair large display text with compact supporting labels. Use weight, size, and color roles to establish hierarchy; reserve emphasized styles for the most important content. Keep small-label tracking subtle.

**Font licensing:** the repository documents Google Sans Flex as OFL-1.1 licensed and deliberately avoids Google Sans Text for third-party use. Check the license for any font you bundle; do not infer permission from a font being downloadable or served by a web endpoint.

## Layout and components

- **Navigation:** a centered, floating pill dock at the bottom, with a moving selected-state indicator and a separate circular action when relevant. The dock reserves a stable footprint as the action appears or disappears.
- **Primary action:** make the main action easy to reach and visually distinct, while tying it to the current screen’s task.
- **Rows and cards:** give each row clear title/supporting-text hierarchy and comfortable touch space. Group related settings into soft containers; avoid over-cardifying simple lists.
- **Sheets and pickers:** use rounded modal sheets for focused creation or editing flows, keeping choice and confirmation actions obvious.
- **Responsive behavior:** use a single-pane narrow layout and switch to a two-pane arrangement on expanded windows where the content benefits from it. Reflow based on available width, not device labels alone.
- **System bars:** draw edge-to-edge and account for status/navigation insets so the app feels integrated with the device while content remains unobscured.

## Motion and interaction

Use Material Expressive motion as a baseline: spring-based spatial movement with restrained effects. The source’s navigation selection uses a springing indicator; controls have small scale/position feedback; icons animate with short, meaningful gestures. Content can transition between destinations, while navigation remains stable and predictable.

Guidelines for adapting it:

1. Animate state changes that help explain what changed (selection, expansion, starting/stopping, confirmation).
2. Use spring motion for spatial movement and short fades for visibility changes.
3. Keep amplitudes small for routine controls; reserve overshoot or a distinctive flourish for a focal action.
4. Respect reduced-motion preferences, avoid animation that delays task completion, and make the same state understandable without motion.
5. Pair every color and motion cue with a clear label/state for accessibility.

## Adaptation recipe

1. Define the main task and one focal element per screen.
2. Choose an accent hue and define light/dark semantic role palettes, including readable `on*` colors and distinct surface levels.
3. Pick a licensed font and establish display, heading, title, body, and label roles.
4. Build a spacing and shape vocabulary (rounded controls, pill navigation, surface containers) and reuse it.
5. Choose a persistent navigation pattern appropriate to the product; add a detached primary action only when it is contextually useful.
6. Add a small set of motion patterns for selection, reveal, and completion; keep core interaction immediate.
7. Adapt layouts to window width, text scaling, and accessibility needs. Verify contrast, focus order, touch targets, and reduced motion.

## Source pointers

The repository’s implementation is in `app/src/main/java/app/expressiveclock/ui/theme/Theme.kt`, `Palettes.kt`, `Type.kt`, `ClockDock.kt`, screen files under `ui/screens/`, and responsive helpers under `ui/responsive/`. Its README identifies Kotlin, Jetpack Compose, Material 3 Expressive, Google Sans Flex, and expressive animations.

- [Repository README](https://github.com/samuelbeckman/expressive-clock#readme)
- [Theme](https://github.com/samuelbeckman/expressive-clock/blob/main/app/src/main/java/app/expressiveclock/ui/theme/Theme.kt)
- [Palettes](https://github.com/samuelbeckman/expressive-clock/blob/main/app/src/main/java/app/expressiveclock/ui/theme/Palettes.kt)
- [Typography](https://github.com/samuelbeckman/expressive-clock/blob/main/app/src/main/java/app/expressiveclock/ui/theme/Type.kt)
- [Navigation component](https://github.com/samuelbeckman/expressive-clock/blob/main/app/src/main/java/app/expressiveclock/ui/ClockDock.kt)
