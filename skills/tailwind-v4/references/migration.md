# Migration Guide: v3 to v4

> Reference for: Tailwind v4 Expert
> Load when: Upgrading from Tailwind v3, migration, breaking changes

## Major Changes Overview

| Aspect | Tailwind v3 | Tailwind v4 |
|--------|-------------|-------------|
| Configuration | `tailwind.config.js` | `@theme` in CSS |
| CSS Import | `@tailwind base/components/utilities` | `@import 'tailwindcss'` |
| Build Tool | PostCSS plugin | Vite plugin (faster) |
| Color Format | RGB/HSL | OKLCH |
| CSS Variables | Optional | Default |

---

## Step 1: Update Dependencies

### Remove v3 Packages

```bash
npm uninstall tailwindcss postcss autoprefixer
```

### Install v4 Packages

```bash
npm install tailwindcss@^4.0.0 @tailwindcss/vite@^4.0.0
```

---

## Step 2: Update Vite Config

### Before (v3 with PostCSS)

```javascript
// vite.config.js
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'

export default defineConfig({
  plugins: [vue()],
  // PostCSS handled Tailwind
})
```

```javascript
// postcss.config.js
module.exports = {
  plugins: {
    tailwindcss: {},
    autoprefixer: {}
  }
}
```

### After (v4 with Vite Plugin)

```typescript
// vite.config.ts
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'
import tailwindcss from '@tailwindcss/vite'

export default defineConfig({
  plugins: [
    vue(),
    tailwindcss()
  ]
})
```

**Delete** `postcss.config.js` (no longer needed for Tailwind).

---

## Step 3: Update CSS Entry Point

### Before (v3)

```css
/* src/styles/main.css */
@tailwind base;
@tailwind components;
@tailwind utilities;

/* Custom styles */
.btn {
  @apply px-4 py-2 rounded-lg;
}
```

### After (v4)

```css
/* src/styles/main.css */
@import 'tailwindcss';

/* Custom styles in layers */
@layer components {
  .btn {
    @apply px-4 py-2 rounded-lg;
  }
}
```

---

## Step 4: Migrate Configuration

### Before (tailwind.config.js)

```javascript
// tailwind.config.js
module.exports = {
  content: ['./src/**/*.{vue,js,ts}'],
  theme: {
    extend: {
      colors: {
        primary: {
          DEFAULT: '#3B82F6',
          dark: '#2563EB'
        },
        secondary: '#10B981'
      },
      fontFamily: {
        sans: ['Inter', 'sans-serif']
      },
      spacing: {
        '128': '32rem'
      },
      borderRadius: {
        '4xl': '2rem'
      }
    }
  },
  plugins: []
}
```

### After (@theme in CSS)

```css
/* src/styles/main.css */
@import 'tailwindcss';

@theme {
  /* Colors - convert to OKLCH */
  --color-primary: oklch(55% 0.22 250);
  --color-primary-dark: oklch(48% 0.22 250);
  --color-secondary: oklch(60% 0.18 160);

  /* Typography */
  --font-sans: 'Inter', ui-sans-serif, system-ui, sans-serif;

  /* Custom spacing */
  --spacing-128: 32rem;

  /* Custom border radius */
  --radius-4xl: 2rem;
}
```

**Delete** `tailwind.config.js` after migration.

---

## Step 5: Convert Colors to OKLCH

### RGB to OKLCH Conversion

| v3 Color | v4 OKLCH Equivalent |
|----------|-------------------|
| `#3B82F6` (blue-500) | `oklch(55% 0.22 250)` |
| `#2563EB` (blue-600) | `oklch(48% 0.22 250)` |
| `#10B981` (emerald-500) | `oklch(60% 0.18 160)` |
| `#EF4444` (red-500) | `oklch(55% 0.25 25)` |
| `#F59E0B` (amber-500) | `oklch(70% 0.18 85)` |
| `#6B7280` (gray-500) | `oklch(55% 0.02 250)` |

### Online Converter

