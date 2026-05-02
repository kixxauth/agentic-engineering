# Aesthetic Brief: East Coast Printers

Companion document to the East Coast Printers Design Brief. The Design Brief defines the *why*; this brief defines the *feel*. Where the two ever conflict, the Design Brief wins — clarity and trust over visual flair.

---

## 1. Core Visual Metaphor

**"The Local Print Shop, Professionally Dressed."**

Picture the calm, organized front office of a long-running New England print shop: white walls, a clean counter, work samples neatly displayed, a knowledgeable owner who answers the phone on the second ring. The site should feel the same way — orderly, confident, regional, and quietly experienced. Not flashy. Not edgy. Not corporate-cold either.

The Chicago Signs site is the reference for *structure and clarity*; East Coast Printers should feel a shade more reserved and a shade more regional than that benchmark. Where Chicago Signs leans on bright red and bold black display type for energy, East Coast Printers leans on deep blue and burgundy for steadiness.

**One-line north star:** *Authoritative, regional, and frictionless — a quote is always one tap away.*

---

## 2. Color System

The Design Brief sets the palette. This brief defines how each color *behaves*.

### Brand colors

| Role | Name | Hex | Behavior |
|---|---|---|---|
| Primary | Blue | `#374087` | Headlines on white, primary CTA buttons, link color, full-bleed section backgrounds for emphasis blocks. This is the dominant brand color and should appear on every screen above the fold. |
| Secondary | Burgundy | `#64324D` | Used sparingly. Reserved for accents: section underlines, hover states, secondary CTA outlines, pull-quote bars. Should never compete with the blue for dominance. |
| Background | White | `#FFFFFF` | The default canvas. Required behind the logo. Carries roughly 70%+ of total surface area. |

### Functional colors

| Role | Hex | Use |
|---|---|---|
| Ink (body text) | `#1A1A1A` | Body copy and most headings on white. Pure black is avoided to prevent harshness. |
| Muted text | `#5A5A5A` | Secondary copy, captions, form labels. |
| Hairline / divider | `#E5E5E5` | Subtle separators, card borders, input borders. |
| Soft surface | `#F5F6FA` | Alternating section backgrounds for visual rhythm without shouting. A whisper of blue undertone, not gray. |
| Success | `#1F7A4D` | Form confirmation, validation pass. |
| Error | `#B3261E` | Form validation, required-field messaging. |
| Warning | `#9A6B00` | Used rarely; lead-time or stock notices only. |

### Contrast & ratio

- All text must meet **WCAG AA** (4.5:1 for body, 3:1 for large text). Blue on white and ink on white both clear AA comfortably; burgundy on white does as well.
- White text on the primary blue background is permitted and encouraged for full-bleed CTA bands.
- Never set burgundy text on blue, or blue text on burgundy.

---

## 3. Typography System

### Typeface

A single sans-serif family, used throughout. Recommended: **Inter** (primary) with **system-ui** as fallback. Inter offers strong weight variation, excellent legibility on small mobile screens, and a neutral, trustworthy character — closer to the editorial sans-serif feel of the Chicago Signs reference than a geometric or rounded face would be.

Acceptable alternatives: Source Sans 3, IBM Plex Sans. Avoid Montserrat (the Chicago Signs choice — slightly too geometric and overused), and avoid anything with personality (Poppins, Nunito, rounded faces).

```
font-family: "Inter", -apple-system, BlinkMacSystemFont, "Segoe UI", system-ui, sans-serif;
```

### Hierarchy & scale

A modest type scale. The Chicago Signs hero sets headlines large and tight; East Coast Printers should follow that lead — confident headlines, generous body sizing for middle-aged readers on phones.

