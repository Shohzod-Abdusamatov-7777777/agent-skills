# Utilities

> Reference for: Tailwind v4 Expert
> Load when: Spacing, typography, layout, flexbox, grid, borders, effects

## Spacing

### Padding & Margin

```html
<!-- Padding -->
<div class="p-4">All sides</div>
<div class="px-4">Horizontal (left + right)</div>
<div class="py-4">Vertical (top + bottom)</div>
<div class="pt-4 pr-4 pb-4 pl-4">Individual sides</div>

<!-- Margin -->
<div class="m-4">All sides</div>
<div class="mx-auto">Center horizontally</div>
<div class="my-8">Vertical margin</div>
<div class="mt-4 mb-8">Top and bottom</div>

<!-- Negative margin -->
<div class="-mt-4">Negative top margin</div>
<div class="-mx-2">Negative horizontal</div>

<!-- Space between children -->
<div class="space-y-4">Vertical gap</div>
<div class="space-x-4">Horizontal gap</div>
```

### Spacing Scale (Default)

| Class | Value | Pixels |
|-------|-------|--------|
| `p-0` | 0 | 0px |
| `p-1` | 0.25rem | 4px |
| `p-2` | 0.5rem | 8px |
| `p-3` | 0.75rem | 12px |
| `p-4` | 1rem | 16px |
| `p-5` | 1.25rem | 20px |
| `p-6` | 1.5rem | 24px |
| `p-8` | 2rem | 32px |
| `p-10` | 2.5rem | 40px |
| `p-12` | 3rem | 48px |
| `p-16` | 4rem | 64px |
| `p-20` | 5rem | 80px |
| `p-24` | 6rem | 96px |

---

## Typography

### Font Size

```html
<p class="text-xs">Extra small (0.75rem)</p>
<p class="text-sm">Small (0.875rem)</p>
<p class="text-base">Base (1rem)</p>
<p class="text-lg">Large (1.125rem)</p>
<p class="text-xl">Extra large (1.25rem)</p>
<p class="text-2xl">2XL (1.5rem)</p>
<p class="text-3xl">3XL (1.875rem)</p>
<p class="text-4xl">4XL (2.25rem)</p>
<p class="text-5xl">5XL (3rem)</p>
<p class="text-6xl">6XL (3.75rem)</p>
```

### Font Weight

```html
<p class="font-thin">100</p>
<p class="font-light">300</p>
<p class="font-normal">400</p>
<p class="font-medium">500</p>
<p class="font-semibold">600</p>
<p class="font-bold">700</p>
<p class="font-extrabold">800</p>
<p class="font-black">900</p>
```

### Text Alignment & Transform

```html
<p class="text-left">Left aligned</p>
<p class="text-center">Centered</p>
<p class="text-right">Right aligned</p>
<p class="text-justify">Justified</p>

<p class="uppercase">UPPERCASE</p>
<p class="lowercase">lowercase</p>
<p class="capitalize">Capitalize Each Word</p>
<p class="normal-case">Normal case</p>
```

### Line Height & Letter Spacing

```html
<p class="leading-none">Line height 1</p>
<p class="leading-tight">Line height 1.25</p>
<p class="leading-normal">Line height 1.5</p>
<p class="leading-relaxed">Line height 1.75</p>
<p class="leading-loose">Line height 2</p>

<p class="tracking-tighter">-0.05em</p>
<p class="tracking-tight">-0.025em</p>
<p class="tracking-normal">0</p>
<p class="tracking-wide">0.025em</p>
<p class="tracking-wider">0.05em</p>
<p class="tracking-widest">0.1em</p>
```

### Text Decoration & Color

```html
<p class="underline">Underlined</p>
<p class="line-through">Strikethrough</p>
<p class="no-underline">No underline</p>

<p class="text-primary">Primary color</p>
<p class="text-muted">Muted color</p>
<p class="text-foreground/80">80% opacity</p>
```

