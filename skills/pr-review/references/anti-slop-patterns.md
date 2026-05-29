# Anti-Slop Pattern Catalogue

AI code generators converge on the same visual defaults. These 10 patterns are the most common tells. Check for all of them regardless of whether a PRD Visual Direction exists.

## Patterns

### 1. AI default fonts
**What:** `Inter`, `Roboto`, `Open Sans`, `Arial`, `system-ui` used as the primary font.
**Where:** CSS `font-family`, Tailwind `fontFamily` config, Google Fonts imports, `@font-face` declarations.
**Why it matters:** Every AI model defaults to the same safe fonts. Using them signals zero design intent.

### 2. Purple-to-blue gradients
**What:** `linear-gradient` with purple/violet/blue hues, especially on white backgrounds.
**Where:** CSS and Tailwind gradient classes (`from-purple-*`, `to-blue-*`).
**Why it matters:** The single most common AI-generated colour choice. Instantly recognisable.

### 3. Nested cards
**What:** Card components rendered inside other card components.
**Where:** `<Card>` inside `<Card>` in JSX, or nested elements that both have card-like styling (rounded corners + shadow + padding).
**Why it matters:** Creates visual clutter and breaks hierarchy. Real designs rarely nest cards.

### 4. Identical repeating card grids
**What:** The same card structure (icon + heading + text) repeated 3+ times in a grid with identical sizing.
**Where:** Mapped arrays rendering uniform cards.
**Why it matters:** Lazy composition. Real feature grids vary card sizes, emphasis, or layout.

### 5. Side-stripe borders
**What:** `border-left` or `border-right` with width > 1px used as accent stripes on cards, alerts, or list items.
**Where:** Tailwind `border-l-2`, `border-l-4`, `border-r-4`, etc.
**Why it matters:** A crutch for adding "visual interest" without actual design thinking.

### 6. Gradient text
**What:** `background-clip: text` or `-webkit-background-clip: text` combined with a gradient background.
**Where:** Tailwind `bg-clip-text` with `bg-gradient-*`.
**Why it matters:** Overused AI flourish. Rarely intentional in real designs.

### 7. Uniform spacing
**What:** The same gap/padding value used for every container and section.
**Where:** A single spacing value (e.g., `p-4` or `gap-4`) applied uniformly without variation.
**Why it matters:** Real designs use spacing hierarchy — tighter within groups, looser between sections.

### 8. Pure black/white
**What:** `#000`, `#fff`, `#000000`, `#ffffff`, `black`, `white` used for large areas (backgrounds, text colours).
**Where:** CSS and Tailwind. Small uses in borders or shadows are fine.
**Why it matters:** Almost no real design uses pure black or white. Slightly tinted values look more intentional.

### 9. Centre-everything layouts
**What:** Every major container using `text-center` + `mx-auto` or `items-center` + `justify-center` with no asymmetric or left-aligned sections.
**Where:** Tailwind classes on layout containers.
**Why it matters:** Real layouts mix alignment. All-centred is the AI path of least resistance.

### 10. Dark mode with neon accents
**What:** Dark backgrounds (`bg-gray-900`, `bg-black`) combined with bright neon accent colours (`text-cyan-400`, `text-green-400`, glowing box shadows).
**Where:** Tailwind dark mode classes as the default theme without context-driven reasoning.
**Why it matters:** The "developer portfolio" look. Fine if intentional, AI slop if defaulted to.

## Confidence Guidance

- **High (85+):** Exact pattern match — Inter import, `border-l-4` on a card, `bg-clip-text` with gradient
- **Medium (60-80):** Likely pattern but could be intentional — uniform spacing that might be a deliberate choice, all-centred layout on a landing page hero
- **Low (<60):** Ambiguous or subjective — colour "feels" like an AI default but isn't an exact match
