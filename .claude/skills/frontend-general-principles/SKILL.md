---
name: frontend-general-principles
description: Frontend General Design Principles. Covers component design, responsive layout, performance optimization, and engineering practices.
version: 1.0.0
---

# Frontend General Design Principles

Guide for frontend engineering best practices.

---

## 1. Component Design & Decoupling

### Atomic Design

| Level | Description | Examples |
|-------|-------------|----------|
| **Atoms** | Basic building blocks | Button, Input, Label |
| **Molecules** | Simple component groups | SearchBar, FormField |
| **Organisms** | Complex UI sections | Header, Card, Sidebar |
| **Templates** | Page layouts | Dashboard, Login |
| **Pages** | Complete pages | HomePage, ProfilePage |

### Principles

- **Single Responsibility**: Each component does one thing
- **UI/Business Separation**: Keep UI and logic separate
- **Prop Drilling Prevention**: Use Context or composition
- **Reusability**: Build reusable components

---

## 2. Responsive & Adaptive Design

### Breakpoints (Tailwind/Twind)

```javascript
// Common breakpoints
const breakpoints = {
  sm: '640px',   // Mobile landscape
  md: '768px',   // Tablet
  lg: '1024px',  // Desktop
  xl: '128px',   // Large desktop
}
```

### Best Practices

- Mobile-first approach
- Use relative units (rem, em, %)
- Test on actual devices
- Consider Tauri desktop window sizes

---

## 3. Performance Optimization

### Code Splitting

```typescript
// Dynamic import for lazy loading
const HeavyComponent = lazy(() => import('./HeavyComponent'));

// Route-based splitting
<Route path="/dashboard" component={lazy(() => import('./Dashboard'))} />
```

### Resource Loading

```typescript
// Lazy load images
<img loading="lazy" src="image.jpg" />

// Preload critical resources
<link rel="preload" href="/fonts/main.woff2" as="font" />
```

### React Optimization

```typescript
// Memoize expensive computations
const expensiveValue = useMemo(() => computeExpensive(a, b), [a, b]);

// Memoize callbacks
const handleClick = useCallback(() => doSomething(a), [a]);

// Memoize components
const ExpensiveComponent = memo(({ data }) => { ... });
```

---

## 4. Engineering Practices

### Lint Configuration

```json
{
  "rules": {
    "no-unused-vars": "error",
    "no-console": "warn",
    "prefer-const": "error"
  }
}
```

### Git Hooks (Husky)

```json
{
  "hooks": {
    "pre-commit": "lint-staged",
    "commit-msg": "commitlint -E HUSKY_GIT_PARAMS"
  }
}
```

### Testing Strategy

| Level | Tool | What to Test |
|--------|------|---------------|
| Unit | Vitest, Jest | Functions, components |
| Integration | Playwright | User flows |
| E2E | Cypress | Critical paths |

---

## Validation Checklist

- [ ] Components follow Atomic Design
- [ ] Responsive on all breakpoints
- [ ] Code splitting implemented
- [ ] Lazy loading for images
- [ ] Lint rules enforced
- [ ] Tests written
- [ ] CI/CD configured
