# Gemini-Gruvbox-Dark Style Guide

This guide explains what the style does, why it does it, and how the Stylus `@var` font system works in practice, so both humans and models can reason about changes safely.

## 1. High-level Intent
- Target: `gemini.google.com`.
- Theme: Gruvbox-inspired dark UI, high contrast but soft, focused on content.
- Priorities:
  - Configurable typography via `@var` so users can pick fonts.
  - Strong, consistent color tokens instead of hard-coded colors.
  - Subtle micro-interactions for clarity, not decoration.
  - Aggressive font smoothing everywhere for crisp text.

---

## 2. How the font @vars actually work

All the “font magic” is driven by three pieces:
1) User-visible `@var` options.
2) Stylus conditionals that map choices → CSS variables.
3) Global selectors that read those variables and apply real `font-family` stacks.

### 2.1 User options
Defined at Gemini-Gruvbox-Dark.user.css:24-35.

```stylus
@var select newfont 'Text Font' [
    'Test Tiempos Text',
    'Website Default',
    'Geist Sans',
    'Test Tiempos Text Medium',
]

@var select headfont 'Heading Font' [
    'Inherit',
    'Bricolage Grotesque Variable',
]
```

What this means:
- `newfont` is literally a string equal to one of those labels.
- `headfont` is also a string.
- Stylus lets us compare those strings in `if` blocks.

### 2.2 Mapping choices into CSS variables
Key block at Gemini-Gruvbox-Dark.user.css:1464-1469.

```stylus
:root {
    --dropdown-text-font: newfont;
    --fallback-text: sans-serif !important;
}
```

Important details:
- `--dropdown-text-font` is set to the literal `newfont` value.
  - If user picks `Geist Sans` → `--dropdown-text-font: Geist Sans`.
  - If user picks `Test Tiempos Text` → `--dropdown-text-font: Test Tiempos Text`.
- `--fallback-text` is a backup stack if something goes wrong.

Then the conditional application (Gemini-Gruvbox-Dark.user.css:1473-1482):

```stylus
if newfont != 'Website Default' {
    p, li, ul, ol, button, input, textarea, strong, span, div {
        font-family: var(--dropdown-text-font) !important;
    }
} else {
    p, li, ul, ol, button, input, textarea, strong, span, * {
        font-family: system-ui, -apple-system, BlinkMacSystemFont,
                     'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell,
                     'Open Sans', 'Helvetica Neue', sans-serif !important;
    }
}
```

How this works in practice (concrete examples):

Example A: user selects `Geist Sans` as Text Font
- Stylus sets: `newfont = 'Geist Sans'`.
- `--dropdown-text-font: Geist Sans`.
- Condition `newfont != 'Website Default'` is true.
- Resulting CSS:
  ```css
  p, li, ul, ol, button, input, textarea, strong, span, div {
      font-family: Geist Sans !important;
  }
  ```

Example B: user selects `Website Default` as Text Font
- Stylus sets: `newfont = 'Website Default'`.
- Condition `newfont != 'Website Default'` is false.
- Resulting CSS:
  ```css
  p, li, ul, ol, button, input, textarea, strong, span, * {
      font-family: system-ui, -apple-system, BlinkMacSystemFont,
                   'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell,
                   'Open Sans', 'Helvetica Neue', sans-serif !important;
  }
  ```
- So we fall back to a sane, local stack.

Example C: user selects `Test Tiempos Text Medium`
- Stylus sets: `newfont = 'Test Tiempos Text Medium'`.
- `--dropdown-text-font: Test Tiempos Text Medium`.
- All key text selectors use that as primary family; browser then resolves to the imported font.

### 2.3 Heading font logic
Defined at Gemini-Gruvbox-Dark.user.css:1446-1458.

