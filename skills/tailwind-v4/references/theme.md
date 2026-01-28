# @theme Directive

> Reference for: Tailwind v4 Expert
> Load when: Theme configuration, CSS variables, @theme modes, design tokens

## @theme Basics

The `@theme` directive replaces `tailwind.config.js` in Tailwind v4. All customization happens in CSS.

```css
@import 'tailwindcss';

@theme {
  /* Your theme configuration */
  --color-primary: oklch(55% 0.25 250);
  --font-sans: 'Inter', system-ui, sans-serif;
  --spacing: 0.25rem;
}
```

---

## @theme Modes

### Default Mode (CSS Variables)

Generates CSS variables at `:root`, enabling dynamic theming.

```css
@theme {
  --color-brand: oklch(60% 0.2 250);
}

/* Generated CSS */
:root {
  --color-brand: oklch(60% 0.2 250);
}

/* Utility uses variable */
.bg-brand {
  background-color: var(--color-brand);
}
```

**Use case:** Dynamic theming, JavaScript-driven changes, user preferences.

### Inline Mode

Inlines values directly without generating CSS variables.

```css
@theme inline {
  --color-accent: oklch(70% 0.25 30);
}

/* Generated CSS - no variable, direct value */
.bg-accent {
  background-color: oklch(70% 0.25 30);
}
```

**Use case:** Better performance, smaller CSS output, static values.

### Reference Mode

Provides fallback values without emitting CSS variables.

```css
@theme reference {
  --color-gray-500: oklch(55% 0.01 250);
}

/* No :root variable generated, but utilities work */
.text-gray-500 {
  color: oklch(55% 0.01 250);
}
```

**Use case:** Reducing `:root` bloat, maintaining utility support.

### Combining Modes

```css
@import 'tailwindcss';

/* Dynamic brand colors (CSS variables) */
@theme {
  --color-primary: oklch(55% 0.25 250);
  --color-secondary: oklch(65% 0.2 180);
}

/* Static utility colors (inline) */
@theme inline {
  --color-gray-100: oklch(95% 0.01 250);
  --color-gray-200: oklch(90% 0.01 250);
  --color-gray-300: oklch(85% 0.01 250);
}

/* Reference values (no output) */
@theme reference {
  --breakpoint-sm: 640px;
  --breakpoint-md: 768px;
}
```

---

## CSS Variable Naming Conventions

### Colors

```css
@theme {
  /* Brand colors */
  --color-primary: oklch(55% 0.25 250);
  --color-primary-light: oklch(70% 0.2 250);
  --color-primary-dark: oklch(40% 0.25 250);

  /* Semantic colors */
  --color-success: oklch(65% 0.2 145);
  --color-warning: oklch(75% 0.2 85);
  --color-error: oklch(55% 0.25 25);
  --color-info: oklch(60% 0.2 250);

  /* Neutral scale */
  --color-gray-50: oklch(98% 0.005 250);
  --color-gray-100: oklch(95% 0.01 250);
  --color-gray-200: oklch(90% 0.01 250);
  --color-gray-300: oklch(85% 0.015 250);
  --color-gray-400: oklch(70% 0.02 250);
  --color-gray-500: oklch(55% 0.02 250);
  --color-gray-600: oklch(45% 0.02 250);
  --color-gray-700: oklch(35% 0.02 250);
  --color-gray-800: oklch(25% 0.02 250);
  --color-gray-900: oklch(15% 0.02 250);
  --color-gray-950: oklch(10% 0.02 250);
}
```

### Typography

