# Shared Layer

> Reference for: Vue FSD Expert
> Load when: UI kit, utilities, API clients, configs, shared infrastructure

## Shared Layer Purpose

The shared layer contains **project-agnostic** code that could theoretically be extracted into a separate package. It has no knowledge of business domain.

```
src/shared/
├── api/          # HTTP client, interceptors
├── config/       # Environment, constants
├── lib/          # Utility functions
├── ui/           # UI kit components
├── types/        # Global TypeScript types
└── composables/  # Shared composables (optional)
```

**No slices** - shared is organized by technical concern.

---

## API Module

### HTTP Client Setup

```typescript
// shared/api/client.ts
import axios, { type AxiosInstance, type AxiosError } from 'axios'

const API_BASE_URL = import.meta.env.VITE_API_URL || '/api'

export const apiClient: AxiosInstance = axios.create({
  baseURL: API_BASE_URL,
  timeout: 30000,
  headers: {
    'Content-Type': 'application/json'
  }
})

// Request interceptor - add auth token
apiClient.interceptors.request.use(
  (config) => {
    const token = localStorage.getItem('accessToken')
    if (token) {
      config.headers.Authorization = `Bearer ${token}`
    }
    return config
  },
  (error) => Promise.reject(error)
)

// Response interceptor - handle errors
apiClient.interceptors.response.use(
  (response) => response,
  async (error: AxiosError) => {
    if (error.response?.status === 401) {
      // Handle token refresh or logout
      localStorage.removeItem('accessToken')
      window.location.href = '/login'
    }
    return Promise.reject(error)
  }
)
```

### API Types

```typescript
// shared/api/types.ts
export interface ApiResponse<T> {
  data: T
  message?: string
}

export interface ApiError {
  message: string
  code: string
  details?: Record<string, string[]>
}

export interface PaginatedResponse<T> {
  data: T[]
  meta: {
    total: number
    page: number
    limit: number
    totalPages: number
  }
}

export interface PaginationParams {
  page?: number
  limit?: number
  sort?: string
  order?: 'asc' | 'desc'
}
```

### Public API

```typescript
// shared/api/index.ts
export { apiClient } from './client'
export type { ApiResponse, ApiError, PaginatedResponse, PaginationParams } from './types'
```

---

## Config Module

### Environment Configuration

```typescript
// shared/config/env.ts
export const config = {
  apiUrl: import.meta.env.VITE_API_URL || '/api',
  appName: import.meta.env.VITE_APP_NAME || 'My App',
  isDev: import.meta.env.DEV,
  isProd: import.meta.env.PROD,
  mode: import.meta.env.MODE
} as const

export type Config = typeof config
```

### Constants

```typescript
// shared/config/constants.ts
export const ROUTES = {
  HOME: '/',
  LOGIN: '/login',
  REGISTER: '/register',
  DASHBOARD: '/dashboard',
  PROFILE: '/profile',
  CATALOG: '/catalog',
  PRODUCT: '/product/:id',
  CART: '/cart',
  CHECKOUT: '/checkout'
} as const

export const STORAGE_KEYS = {
  ACCESS_TOKEN: 'accessToken',
  REFRESH_TOKEN: 'refreshToken',
  THEME: 'theme',
  LOCALE: 'locale'
} as const

export const PAGINATION = {
  DEFAULT_PAGE: 1,
  DEFAULT_LIMIT: 20,
  MAX_LIMIT: 100
} as const
```

### Public API

```typescript
// shared/config/index.ts
export { config } from './env'
export { ROUTES, STORAGE_KEYS, PAGINATION } from './constants'
```

---

## Lib Module (Utilities)

### Date Utilities

```typescript
// shared/lib/date.ts
import { format, formatDistance, parseISO } from 'date-fns'

export function formatDate(date: string | Date, pattern = 'PP'): string {
  const parsed = typeof date === 'string' ? parseISO(date) : date
  return format(parsed, pattern)
}

export function formatDateTime(date: string | Date): string {
  return formatDate(date, 'PPp')
}

export function formatRelative(date: string | Date): string {
  const parsed = typeof date === 'string' ? parseISO(date) : date
  return formatDistance(parsed, new Date(), { addSuffix: true })
}
```

### String Utilities

```typescript
// shared/lib/string.ts
export function capitalize(str: string): string {
  return str.charAt(0).toUpperCase() + str.slice(1)
}

export function truncate(str: string, length: number): string {
  if (str.length <= length) return str
  return str.slice(0, length) + '...'
}

export function slugify(str: string): string {
  return str
    .toLowerCase()
    .replace(/[^\w\s-]/g, '')
    .replace(/\s+/g, '-')
    .replace(/-+/g, '-')
    .trim()
}

export function generateId(): string {
  return Math.random().toString(36).substring(2, 9)
}
```

### Number Utilities