---

## Layout

### Display

```html
<div class="block">Block</div>
<span class="inline">Inline</span>
<div class="inline-block">Inline block</div>
<div class="flex">Flex container</div>
<div class="inline-flex">Inline flex</div>
<div class="grid">Grid container</div>
<div class="hidden">Hidden</div>
```

### Width & Height

```html
<!-- Fixed widths -->
<div class="w-4">1rem</div>
<div class="w-24">6rem</div>
<div class="w-64">16rem</div>
<div class="w-full">100%</div>
<div class="w-screen">100vw</div>

<!-- Percentage -->
<div class="w-1/2">50%</div>
<div class="w-1/3">33.333%</div>
<div class="w-2/3">66.666%</div>
<div class="w-1/4">25%</div>
<div class="w-3/4">75%</div>

<!-- Max width -->
<div class="max-w-sm">24rem</div>
<div class="max-w-md">28rem</div>
<div class="max-w-lg">32rem</div>
<div class="max-w-xl">36rem</div>
<div class="max-w-2xl">42rem</div>
<div class="max-w-screen-xl">1280px</div>

<!-- Height -->
<div class="h-screen">100vh</div>
<div class="min-h-screen">min-height: 100vh</div>
<div class="h-full">100%</div>
```

### Position

```html
<div class="relative">Relative</div>
<div class="absolute">Absolute</div>
<div class="fixed">Fixed</div>
<div class="sticky">Sticky</div>

<!-- Positioning -->
<div class="absolute top-0 left-0">Top left</div>
<div class="absolute top-0 right-0">Top right</div>
<div class="absolute bottom-0 left-0">Bottom left</div>
<div class="absolute inset-0">All edges = 0</div>
<div class="absolute inset-x-0 top-0">Full width at top</div>

<!-- Z-index -->
<div class="z-0">z-index: 0</div>
<div class="z-10">z-index: 10</div>
<div class="z-50">z-index: 50</div>
```

---

## Flexbox

```html
<!-- Container -->
<div class="flex">Flex row (default)</div>
<div class="flex flex-col">Flex column</div>
<div class="flex flex-row-reverse">Row reverse</div>
<div class="flex flex-col-reverse">Column reverse</div>

<!-- Wrap -->
<div class="flex flex-wrap">Wrap</div>
<div class="flex flex-nowrap">No wrap</div>

<!-- Justify (main axis) -->
<div class="flex justify-start">Start</div>
<div class="flex justify-center">Center</div>
<div class="flex justify-end">End</div>
<div class="flex justify-between">Space between</div>
<div class="flex justify-around">Space around</div>
<div class="flex justify-evenly">Space evenly</div>

<!-- Align (cross axis) -->
<div class="flex items-start">Start</div>
<div class="flex items-center">Center</div>
<div class="flex items-end">End</div>
<div class="flex items-stretch">Stretch</div>
<div class="flex items-baseline">Baseline</div>

<!-- Gap -->
<div class="flex gap-4">Gap all</div>
<div class="flex gap-x-4 gap-y-2">Different x/y gap</div>

<!-- Flex items -->
<div class="flex-1">Grow and shrink</div>
<div class="flex-none">Don't grow or shrink</div>
<div class="flex-grow">Grow only</div>
<div class="flex-shrink-0">Don't shrink</div>
```

---

## Grid

