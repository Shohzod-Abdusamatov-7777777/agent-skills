# Animations

> Reference for: Tailwind v4 Expert
> Load when: Keyframes, transitions, custom animations, motion

## Transitions

### Basic Transitions

```html
<!-- Transition all properties -->
<div class="transition">Transition all</div>

<!-- Specific properties -->
<div class="transition-colors">Colors only</div>
<div class="transition-opacity">Opacity only</div>
<div class="transition-shadow">Shadow only</div>
<div class="transition-transform">Transform only</div>

<!-- Combined -->
<button class="
  bg-primary text-white
  hover:bg-primary-dark
  transition-colors duration-200
">
  Hover me
</button>
```

### Duration

```html
<div class="transition duration-75">75ms</div>
<div class="transition duration-100">100ms</div>
<div class="transition duration-150">150ms (default)</div>
<div class="transition duration-200">200ms</div>
<div class="transition duration-300">300ms</div>
<div class="transition duration-500">500ms</div>
<div class="transition duration-700">700ms</div>
<div class="transition duration-1000">1000ms</div>
```

### Timing Functions

```html
<div class="transition ease-linear">Linear</div>
<div class="transition ease-in">Ease in</div>
<div class="transition ease-out">Ease out</div>
<div class="transition ease-in-out">Ease in-out</div>
```

### Delay

```html
<div class="transition delay-75">75ms delay</div>
<div class="transition delay-100">100ms delay</div>
<div class="transition delay-150">150ms delay</div>
<div class="transition delay-200">200ms delay</div>
<div class="transition delay-300">300ms delay</div>
<div class="transition delay-500">500ms delay</div>
```

---

## Built-in Animations

### Spin

```html
<svg class="animate-spin h-5 w-5">
  <!-- Loading spinner -->
</svg>

<button class="flex items-center gap-2" disabled>
  <svg class="animate-spin h-4 w-4">...</svg>
  Loading...
</button>
```

### Ping

```html
<!-- Notification badge with ping effect -->
<span class="relative flex h-3 w-3">
  <span class="animate-ping absolute inline-flex h-full w-full rounded-full bg-primary opacity-75"></span>
  <span class="relative inline-flex rounded-full h-3 w-3 bg-primary"></span>
</span>
```

### Pulse

```html
<!-- Skeleton loading -->
<div class="animate-pulse">
  <div class="h-4 bg-gray-300 rounded w-3/4 mb-2"></div>
  <div class="h-4 bg-gray-300 rounded w-1/2"></div>
</div>
```

### Bounce

```html
<!-- Attention indicator -->
<div class="animate-bounce">
  ↓ Scroll down
</div>
```

---

## Custom Animations in @theme

### Define Keyframes

```css
@theme inline {
  /* Fade in animation */
  @keyframes fade-in {
    from {
      opacity: 0;
    }
    to {
      opacity: 1;
    }
  }
  --animate-fade-in: fade-in 0.3s ease-out;

  /* Fade in up */
  @keyframes fade-in-up {
    from {
      opacity: 0;
      transform: translateY(10px);
    }
    to {
      opacity: 1;
      transform: translateY(0);
    }
  }
  --animate-fade-in-up: fade-in-up 0.4s ease-out;

  /* Fade in down */
  @keyframes fade-in-down {
    from {
      opacity: 0;
      transform: translateY(-10px);
    }
    to {
      opacity: 1;
      transform: translateY(0);
    }
  }
  --animate-fade-in-down: fade-in-down 0.4s ease-out;

  /* Scale in */
  @keyframes scale-in {
    from {
      opacity: 0;
      transform: scale(0.95);
    }
    to {
      opacity: 1;
      transform: scale(1);
    }
  }
  --animate-scale-in: scale-in 0.2s ease-out;

  /* Slide in from right */
  @keyframes slide-in-right {
    from {
      transform: translateX(100%);
    }
    to {
      transform: translateX(0);
    }
  }
  --animate-slide-in-right: slide-in-right 0.3s ease-out;

  /* Slide in from left */
  @keyframes slide-in-left {
    from {
      transform: translateX(-100%);
    }
    to {
      transform: translateX(0);
    }
  }
  --animate-slide-in-left: slide-in-left 0.3s ease-out;

  /* Shake */
  @keyframes shake {
    0%, 100% { transform: translateX(0); }
    25% { transform: translateX(-5px); }
    75% { transform: translateX(5px); }
  }
  --animate-shake: shake 0.5s ease-in-out;

  /* Wiggle */
  @keyframes wiggle {
    0%, 100% { transform: rotate(0deg); }
    25% { transform: rotate(-3deg); }
    75% { transform: rotate(3deg); }
  }
  --animate-wiggle: wiggle 0.3s ease-in-out;
}
```

### Use Custom Animations

```html
<div class="animate-fade-in">Fades in</div>
<div class="animate-fade-in-up">Fades in from below</div>
<div class="animate-scale-in">Scales in</div>
<div class="animate-slide-in-right">Slides from right</div>
<div class="animate-shake">Shakes on error</div>
```

---

## Transform Utilities

### Scale