```css
@theme {
  /* Font families */
  --font-sans: 'Inter', ui-sans-serif, system-ui, sans-serif;
  --font-serif: 'Merriweather', ui-serif, Georgia, serif;
  --font-mono: 'Fira Code', ui-monospace, monospace;

  /* Font sizes */
  --text-xs: 0.75rem;
  --text-sm: 0.875rem;
  --text-base: 1rem;
  --text-lg: 1.125rem;
  --text-xl: 1.25rem;
  --text-2xl: 1.5rem;
  --text-3xl: 1.875rem;
  --text-4xl: 2.25rem;

  /* Line heights */
  --leading-none: 1;
  --leading-tight: 1.25;
  --leading-normal: 1.5;
  --leading-relaxed: 1.75;

  /* Font weights */
  --font-weight-normal: 400;
  --font-weight-medium: 500;
  --font-weight-semibold: 600;
  --font-weight-bold: 700;

  /* Letter spacing */
  --tracking-tight: -0.025em;
  --tracking-normal: 0;
  --tracking-wide: 0.025em;
}
```

### Spacing

```css
@theme {
  /* Base spacing unit (default: 0.25rem = 4px) */
  --spacing: 0.25rem;

  /* Custom spacing values */
  --spacing-xs: 0.25rem;    /* 4px */
  --spacing-sm: 0.5rem;     /* 8px */
  --spacing-md: 1rem;       /* 16px */
  --spacing-lg: 1.5rem;     /* 24px */
  --spacing-xl: 2rem;       /* 32px */
  --spacing-2xl: 3rem;      /* 48px */
  --spacing-3xl: 4rem;      /* 64px */
}
```

### Sizing

```css
@theme {
  /* Width/Height values */
  --size-xs: 20rem;
  --size-sm: 24rem;
  --size-md: 28rem;
  --size-lg: 32rem;
  --size-xl: 36rem;
  --size-2xl: 42rem;
  --size-3xl: 48rem;
  --size-4xl: 56rem;
  --size-5xl: 64rem;
  --size-6xl: 72rem;
  --size-7xl: 80rem;
}
```

### Breakpoints

```css
@theme {
  --breakpoint-sm: 640px;
  --breakpoint-md: 768px;
  --breakpoint-lg: 1024px;
  --breakpoint-xl: 1280px;
  --breakpoint-2xl: 1536px;
}
```

### Shadows

```css
@theme {
  --shadow-sm: 0 1px 2px 0 rgb(0 0 0 / 0.05);
  --shadow: 0 1px 3px 0 rgb(0 0 0 / 0.1), 0 1px 2px -1px rgb(0 0 0 / 0.1);
  --shadow-md: 0 4px 6px -1px rgb(0 0 0 / 0.1), 0 2px 4px -2px rgb(0 0 0 / 0.1);
  --shadow-lg: 0 10px 15px -3px rgb(0 0 0 / 0.1), 0 4px 6px -4px rgb(0 0 0 / 0.1);
  --shadow-xl: 0 20px 25px -5px rgb(0 0 0 / 0.1), 0 8px 10px -6px rgb(0 0 0 / 0.1);
  --shadow-2xl: 0 25px 50px -12px rgb(0 0 0 / 0.25);
  --shadow-inner: inset 0 2px 4px 0 rgb(0 0 0 / 0.05);
}
```

### Border Radius

```css
@theme {
  --radius-none: 0;
  --radius-sm: 0.125rem;
  --radius: 0.25rem;
  --radius-md: 0.375rem;
  --radius-lg: 0.5rem;
  --radius-xl: 0.75rem;
  --radius-2xl: 1rem;
  --radius-3xl: 1.5rem;
  --radius-full: 9999px;
}
```

---

## Semantic Design Tokens

### Two-Tier Token System

```css
@theme {
  /* Primitive tokens (raw values) */
  --color-blue-500: oklch(55% 0.25 250);
  --color-blue-600: oklch(45% 0.25 250);
  --color-green-500: oklch(65% 0.2 145);
  --color-red-500: oklch(55% 0.25 25);

  /* Semantic tokens (reference primitives) */
  --color-primary: var(--color-blue-500);
  --color-primary-hover: var(--color-blue-600);
  --color-success: var(--color-green-500);
  --color-error: var(--color-red-500);

  /* Component tokens */
  --color-button-primary: var(--color-primary);
  --color-button-primary-hover: var(--color-primary-hover);
  --color-input-border: var(--color-gray-300);
  --color-input-focus: var(--color-primary);
}
```

