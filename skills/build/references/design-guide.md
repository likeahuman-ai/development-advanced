# Design Guide for Implementers

Read this before implementing any ticket that involves frontend or UI work. Check the PRD's **Visual Direction** section first — it has the approved design choices for this project. This guide provides the principles behind those choices.

---

## The AI Slop Test

> If you showed this interface to someone and said "AI made this," would they believe you immediately? If yes, rethink your choices.

Every AI model learned from the same templates. Without intervention, they all produce the same defaults: Inter font, purple gradients on white, identical card grids, rounded rectangles with drop shadows. Your job is to build something that looks intentionally designed for THIS project.

---

## What NOT to Do

These are the most recognisable AI-generated design patterns. Avoid them unless the PRD explicitly calls for them:

1. **Generic fonts** — Don't default to Inter, Roboto, Open Sans, or system fonts when the project has personality. Check the PRD's Visual Direction for the approved font pairing.
2. **Purple-to-blue gradients** — Especially on white backgrounds. The most overused AI colour scheme.
3. **Nested cards** — Never put cards inside cards. Use spacing, typography, and subtle dividers for hierarchy within a card.
4. **Identical card grids** — Icon + heading + text, repeated endlessly in the same size. Vary card sizes, span columns, or mix with non-card content.
5. **Side-stripe borders** — No `border-left` or `border-right` thicker than 1px as coloured accent stripes on cards, alerts, or list items. Use background tints, leading icons, or full borders instead.
6. **Gradient text** — No `background-clip: text` with gradients. Use solid colours for text. Emphasise with weight or size, not gradient fill.
7. **Same spacing everywhere** — Uniform padding and gaps kill visual hierarchy. Vary spacing intentionally.
8. **Centre everything** — Left-aligned text with asymmetric layouts feels more designed than centring every element.
9. **Pure black and white** — Don't use `#000` or `#fff`. Tint them slightly toward the brand hue. Even a subtle tint creates cohesion.
10. **Dark-mode-with-glowing-accents by default** — Don't pick dark mode because it "looks cool." Don't pick light mode "to be safe." Derive the theme from who uses the product and when.

---

## What TO Do

### Typography

- **Check the PRD first.** If Visual Direction specifies fonts, use those.
- **Fewer sizes, more contrast.** A 5-size type scale covers most needs. Use at least a 1.25 ratio between steps — sizes too close together (14px, 15px, 16px) create muddy hierarchy.
- **Pair fonts with contrast.** A distinctive display font + a refined body font. Contrast on structure (serif + sans), personality (geometric + humanist), or proportion (condensed + wide). One well-chosen family in multiple weights also works.
- **Cap line length** at 65-75 characters for body text. Use `max-width` with `ch` units.
- **Body text minimum** 16px / 1rem. Smaller strains eyes and fails accessibility on mobile.

### Colour

- **Dominant + accent.** One main colour with sharp accents outperforms timid, evenly-distributed palettes. Accents work because they're rare — overuse kills their power.
- **Tint your neutrals.** Add a tiny colour cast toward the brand hue. Even a subtle hint creates subconscious cohesion between brand colours and UI surfaces.
- **60-30-10 rule** by visual weight: 60% neutral surfaces, 30% secondary (text, borders), 10% accent (CTAs, highlights).
- **Grey on coloured backgrounds looks dead.** Use a darker shade of the background colour instead, or transparency.

### Layout & Spacing

- **Use a spacing scale.** Consistent values from a defined set — 4, 8, 12, 16, 24, 32, 48, 64px. Framework scales (like Tailwind's) work too. The point is consistency, not specific numbers.
- **Vary spacing for hierarchy.** Tight grouping (8-12px) for related elements. Generous separation (48-96px) between sections. If everything has the same gap, nothing has hierarchy.
- **Use `gap` for sibling spacing** instead of margins. Eliminates margin collapse.
- **Cards are not required.** Spacing and alignment create visual grouping naturally. Only use cards when content is truly distinct and actionable.
- **The squint test.** Blur your eyes (metaphorically). Can you still identify the most important element, the second most important, and clear groupings? If everything looks the same weight, you have a hierarchy problem.

### Atmosphere & Depth

- **Create depth**, not flatness. Layered backgrounds, subtle textures, thoughtful shadows — not flat solid colours everywhere.
- **Shadows should be subtle.** If you can clearly see it, it's probably too strong. Shadows create elevation, not decoration.
- **Background variety.** A slight gradient, a subtle pattern, or a soft texture adds atmosphere without being distracting.

### Motion

- **One well-orchestrated moment** beats scattered micro-interactions. A staggered page load with reveals creates more delight than random hover effects.
- **Use ease-out** (quart, quint, or expo) for natural deceleration. Never bounce or elastic — they feel dated.
- **Only animate `transform` and `opacity`.** Animating layout properties (width, height, padding) causes jank.
- **Respect `prefers-reduced-motion`.** Always.

---

## Quick Checklist

Before marking a UI ticket as done:

- [ ] Fonts match the PRD's Visual Direction (not AI defaults)
- [ ] Colour palette has a clear dominant + accent (not evenly distributed)
- [ ] Spacing varies between sections (not uniform everywhere)
- [ ] No nested cards, no identical repeating card grids
- [ ] No side-stripe borders, no gradient text
- [ ] Body text is 16px+ and line length is capped
- [ ] The squint test passes — clear hierarchy visible at a glance
