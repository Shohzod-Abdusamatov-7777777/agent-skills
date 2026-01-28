# VueUse

> Reference for: Vue Expert
> Load when: Using VueUse composables, utility functions, browser APIs, sensors, animations

## Installation

```bash
npm install @vueuse/core
# Optional packages
npm install @vueuse/integrations  # Third-party integrations
npm install @vueuse/motion        # Animation utilities
npm install @vueuse/head          # Document head management
```

## Core Utilities

### State Management

```typescript
import {
  useStorage,
  useLocalStorage,
  useSessionStorage,
  useRefHistory,
  useDebouncedRef,
  useThrottledRef
} from '@vueuse/core'

// Persistent state in localStorage
const user = useLocalStorage('user', { name: '', email: '' })
user.value.name = 'John' // Auto-saves to localStorage

// Session storage
const token = useSessionStorage('token', '')

// Generic storage (auto-detects localStorage/sessionStorage)
const settings = useStorage('settings', { theme: 'dark' }, localStorage)

// Track ref history with undo/redo
const counter = ref(0)
const { history, undo, redo, canUndo, canRedo } = useRefHistory(counter, {
  capacity: 10 // Keep last 10 states
})

// Debounced ref - delays updates
const search = ref('')
const debouncedSearch = useDebouncedRef(search, 300)

// Throttled ref - limits update frequency
const scroll = ref(0)
const throttledScroll = useThrottledRef(scroll, 100)
```

### Computed Utilities

```typescript
import {
  computedAsync,
  computedEager,
  computedWithControl,
  reactiveComputed
} from '@vueuse/core'

// Async computed with loading state
const userId = ref(1)
const { state: user, isLoading } = computedAsync(
  async () => {
    const response = await fetch(`/api/users/${userId.value}`)
    return response.json()
  },
  null, // Initial value
  { lazy: true }
)

// Eager computed - updates synchronously (bypasses Vue's lazy evaluation)
const count = ref(0)
const doubled = computedEager(() => count.value * 2)

// Computed with manual trigger control
const source = ref('hello')
const { result, trigger } = computedWithControl(
  () => source.value, // Dependencies
  () => source.value.toUpperCase() // Computation
)
// result.value won't update until trigger() is called
```

### Reactivity Utilities

```typescript
import {
  syncRef,
  syncRefs,
  toReactive,
  toRefs,
  reactiveOmit,
  reactivePick,
  createGlobalState,
  createSharedComposable
} from '@vueuse/core'

// Two-way sync between refs
const a = ref(1)
const b = ref(2)
syncRef(a, b) // Changes to a reflect in b and vice versa

// One-way sync (multiple sources to one target)
const source1 = ref('a')
const source2 = ref('b')
const target = ref('')
syncRefs([source1, source2], target)

// Convert ref to reactive object
const refObj = ref({ name: 'John', age: 30 })
const reactiveObj = toReactive(refObj)
console.log(reactiveObj.name) // No .value needed

// Pick/omit reactive properties
const state = reactive({ name: 'John', age: 30, email: 'john@example.com' })
const picked = reactivePick(state, 'name', 'email')
const omitted = reactiveOmit(state, 'email')

// Global state (singleton across components)
const useGlobalCounter = createGlobalState(() => {
  const count = ref(0)
  const increment = () => count.value++
  return { count, increment }
})

// Shared composable (reuses same instance)
const useSharedMouse = createSharedComposable(useMouse)
```

## Browser APIs

### DOM & Elements

```typescript
import {
  useElementSize,
  useElementBounding,
  useElementVisibility,
  useIntersectionObserver,
  useMutationObserver,
  useResizeObserver,
  useWindowSize,
  useWindowScroll,
  useScroll
} from '@vueuse/core'

// Element dimensions
const el = ref<HTMLElement | null>(null)
const { width, height } = useElementSize(el)

// Element position and size
const { x, y, top, left, right, bottom, width, height } = useElementBounding(el)

// Visibility detection
const isVisible = useElementVisibility(el)

// Intersection observer
const target = ref<HTMLElement | null>(null)
const { isIntersecting, stop } = useIntersectionObserver(
  target,
  ([entry]) => {
    console.log('Visibility:', entry.isIntersecting)
  },
  { threshold: 0.5 }
)

// Window dimensions
const { width: windowWidth, height: windowHeight } = useWindowSize()

// Window scroll position
const { x: scrollX, y: scrollY } = useWindowScroll()

// Element scroll
const scrollEl = ref<HTMLElement | null>(null)
const { x, y, isScrolling, arrivedState, directions } = useScroll(scrollEl)
// arrivedState: { left, right, top, bottom }
// directions: { left, right, top, bottom }
```

