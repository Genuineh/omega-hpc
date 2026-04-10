---
name: frontend-design
description: Frontend Design Excellence. Creates distinctive, production-grade frontend interfaces with high design quality. Use when building web components, pages, or applications.
version: 1.0.0
---

# Frontend Design Excellence

Guide for creating distinctive, production-grade frontend interfaces.

---

## Design Thinking

Before coding, understand context and commit to a bold aesthetic direction:

### Key Questions

- **Purpose**: What problem does this interface solve? Who uses it?
- **Tone**: Choose an extreme aesthetic - brutalist, retro-futuristic, luxury, playful, editorial, etc.
- **Constraints**: Technical requirements, framework, performance, accessibility
- **Differentiation**: What makes this unforgettable? What's the memorable element?

### Aesthetic Directions

Choose one and execute with precision:

| Direction | Description |
|----------|-------------|
| **Brutalist/Raw** | Bold typography, high contrast, exposed structure |
| **Luxury/Refined** | Premium feel, subtle animations, elegant spacing |
| **Retro-Futuristic** | Nostalgic yet forward-looking, neon accents |
| **Minimal/Editorial** | Clean, magazine-style, strong typography |
| **Playful/Toy-like** | Fun, colorful, bouncy animations |
| **Organic/Natural** | Soft shapes, earthy tones, calm feel |
| **Industrial** | Utilitarian, exposed materials, technical aesthetic |

---

## Frontend Aesthetics Guidelines

### 1. Typography

```css
/* Good: Distinctive font choices */
@import url('https://fonts.googleapis.com/css2?family=Playfair+Display:wght@400;700&family=DM+Sans:wght@400;500&display=swap');

:root {
  --font-display: 'Playfair Display', serif;
  --font-body: 'DM Sans', sans-serif;
}

/* Avoid generic fonts like Arial, Roboto, Inter */
```

### 2. Color & Theme

```css
:root {
  /* Bold: Dominant color with sharp accents */
  --color-primary: #1a1a2e;
  --color-accent: #e94560;
  --color-surface: #16213e;

  /* Use CSS variables for consistency */
  --shadow-sm: 0 2px 8px rgba(0,0,0,0.1);
  --shadow-lg: 0 8px 32px rgba(0,0,0,0.2);
}
```

### 3. Motion & Animation

```css
/* Staggered reveals on load */
.hero-title {
  animation: fadeInUp 0.6s ease-out;
}
.hero-subtitle {
  animation: fadeInUp 0.6s ease-out 0.1s both;
}
.hero-cta {
  animation: fadeInUp 0.6s ease-out 0.2s both;
}

@keyframes fadeInUp {
  from {
    opacity: 0;
    transform: translateY(20px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

/* Hover states that surprise */
.button:hover {
  transform: scale(1.05);
  box-shadow: 0 8px 24px rgba(233, 69, 96, 0.4);
}
```

### 4. Spatial Composition

```css
/* Unexpected layouts */
.hero {
  display: grid;
  grid-template-columns: 1fr 1.5fr;
  gap: 4rem;
  align-items: center;
}

/* Asymmetry */
.featured-card {
  grid-column: span 2;
  transform: translateY(-2rem);
}

/* Generous negative space */
.section {
  padding: 8rem 2rem;
}
```

### 5. Backgrounds & Visual Details

```css
/* Gradient mesh */
.hero-bg {
  background:
    radial-gradient(at 40% 20%, hsla(280, 90%, 70%, 0.3) 0px, transparent 50%),
    radial-gradient(at 80% 0%, hsla(189, 100%, 56%, 0.2) 0px, transparent 50%),
    radial-gradient(at 0% 50%, hsla(355, 85%, 93%, 0.1) 0px, transparent 50%);
}

/* Noise texture overlay */
.noise::before {
  content: '';
  position: absolute;
  inset: 0;
  background: url('/noise.png');
  opacity: 0.03;
  pointer-events: none;
}

/* Dramatic shadows */
.card {
  box-shadow:
    0 4px 6px rgba(0,0,0,0.02),
    0 10px 20px rgba(0,0,0,0.04),
    0 30px 60px rgba(0,0,0,0.08);
}
```

---

## Design Anti-Patterns to Avoid

| ❌ Avoid | ✅ Instead |
|---------|-----------|
| Generic fonts (Arial, Roboto, Inter) | Distinctive, characterful fonts |
| Purple gradients on white | Bold color choices with purpose |
| Predictable layouts | Unexpected compositions |
| Cookie-cutter components | Context-specific design |
| System fonts | Thoughtful typography pairing |

---

## Implementation Guidelines

### Match Complexity to Vision

- **Maximalist designs**: Need elaborate code, extensive animations
- **Minimalist designs**: Need restraint, precision, careful spacing
- **Elegance** comes from executing the vision well

### Production Checklist

- [ ] Typography is distinctive, not generic
- [ ] Color palette is cohesive and bold
- [ ] Animations are purposeful, not scattered
- [ ] Layout breaks conventions intentionally
- [ ] Backgrounds create atmosphere
- [ ] Hover states delight
- [ ] Responsive on all breakpoints
- [ ] Accessible (contrast, focus states)

---

## Code Example: Hero Section

```tsx
export function Hero() {
  return (
    <section className="hero">
      <div className="hero-bg noise" />
      <div className="hero-content">
        <h1 className="hero-title">
          Design Without
          <span className="gradient-text">Compromise</span>
        </h1>
        <p className="hero-subtitle">
          Build interfaces that capture attention and deliver results.
        </p>
        <div className="hero-actions">
          <button className="btn-primary">Get Started</button>
          <button className="btn-secondary">View Work</button>
        </div>
      </div>
      <div className="hero-visual">
        <div className="floating-card card-1">
          <span className="icon">⚡</span>
          <span>Lightning Fast</span>
        </div>
        <div className="floating-card card-2">
          <span className="icon">🎨</span>
          <span>Beautifully Designed</span>
        </div>
      </div>
    </section>
  );
}
```

```css
.hero {
  min-height: 100vh;
  display: grid;
  grid-template-columns: 1fr 1fr;
  align-items: center;
  gap: 4rem;
  padding: 4rem;
  position: relative;
  overflow: hidden;
}

.gradient-text {
  background: linear-gradient(135deg, #e94560, #0f3460);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
}

.floating-card {
  position: absolute;
  background: rgba(255,255,255,0.1);
  backdrop-filter: blur(20px);
  border: 1px solid rgba(255,255,255,0.2);
  padding: 1.5rem 2rem;
  border-radius: 1rem;
  animation: float 6s ease-in-out infinite;
}

.card-1 { top: 20%; right: 10%; animation-delay: 0s; }
.card-2 { bottom: 20%; right: 25%; animation-delay: 2s; }

@keyframes float {
  0%, 100% { transform: translateY(0); }
  50% { transform: translateY(-20px); }
}
```

---

## Remember

- Commit to a clear aesthetic direction
- Execute with precision and intentionality
- Make bold choices, not safe ones
- Create something unforgettable
- Show what can be truly created when thinking outside the box