| Token | Mobile | Desktop | Weight | Tracking |
|---|---|---|---|---|
| Display (H1, hero) | 36px / 1.1 | 56px / 1.05 | 700 | -0.02em |
| H2 | 28px / 1.15 | 40px / 1.15 | 700 | -0.01em |
| H3 | 22px / 1.25 | 28px / 1.25 | 600 | normal |
| H4 / eyebrow | 14px / 1.3 | 14px / 1.3 | 600, uppercase, +0.08em | tracked |
| Body | 17px / 1.6 | 18px / 1.6 | 400 | normal |
| Small / caption | 14px / 1.5 | 14px / 1.5 | 400 | normal |
| Button | 16px | 16px | 600 | +0.01em |

**Rules:**
- Headlines are blue (`#374087`) or ink (`#1A1A1A`). Burgundy headlines only as a one-per-page accent.
- Body copy is always ink, never blue.
- No more than three weights on a page (400, 600, 700). No italics in UI; italics only inside body prose if a customer testimonial requires it.
- No all-caps display text. Eyebrow labels only.

### Section headlines

Borrow one device from Chicago Signs: a short **burgundy underline bar** (3–4px tall, ~48px wide) sitting just below H2 section headings. It is the single accent flourish allowed in the design and should appear at most once per section.

---

## 4. Layout & Grid Philosophy

### Density

**Airy, but not minimalist.** The page should breathe — generous vertical rhythm between sections (96px desktop / 64px mobile) — but each section should be information-rich enough that a price-sensitive, time-constrained buyer can scan and act. Avoid the "one giant headline per scroll" pattern; that wastes the user's time.

### Grid

- **Desktop:** 12-column grid, 1200px max content width, 24px gutters.
- **Tablet:** 8-column, fluid.
- **Mobile (primary):** Single column, 16px side padding, full-bleed photography where it earns its keep.

Layouts are **orthogonal and grounded** — content sits squarely on the grid. No tilted cards, no overlapping diagonal slices, no "broken grid" experiments. The Chicago Signs hero uses a subtle offset photo collage; East Coast Printers should *not* copy that. Keep imagery rectangular and aligned.

### Section rhythm

Alternate three section treatments to create rhythm without volume:
1. **White** background, ink + blue type (default).
2. **Soft surface** (`#F5F6FA`) background, ink + blue type (every 2–3 sections).
3. **Primary blue** full-bleed (`#374087`) with white type (used 1–2 times per page max, reserved for high-emphasis CTA bands).

Never use burgundy as a full-bleed background.

### Persistent contact

The phone number and email address must be visible at all times:
- **Top bar:** thin white strip above the main nav, ink text, phone + email + hours, right-aligned. Collapses on mobile into a tap-to-call icon pinned in the header.
- **Footer:** repeated prominently, large enough to read at arm's length.
- **Mobile:** a sticky bottom bar with two buttons — "Call" (primary blue) and "Get a Quote" (burgundy outline) — visible on every page.

---

## 5. Components

### Buttons

- **Primary:** solid blue `#374087`, white text, 4px corner radius, 14px vertical / 24px horizontal padding, 600 weight. Hover: darken 8%.
- **Secondary:** white fill, blue 1.5px border, blue text. Hover: fill blue 8% tint.
- **Tertiary / inline:** blue text, underline on hover only.
- Burgundy is reserved for the sticky mobile "Get a Quote" CTA, where it needs to differentiate from the blue "Call" button. Otherwise buttons are blue.
- Corners are **lightly rounded (4px)**, never pill-shaped, never sharp 0px. This matches the conservative, professional tone — neither playful nor severe.

### Cards (service categories, portfolio tiles)

- White background, 1px hairline border (`#E5E5E5`), 6px corner radius.
- Subtle shadow on hover only: `0 4px 12px rgba(55, 64, 135, 0.08)`. No resting shadow.
- Image-on-top, headline + one-line description below, single text link at the bottom ("Get a quote →" in blue).

### Forms (the conversion path)