### Events & Input

```typescript
import {
  useEventListener,
  useMouse,
  useMousePressed,
  useKeyModifier,
  useMagicKeys,
  onClickOutside,
  onKeyStroke,
  useFocus,
  useFocusWithin
} from '@vueuse/core'

// Generic event listener (auto-cleanup)
const el = ref<HTMLElement | null>(null)
useEventListener(el, 'click', (event) => {
  console.log('Clicked!', event)
})
useEventListener(window, 'resize', () => console.log('Resized'))

// Mouse position
const { x, y } = useMouse()

// Mouse pressed state
const { pressed } = useMousePressed()

// Keyboard modifiers
const shift = useKeyModifier('Shift')
const ctrl = useKeyModifier('Control')

// Magic keys - reactive key states
const { current, ctrl_s, escape, arrowUp } = useMagicKeys()
whenever(ctrl_s, () => {
  console.log('Ctrl+S pressed')
})

// Click outside detection
const modal = ref<HTMLElement | null>(null)
onClickOutside(modal, () => {
  closeModal()
})

// Key stroke handler
onKeyStroke('Escape', () => closeModal())
onKeyStroke(['w', 'W'], () => moveUp())
onKeyStroke('ArrowUp', (e) => {
  e.preventDefault()
  moveUp()
})

// Focus management
const input = ref<HTMLInputElement | null>(null)
const { focused } = useFocus(input, { initialValue: true })

// Focus within container
const container = ref<HTMLElement | null>(null)
const { focused: focusedWithin } = useFocusWithin(container)
```

### Clipboard & Media

```typescript
import {
  useClipboard,
  usePermission,
  useMediaQuery,
  usePreferredDark,
  usePreferredLanguages,
  useColorMode,
  useDark
} from '@vueuse/core'

// Clipboard
const { text, copy, copied, isSupported } = useClipboard()
async function copyText() {
  await copy('Hello World')
  if (copied.value) {
    console.log('Copied!')
  }
}

// Permission status
const micPermission = usePermission('microphone')
// micPermission.value: 'granted' | 'denied' | 'prompt'

// Media queries
const isLargeScreen = useMediaQuery('(min-width: 1024px)')
const prefersDark = usePreferredDark()
const languages = usePreferredLanguages()

// Color mode management
const mode = useColorMode({
  attribute: 'data-theme',
  modes: {
    light: 'light',
    dark: 'dark',
    auto: 'auto'
  }
})

// Simple dark mode toggle
const isDark = useDark()
function toggleDark() {
  isDark.value = !isDark.value
}
```

## Network & Data

### Fetch & API

```typescript
import {
  useFetch,
  useAsyncState,
  useLastChanged,
  watchDebounced,
  watchThrottled
} from '@vueuse/core'

// Basic fetch
const { data, error, isFetching, execute } = useFetch('/api/users')

// Fetch with options
const { data: user } = useFetch('/api/users/1', {
  refetch: true, // Auto-refetch when URL changes
  beforeFetch({ url, options, cancel }) {
    const token = localStorage.getItem('token')
    if (!token) cancel()
    options.headers = {
      ...options.headers,
      Authorization: `Bearer ${token}`
    }
    return { options }
  },
  afterFetch(ctx) {
    ctx.data = ctx.data.map(transformUser)
    return ctx
  },
  onFetchError(ctx) {
    if (ctx.response?.status === 401) {
      router.push('/login')
    }
    return ctx
  }
}).json()

// POST request
const { post } = useFetch('/api/users').post().json()
await post({ name: 'John', email: 'john@example.com' })

// Async state with loading
const { state, isReady, isLoading, error, execute } = useAsyncState(
  async () => {
    const response = await api.getUsers()
    return response.data
  },
  [], // Initial state
  { immediate: true, resetOnExecute: false }
)

// Track last changed timestamp
const data = ref({ name: 'John' })
const lastChanged = useLastChanged(data)

// Debounced watch
const search = ref('')
watchDebounced(
  search,
  async (value) => {
    const results = await api.search(value)
  },
  { debounce: 300 }
)

// Throttled watch
watchThrottled(
  scrollY,
  (value) => console.log('Scroll:', value),
  { throttle: 100 }
)
```

### WebSocket