```typescript
// shared/lib/number.ts
export function formatCurrency(
  amount: number,
  currency = 'USD',
  locale = 'en-US'
): string {
  return new Intl.NumberFormat(locale, {
    style: 'currency',
    currency
  }).format(amount)
}

export function formatNumber(
  num: number,
  locale = 'en-US'
): string {
  return new Intl.NumberFormat(locale).format(num)
}

export function clamp(value: number, min: number, max: number): number {
  return Math.min(Math.max(value, min), max)
}
```

### Async Utilities

```typescript
// shared/lib/async.ts
export function debounce<T extends (...args: any[]) => void>(
  fn: T,
  delay: number
): (...args: Parameters<T>) => void {
  let timeoutId: ReturnType<typeof setTimeout>

  return (...args: Parameters<T>) => {
    clearTimeout(timeoutId)
    timeoutId = setTimeout(() => fn(...args), delay)
  }
}

export function throttle<T extends (...args: any[]) => void>(
  fn: T,
  delay: number
): (...args: Parameters<T>) => void {
  let lastCall = 0

  return (...args: Parameters<T>) => {
    const now = Date.now()
    if (now - lastCall >= delay) {
      lastCall = now
      fn(...args)
    }
  }
}

export function sleep(ms: number): Promise<void> {
  return new Promise(resolve => setTimeout(resolve, ms))
}
```

### Storage Utilities

```typescript
// shared/lib/storage.ts
export const storage = {
  get<T>(key: string): T | null {
    try {
      const item = localStorage.getItem(key)
      return item ? JSON.parse(item) : null
    } catch {
      return null
    }
  },

  set<T>(key: string, value: T): void {
    try {
      localStorage.setItem(key, JSON.stringify(value))
    } catch {
      // Storage full or unavailable
    }
  },

  remove(key: string): void {
    localStorage.removeItem(key)
  },

  clear(): void {
    localStorage.clear()
  }
}
```

### Public API

```typescript
// shared/lib/index.ts
export { formatDate, formatDateTime, formatRelative } from './date'
export { capitalize, truncate, slugify, generateId } from './string'
export { formatCurrency, formatNumber, clamp } from './number'
export { debounce, throttle, sleep } from './async'
export { storage } from './storage'
```

---

## UI Module

### Button Component

```vue
<!-- shared/ui/Button/Button.vue -->
<script setup lang="ts">
interface Props {
  variant?: 'primary' | 'secondary' | 'outline' | 'ghost' | 'danger'
  size?: 'sm' | 'md' | 'lg'
  loading?: boolean
  disabled?: boolean
  block?: boolean
  type?: 'button' | 'submit' | 'reset'
}

withDefaults(defineProps<Props>(), {
  variant: 'primary',
  size: 'md',
  loading: false,
  disabled: false,
  block: false,
  type: 'button'
})
</script>

<template>
  <button
    :type="type"
    :disabled="disabled || loading"
    :class="[
      'btn',
      `btn--${variant}`,
      `btn--${size}`,
      { 'btn--block': block, 'btn--loading': loading }
    ]"
  >
    <span v-if="loading" class="btn__spinner" />
    <slot />
  </button>
</template>

<style scoped>
.btn {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  gap: 0.5rem;
  font-weight: 500;
  border-radius: 0.5rem;
  transition: all 0.2s;
  cursor: pointer;
  border: none;
}

.btn--primary {
  background: var(--color-primary);
  color: white;
}

.btn--primary:hover:not(:disabled) {
  background: var(--color-primary-dark);
}

.btn--secondary {
  background: var(--color-secondary);
  color: white;
}

.btn--outline {
  background: transparent;
  border: 1px solid var(--color-border);
  color: var(--color-text);
}

.btn--ghost {
  background: transparent;
  color: var(--color-text);
}

.btn--danger {
  background: var(--color-danger);
  color: white;
}

.btn--sm { padding: 0.375rem 0.75rem; font-size: 0.875rem; }
.btn--md { padding: 0.5rem 1rem; font-size: 1rem; }
.btn--lg { padding: 0.75rem 1.5rem; font-size: 1.125rem; }

.btn--block { width: 100%; }

.btn:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.btn__spinner {
  width: 1em;
  height: 1em;
  border: 2px solid currentColor;
  border-right-color: transparent;
  border-radius: 50%;
  animation: spin 0.75s linear infinite;
}

@keyframes spin {
  to { transform: rotate(360deg); }
}
</style>
```

```typescript
// shared/ui/Button/index.ts
export { default as Button } from './Button.vue'
```

### Input Component