Forms are the most important interactive surface on the site. Treat them with care.
- Labels above inputs, never floating-label or placeholder-as-label.
- Inputs: 48px tall minimum (thumb-friendly), 1px `#E5E5E5` border, 4px radius, white fill. Focus state: 2px blue border.
- Required-field asterisks in burgundy.
- Submit button is full-width on mobile, primary blue.
- Inline validation, never modal alerts.

### Iconography

- **Line icons**, 1.5px stroke, rounded caps. Recommended set: Lucide or Phosphor (Regular weight).
- Icons sit in blue or ink, never burgundy.
- No illustrative or 3D icons. No emoji in UI.

---

## 6. Material & Texture

### Surface treatment

**Flat, ink-on-paper.** No gradients on backgrounds. No glassmorphism, no neumorphism, no animated mesh blobs. The only depth cues are:
- The hover shadow on cards (above).
- The thin top-bar shadow under the sticky header when scrolled (`0 1px 0 rgba(0,0,0,0.06)`).

### Photography

This is the single biggest visual lever and the area where the current site likely fails hardest. Rules:
- **Real work, real customers.** Product shots of actual printed apparel and signage East Coast Printers has produced. No generic stock photos of smiling people in headsets.
- **New England signal.** Where appropriate, lean into regional cues — Vermont team apparel, local business branded gear, fall/winter palettes. This is a quiet differentiator from a national competitor like Chicago Signs.
- **Consistent treatment:** natural light, neutral backgrounds, no heavy filters, no duotones. The Chicago Signs reference uses crisp daylight product shots — match that quality bar.
- **Aspect ratios:** 4:3 or 1:1 for product cards, 16:9 for hero. Rectangular and aligned to the grid — no tilted or overlapping collages.
- Photos never sit behind the logo (Design Brief requirement) and never have text laid directly on top without a solid color band beneath.

### Motion

Motion is functional, not decorative.
- Transitions: 150–200ms, ease-out.
- Allowed: button hover color shift, card hover shadow, mobile menu slide-in, form focus ring.
- Disallowed: parallax, scroll-triggered reveals, autoplaying carousels, animated hero text, lottie animations.
- Respect `prefers-reduced-motion`.

---

## 7. The Logo

Per the Design Brief: always on a white background. Practical implications for layout:
- The header and footer must always have a white logo lockup zone, even when the surrounding section is blue or soft-surface. If the header sits over a blue band, the logo lives in a white pill or white top strip above it.
- Minimum clear space around the logo: equal to the height of the logo's "E" character.
- Minimum logo height: 32px mobile, 40px desktop.

---

## 8. What Good Looks Like (and What It Doesn't)

### Looks like

- Chicago Signs' overall structure: clear nav, bold headline, prominent quote CTA, service categories as cards, professional product photography, repeated CTAs throughout.
- The visual restraint of a regional law firm or community bank site — but warmer, with real product photography doing the emotional work.
- Type-forward sections where a confident headline and one short paragraph are enough. No filler.

### Does not look like

- The current East Coast Printers site (visually outdated, low trust).
- Trendy DTC brands (pastel palettes, oversized type, playful illustration).
- Print-industry clichés: ink splatters, halftone overlays, distressed textures, "grunge" type. The Chicago Signs site has a hint of this in one section (paint-splatter behind the apparel collage) — **East Coast Printers should not adopt it.**
- Competing red-white-and-blue patriotic styling. Our blue is deep and reserved, not flag-bright.
- Dark mode. Not in scope; the brand requires white backgrounds.

---

## 9. Decision Heuristic

When a design decision isn't covered here, fall back to this order:
1. **Does it make the phone number, email, or quote form easier to reach?** Ship it.
2. **Would a 55-year-old office manager on an iPhone in dim light find this clear?** If not, simplify.
3. **Is it more conservative than the Chicago Signs reference?** Good. If it's *less* conservative, reconsider.
4. **Does it earn its complexity?** If it's decorative, cut it.

When in doubt, go more conservative. The brand earns trust through competence, not flair.