```typescript
import { useWebSocket } from '@vueuse/core'

const {
  status,
  data,
  send,
  open,
  close
} = useWebSocket('wss://api.example.com/ws', {
  autoReconnect: {
    retries: 3,
    delay: 1000,
    onFailed() {
      console.log('Failed to reconnect')
    }
  },
  heartbeat: {
    message: 'ping',
    interval: 30000,
    pongTimeout: 5000
  },
  onConnected(ws) {
    console.log('Connected')
    send(JSON.stringify({ type: 'subscribe', channel: 'updates' }))
  },
  onMessage(ws, event) {
    const message = JSON.parse(event.data)
    console.log('Received:', message)
  },
  onDisconnected(ws, event) {
    console.log('Disconnected:', event.code)
  },
  onError(ws, event) {
    console.error('WebSocket error:', event)
  }
})

// status: 'CONNECTING' | 'OPEN' | 'CLOSING' | 'CLOSED'
```

## Sensors

### Device Sensors

```typescript
import {
  useDeviceMotion,
  useDeviceOrientation,
  useGeolocation,
  useBattery,
  useNetwork,
  useOnline,
  useVibrate
} from '@vueuse/core'

// Device motion (accelerometer)
const { acceleration, accelerationIncludingGravity, rotationRate } = useDeviceMotion()

// Device orientation
const { alpha, beta, gamma, absolute } = useDeviceOrientation()

// Geolocation
const { coords, locatedAt, error, resume, pause } = useGeolocation({
  enableHighAccuracy: true,
  maximumAge: 30000,
  timeout: 27000
})
// coords: { latitude, longitude, altitude, accuracy, ... }

// Battery status
const { charging, chargingTime, dischargingTime, level } = useBattery()

// Network status
const { isOnline, offlineAt, downlink, effectiveType, type } = useNetwork()
// effectiveType: 'slow-2g' | '2g' | '3g' | '4g'

// Simple online check
const online = useOnline()

// Vibration
const { vibrate, stop, isSupported } = useVibrate({ pattern: [100, 200, 100] })
```

### User Activity

```typescript
import {
  useIdle,
  usePageLeave,
  useDocumentVisibility,
  useActiveElement
} from '@vueuse/core'

// Idle detection
const { idle, lastActive } = useIdle(5 * 60 * 1000) // 5 minutes
watch(idle, (isIdle) => {
  if (isIdle) {
    console.log('User is idle')
  }
})

// Page leave detection
const isLeft = usePageLeave()

// Document visibility
const visibility = useDocumentVisibility()
watch(visibility, (state) => {
  if (state === 'hidden') {
    pauseVideo()
  }
})

// Active element tracking
const activeElement = useActiveElement()
```

## Animations & Timing

```typescript
import {
  useInterval,
  useTimeout,
  useTimestamp,
  useNow,
  useDateFormat,
  useRafFn,
  useTransition
} from '@vueuse/core'

// Interval with controls
const { counter, pause, resume, isActive } = useInterval(1000, {
  controls: true,
  immediate: false
})

// Timeout with controls
const { isPending, start, stop } = useTimeout(3000, { controls: true })

// Current timestamp (reactive)
const timestamp = useTimestamp({ interval: 1000 })
const now = useNow({ interval: 'requestAnimationFrame' })

// Date formatting
const formatted = useDateFormat(now, 'YYYY-MM-DD HH:mm:ss')

// requestAnimationFrame loop
const { pause, resume, isActive } = useRafFn(({ delta, timestamp }) => {
  // Runs every animation frame
  updateAnimation(delta)
})

// Value transitions/tweening
const source = ref(0)
const output = useTransition(source, {
  duration: 1000,
  transition: [0.4, 0, 0.2, 1] // Cubic bezier
})
source.value = 100 // output transitions smoothly to 100
```

## Utilities

### Array & Collection

```typescript
import {
  useArrayFilter,
  useArrayMap,
  useArrayReduce,
  useArrayFind,
  useArrayFindIndex,
  useArraySome,
  useArrayEvery,
  useArrayUnique,
  useSorted
} from '@vueuse/core'

const list = ref([1, 2, 3, 4, 5])

// Reactive array operations
const filtered = useArrayFilter(list, (n) => n > 2) // [3, 4, 5]
const doubled = useArrayMap(list, (n) => n * 2) // [2, 4, 6, 8, 10]
const sum = useArrayReduce(list, (acc, n) => acc + n, 0) // 15
const found = useArrayFind(list, (n) => n > 3) // 4
const hasEven = useArraySome(list, (n) => n % 2 === 0) // true
const allPositive = useArrayEvery(list, (n) => n > 0) // true

// Unique values
const items = ref([1, 2, 2, 3, 3, 3])
const unique = useArrayUnique(items) // [1, 2, 3]

// Sorted array
const users = ref([{ name: 'John', age: 30 }, { name: 'Jane', age: 25 }])
const sorted = useSorted(users, (a, b) => a.age - b.age)
```

