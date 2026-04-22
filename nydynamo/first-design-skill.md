---
name: ui-design
description: This skill describes the design aesthetic for this site so that that you can stay consistent to your aesthetic as you upgrade the user interface. Use this skill whenever you are making changes to the UI which could impact the aesthetic of the site.
---

## Aesthetic: Modern Athletic Grit

The NY Dynamo site uses a high-energy, high-contrast style found in Nike/Under Armour marketing. It's professional and aggressive — dark backgrounds make the orange accent and cinematic photography pop. The tone is "premium yet tough," exactly right for a Tier 1 hockey development program.

Commit to this direction with precision. Bold maximalism and refined minimalism both work — the key is intentionality, not intensity.

## Quick Reference

| Concept | Implementation |
|---------|---------------|
| Background (dark) | `--color-background-dark` (`#000`) |
| Background (light) | `--color-background-light` (`#fff`) |
| Text on dark | `--color-text-on-dark` (warm off-white) |
| Text on light | `--color-text-on-light` (warm near-black) |
| Orange accent (on light bg) | `--color-primary` |
| Orange accent (on dark bg) | `--color-primary-on-dark` |
| Muted text (on dark bg) | `--color-secondary-on-dark` |
| Heading / display font | `--font-styled` (Exo 2) |
| Body font | `--font-sans` (system sans-serif) |
| Max page width | `--max-site-width` (1024px) with `--site-width-margin` (4vw) |
| Width container | `.site-width-container` class |

## Color System

All brand colors use a warm hue (hsl channel 20). There are separate on-light and on-dark variants tuned for contrast:

- **On dark backgrounds**: Use `--color-primary-on-dark` for orange accents and `--color-secondary-on-dark` for muted/supporting text. Standard text uses `--color-text-on-dark`.
- **On light backgrounds**: Use `--color-primary` for orange accents and `--color-text-on-light` for body text.
- **Opacity treatments**: Use Tailwind's opacity modifier syntax for subtle effects, e.g. `border-(--color-primary-on-dark)/20` or `text-(--color-secondary-on-dark)/60`.

## Typography Patterns

The display font is **Exo 2** (`--font-styled`), used for all headings, brand text, and UI labels. Body text uses the system sans-serif (`--font-sans`).

**Brand heading** (site header — the definitive pattern):
- `font-family: var(--font-styled)`, `text-transform: uppercase`, `font-style: italic`, `font-weight: 700`
- Wide letter-spacing (`0.04em`–`0.1em`)
- The "HOCKEY" badge adds `transform: skew(-10deg)` with an orange background

**Section headings** (e.g. footer column headers):
- `font-family: var(--font-styled)`, `uppercase`, `italic`, `font-weight: 700`
- Small size (`0.75rem`), wide letter-spacing (`0.1em`)
- Orange color (`--color-primary-on-dark` on dark backgrounds)
- See `.site-footer__heading` in `css/custom.css` for the reference implementation

**Body text**: System sans-serif, `text-sm` (0.875rem) for supporting content, default size for main content.

## Link Patterns

- **On dark backgrounds**: `text-(--color-text-on-dark)` default, `hover:text-(--color-primary-on-dark) hover:underline transition-colors` on hover.
- **Accent links** (calls to action): Use `text-(--color-primary-on-dark)` as the default color with `hover:underline`.
- Always add `rel="noopener"` to external links with `target="_blank"`.

## Torn-Paper Edge

The signature visual element — a distressed, torn-paper transition between sections. Implemented as a `::before` or `::after` pseudo-element:

```css
.element::before {
    content: '';
    display: block;
    height: 30px;
    width: 100%;
    background-image: url(/images/papier-top.svg);   /* or papier-bottom.svg */
    background-repeat: no-repeat;
    background-size: cover;
    position: absolute;
    top: -30px;                                       /* or bottom: -30px */
}
```

- `papier-top.svg` — torn edge above a dark section (content flows down into dark)
- `papier-bottom.svg` — torn edge below a dark section (dark flows down into content)
- The parent element needs `position: relative` for the absolute positioning to work.

## Responsive Breakpoints

Use Tailwind's standard breakpoints. Mobile-first approach — write base styles for mobile, then layer up:

| Prefix | Min-width | Typical use |
|--------|-----------|-------------|
| (none) | 0 | Single column, stacked layout |
| `sm:` | 640px | 2-column grids, side-by-side elements |
| `md:` | 768px | Full multi-column layouts |
| `lg:` | 1024px | Matches `--max-site-width`, fine-tuning |
| `xl:` | 1280px | Large hero images, generous spacing |

## Icons

Use inline SVGs for social media and custom icons (Material Symbols is not loaded via CDN). Standard sizing: `w-4 h-4 fill-current` for inline icons next to text.

## Design Principles

- **Color & Theme**: Dominant dark backgrounds with sharp orange accents. Don't distribute colors evenly — let black dominate and orange punctuate.
- **Typography**: Pair Exo 2 display text (bold, italic, uppercase) with clean system sans-serif body text. Italics suggest forward motion and speed.
- **Spatial Composition**: Generous negative space. Asymmetry and overlap where it serves the content. Grid-breaking elements for visual interest.
- **Photography**: Cinematic, tight crops with shallow depth-of-field. Storytelling over posed portraits.
- **Motion**: CSS-only transitions for hover states and micro-interactions. Do *not* use a sticky header or scroll-triggered animations.
- **Restraint**: Match complexity to the vision. Elegance comes from executing well, not adding more. Don't over-decorate.