```stylus
:root {
    --default-heading-font: headfont;
}

if headfont != 'Inherit' {
    h1, h2, h3, h4, h5, h6 {
        font-family: var(--default-heading-font), system-ui, -apple-system,
                     sans-serif !important;
    }
} else {
    h1, h2, h3, h4, h5, h6 {
        font-family: newfont, sans-serif !important;
    }
}
```

Behavior:
- If user chooses `Bricolage Grotesque Variable` → headings use that.
- If user selects `Inherit` → headings follow whatever `newfont` is using.

### 2.4 `LoadWebFont` behavior

```stylus
if LoadWebFont {
    p, div, span, button, body, html, p, li, ul, ol {
        font-family: 'Tiempos Text Regular' !important;
    }
}
```

- This acts as an override: forces `Tiempos Text Regular` across base text when toggled.
- Rationale: one-click “use the shipped editorial font everywhere”.

This is the core: @vars → strings → CSS custom properties → consistent font-family decisions.

---

## 3. Color system (why variables exist)

All colors live in `:root` (Gemini-Gruvbox-Dark.user.css:355-476) so components never hard-code raw hexes.

Key roles:
- Surfaces: `--bg-def-dark`, `--bg-dark-new`, `--bg1-dark`, etc.
- Text:
  - Primary: `--text-normal`, `--text-offwhite`, `--text-offwhite-faded`.
  - Muted: `--text--faded`, `--text--veryfade`.
- Interaction / accents:
  - `--faded-*` for active, hover, and important UI.
  - `--dim-*` for borders, quiet accents.

Guidelines:
- New components MUST use these tokens, not ad-hoc hex values.
- Use `--faded-green` / `--dim-green` for confirm/primary accents.
- Use `--faded-blue` for link-like states and neutral actions.

---

## 4. Font smoothing (non-optional)

Any selector that influences text uses:

```css
-webkit-font-smoothing: antialiased !important;
-moz-osx-font-smoothing: grayscale !important;
text-rendering: optimizeLegibility !important;
```

Why:
- Many different webfonts + dark background → smoothing keeps edges clean.
- Ensures consistency across Chrome/Safari/Firefox/macOS/Windows.

When adding styles:
- If you set `font-family`, `font-size`, or text color, you must include these three lines.

---

## 5. Components (what they look like and why)

Below: each major piece, its selectors, and the design reasoning.

### 5.1 Layout / Base

- `html, body`:
  - Background: `#171718` (matches `--bg-def-dark`).
  - Rationale: neutral canvas for all other surfaces.

- Paragraphs & Lists:
  - `p`: `--text-offwhite-faded`.
  - `li, ol, ul`: `--text-offwhite`; markers in `--dim-green`.
  - Rationale: content-first, subtle emphasis on list bullets.

- Headings `h1`–`h6`:
  - Each bound to a distinct Gruvbox accent.
  - Rationale: semantic levels are visually distinct without extra layout.

### 5.2 Links

- `a`:
  - Color: `--faded-blue`, weight 600, JetBrains Mono Variable.
  - Hover: small upward shift + soft glow.
  - Rationale: links feel technical and precise; consistent with code-y aesthetic.

---

### 5.3 Chat history & conversations

- `.infinite-scroller.chat-history`:
  - Dark card background, thin border, soft hover shadow.
  - Rationale: separates history from canvas without heavy chrome.

- `.conversation` + `.mat-mdc-tooltip-trigger.conversation`:
  - Pill-like chip with transparent border.
  - Hover: `--bg-def-dark` fill, `--bg2-dark` border, `--faded-green` text, lift.
  - Rationale: indicates "clickable session" in a clean, touch-friendly way.

---

### 5.4 Chat bubbles & inline actions

- User bubble text: `p.query-text-line.ng-star-inserted`:
  - `--bg-dark-new` bubble, `--text-normal` text, rounded.
  - Hover: `--faded-blue` border, tiny lift, subtle shadow.