### Object Utilities

```typescript
import {
  useObjectUrl,
  useBase64,
  useCloned
} from '@vueuse/core'

// Object URL for files/blobs
const file = ref<File | null>(null)
const url = useObjectUrl(file) // Auto-creates and revokes object URL

// Base64 encoding
const { base64, execute } = useBase64('Hello World')
// base64.value = 'SGVsbG8gV29ybGQ='

// Deep clone with reactivity
const original = ref({ nested: { value: 1 } })
const { cloned, sync } = useCloned(original, { deep: true })
// cloned is independent reactive copy
// sync() updates cloned to match original
```

### Lifecycle & Control Flow

```typescript
import {
  whenever,
  until,
  watchOnce,
  watchAtMost,
  watchPausable,
  watchTriggerable,
  tryOnMounted,
  tryOnUnmounted
} from '@vueuse/core'

// Watch when truthy
const ready = ref(false)
whenever(ready, () => {
  console.log('Ready!')
})

// Wait until condition
const data = ref(null)
await until(data).not.toBeNull()
await until(data).toBe('expected')
await until(data).toMatch(v => v?.length > 0)
await until(data).changed({ timeout: 5000 })

// Watch only once
watchOnce(source, (value) => {
  console.log('First change:', value)
})

// Watch limited times
watchAtMost(source, (value) => {
  console.log('Changed:', value)
}, { count: 3 })

// Pausable watch
const { pause, resume, isActive } = watchPausable(source, (value) => {
  console.log(value)
})

// Manually triggerable watch
const { trigger, ignoreUpdates } = watchTriggerable(source, (value) => {
  console.log(value)
})
trigger() // Manually trigger callback

// Safe lifecycle hooks (won't error outside component)
tryOnMounted(() => console.log('Mounted'))
tryOnUnmounted(() => console.log('Unmounted'))
```

## Integrations (@vueuse/integrations)

```typescript
// Axios
import { useAxios } from '@vueuse/integrations/useAxios'
import axios from 'axios'

const { data, isFinished, isLoading, error, execute } = useAxios('/api/users', axios)

// Draggable
import { useDraggable } from '@vueuse/core'

const el = ref<HTMLElement | null>(null)
const { x, y, style, isDragging } = useDraggable(el, {
  initialValue: { x: 100, y: 100 }
})

// QRCode
import { useQRCode } from '@vueuse/integrations/useQRCode'

const text = ref('https://example.com')
const qrcode = useQRCode(text)
// <img :src="qrcode" />

// Cookies
import { useCookies } from '@vueuse/integrations/useCookies'

const cookies = useCookies(['token'])
cookies.set('token', 'abc123', { path: '/', maxAge: 3600 })
const token = cookies.get('token')

// JWT
import { useJwt } from '@vueuse/integrations/useJwt'

const token = ref('eyJhbGciOiJIUzI1NiIs...')
const { header, payload } = useJwt(token)
```

## Best Practices

### DO
- Use auto-cleanup composables instead of manual event listeners
- Leverage `useStorage` for persistent state
- Use `watchDebounced`/`watchThrottled` for expensive operations
- Combine with Pinia for complex state management
- Use `createSharedComposable` for singleton patterns

### DON'T
- Don't forget to handle loading/error states from async composables
- Don't use `useFetch` for complex API needs (use Axios/custom)
- Don't overuse reactive utilities for simple cases
- Don't ignore TypeScript types - VueUse is fully typed

## Common Import Patterns

```typescript
// State
import { useStorage, useLocalStorage, useRefHistory, useDebouncedRef } from '@vueuse/core'

// DOM
import { useElementSize, useWindowSize, useScroll, useIntersectionObserver } from '@vueuse/core'

// Events
import { useEventListener, useMouse, onClickOutside, onKeyStroke, useMagicKeys } from '@vueuse/core'

// Browser
import { useClipboard, useMediaQuery, useDark, useColorMode } from '@vueuse/core'

// Network
import { useFetch, useAsyncState, useWebSocket, useOnline } from '@vueuse/core'

// Sensors
import { useGeolocation, useBattery, useNetwork, useIdle } from '@vueuse/core'

// Animation
import { useInterval, useTimeout, useTransition, useRafFn } from '@vueuse/core'

// Utilities
import { whenever, until, watchDebounced, watchPausable, createGlobalState } from '@vueuse/core'
```