### Theme Switching with CSS Variables

```css
@theme {
  /* Light theme (default) */
  --color-background: oklch(100% 0 0);
  --color-foreground: oklch(15% 0.02 250);
  --color-surface: oklch(98% 0.005 250);
  --color-border: oklch(90% 0.01 250);
}

/* Override variables for dark mode */
@media (prefers-color-scheme: dark) {
  :root {
    --color-background: oklch(15% 0.02 250);
    --color-foreground: oklch(95% 0.01 250);
    --color-surface: oklch(20% 0.02 250);
    --color-border: oklch(30% 0.02 250);
  }
}

/* Manual dark mode class */
.dark {
  --color-background: oklch(15% 0.02 250);
  --color-foreground: oklch(95% 0.01 250);
  --color-surface: oklch(20% 0.02 250);
  --color-border: oklch(30% 0.02 250);
}
```

---

## Complete Theme Example

```css
@import 'tailwindcss';

@theme {
  /* ===== COLORS ===== */
  /* Brand */
  --color-primary: oklch(55% 0.25 250);
  --color-primary-light: oklch(70% 0.2 250);
  --color-primary-dark: oklch(40% 0.25 250);
  --color-secondary: oklch(65% 0.15 180);

  /* Semantic */
  --color-success: oklch(65% 0.2 145);
  --color-warning: oklch(75% 0.2 85);
  --color-error: oklch(55% 0.25 25);
  --color-info: oklch(60% 0.2 250);

  /* Surfaces */
  --color-background: oklch(100% 0 0);
  --color-foreground: oklch(15% 0.02 250);
  --color-surface: oklch(98% 0.005 250);
  --color-muted: oklch(95% 0.01 250);
  --color-border: oklch(90% 0.01 250);

  /* ===== TYPOGRAPHY ===== */
  --font-sans: 'Inter', ui-sans-serif, system-ui, sans-serif;
  --font-mono: 'JetBrains Mono', ui-monospace, monospace;

  /* ===== SPACING ===== */
  --spacing: 0.25rem;

  /* ===== BORDERS ===== */
  --radius: 0.5rem;
  --radius-lg: 0.75rem;
  --radius-full: 9999px;

  /* ===== SHADOWS ===== */
  --shadow-sm: 0 1px 2px 0 rgb(0 0 0 / 0.05);
  --shadow: 0 1px 3px 0 rgb(0 0 0 / 0.1), 0 1px 2px -1px rgb(0 0 0 / 0.1);
  --shadow-lg: 0 10px 15px -3px rgb(0 0 0 / 0.1);

  /* ===== TRANSITIONS ===== */
  --transition-fast: 150ms;
  --transition-normal: 200ms;
  --transition-slow: 300ms;

  /* ===== BREAKPOINTS ===== */
  --breakpoint-sm: 640px;
  --breakpoint-md: 768px;
  --breakpoint-lg: 1024px;
  --breakpoint-xl: 1280px;
  --breakpoint-2xl: 1536px;
}
```

---

## Quick Reference

| Variable Pattern | Example | Utility |
|-----------------|---------|---------|
| `--color-{name}` | `--color-primary` | `bg-primary`, `text-primary` |
| `--font-{family}` | `--font-sans` | `font-sans` |
| `--text-{size}` | `--text-lg` | `text-lg` |
| `--spacing` | `--spacing: 0.25rem` | `p-4` = 1rem |
| `--radius-{size}` | `--radius-lg` | `rounded-lg` |
| `--shadow-{size}` | `--shadow-md` | `shadow-md` |
| `--breakpoint-{size}` | `--breakpoint-md` | `md:` prefix |