Use [oklch.com](https://oklch.com) to convert colors.

### Preserve RGB if Needed

```css
@theme {
  /* You can still use RGB if needed */
  --color-legacy: rgb(59, 130, 246);

  /* But OKLCH is recommended */
  --color-primary: oklch(55% 0.22 250);
}
```

---

## Step 6: Update Dark Mode

### Before (v3)

```javascript
// tailwind.config.js
module.exports = {
  darkMode: 'class', // or 'media'
}
```

```html
<div class="bg-white dark:bg-gray-900">...</div>
```

### After (v4)

Dark mode works automatically. You can use:

#### Option A: CSS Variables (Recommended)

```css
@theme {
  --color-background: oklch(100% 0 0);
  --color-foreground: oklch(15% 0.02 250);
}

.dark {
  --color-background: oklch(15% 0.02 250);
  --color-foreground: oklch(95% 0.01 250);
}
```

```html
<div class="bg-background text-foreground">
  Automatically switches with .dark class
</div>
```

#### Option B: dark: Variant (Same as v3)

```html
<div class="bg-white dark:bg-gray-900">
  Still works the same way
</div>
```

---

## Step 7: Update Plugins

### v3 Plugins Need Updates

Most v3 plugins won't work in v4. Check for v4-compatible versions.

### Common Plugin Alternatives

| v3 Plugin | v4 Approach |
|-----------|-------------|
| `@tailwindcss/forms` | Built-in or use @layer |
| `@tailwindcss/typography` | Use prose classes or @layer |
| Custom plugins | Convert to @theme or @layer |

### Converting Custom Plugin

```javascript
// v3 plugin
module.exports = {
  plugins: [
    function({ addUtilities }) {
      addUtilities({
        '.text-shadow': {
          'text-shadow': '2px 2px 4px rgba(0,0,0,0.1)'
        }
      })
    }
  ]
}
```

```css
/* v4 equivalent */
@layer utilities {
  .text-shadow {
    text-shadow: 2px 2px 4px rgb(0 0 0 / 0.1);
  }
}
```

---

## Step 8: Update Content Detection

### Before (v3)

```javascript
// tailwind.config.js
module.exports = {
  content: [
    './src/**/*.{vue,js,ts,jsx,tsx}',
    './index.html'
  ]
}
```

### After (v4)

Content detection is automatic. No configuration needed for standard setups.

If you need custom paths:

```css
@source "../components/**/*.vue";
@source "../pages/**/*.vue";
```

---

## Common Migration Issues

### Issue 1: Colors Look Different

OKLCH colors may look slightly different from RGB.

**Solution:** Adjust OKLCH values until colors match visually.

### Issue 2: Custom Classes Not Working

Custom utilities defined in config don't migrate automatically.

**Solution:** Move to `@layer utilities` or `@theme`.

### Issue 3: Plugin Errors

v3 plugins throw errors in v4.

**Solution:** Remove plugins and recreate functionality with `@layer`.

### Issue 4: PostCSS Conflicts

Old PostCSS config conflicts with v4.

**Solution:** Remove `postcss.config.js` or update it:

```javascript
// postcss.config.js (if still needed for other plugins)
export default {
  plugins: {
    // Don't include tailwindcss here
    autoprefixer: {} // Only if needed for other CSS
  }
}
```

---

## Gradual Migration Strategy

### Phase 1: Keep v3, Prepare

1. Audit your `tailwind.config.js`
2. List all custom colors, fonts, spacing
3. Convert colors to OKLCH (on paper)

### Phase 2: Create New Branch

```bash
git checkout -b tailwind-v4-migration
```

### Phase 3: Update Dependencies

```bash
npm uninstall tailwindcss postcss autoprefixer
npm install tailwindcss@^4.0.0 @tailwindcss/vite@^4.0.0
```

### Phase 4: Update Configs

1. Update `vite.config.ts`
2. Create new CSS with `@import 'tailwindcss'`
3. Add `@theme` configuration
4. Delete `tailwind.config.js`
5. Delete `postcss.config.js`

### Phase 5: Test

```bash
npm run dev
```

Fix any issues, then:

```bash
npm run build
```

### Phase 6: Merge

```bash
git checkout main
git merge tailwind-v4-migration
```

---

## Quick Reference Cheatsheet

| v3 | v4 |
|----|-----|
| `@tailwind base;` | `@import 'tailwindcss';` |
| `tailwind.config.js` | `@theme { }` in CSS |
| `theme.extend.colors` | `--color-*` in @theme |
| `theme.extend.spacing` | `--spacing-*` in @theme |
| `theme.extend.fontFamily` | `--font-*` in @theme |
| PostCSS plugin | Vite plugin |
| `content: [...]` | Automatic detection |
| `rgb(59, 130, 246)` | `oklch(55% 0.22 250)` |
| Custom plugin | `@layer utilities { }` |