- Query action buttons: `.query-content .action-button`:
  - Circular, dark, `--bg3-dark` border.
  - Hover: `--faded-green` border + glow + scale.
  - Icon in `--faded-green` with gentle scale on hover.
  - Rationale: micro-interactions confirm Copy/Edit without stealing focus.

- Temp chat button: `button[data-test-id="temp-chat-button"]`:
  - Circular, dark base, aqua border/glow on hover; icon fills tween aqua → green.
  - Rationale: distinct from normal actions, but same visual language.

---

### 5.5 Sidebar, Gems, and toolbox

- Sidenav shell: `bard-sidenav-container > *`:
  - Dark card + border + radius.

- Sidebar actions: `.side-nav-action-button span.gds-body-m`:
  - `--dim-green`, small, medium weight → utility, not headline.

- Gems: `.bot-name.gds-body-m`, `h1.title.gds-label-l`:
  - Blue names and green section titles for quick scanning.

- Sidebar input: `input-area-v2 > div.input-area`:
  - Dark panel with thick border; no glow to avoid distraction.

- Toolbox: `.toolbox-drawer-menu-item` + children:
  - Dark pill-like entries, green icons, muted labels.
  - Hover tightens border; no jumping layout.

---

### 5.6 Primary pills (Write / Build / Deep Research / Learn)

- `button.mat-ripple.card.card-legacy`:
  - Base: dark pill, `--bg1-dark` border.
  - Hover: `--dim-green` border, horizontal padding expansion.
- Label `.card-label.gds-label-l`:
  - Base: `--dim-green`; hover: fades to `--text--faded` and bolds.

Rationale:
- These are the main entry points; they share one pill language so the page feels intentional.

---

### 5.7 Model selector

- Trigger `button.gds-mode-switch-button.logo-pill-btn`:
  - Dark pill, `--dim-green` text, hover border.
- Menu `.mat-mdc-menu-content`:
  - Dark panel with `--bg2-dark` border.
- Model labels:
  - Names `.gds-label-m`: offwhite.
  - Descriptions `.mode-desc.gds-label-m-alt`: `--faded-blue`.
  - Title `span.gds-label-l.title`: `--dim-green`.

Rationale:
- Make model choice feel like part of the theme, not a stock Material menu.

---

### 5.8 Canvas / Code editor

- `xap-code-editor` and Monaco internals:
  - Background: `--bg-def-dark` everywhere.
  - Font: Geist Mono for text, numbers, gutters.
  - Active line number: `--dim-green`.
- Code tokens under `code[data-test-id="code-content"] .hljs-*`:
  - Implement classic Gruvbox mappings (keywords orange, strings green, etc.).

Rationale:
- If you’re coding or reading code output, it should look like a real Gruvbox editor.

---

### 5.9 Citations & sources

- Inline chip: `.source-inline-chip-container > button`:
  - Compact pill, hover stretch + lift; icon animates slightly.
- Sidebar citations:
  - Structure uses `--text-normal`, `--faded-green`, `--faded-yellow`, etc. to separate title/URL/snippet.

Rationale:
- Make citations legible and trustworthy, in the same design language.

---

## 6. Hiding clutter (config-driven)

- `noLogo`, `noUpgrade`, `noGoogle`, `noAddress` are hooks for Stylus `if` blocks to hide decorative or marketing UI.
- `capabilities-disclaimer` is visually collapsed (height/opacity) instead of hard `display: none`.
- Upgrade upsell is removed.

Principle:
- Let users run a minimal, focused Gemini surface while keeping the theme stable.

---

## 7. When extending this theme

- Read from existing tokens: use `--bg-*`, `--text-*`, `--faded-*`, `--dim-*`.
- Respect the font system:
  - Use `var(--dropdown-text-font)` / `headfont` behavior rather than hard-coding.
- Always include font smoothing on new text selectors.
- Use subtle transforms and 0.2s–0.4s transitions; avoid long or janky animations.
- Place new rules next to related components (model selector styles with model selector, etc.).
