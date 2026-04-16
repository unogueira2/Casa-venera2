---
name: tailwind-v4-patterns
description: Conventions and changes for Tailwind CSS v4 (released 2025) — the CSS-first config, @theme directive, native CSS variables, and differences from v3. Use this skill whenever the user is working with Tailwind CSS, especially on projects started in 2025 or later, or when they mention `tailwind.config.js`, `@theme`, design tokens, or migration from v3. Also trigger for any frontend work on The PotFather or other Next.js projects since those should default to v4. Critical: v4 removed tailwind.config.js as the primary config surface — if Claude writes v3-style JS config for a v4 project, the build will silently break.
---

# Tailwind CSS v4 Patterns

Tailwind v4 (stable Jan 2025) is a significant architectural shift from v3. The headline change: **config is now CSS-first**, not JavaScript-first.

## Installation (v4)

```bash
npm install tailwindcss @tailwindcss/postcss
```

For Vite/Next.js plugin style:
```bash
npm install tailwindcss @tailwindcss/vite
# or
npm install tailwindcss @tailwindcss/postcss
```

`postcss.config.js`:
```js
export default {
  plugins: {
    "@tailwindcss/postcss": {},
  },
}
```

Import in your main CSS file (NOT `@tailwind base; @tailwind components; @tailwind utilities;` anymore):

```css
@import "tailwindcss";
```

That single import replaces the three old directives.

## CSS-First Configuration

v3 used `tailwind.config.js` with a JS object. v4 uses the `@theme` directive in CSS.

```css
@import "tailwindcss";

@theme {
  --color-pf-black: #0A0705;
  --color-pf-burgundy: #4A0E0E;
  --color-pf-gold: #C9A55C;
  --color-pf-cream: #EFE6D2;

  --font-display: "Cinzel", serif;
  --font-body: "Lora", Georgia, serif;
  --font-ui: "Inter", sans-serif;

  --spacing-128: 32rem;

  --breakpoint-3xl: 1920px;

  --radius-card: 0.75rem;
  --radius-button: 0.25rem;

  --animate-fade-up: fadeUp 0.6s ease-out;
}

@keyframes fadeUp {
  from { opacity: 0; transform: translateY(12px); }
  to { opacity: 1; transform: translateY(0); }
}
```

These automatically become utilities: `bg-pf-black`, `text-pf-gold`, `font-display`, `rounded-card`, `animate-fade-up`.

## Naming Conventions for Theme Variables

| Prefix | Generates | Example |
|--------|-----------|---------|
| `--color-*` | color utilities | `--color-brand` → `bg-brand`, `text-brand` |
| `--font-*` | font-family | `--font-display` → `font-display` |
| `--text-*` | font-size | `--text-hero` → `text-hero` |
| `--spacing-*` | spacing scale | `--spacing-128` → `p-128`, `m-128`, `gap-128` |
| `--radius-*` | border-radius | `--radius-card` → `rounded-card` |
| `--shadow-*` | box-shadow | `--shadow-lift` → `shadow-lift` |
| `--breakpoint-*` | responsive | `--breakpoint-3xl` → `3xl:` prefix |
| `--animate-*` | animation | `--animate-fade-up` → `animate-fade-up` |
| `--ease-*` | timing-function | `--ease-smooth` → `ease-smooth` |

## What's Different from v3

1. **No `tailwind.config.js`** (can still opt into legacy JS config via `@config`, but don't unless migrating).
2. **All theme values are real CSS variables** at runtime — you can `var(--color-pf-gold)` in your own CSS.
3. **Automatic content detection** — no more `content: ['./src/**/*.{ts,tsx}']`. Tailwind finds files via a heuristic. Disable with `@source none` or override with `@source "./path"`.
4. **Dark mode**: still works, but enable via `@variant dark (...)`:
   ```css
   @variant dark (&:where(.dark, .dark *));
   ```
5. **New utilities**: `@starting-style`, `field-sizing-content`, container queries built-in (no plugin).
6. **3D transforms** native: `rotate-x-12`, `rotate-y-45`, `perspective-1000`.
7. **`color-mix()` support** in opacity: `bg-pf-gold/20` uses color-mix, smoother blending.

## Custom Variants

```css
@custom-variant hocus (&:is(:hover, :focus-visible));

/* usage: hocus:bg-pf-gold-bright */
```

## @utility for Custom Utilities

```css
@utility tab-4 {
  tab-size: 4;
}

@utility scrollbar-hidden {
  scrollbar-width: none;
  &::-webkit-scrollbar {
    display: none;
  }
}
```

Now `tab-4` and `scrollbar-hidden` work as classes, including responsive prefixes (`md:scrollbar-hidden`).

## Migration from v3

If the project still has `tailwind.config.js`:

Option A — **Migrate to CSS-first** (recommended for new-ish projects):
1. Delete (or archive) `tailwind.config.js`
2. Move `theme.extend` values into `@theme { }` in CSS, renaming per table above
3. Update `postcss.config` or Vite plugin
4. Replace `@tailwind base/components/utilities` with `@import "tailwindcss"`

Option B — **Keep JS config** via the compatibility layer:
```css
@import "tailwindcss";
@config "../tailwind.config.js";
```
Works but loses some v4 benefits; use only for gradual migration.

## Common Gotchas

1. **Old `@tailwind` directives don't error** in v4, but they don't do anything either. Silent failure.
2. **`theme()` function still works** in CSS but prefer `var(--color-pf-gold)` directly.
3. **Arbitrary values unchanged**: `bg-[#4A0E0E]`, `p-[17px]` all still work.
4. **Plugins**: v4 has its own plugin API. Many v3 plugins (e.g., `@tailwindcss/typography`) now work but via imports:
   ```css
   @plugin "@tailwindcss/typography";
   ```
5. **Content sources** auto-detected from the directory of your CSS file upward. If utilities are missing, add `@source "./src"` explicitly.
6. **`@apply` still works**, but v4 prefers composing utilities in HTML over `@apply` in CSS.

## Performance Notes

- v4 is significantly faster (Rust-based engine, 5-10x faster builds)
- CSS output is smaller with `color-mix`-based opacity
- Lightning CSS replaces PostCSS in many toolchains

## Quick reference: v3 → v4 equivalents

| v3 | v4 |
|----|----|
| `tailwind.config.js` `theme.extend.colors.brand` | `@theme { --color-brand: #... }` |
| `tailwind.config.js` `content: [...]` | auto-detected, or `@source "..."` |
| `@tailwind base; @tailwind components; @tailwind utilities;` | `@import "tailwindcss";` |
| `require("@tailwindcss/typography")` in config | `@plugin "@tailwindcss/typography";` in CSS |
| `dark:` default | same, but configurable via `@variant dark` |
| `screens: { '2xl': '1536px' }` | `@theme { --breakpoint-2xl: 1536px; }` |

## Claude's default behavior for this skill

- Assume v4 for any Tailwind project started in 2025+ unless told otherwise.
- If an existing project uses v3, confirm version (`npm ls tailwindcss`) before writing config.
- Prefer CSS-first `@theme` over `@config` JS escape hatch.
- Use semantic design-token names (`--color-pf-burgundy`) not raw hex in utility classes.
- When writing components, compose utilities directly in JSX; avoid `@apply` except for true repeated primitives.
