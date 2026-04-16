---
name: godfather-design-system
description: Design system, visual tokens, and component patterns for The PotFather — a hemp/THCA e-commerce brand with aesthetic inspired by The Godfather film (Coppola, 1972). Use this skill whenever the user is working on The PotFather UI, mentions Godfather-themed design, requests mockups or components for the PotFather storefront, asks about typography/color/motion for this brand, or wants to generate any visual asset (hero, PDP, PLP, cart, checkout, mystery box page, email template). Also trigger for any request about "premium cannabis branding" or "mafia-inspired e-commerce design" in the context of Global's projects. Enforces brand consistency so the site feels cinematic, old-money, and trustworthy rather than stoner-cliché.
---

# The PotFather Design System

A cinematic, old-world luxury aesthetic inspired by The Godfather (1972). The brand should feel like a speakeasy meets an Italian tailor's shop — NOT like a typical green-leaf-and-tie-dye cannabis site. Authority, discretion, craft.

## Brand Principles

1. **Cinematic, not cartoonish.** Reference the film's sepia/amber color grading, backlit windows, deep shadows. Avoid literal mafia iconography (no tommy guns, no cartoon dons).
2. **Old money, not loud money.** Serif typography, understated gold, aged textures. No neon, no holographic, no drop-shadow gradients.
3. **Craft over hype.** Product photography should feel like still-life painting, not Instagram flat-lay.
4. **Discretion signals trust.** In a regulated vertical, a calm, confident aesthetic converts better than aggressive "420" marketing.

## Color Tokens

```css
:root {
  /* Core */
  --pf-black: #0A0705;           /* near-black, warm */
  --pf-ink: #1A1512;             /* card / elevated bg */
  --pf-burgundy: #4A0E0E;        /* primary accent, wine */
  --pf-burgundy-deep: #2C0606;   /* hover / pressed */
  --pf-gold: #C9A55C;            /* CTA, highlights */
  --pf-gold-bright: #E6C878;     /* hover on gold */
  --pf-gold-muted: #8C7340;      /* secondary gold */

  /* Neutrals (warm-tinted) */
  --pf-cream: #EFE6D2;           /* primary text on dark */
  --pf-parchment: #D9CCAE;       /* body text on dark */
  --pf-smoke: #6B5E4A;           /* muted text, borders */
  --pf-shadow: #000000CC;        /* overlay */

  /* Functional */
  --pf-success: #4A6B3A;         /* olive green, not bright */
  --pf-warning: #B8860B;         /* dark goldenrod */
  --pf-error: #8B1A1A;           /* blood red */
}
```

**Usage rules:**
- Default backgrounds: `--pf-black` or `--pf-ink`
- Body text on dark: `--pf-parchment` (never pure white)
- Headlines: `--pf-cream` or `--pf-gold`
- Primary CTA: `--pf-burgundy` bg + `--pf-cream` text, OR `--pf-gold` bg + `--pf-black` text
- Never use pure `#000` or `#FFF` — always warm-tinted blacks and creams
- Green is forbidden EXCEPT `--pf-success` for order-confirmed states, used sparingly

## Typography

**Heading font: Cinzel** (Google Fonts) — Roman inscriptional capitals, matches the Godfather title card.
**Alt heading: Trajan Pro** (if licensed) or **Cormorant Garamond** as free fallback.
**Body font: Lora** or **Crimson Pro** (serif, readable at long length).
**UI/numeric font: Inter** (sans, for tiny UI chrome only — prices, badges, nav).

```css
--font-display: 'Cinzel', 'Trajan Pro', serif;
--font-body: 'Lora', 'Crimson Pro', Georgia, serif;
--font-ui: 'Inter', system-ui, sans-serif;
```

**Type scale** (mobile-first, 1.25 ratio):
| Token | Size | Usage |
|-------|------|-------|
| `--text-xs` | 0.75rem | Badges, legal footnotes |
| `--text-sm` | 0.875rem | UI labels |
| `--text-base` | 1rem | Body |
| `--text-lg` | 1.125rem | Lead paragraph |
| `--text-xl` | 1.5rem | Card titles |
| `--text-2xl` | 2rem | Section titles |
| `--text-3xl` | 2.75rem | Page titles |
| `--text-4xl` | 3.75rem | Hero (desktop 5rem+) |

**Rules:**
- Display type: uppercase, `letter-spacing: 0.08em`, no bold (Cinzel is display)
- Body type: natural case, `line-height: 1.7`, `letter-spacing: 0.01em`
- Prices: `--font-ui`, tabular-nums, medium weight
- NEVER mix more than two families on one screen

## Spacing & Layout

8px base grid. Tokens: `4 / 8 / 12 / 16 / 24 / 32 / 48 / 64 / 96 / 128`.

- Max content width: `1280px` with `--gutter: clamp(1rem, 4vw, 3rem)` on sides
- Section vertical rhythm: `96px` desktop, `64px` mobile
- Cards: `24px` internal padding, `12px` radius max
- Buttons: `12px 24px` padding, `4px` radius (sharper than cards — reads more authoritative)

## Iconography & Ornament

- Line icons only (1.5px stroke), **Lucide** as base library
- Custom ornaments: fleur-de-lis dividers, laurel wreaths, monogram "P" interlocked with "F"
- NEVER use cannabis leaf icon — use subtle trichome patterns or smoke whispers instead
- Favicon/logo: interlocked PF monogram in gold on black, inspired by the Godfather puppeteer hand

