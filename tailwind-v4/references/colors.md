# OKLCH Colors

> Reference for: Tailwind v4 Expert
> Load when: Color system, OKLCH format, palettes, semantic colors, dark mode colors

## Why OKLCH?

Tailwind v4 uses OKLCH as the default color format because it's **perceptually uniform** - colors with the same lightness value actually look equally bright to human eyes.

### OKLCH vs RGB/HSL

| Format | Perceptually Uniform | Wide Gamut | Easy to Manipulate |
|--------|---------------------|------------|-------------------|
| RGB | No | No | No |
| HSL | No | No | Yes (hue) |
| OKLCH | **Yes** | **Yes** | **Yes** |

---

## OKLCH Syntax

```
oklch(L% C H)

L = Lightness (0% to 100%)
C = Chroma (0 to ~0.4, typically 0.1-0.3)
H = Hue (0 to 360 degrees)
```

### Lightness (L)
- `0%` = Black
- `50%` = Middle gray/color
- `100%` = White

### Chroma (C)
- `0` = Grayscale (no color)
- `0.1` = Muted/pastel
- `0.2` = Saturated
- `0.3+` = Vivid/intense

### Hue (H)
- `0` = Pink/Red
- `30` = Orange
- `60` = Yellow
- `90` = Lime
- `145` = Green
- `180` = Cyan
- `220` = Blue
- `270` = Purple
- `330` = Magenta

---

## Color Palette Examples

### Primary Blue Palette

```css
@theme {
  --color-blue-50: oklch(97% 0.02 250);
  --color-blue-100: oklch(93% 0.04 250);
  --color-blue-200: oklch(87% 0.08 250);
  --color-blue-300: oklch(77% 0.12 250);
  --color-blue-400: oklch(65% 0.18 250);
  --color-blue-500: oklch(55% 0.22 250);   /* Base */
  --color-blue-600: oklch(48% 0.22 250);
  --color-blue-700: oklch(40% 0.20 250);
  --color-blue-800: oklch(33% 0.16 250);
  --color-blue-900: oklch(27% 0.12 250);
  --color-blue-950: oklch(20% 0.08 250);
}
```

### Emerald/Green Palette

```css
@theme {
  --color-emerald-50: oklch(97% 0.02 160);
  --color-emerald-100: oklch(93% 0.04 160);
  --color-emerald-200: oklch(87% 0.08 160);
  --color-emerald-300: oklch(77% 0.14 160);
  --color-emerald-400: oklch(67% 0.18 160);
  --color-emerald-500: oklch(60% 0.20 160);   /* Base */
  --color-emerald-600: oklch(52% 0.18 160);
  --color-emerald-700: oklch(44% 0.16 160);
  --color-emerald-800: oklch(36% 0.12 160);
  --color-emerald-900: oklch(28% 0.08 160);
  --color-emerald-950: oklch(20% 0.06 160);
}
```

### Red/Error Palette

```css
@theme {
  --color-red-50: oklch(97% 0.02 25);
  --color-red-100: oklch(93% 0.05 25);
  --color-red-200: oklch(87% 0.10 25);
  --color-red-300: oklch(77% 0.16 25);
  --color-red-400: oklch(65% 0.22 25);
  --color-red-500: oklch(55% 0.25 25);   /* Base */
  --color-red-600: oklch(48% 0.24 25);
  --color-red-700: oklch(40% 0.20 25);
  --color-red-800: oklch(33% 0.16 25);
  --color-red-900: oklch(27% 0.12 25);
  --color-red-950: oklch(20% 0.08 25);
}
```

### Amber/Warning Palette

```css
@theme {
  --color-amber-50: oklch(98% 0.02 85);
  --color-amber-100: oklch(95% 0.05 85);
  --color-amber-200: oklch(90% 0.10 85);
  --color-amber-300: oklch(83% 0.15 85);
  --color-amber-400: oklch(75% 0.18 85);
  --color-amber-500: oklch(70% 0.20 85);   /* Base */
  --color-amber-600: oklch(60% 0.18 85);
  --color-amber-700: oklch(50% 0.16 85);
  --color-amber-800: oklch(42% 0.12 85);
  --color-amber-900: oklch(35% 0.08 85);
  --color-amber-950: oklch(25% 0.06 85);
}
```

### Neutral/Gray Palette

```css
@theme {
  --color-gray-50: oklch(98% 0.005 250);
  --color-gray-100: oklch(95% 0.008 250);
  --color-gray-200: oklch(90% 0.010 250);
  --color-gray-300: oklch(83% 0.012 250);
  --color-gray-400: oklch(70% 0.015 250);
  --color-gray-500: oklch(55% 0.015 250);   /* Base */
  --color-gray-600: oklch(45% 0.015 250);
  --color-gray-700: oklch(35% 0.015 250);
  --color-gray-800: oklch(25% 0.015 250);
  --color-gray-900: oklch(18% 0.015 250);
  --color-gray-950: oklch(12% 0.015 250);
}
```

---

## Semantic Color System

### Light Mode

