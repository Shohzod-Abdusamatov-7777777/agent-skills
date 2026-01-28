# Responsive & Dark Mode

> Reference for: Tailwind v4 Expert
> Load when: Breakpoints, dark mode, variants, media queries, container queries

## Responsive Breakpoints

### Default Breakpoints

| Prefix | Min Width | CSS |
|--------|-----------|-----|
| `sm:` | 640px | `@media (min-width: 640px)` |
| `md:` | 768px | `@media (min-width: 768px)` |
| `lg:` | 1024px | `@media (min-width: 1024px)` |
| `xl:` | 1280px | `@media (min-width: 1280px)` |
| `2xl:` | 1536px | `@media (min-width: 1536px)` |

### Mobile-First Approach

```html
<!-- Base styles for mobile, then add for larger screens -->
<div class="
  w-full          /* Mobile: full width */
  md:w-1/2        /* Tablet: half width */
  lg:w-1/3        /* Desktop: third width */
">
  Responsive width
</div>

<div class="
  flex flex-col     /* Mobile: stack vertically */
  md:flex-row       /* Tablet+: horizontal row */
">
  Responsive layout
</div>

<h1 class="
  text-2xl          /* Mobile: smaller */
  md:text-3xl       /* Tablet: medium */
  lg:text-4xl       /* Desktop: larger */
">
  Responsive text
</h1>
```

### Custom Breakpoints

```css
@theme {
  --breakpoint-xs: 475px;
  --breakpoint-sm: 640px;
  --breakpoint-md: 768px;
  --breakpoint-lg: 1024px;
  --breakpoint-xl: 1280px;
  --breakpoint-2xl: 1536px;
  --breakpoint-3xl: 1920px;
}
```

```html
<div class="xs:flex 3xl:hidden">Custom breakpoints</div>
```

### Max-Width Breakpoints

```html
<!-- Tailwind v4 supports max-width with 'max-' prefix -->
<div class="max-md:hidden">Hidden on mobile (< 768px)</div>
<div class="max-lg:flex-col">Column on mobile/tablet</div>
```

### Range Breakpoints

```html
<!-- Only between md and lg -->
<div class="md:max-lg:bg-blue-500">Blue only on tablets</div>
```

---

## Dark Mode

### Setup Options

#### 1. Media Query (Automatic)

Follows system preference automatically.

```css
@import 'tailwindcss';

@theme {
  /* Light mode colors (default) */
  --color-background: oklch(100% 0 0);
  --color-foreground: oklch(15% 0.02 250);
}

/* Dark mode overrides */
@media (prefers-color-scheme: dark) {
  :root {
    --color-background: oklch(15% 0.02 250);
    --color-foreground: oklch(95% 0.01 250);
  }
}
```

#### 2. Class-Based (Manual Toggle)

User can toggle dark mode.

```css
@import 'tailwindcss';

@theme {
  --color-background: oklch(100% 0 0);
  --color-foreground: oklch(15% 0.02 250);
}

/* Dark mode when .dark class is on html/body */
.dark {
  --color-background: oklch(15% 0.02 250);
  --color-foreground: oklch(95% 0.01 250);
}
```

```html
<html class="dark">
  <body class="bg-background text-foreground">
    <!-- Content uses CSS variables, automatically dark -->
  </body>
</html>
```

#### 3. Using dark: Variant

```html
<div class="
  bg-white dark:bg-gray-900
  text-gray-900 dark:text-white
  border-gray-200 dark:border-gray-700
">
  Dark mode with utilities
</div>

<button class="
  bg-blue-500 hover:bg-blue-600
  dark:bg-blue-600 dark:hover:bg-blue-500
">
  Button with dark mode
</button>
```

### Dark Mode Toggle (Vue)

```vue
<script setup lang="ts">
import { ref, onMounted } from 'vue'

const isDark = ref(false)

onMounted(() => {
  // Check saved preference or system preference
  isDark.value = localStorage.getItem('theme') === 'dark' ||
    (!localStorage.getItem('theme') && window.matchMedia('(prefers-color-scheme: dark)').matches)
  updateTheme()
})

function toggleDark() {
  isDark.value = !isDark.value
  localStorage.setItem('theme', isDark.value ? 'dark' : 'light')
  updateTheme()
}

function updateTheme() {
  document.documentElement.classList.toggle('dark', isDark.value)
}
</script>

<template>
  <button @click="toggleDark" class="p-2 rounded-lg bg-surface">
    <span v-if="isDark">🌙</span>
    <span v-else>☀️</span>
  </button>
</template>
```

---

## State Variants

### Hover, Focus, Active

```html
<button class="
  bg-primary
  hover:bg-primary-dark
  focus:ring-2 focus:ring-primary/50
  active:scale-95
">
  Interactive button
</button>

<input class="
  border border-gray-300
  hover:border-gray-400
  focus:border-primary focus:ring-2 focus:ring-primary/20
  focus:outline-none
"/>

<a class="
  text-primary
  hover:text-primary-dark
  hover:underline
">
  Link
</a>
```

