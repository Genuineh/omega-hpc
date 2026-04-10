---
name: frontend-styling-twind
description: Twind/Tailwind Styling Guide. Covers design system configuration, component patterns, and dynamic styling.
version: 1.0.0
---

# Twind/Tailwind Styling Guide

Guide for consistent styling with Twind or Tailwind.

---

## 1. Design System Configuration

### Theme Setup

```javascript
// tailwind.config.js
module.exports = {
  theme: {
    extend: {
      colors: {
        brand: {
          50: '#f0f9ff',
          500: '#0ea5e9',
          900: '#0c4a6e',
        },
      },
      fontFamily: {
        sans: ['Inter', 'system-ui', 'sans-serif'],
        mono: ['JetBrains Mono', 'monospace'],
      },
      spacing: {
        '18': '4.5rem',
        '88': '22rem',
      },
    },
  },
};
```

### Twind Configuration

```javascript
// twind.config.ts
import { defineConfig } from '@twind/core';
import presetTailwind from '@twind/preset-tailwind';

export default defineConfig({
  presets: [presetTailwind()],
  theme: {
    colors: {
      brand: {
        50: '#f0f9ff',
        500: '#0ea5e9',
        900: '#0c4a6e',
      },
    },
  },
});
```

---

## 2. Component Patterns

### Reusable Components

```typescript
// components/Button.tsx
interface ButtonProps extends React.ButtonHTMLAttributes<HTMLButtonElement> {
  variant?: 'primary' | 'secondary' | 'danger';
  size?: 'sm' | 'md' | 'lg';
}

export function Button({
  variant = 'primary',
  size = 'md',
  className = '',
  ...props
}: ButtonProps) {
  const baseStyles = 'font-medium rounded transition-colors focus:outline-none focus:ring-2';

  const variants = {
    primary: 'bg-brand-500 text-white hover:bg-brand-600',
    secondary: 'bg-gray-200 text-gray-800 hover:bg-gray-300',
    danger: 'bg-red-500 text-white hover:bg-red-600',
  };

  const sizes = {
    sm: 'px-3 py-1.5 text-sm',
    md: 'px-4 py-2 text-base',
    lg: 'px-6 py-3 text-lg',
  };

  return (
    <button
      className={`${baseStyles} ${variants[variant]} ${sizes[size]} ${className}`}
      {...props}
    />
  );
}
```

### Composing Classes

```typescript
// Utility for common patterns
function cn(...classes: (string | boolean | undefined)[]) {
  return classes.filter(Boolean).join(' ');
}

// Usage
<div className={cn(
  'flex items-center justify-between',
  isActive && 'bg-blue-50',
)} />
```

---

## 3. Dynamic Styling

### Conditional Classes

```typescript
// Using clsx or custom utility
<div
  className={`
    p-4 rounded
    ${isActive ? 'bg-blue-500' : 'bg-gray-100'}
    ${isDisabled ? 'opacity-50 cursor-not-allowed' : ''}
  `}
/>
```

### Responsive Styles

```typescript
// Mobile-first approach
<div className="
  w-full          // Mobile
  md:w-1/2       // Tablet
  lg:w-1/3       // Desktop
">
  Content
</div>
```

### State-Based Styles

```typescript
<input
  className={`
    border rounded px-3 py-2
    focus:ring-2 focus:ring-blue-500
    ${error ? 'border-red-500' : 'border-gray-300'}
  `}
/>
```

---

## 4. Best Practices

- **No Magic Numbers**: Use design tokens
- **Consistent Spacing**: Use spacing scale
- **Color Variables**: Define brand colors once
- **Component First**: Build reusable components
- **Responsive Mobile-First**: Start with mobile, add breakpoints

---

## Checklist

- [ ] Design tokens defined
- [ ] Color system consistent
- [ ] Spacing scale used
- [ ] Reusable components built
- [ ] No inline magic numbers
- [ ] Responsive design tested