```html
<div class="scale-50">50%</div>
<div class="scale-75">75%</div>
<div class="scale-90">90%</div>
<div class="scale-95">95%</div>
<div class="scale-100">100% (default)</div>
<div class="scale-105">105%</div>
<div class="scale-110">110%</div>
<div class="scale-125">125%</div>
<div class="scale-150">150%</div>

<!-- Axis-specific -->
<div class="scale-x-50">Horizontal 50%</div>
<div class="scale-y-150">Vertical 150%</div>

<!-- Hover effect -->
<div class="hover:scale-105 transition-transform">
  Grows on hover
</div>
```

### Rotate

```html
<div class="rotate-0">0deg</div>
<div class="rotate-45">45deg</div>
<div class="rotate-90">90deg</div>
<div class="rotate-180">180deg</div>
<div class="-rotate-45">-45deg</div>
<div class="-rotate-90">-90deg</div>

<!-- Hover effect -->
<div class="hover:rotate-12 transition-transform">
  Rotates on hover
</div>
```

### Translate

```html
<div class="translate-x-4">Move right 1rem</div>
<div class="translate-y-4">Move down 1rem</div>
<div class="-translate-x-4">Move left 1rem</div>
<div class="-translate-y-4">Move up 1rem</div>
<div class="translate-x-1/2">Move right 50%</div>
<div class="-translate-y-full">Move up 100%</div>

<!-- Center absolutely positioned element -->
<div class="absolute top-1/2 left-1/2 -translate-x-1/2 -translate-y-1/2">
  Centered
</div>
```

### Skew

```html
<div class="skew-x-3">Skew X 3deg</div>
<div class="skew-y-6">Skew Y 6deg</div>
<div class="-skew-x-3">Skew X -3deg</div>
```

### Transform Origin

```html
<div class="origin-center">Center (default)</div>
<div class="origin-top">Top</div>
<div class="origin-top-right">Top right</div>
<div class="origin-right">Right</div>
<div class="origin-bottom-right">Bottom right</div>
<div class="origin-bottom">Bottom</div>
<div class="origin-bottom-left">Bottom left</div>
<div class="origin-left">Left</div>
<div class="origin-top-left">Top left</div>
```

---

## Animation Examples

### Button with Hover Effect

```html
<button class="
  px-6 py-3
  bg-primary text-white
  rounded-lg
  transform
  transition-all duration-200
  hover:bg-primary-dark
  hover:scale-105
  hover:shadow-lg
  active:scale-95
">
  Click me
</button>
```

### Card with Hover Animation

```html
<div class="
  p-6 bg-surface rounded-xl shadow
  transform transition-all duration-300
  hover:-translate-y-1
  hover:shadow-xl
">
  <h3>Card Title</h3>
  <p>Card content with smooth hover effect.</p>
</div>
```

### Loading Skeleton

```html
<div class="animate-pulse space-y-4">
  <div class="h-4 bg-gray-300 rounded w-3/4"></div>
  <div class="h-4 bg-gray-300 rounded"></div>
  <div class="h-4 bg-gray-300 rounded w-5/6"></div>
  <div class="flex gap-4">
    <div class="h-12 w-12 bg-gray-300 rounded-full"></div>
    <div class="flex-1 space-y-2">
      <div class="h-4 bg-gray-300 rounded"></div>
      <div class="h-4 bg-gray-300 rounded w-2/3"></div>
    </div>
  </div>
</div>
```

### Dropdown Menu Animation

```html
<!-- Closed state -->
<div class="
  opacity-0
  scale-95
  pointer-events-none
  transition-all duration-200
">
  Menu content
</div>

<!-- Open state -->
<div class="
  opacity-100
  scale-100
  pointer-events-auto
  transition-all duration-200
">
  Menu content
</div>
```

### Modal Animation

```css
@theme inline {
  @keyframes modal-in {
    from {
      opacity: 0;
      transform: scale(0.95) translateY(-10px);
    }
    to {
      opacity: 1;
      transform: scale(1) translateY(0);
    }
  }
  --animate-modal-in: modal-in 0.2s ease-out;

  @keyframes backdrop-in {
    from { opacity: 0; }
    to { opacity: 1; }
  }
  --animate-backdrop-in: backdrop-in 0.2s ease-out;
}
```

```html
<div class="fixed inset-0 bg-black/50 animate-backdrop-in">
  <div class="bg-white rounded-xl p-6 animate-modal-in">
    Modal content
  </div>
</div>
```

---

## Respecting Motion Preferences

```html
<!-- Disable animations for users who prefer reduced motion -->
<div class="
  transition-transform duration-300
  hover:scale-105
  motion-reduce:transition-none
  motion-reduce:hover:scale-100
">
  Respects user preferences
</div>

<!-- Only animate when motion is OK -->
<div class="
  motion-safe:animate-bounce
  motion-reduce:animate-none
">
  Bounces only if safe
</div>
```

---

## Quick Reference

| Category | Classes |
|----------|---------|
| Transitions | `transition`, `transition-colors`, `duration-200`, `ease-in-out`, `delay-100` |
| Built-in | `animate-spin`, `animate-ping`, `animate-pulse`, `animate-bounce` |
| Scale | `scale-95`, `scale-100`, `scale-105`, `hover:scale-110` |
| Rotate | `rotate-45`, `rotate-90`, `-rotate-45`, `hover:rotate-6` |
| Translate | `translate-x-4`, `-translate-y-2`, `translate-x-1/2` |
| Origin | `origin-center`, `origin-top`, `origin-bottom-left` |
| Custom | Define in `@theme inline { @keyframes ... }` |
| Motion | `motion-reduce:`, `motion-safe:` |
