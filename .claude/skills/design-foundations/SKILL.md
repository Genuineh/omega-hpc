---
name: design-foundations
description: Advanced design foundations for software products. Fuses mathematical proportion, color semantics, and industrial-grade aesthetic derivation for interface and design-system work.
version: 1.1.0
---

# Design Foundations

Guide for software product design where visual quality should emerge from mathematical consistency, semantic color logic, and a clear aesthetic thesis.

---

## When to Use

Use this skill when you need to:

- Define or critique the visual logic of a product, feature, or design system
- Establish a reusable visual language before implementation
- Improve an interface that feels noisy, inconsistent, flat, or visually immature
- Make deliberate decisions about scale, spacing, radius, hierarchy, palette, or elevation

Use `frontend-design` for implementation-heavy UI execution.
Use `product-ux` when the main problem is flow, interaction, or usability structure.

---

## Core Philosophy: Systematic Aesthetics

Design is not a pile of decorative moves. It is the visible expression of logic.

**Visual Quality = (Mathematical Consistency x Emotional Tension) / Visual Noise**

The system should produce ordered contrast:

- **Mathematics** creates proportion, rhythm, and repeatability
- **Aesthetics** creates mood, character, and tension
- **Color** creates hierarchy, semantics, and emphasis

Failure modes:

- Aesthetic without structure becomes decoration
- Structure without aesthetics becomes sterile
- Color without value logic becomes noise

---

## 1. Aesthetic Thesis

Before pushing pixels, define the visual axioms of the system.

Specify:

- **Temperament Anchor**: precision technical, soft consumer, brutalist editorial, ceremonial, playful, etc.
- **Density Model**: sparse, balanced, high-density
- **Texture Model**: flat, tactile, luminous, layered
- **Contrast Posture**: soft, restrained, medium, sharp
- **Lighting Model**: top-down, offset, or intentionally flat
- **Layering Logic**: physical stacking, planar separation, frosted translucency

The interface should read as one point of view, not a set of local compromises.

### Aesthetic Checks

- Can the visual language be described in one sentence?
- Do typography, geometry, motion, and color support the same thesis?
- If branding is removed, does the product still have a recognizable visual signature?

---

## 2. Mathematical Order

Forbid magic numbers. Spacing, type, radius, and component dimensions should come from a defined sequence.

### Scale Functions

Choose a constant `k` and use it consistently:

- **High-density systems**: `1.125`
- **Balanced systems**: `1.25`
- **Expressive systems**: `1.333` to `1.5`

Use the scale explicitly:

- `size_n = base x k^n` for type scale
- Linear or geometric progressions from a base unit for spacing
- Radius steps derived from the same proportional logic

### Base Unit

Choose a base unit such as `4px` or `8px` and derive:

- Margins
- Gutters
- Padding
- Control heights
- Icon containers

Break the grid only when the exception is stronger than the rule.

### Radius Hierarchy

Nested corners should preserve concentric harmony.

- `r_inner = r_outer - padding`

This prevents the visual clash common in amateur nested surfaces.

### Rhythm Rules

- Similar elements repeat similar intervals
- Hierarchy breaks should be clearly larger, not slightly different
- Large surfaces need internal alignment, not only outer alignment

Good rhythm makes a screen feel inevitable instead of assembled.

---

## 3. Color Semantics and Value Architecture

Color is a functional container, not a bag of favorite swatches.

### Define Roles First

Specify roles before exact values:

- **Canvas**: page or app background
- **Surface**: cards, panels, raised planes
- **Ink**: primary, secondary, and muted text
- **Accent**: brand emphasis and high-value interaction points
- **Signal**: success, warning, danger, info
- **Border**: separation, restraint, and edge control

### Control the Three Variables

Every palette must manage:

- **Hue**: color family
- **Value**: lightness or darkness
- **Chroma**: intensity or saturation

Most broken interfaces fail in value architecture, not hue selection.

### The Grayscale Test

Before adding hue, the interface should work in grayscale.

- Surface-to-surface contrast should remain perceptible
- Typography must satisfy WCAG AA against its actual surface
- Primary hierarchy should still read without relying on color

### Distribution Heuristics

Use color sparingly and by role:

- **60% neutrals**: canvas and structure
- **30% secondary/supporting tones**: subdued text, icons, inactive states
- **10% accent or signal**: brand energy, calls to action, critical emphasis

This is a heuristic, not a law.

### Practical Rules

- Use value contrast before adding more hues
- Reserve high-chroma color for actions, alerts, or focal points
- Let neutrals carry most of the interface
- Keep brand color and signal color distinct when possible
- If everything is vivid, nothing is important

---

## 4. Geometry, Elevation, and Light

Shape language and elevation should express the product's character and physical logic.

### Geometry

- **Sharp geometry**: technical, exact, fast
- **Soft radii**: approachable, consumer-oriented, calm
- **High symmetry**: stable, institutional, formal
- **Controlled asymmetry**: editorial, energetic, contemporary

Do not mix incompatible geometries casually.

### Elevation Steps

Define a small elevation ladder and reuse it:

- **Level 0**: flat canvas, no shadow
- **Level 1**: raised controls and cards
- **Level 2**: floating overlays, popovers, dialogs

Example starting point:

- **Level 1**: `2px` y-offset, `4px` blur, `10%` opacity
- **Level 2**: `8px` y-offset, `16px` blur, `15%` opacity

These are seeds, not universal truth. Keep the progression mathematically related.

### Ambient Tinting

Professional shadows are rarely pure black.

- Infuse `2-5%` of the dominant brand hue into shadows when the visual language benefits from a more integrated atmosphere

This helps shadows feel luminous and intentional instead of dirty or detached.

---

## 5. Synthesis Workflow

When designing or reviewing a product surface:

1. Define the aesthetic thesis and desired emotional tension.
2. Establish atomic scales for type, space, and radius.
3. Map the grayscale value landscape for canvas, surface, and elevated layers.
4. Assign semantic color roles before final hue selection.
5. Apply accent only to high-value interaction points and focal hierarchy.
6. Check nested geometry for concentric harmony.
7. Stress-test the layout in grayscale and in dense states.

---

## 6. Anti-Patterns

Avoid these failures:

- Random pixel values outside the scale
- Mixed radius logic with no concentric relationship
- Accent color used on every interactive or semantic state
- Decoration added to hide weak structure
- Shadows with no global lighting logic
- Trend-following disconnected from product tone or user context
- Geometry that contradicts the aesthetic thesis

---

## Quality Gate

- [ ] The interface has a clear aesthetic thesis
- [ ] Type, spacing, and radius follow a visible proportional system
- [ ] There are no unexplained magic numbers
- [ ] Nested corners follow concentric logic where applicable
- [ ] Hierarchy remains legible in grayscale
- [ ] Text contrast meets WCAG AA against its actual surface
- [ ] Accent color is scarce, intentional, and role-based
- [ ] Geometry and elevation reinforce the same product character
- [ ] Shadows follow a consistent lighting model