## Texture & Imagery

- Background texture: subtle paper grain or dark velvet (`opacity: 0.04`) overlaid on solid colors
- Photography treatment: warm tint (`+15` temp), lifted blacks (film-like), subtle grain
- Product shots: dark background, single raking light, shallow depth of field
- Hero imagery: candle-lit, wood-paneled, cinema aspect (21:9 desktop)

## Motion

Understated, slow, confident. Never bouncy or playful.

```css
--ease-smooth: cubic-bezier(0.4, 0, 0.2, 1);
--ease-authority: cubic-bezier(0.77, 0, 0.175, 1);
--dur-quick: 200ms;
--dur-base: 400ms;
--dur-slow: 800ms;
```

- Page transitions: 400ms fade + 8px rise
- Hover states: 200ms, subtle (1-3% brightness shift, never scale)
- Hero reveal on load: 1200ms staggered fade from bottom
- Framer Motion for complex sequences; CSS transitions for atomic hovers
- Reduce motion respected via `@media (prefers-reduced-motion)`

## Component Patterns

### Button (Primary)
```tsx
<button className="
  font-ui text-sm uppercase tracking-[0.15em] font-medium
  bg-[--pf-burgundy] text-[--pf-cream]
  px-6 py-3 rounded-sm
  border border-[--pf-gold]/20
  transition-all duration-200
  hover:bg-[--pf-burgundy-deep] hover:border-[--pf-gold]/60
  active:translate-y-[1px]
">
  Add to Collection
</button>
```

Copy convention: never "Buy Now" or "Add to Cart" — use "Add to Collection", "Reserve", "Acquire", "Join the Family" (for the mystery box).

### Card (Product)
- Dark ink background with 1px gold border at 20% opacity
- Aspect-ratio 4:5 image top
- Title in Cinzel uppercase, small
- Price + cannabinoid % side-by-side in Inter
- On hover: border brightens to 60% gold, slight lift (`translateY(-2px)`, `box-shadow` deep)

### Hero Section
- Full viewport height, dark cinematic photo
- Headline in Cinzel, 4-5xl, centered, letter-spaced wide
- Tagline in body serif italic
- Single gold CTA, small, placed low with generous breathing room
- Vignette overlay gradient bottom-up

### Navigation
- Thin horizontal bar, all caps UI font, small text
- Logo centered or far left
- No mega-menus — use quiet dropdown or full-screen overlay with editorial layout
- Cart icon: monogram-framed, count badge in burgundy

### Forms
- Input: transparent bg, bottom-border only (`1px solid --pf-gold-muted`), focus pulls border to gold-bright
- Labels: small caps UI, above field
- Errors: blood-red text below, italic

### The Mystery Box Page
Treatment: "Join the Family." Present as membership, not subscription. Include:
- Cinematic hero: candlelit wooden table with the box, whisper of smoke
- Tier names: *Consigliere / Capo / Don* (monthly / quarterly / annual commitment)
- Copy voice: oath-like, discreet. "Once a month, a package arrives. No explanations needed."
- Unboxing gallery from past months
- Testimonials styled as handwritten letter scans

## Copy Voice

- **Confident, discreet, classical.** Short sentences. Italian-tinged vocabulary sparingly (*la famiglia*, *omertà*, *consigliere*).
- **Never**: "dope", "stoned", "blazed", "420", "high AF", emojis (except 🥀 or 🌹 used once per page max).
- **Taglines in rotation**:
  - "An offer you cannot refuse."
  - "Quality is a family business."
  - "We keep our promises in writing."
  - "Discretion. Craft. Ritual."

## Accessibility (non-negotiable)

- Contrast: all text on dark must hit 4.5:1 minimum (parchment on black passes; gold on black passes for large text only)
- Focus rings: 2px gold outline, offset 2px — always visible
- All interactive elements have ≥44px tap target
- Motion respects `prefers-reduced-motion`
- Form errors announced via `aria-live="polite"`
- Skip-to-content link on every page (hidden until focus)

## Responsive Breakpoints

```
sm: 640px   - phone landscape
md: 768px   - tablet
lg: 1024px  - desktop
xl: 1280px  - large desktop
2xl: 1536px - wide
```

Design mobile-first. The hero must sing on a 375px iPhone before anything else.

## Forbidden (hard rules)

- No cannabis leaf iconography anywhere
- No neon, glow, or drop shadow effects on text
- No sans-serif display headlines
- No pure black (`#000`) or pure white (`#FFF`)
- No green as primary brand color
- No stock photos of people smoking
- No emoji in UI chrome (product cards, nav, buttons)
- No "Buy Now" / "Add to Cart" literal copy — always brand-voiced alternatives
- No Comic Sans, Papyrus, or any script font that isn't a licensed luxury script

## Quick reference for Claude when generating code

When asked to build a UI component for The PotFather:
1. Pull colors and fonts from tokens above
2. Default to dark mode (there is no light mode for this brand)
3. Use serif display for headings, sans for UI chrome only
4. Keep motion subtle (200-400ms, smooth easing)
5. Apply brand copy voice to all user-facing strings
6. Include accessibility (focus states, aria labels, contrast)
7. Build mobile-first, then enhance up
