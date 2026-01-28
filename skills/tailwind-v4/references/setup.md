# Setup & Configuration

> Reference for: Tailwind v4 Expert
> Load when: Installation, Vite config, CSS entry, project setup

## Installation

### New Project with Vite

```bash
# Create Vite project
npm create vite@latest my-app -- --template vue-ts
cd my-app

# Install Tailwind v4
npm install tailwindcss @tailwindcss/vite
```

### Vite Configuration

```typescript
// vite.config.ts
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'
import tailwindcss from '@tailwindcss/vite'

export default defineConfig({
  plugins: [
    vue(),
    tailwindcss()  // Add Tailwind plugin
  ]
})
```

### CSS Entry Point

```css
/* src/styles/main.css */

/* Import Tailwind - this single import includes base, components, utilities */
@import 'tailwindcss';

/* Your custom theme configuration */
@theme {
  /* Custom colors, fonts, spacing, etc. */
  --color-primary: oklch(55% 0.25 250);
  --color-secondary: oklch(65% 0.2 180);
}

/* Custom component styles (optional) */
@layer components {
  .btn {
    @apply px-4 py-2 rounded-lg font-medium transition-colors;
  }
}
```

### Import in Main File

```typescript
// src/main.ts
import { createApp } from 'vue'
import App from './App.vue'
import './styles/main.css'  // Import your CSS entry point

createApp(App).mount('#app')
```

---

## Project Structure

```
src/
├── styles/
│   ├── main.css           # CSS entry point with @import 'tailwindcss'
│   ├── theme.css          # @theme configuration (optional - can be in main.css)
│   └── components.css     # Custom component styles (optional)
├── components/
├── App.vue
└── main.ts
```

### Split Theme Configuration (Optional)

```css
/* src/styles/main.css */
@import 'tailwindcss';
@import './theme.css';
@import './components.css';
```

```css
/* src/styles/theme.css */
@theme {
  /* Colors */
  --color-primary: oklch(55% 0.25 250);
  --color-secondary: oklch(65% 0.2 180);

  /* Typography */
  --font-sans: 'Inter', system-ui, sans-serif;

  /* Spacing */
  --spacing: 0.25rem;
}
```

---

## TypeScript Support

### Type Definitions

```typescript
// src/types/tailwind.d.ts
declare module 'tailwindcss' {
  const content: string
  export default content
}
```

### VSCode Settings

```json
// .vscode/settings.json
{
  "css.validate": false,
  "tailwindCSS.experimental.classRegex": [
    ["class=\"([^\"]*)", "\"([^\"]*)\""],
    ["@apply\\s+([^;]*)", "([^\\s;]*)"]
  ],
  "editor.quickSuggestions": {
    "strings": true
  }
}
```

---

## PostCSS (Alternative Setup)

If you need PostCSS for other plugins:

```bash
npm install tailwindcss @tailwindcss/postcss postcss
```

```javascript
// postcss.config.js
export default {
  plugins: {
    '@tailwindcss/postcss': {}
  }
}
```

```css
/* src/styles/main.css */
@import 'tailwindcss';
```

---

## Nuxt 3 Integration

```bash
npm install tailwindcss @tailwindcss/vite
```

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  vite: {
    plugins: [
      require('@tailwindcss/vite').default
    ]
  },
  css: ['~/assets/css/main.css']
})
```

```css
/* assets/css/main.css */
@import 'tailwindcss';

@theme {
  --color-primary: oklch(55% 0.25 250);
}
```

---

## Content Detection

Tailwind v4 automatically detects content files. No configuration needed for:
- `.vue`, `.jsx`, `.tsx` files
- HTML files
- JavaScript/TypeScript files

### Custom Content Paths (if needed)

```css
/* main.css */
@import 'tailwindcss';

@source "../components/**/*.vue";
@source "../pages/**/*.vue";
```

---

## Build Optimization

### Production Build

```bash
npm run build
```

Tailwind v4 automatically:
- Removes unused CSS
- Minifies output
- Optimizes for production

### Analyze Bundle Size

```bash
npm install -D rollup-plugin-visualizer
```

```typescript
// vite.config.ts
import { visualizer } from 'rollup-plugin-visualizer'

export default defineConfig({
  plugins: [
    vue(),
    tailwindcss(),
    visualizer({ open: true })
  ]
})
```

---

## Development Tools

### Tailwind CSS IntelliSense (VSCode)

Install the official extension for:
- Autocomplete for utility classes
- Syntax highlighting for @theme
- Hover preview of CSS output
- Linting for class conflicts

### Browser DevTools

Tailwind v4 uses CSS variables, visible in DevTools:
- Inspect computed CSS variables
- Live edit theme values
- Debug responsive breakpoints

---

## Quick Reference

| Task | Command/File |
|------|--------------|
| Install | `npm install tailwindcss @tailwindcss/vite` |
| Vite plugin | `import tailwindcss from '@tailwindcss/vite'` |
| CSS import | `@import 'tailwindcss'` |
| Theme config | `@theme { ... }` in CSS |
| Custom source | `@source "path/**/*.vue"` |
| Build | `npm run build` (auto-optimized) |