```html
<!-- Columns -->
<div class="grid grid-cols-1">1 column</div>
<div class="grid grid-cols-2">2 columns</div>
<div class="grid grid-cols-3">3 columns</div>
<div class="grid grid-cols-4">4 columns</div>
<div class="grid grid-cols-12">12 columns</div>

<!-- Responsive grid -->
<div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3">
  Responsive columns
</div>

<!-- Gap -->
<div class="grid grid-cols-3 gap-4">Gap all</div>
<div class="grid grid-cols-3 gap-x-4 gap-y-8">Different x/y gap</div>

<!-- Span -->
<div class="col-span-2">Span 2 columns</div>
<div class="col-span-full">Span all columns</div>
<div class="col-start-2 col-end-4">Start col 2, end col 4</div>

<!-- Rows -->
<div class="grid grid-rows-3">3 rows</div>
<div class="row-span-2">Span 2 rows</div>

<!-- Auto columns/rows -->
<div class="grid auto-cols-fr">Auto columns with fr</div>
<div class="grid auto-rows-min">Auto rows min-content</div>

<!-- Place items -->
<div class="grid place-items-center">Center both axes</div>
<div class="grid place-content-center">Center content</div>
```

---

## Borders

```html
<!-- Border width -->
<div class="border">1px border</div>
<div class="border-2">2px border</div>
<div class="border-4">4px border</div>
<div class="border-t">Top only</div>
<div class="border-r">Right only</div>
<div class="border-b">Bottom only</div>
<div class="border-l">Left only</div>
<div class="border-x">Left + right</div>
<div class="border-y">Top + bottom</div>

<!-- Border color -->
<div class="border border-gray-300">Gray border</div>
<div class="border border-primary">Primary border</div>
<div class="border border-transparent">Transparent</div>

<!-- Border radius -->
<div class="rounded-none">0</div>
<div class="rounded-sm">0.125rem</div>
<div class="rounded">0.25rem</div>
<div class="rounded-md">0.375rem</div>
<div class="rounded-lg">0.5rem</div>
<div class="rounded-xl">0.75rem</div>
<div class="rounded-2xl">1rem</div>
<div class="rounded-full">9999px</div>

<!-- Specific corners -->
<div class="rounded-t-lg">Top corners</div>
<div class="rounded-b-lg">Bottom corners</div>
<div class="rounded-tl-lg">Top-left only</div>
```

---

## Effects

### Shadows

```html
<div class="shadow-sm">Small shadow</div>
<div class="shadow">Default shadow</div>
<div class="shadow-md">Medium shadow</div>
<div class="shadow-lg">Large shadow</div>
<div class="shadow-xl">Extra large</div>
<div class="shadow-2xl">2XL shadow</div>
<div class="shadow-inner">Inner shadow</div>
<div class="shadow-none">No shadow</div>
```

### Opacity

```html
<div class="opacity-0">0%</div>
<div class="opacity-25">25%</div>
<div class="opacity-50">50%</div>
<div class="opacity-75">75%</div>
<div class="opacity-100">100%</div>
```

### Overflow

```html
<div class="overflow-auto">Auto scrollbars</div>
<div class="overflow-hidden">Hidden</div>
<div class="overflow-visible">Visible</div>
<div class="overflow-scroll">Always scroll</div>
<div class="overflow-x-auto">Horizontal auto</div>
<div class="overflow-y-auto">Vertical auto</div>
```

### Cursor

```html
<div class="cursor-pointer">Pointer</div>
<div class="cursor-default">Default</div>
<div class="cursor-not-allowed">Not allowed</div>
<div class="cursor-wait">Wait</div>
<div class="cursor-grab">Grab</div>
```

---

## Quick Reference

| Category | Common Classes |
|----------|---------------|
| Spacing | `p-4`, `m-4`, `px-4`, `my-auto`, `space-y-4`, `gap-4` |
| Typography | `text-lg`, `font-bold`, `text-center`, `leading-relaxed` |
| Layout | `flex`, `grid`, `block`, `hidden`, `w-full`, `h-screen` |
| Flexbox | `flex-col`, `justify-center`, `items-center`, `gap-4` |
| Grid | `grid-cols-3`, `col-span-2`, `gap-4`, `place-items-center` |
| Borders | `border`, `rounded-lg`, `border-gray-300` |
| Effects | `shadow-lg`, `opacity-50`, `overflow-hidden` |
