# Rust GUI Comparison: Iced vs. Slint vs. Egui

Based on the discussion in [r/rust](https://www.reddit.com/r/rust/) (*"How do popular Rust UI libraries compare? Iced vs Slint vs Egui"*), each framework excels in different scenarios depending on architecture, target platforms, licensing, and UI complexity.

---

## Quick Comparison

| Library | Paradigm / Style | Strengths | Trade-offs & Caveats |
| :--- | :--- | :--- | :--- |
| **[Egui](https://github.com/emilk/egui)** | Immediate mode (supports reactive redraws via [eframe](http://www.egui.rs/)) | Fast prototyping, pure Rust, compiles to WASM, rich ecosystem | Immediate-mode mental model; less flexible visual customization |
| **[Slint](https://github.com/slint-ui/slint)** | Retained mode with declarative `.slint` markup (QML-like) | Polished feel, closer to native/embedded workflows, strong docs | Dual-licensing (check commercial terms); occasional platform quirks |
| **[Iced](https://github.com/iced-rs/iced)** | Retained mode inspired by The Elm Architecture | Structured state flow, MIT license, active [COSMIC](https://github.com/pop-os/iced) / System76 backing | API evolution & breaking changes; docs/examples can lag behind |

---

## Detailed Breakdown

### 1. Egui
* **Best for:** Rapid prototypes, developer tools, utilities, internal apps, and browser/WASM targets.
* **Key takeaways from the thread:**
  * Easy to get up and running quickly with minimal boilerplate.
  * Runs smoothly in browsers via WebAssembly and has a large community with many supporting crates.
  * Can run reactively rather than redrawing unnecessarily every frame, though animations/continuous updates still drive redraws.
  * Custom styling and heavy visual polish can feel more constrained than retained-mode frameworks.

### 2. Slint
* **Best for:** Polished UIs, embedded devices, and projects that like declarative UI markup.
* **Key takeaways from the thread:**
  * Uses a declarative markup language similar to QML that compiles into Rust.
  * Often praised for feeling productive during implementation and fitting embedded/native-leaning targets well.
  * **Licensing matters:** Review its licensing terms carefully before using it in proprietary or commercial projects.
  * Watch out for platform-specific edge cases (such as Linux vs. Windows behavior) and data conversions between Rust and Slint types.

### 3. Iced
* **Best for:** Structured, medium-to-large pure-Rust applications and projects preferring the MIT license.
* **Key takeaways from the thread:**
  * Uses an Elm-like architecture (state, messages, update, view), which many developers find clean and predictable as apps grow.
  * Benefits from System76's work on the [COSMIC](https://github.com/pop-os/iced) desktop and related crates (such as text rendering and windowing improvements).
  * APIs and examples have evolved across versions, so code written for one release may need updates later.

---

## Other Mentioned Alternatives
* **[Dioxus](https://www.reddit.com/r/rust/comments/1iavpit/comment/m9ecpio/):** Familiar if you like React / React Native-style component patterns and multi-platform reach.
* **Tauri:** Uses web technologies for the UI with a Rust backend; great web UI flexibility, though packaging/distribution details should be evaluated for your target users.
* **Vizia:** Retained-mode library with Elm-inspired state handling, reactive bindings, and CSS/layout styling concepts.
* **GPUI:** High-responsiveness UI engine behind [Zed](https://zed.dev/), though still evolving in terms of standalone widget ecosystem and first-party target parity.

---

## Which One Should You Pick?
* Choose **Egui** if you want speed to first working app, simple tooling, or WASM support.
* Choose **Slint** if you want declarative UI definitions and embedded/desktop polish—and your licensing requirements fit.
* Choose **Iced** if you want a pure-Rust, MIT-licensed codebase with Elm-style state structure.