```css
@theme {
  /* Background layers */
  --color-background: oklch(100% 0 0);           /* Pure white */
  --color-surface: oklch(98% 0.005 250);         /* Subtle gray */
  --color-surface-raised: oklch(100% 0 0);       /* Cards, modals */

  /* Text colors */
  --color-foreground: oklch(15% 0.02 250);       /* Primary text */
  --color-muted: oklch(45% 0.02 250);            /* Secondary text */
  --color-placeholder: oklch(60% 0.015 250);     /* Placeholder text */

  /* Borders */
  --color-border: oklch(88% 0.01 250);           /* Default border */
  --color-border-hover: oklch(80% 0.015 250);    /* Hover state */
  --color-ring: oklch(55% 0.22 250);             /* Focus ring */

  /* Interactive */
  --color-primary: oklch(55% 0.22 250);          /* Primary action */
  --color-primary-hover: oklch(48% 0.22 250);    /* Primary hover */
  --color-primary-foreground: oklch(100% 0 0);   /* Text on primary */

  /* Status */
  --color-success: oklch(60% 0.18 145);
  --color-success-foreground: oklch(100% 0 0);
  --color-warning: oklch(70% 0.18 85);
  --color-warning-foreground: oklch(20% 0.05 85);
  --color-error: oklch(55% 0.22 25);
  --color-error-foreground: oklch(100% 0 0);
}
```

### Dark Mode

```css
/* Dark mode overrides */
.dark,
[data-theme="dark"] {
  /* Background layers */
  --color-background: oklch(15% 0.02 250);
  --color-surface: oklch(20% 0.02 250);
  --color-surface-raised: oklch(25% 0.02 250);

  /* Text colors */
  --color-foreground: oklch(95% 0.01 250);
  --color-muted: oklch(65% 0.015 250);
  --color-placeholder: oklch(50% 0.015 250);

  /* Borders */
  --color-border: oklch(30% 0.02 250);
  --color-border-hover: oklch(40% 0.02 250);
  --color-ring: oklch(60% 0.20 250);

  /* Interactive - slightly brighter for dark mode */
  --color-primary: oklch(60% 0.20 250);
  --color-primary-hover: oklch(65% 0.18 250);
  --color-primary-foreground: oklch(15% 0.02 250);
}
```

---

## Creating Color Scales

### Formula for Consistent Palettes

```
Light shades (50-200):  L: 87-97%,  C: 0.02-0.08
Medium shades (300-500): L: 55-77%,  C: 0.12-0.22
Dark shades (600-950):   L: 12-48%,  C: 0.08-0.22
```

### Generate Palette from Base Color

```css
/* Base: oklch(55% 0.22 250) - blue-500 */

@theme {
  /* Lighter: increase L, decrease C */
  --color-brand-50: oklch(97% 0.02 250);
  --color-brand-100: oklch(93% 0.05 250);
  --color-brand-200: oklch(87% 0.10 250);
  --color-brand-300: oklch(77% 0.15 250);
  --color-brand-400: oklch(65% 0.20 250);

  /* Base */
  --color-brand-500: oklch(55% 0.22 250);

  /* Darker: decrease L, maintain/decrease C */
  --color-brand-600: oklch(48% 0.22 250);
  --color-brand-700: oklch(40% 0.20 250);
  --color-brand-800: oklch(33% 0.16 250);
  --color-brand-900: oklch(25% 0.12 250);
  --color-brand-950: oklch(18% 0.08 250);
}
```

---

## Color with Opacity

```css
/* Using oklch with alpha channel */
@theme {
  --color-overlay: oklch(0% 0 0 / 0.5);           /* 50% black overlay */
  --color-backdrop: oklch(0% 0 0 / 0.75);         /* 75% black backdrop */
  --color-highlight: oklch(55% 0.22 250 / 0.1);   /* 10% primary highlight */
}

/* In utilities */
<div class="bg-primary/10">   <!-- 10% opacity -->
<div class="bg-primary/50">   <!-- 50% opacity -->
<div class="text-foreground/80"> <!-- 80% opacity -->
```

---

## Hue Reference Chart

| Color | Hue (H) | Example |
|-------|---------|---------|
| Red | 25 | `oklch(55% 0.25 25)` |
| Orange | 50 | `oklch(65% 0.20 50)` |
| Amber | 85 | `oklch(70% 0.18 85)` |
| Yellow | 100 | `oklch(85% 0.15 100)` |
| Lime | 125 | `oklch(75% 0.20 125)` |
| Green | 145 | `oklch(60% 0.20 145)` |
| Emerald | 160 | `oklch(60% 0.18 160)` |
| Teal | 180 | `oklch(55% 0.15 180)` |
| Cyan | 200 | `oklch(70% 0.15 200)` |
| Sky | 220 | `oklch(65% 0.18 220)` |
| Blue | 250 | `oklch(55% 0.22 250)` |
| Indigo | 270 | `oklch(50% 0.22 270)` |
| Violet | 290 | `oklch(55% 0.22 290)` |
| Purple | 300 | `oklch(55% 0.22 300)` |
| Fuchsia | 320 | `oklch(60% 0.25 320)` |
| Pink | 350 | `oklch(65% 0.20 350)` |
| Rose | 10 | `oklch(60% 0.22 10)` |

---

## Quick Reference

| Task | OKLCH Pattern |
|------|---------------|
| Lighten color | Increase L, decrease C slightly |
| Darken color | Decrease L, decrease C slightly |
| Mute color | Decrease C (toward 0) |
| Vivid color | Increase C (toward 0.3) |
| Shift hue | Change H value (0-360) |
| Add opacity | Add `/opacity` (e.g., `oklch(55% 0.22 250 / 0.5)`) |
| Grayscale | Set C to 0 or very low (0.01) |