### Focus Variants

```html
<input class="
  focus:outline-none       /* Remove default outline */
  focus:ring-2             /* Add ring */
  focus:ring-primary       /* Ring color */
  focus:ring-offset-2      /* Ring offset */
  focus-visible:ring-2     /* Only keyboard focus */
  focus-within:ring-2      /* When child is focused */
"/>
```

### Group & Peer

```html
<!-- Group: parent hover affects children -->
<div class="group p-4 border rounded-lg hover:border-primary">
  <h3 class="group-hover:text-primary">Title</h3>
  <p class="group-hover:text-muted">Description</p>
  <span class="opacity-0 group-hover:opacity-100">→</span>
</div>

<!-- Peer: sibling state affects element -->
<input type="checkbox" class="peer" />
<label class="peer-checked:text-primary peer-checked:font-bold">
  Label changes when checkbox is checked
</label>

<input class="peer" placeholder="Email" />
<p class="hidden peer-invalid:block text-error">
  Please enter a valid email
</p>
```

### Disabled & Checked States

```html
<button class="
  disabled:opacity-50
  disabled:cursor-not-allowed
  disabled:bg-gray-300
" disabled>
  Disabled button
</button>

<input type="checkbox" class="
  checked:bg-primary
  checked:border-primary
"/>

<input type="radio" class="
  checked:bg-primary
  checked:border-primary
"/>
```

### First, Last, Odd, Even

```html
<ul>
  <li class="first:pt-0 last:pb-0 py-2 border-b last:border-0">Item 1</li>
  <li class="first:pt-0 last:pb-0 py-2 border-b last:border-0">Item 2</li>
  <li class="first:pt-0 last:pb-0 py-2 border-b last:border-0">Item 3</li>
</ul>

<table>
  <tr class="odd:bg-gray-50 even:bg-white">...</tr>
  <tr class="odd:bg-gray-50 even:bg-white">...</tr>
</table>
```

---

## Container Queries (New in v4)

Container queries allow styling based on parent container size, not viewport.

```html
<!-- Define container -->
<div class="@container">
  <!-- Style based on container width -->
  <div class="@md:flex @lg:grid @lg:grid-cols-2">
    <div class="@sm:text-lg @md:text-xl">
      Responsive to container, not viewport
    </div>
  </div>
</div>
```

### Named Containers

```html
<div class="@container/sidebar">
  <div class="@md/sidebar:flex">
    Only responds to sidebar container
  </div>
</div>

<div class="@container/main">
  <div class="@lg/main:grid">
    Only responds to main container
  </div>
</div>
```

---

## Print Styles

```html
<div class="print:hidden">
  Hidden when printing
</div>

<div class="hidden print:block">
  Only visible when printing
</div>

<div class="bg-white print:bg-transparent">
  No background when printing
</div>
```

---

## Motion & Accessibility

### Reduced Motion

```html
<div class="
  transition-transform duration-300
  motion-reduce:transition-none
  motion-reduce:transform-none
">
  Respects reduced motion preference
</div>

<div class="
  animate-bounce
  motion-safe:animate-bounce
  motion-reduce:animate-none
">
  Bounces only if motion is safe
</div>
```

### Contrast & Accessibility

```html
<div class="
  contrast-more:border-2
  contrast-more:border-black
">
  Higher contrast when requested
</div>
```

---

## Responsive Component Example

```html
<!-- Card that adapts to screen size -->
<article class="
  /* Base (mobile) */
  p-4
  flex flex-col
  gap-4
  bg-surface
  rounded-lg
  shadow-sm

  /* Tablet */
  md:p-6
  md:flex-row
  md:gap-6

  /* Desktop */
  lg:p-8

  /* Dark mode */
  dark:bg-surface-dark
  dark:shadow-lg
">
  <img class="
    w-full h-48 object-cover rounded-lg
    md:w-48 md:h-auto
    lg:w-64
  " src="..." alt="...">

  <div class="flex flex-col gap-2">
    <h2 class="
      text-xl font-bold
      md:text-2xl
      lg:text-3xl
    ">
      Card Title
    </h2>
    <p class="
      text-muted
      text-sm
      md:text-base
    ">
      Card description that adapts to screen size.
    </p>
  </div>
</article>
```

---

## Quick Reference

| Variant | Description |
|---------|-------------|
| `sm:`, `md:`, `lg:`, `xl:`, `2xl:` | Min-width breakpoints |
| `max-sm:`, `max-md:` | Max-width breakpoints |
| `dark:` | Dark mode |
| `hover:`, `focus:`, `active:` | Interactive states |
| `group-hover:`, `peer-checked:` | Relative states |
| `first:`, `last:`, `odd:`, `even:` | Child position |
| `disabled:`, `checked:`, `invalid:` | Form states |
| `@container`, `@md:` | Container queries |
| `print:` | Print styles |
| `motion-reduce:`, `motion-safe:` | Motion preferences |