```vue
<!-- shared/ui/Input/Input.vue -->
<script setup lang="ts">
interface Props {
  modelValue: string | number
  type?: 'text' | 'email' | 'password' | 'number' | 'tel' | 'url'
  label?: string
  placeholder?: string
  error?: string
  disabled?: boolean
  required?: boolean
}

const props = withDefaults(defineProps<Props>(), {
  type: 'text',
  disabled: false,
  required: false
})

const emit = defineEmits<{
  'update:modelValue': [value: string | number]
}>()

function handleInput(event: Event) {
  const target = event.target as HTMLInputElement
  emit('update:modelValue', props.type === 'number' ? Number(target.value) : target.value)
}
</script>

<template>
  <div class="input-wrapper">
    <label v-if="label" class="input-label">
      {{ label }}
      <span v-if="required" class="input-required">*</span>
    </label>
    <input
      :type="type"
      :value="modelValue"
      :placeholder="placeholder"
      :disabled="disabled"
      :required="required"
      :class="['input', { 'input--error': error }]"
      @input="handleInput"
    />
    <span v-if="error" class="input-error">{{ error }}</span>
  </div>
</template>

<style scoped>
.input-wrapper {
  display: flex;
  flex-direction: column;
  gap: 0.25rem;
}

.input-label {
  font-size: 0.875rem;
  font-weight: 500;
  color: var(--color-text);
}

.input-required {
  color: var(--color-danger);
}

.input {
  padding: 0.5rem 0.75rem;
  border: 1px solid var(--color-border);
  border-radius: 0.5rem;
  font-size: 1rem;
  transition: border-color 0.2s, box-shadow 0.2s;
}

.input:focus {
  outline: none;
  border-color: var(--color-primary);
  box-shadow: 0 0 0 3px var(--color-primary-light);
}

.input--error {
  border-color: var(--color-danger);
}

.input:disabled {
  background: var(--color-bg-disabled);
  cursor: not-allowed;
}

.input-error {
  font-size: 0.75rem;
  color: var(--color-danger);
}
</style>
```

### Modal Component

```vue
<!-- shared/ui/Modal/Modal.vue -->
<script setup lang="ts">
import { onMounted, onUnmounted } from 'vue'

interface Props {
  show: boolean
  title?: string
  size?: 'sm' | 'md' | 'lg' | 'xl'
  closable?: boolean
}

const props = withDefaults(defineProps<Props>(), {
  size: 'md',
  closable: true
})

const emit = defineEmits<{
  close: []
}>()

function handleClose() {
  if (props.closable) {
    emit('close')
  }
}

function handleKeydown(e: KeyboardEvent) {
  if (e.key === 'Escape' && props.closable) {
    emit('close')
  }
}

onMounted(() => {
  document.addEventListener('keydown', handleKeydown)
  document.body.style.overflow = 'hidden'
})

onUnmounted(() => {
  document.removeEventListener('keydown', handleKeydown)
  document.body.style.overflow = ''
})
</script>

<template>
  <Teleport to="body">
    <Transition name="modal">
      <div v-if="show" class="modal-overlay" @click.self="handleClose">
        <div :class="['modal', `modal--${size}`]">
          <div v-if="title || closable" class="modal__header">
            <h2 v-if="title" class="modal__title">{{ title }}</h2>
            <button v-if="closable" class="modal__close" @click="handleClose">
              &times;
            </button>
          </div>
          <div class="modal__body">
            <slot />
          </div>
          <div v-if="$slots.footer" class="modal__footer">
            <slot name="footer" />
          </div>
        </div>
      </div>
    </Transition>
  </Teleport>
</template>
```

### UI Kit Public API

```typescript
// shared/ui/index.ts
export { Button } from './Button'
export { Input } from './Input'
export { Modal } from './Modal'
export { Avatar } from './Avatar'
export { Card } from './Card'
export { Badge } from './Badge'
export { Spinner } from './Spinner'
export { Alert } from './Alert'
```

---

## Types Module

```typescript
// shared/types/index.ts

// Generic utility types
export type Nullable<T> = T | null
export type Optional<T> = T | undefined
export type Maybe<T> = T | null | undefined

// ID types
export type ID = number | string

// Common entity base
export interface BaseEntity {
  id: ID
  createdAt: string
  updatedAt: string
}

// Form state
export interface FormState<T> {
  values: T
  errors: Partial<Record<keyof T, string>>
  touched: Partial<Record<keyof T, boolean>>
  isSubmitting: boolean
  isValid: boolean
}

// Async state
export type AsyncState<T> =
  | { status: 'idle' }
  | { status: 'loading' }
  | { status: 'success'; data: T }
  | { status: 'error'; error: Error }
```

---

## Complete Shared Index

```typescript
// shared/index.ts (optional - for convenience)
export * from './api'
export * from './config'
export * from './lib'
export * from './ui'
export * from './types'
```

## Quick Reference

| Module | Contents |
|--------|----------|
| api/ | HTTP client, interceptors, API types |
| config/ | Environment, constants, feature flags |
| lib/ | Utility functions (date, string, async) |
| ui/ | Reusable UI components (Button, Input, Modal) |
| types/ | Global TypeScript types and interfaces |
| composables/ | Shared Vue composables (optional) |
