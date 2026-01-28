# TypeScript with Vue 3

> Reference for: Vue Expert
> Load when: TypeScript integration, typing props, generic components

## Component Props Typing

```vue
<script setup lang="ts">
// Basic interface
interface Props {
  title: string
  count: number
  items: string[]
  optional?: boolean
}

const props = defineProps<Props>()

// Props with defaults
const propsWithDefaults = withDefaults(defineProps<Props>(), {
  count: 0,
  items: () => [],
  optional: false
})

// Union types
interface PropsWithUnion {
  status: 'success' | 'error' | 'warning'
  size: 'sm' | 'md' | 'lg'
}

// Complex types
interface User {
  id: number
  name: string
  email: string
}

interface ComplexProps {
  user: User
  users: User[]
  callback: (id: number) => void
  config: Record<string, unknown>
}

const complexProps = defineProps<ComplexProps>()
</script>
```

## Emits Typing

```vue
<script setup lang="ts">
// Type-safe emits
interface Emits {
  (e: 'update', value: string): void
  (e: 'delete', id: number): void
  (e: 'submit', payload: { name: string; email: string }): void
}

const emit = defineEmits<Emits>()

// Usage
function handleUpdate(value: string) {
  emit('update', value) // Type-safe
  // emit('update', 123) // Error: number not assignable to string
}

// Alternative syntax
type EmitsType = {
  update: [value: string]
  delete: [id: number]
  submit: [payload: { name: string; email: string }]
}

const emit2 = defineEmits<EmitsType>()
</script>
```

## Ref Typing

```vue
<script setup lang="ts">
import { ref, Ref } from 'vue'

// Type inference
const count = ref(0) // Ref<number>
const message = ref('hello') // Ref<string>

// Explicit typing
const user = ref<User | null>(null)
const items = ref<string[]>([])

// Complex types
interface FormData {
  username: string
  email: string
  age: number
}

const form = ref<FormData>({
  username: '',
  email: '',
  age: 0
})

// Ref as function parameter
function updateCount(countRef: Ref<number>) {
  countRef.value++
}

updateCount(count)
</script>
```

## Reactive Typing

```vue
<script setup lang="ts">
import { reactive } from 'vue'

interface State {
  count: number
  user: {
    name: string
    email: string
  }
  items: string[]
}

// Explicit typing
const state = reactive<State>({
  count: 0,
  user: {
    name: '',
    email: ''
  },
  items: []
})

// Type inference
const inferredState = reactive({
  count: 0, // number
  message: 'hello', // string
  active: true // boolean
})
</script>
```

## Computed Typing

```vue
<script setup lang="ts">
import { ref, computed, ComputedRef } from 'vue'

const count = ref(0)

// Type inference
const doubled = computed(() => count.value * 2) // ComputedRef<number>

// Explicit typing
const tripled = computed<number>(() => count.value * 3)

// Complex computed
interface User {
  firstName: string
  lastName: string
}

const user = ref<User>({ firstName: 'John', lastName: 'Doe' })

const fullName = computed<string>(() => {
  return `${user.value.firstName} ${user.value.lastName}`
})

// Writable computed with typing
const fullNameWritable = computed<string>({
  get() {
    return `${user.value.firstName} ${user.value.lastName}`
  },
  set(value: string) {
    const [first, last] = value.split(' ')
    user.value.firstName = first
    user.value.lastName = last
  }
})
</script>
```

## Template Ref Typing

```vue
<script setup lang="ts">
import { ref, onMounted } from 'vue'

// HTML element refs
const inputRef = ref<HTMLInputElement | null>(null)
const divRef = ref<HTMLDivElement | null>(null)

onMounted(() => {
  inputRef.value?.focus()
  if (divRef.value) {
    divRef.value.scrollTop = 100
  }
})

// Component refs
import ChildComponent from './ChildComponent.vue'

const childRef = ref<InstanceType<typeof ChildComponent> | null>(null)

onMounted(() => {
  childRef.value?.someMethod()
})
</script>

<template>
  <input ref="inputRef" />
  <div ref="divRef">Content</div>
  <ChildComponent ref="childRef" />
</template>
```

## Composables Typing

```typescript
// composables/useCounter.ts
import { ref, computed, Ref, ComputedRef } from 'vue'

interface UseCounterReturn {
  count: Ref<number>
  doubled: ComputedRef<number>
  increment: () => void
  decrement: () => void
  reset: () => void
}

export function useCounter(initialValue = 0): UseCounterReturn {
  const count = ref(initialValue)
  const doubled = computed(() => count.value * 2)

  function increment() {
    count.value++
  }

  function decrement() {
    count.value--
  }

  function reset() {
    count.value = initialValue
  }

  return {
    count,
    doubled,
    increment,
    decrement,
    reset
  }
}

// composables/useFetch.ts
interface UseFetchOptions<T> {
  immediate?: boolean
  transform?: (data: unknown) => T
}

interface UseFetchReturn<T> {
  data: Ref<T | null>
  error: Ref<Error | null>
  loading: Ref<boolean>
  execute: () => Promise<void>
}

export function useFetch<T = unknown>(
  url: string,
  options: UseFetchOptions<T> = {}
): UseFetchReturn<T> {
  const data = ref<T | null>(null)
  const error = ref<Error | null>(null)
  const loading = ref(false)

  async function execute() {
    loading.value = true
    error.value = null

    try {
      const response = await fetch(url)
      const json = await response.json()
      data.value = options.transform ? options.transform(json) : json
    } catch (e) {
      error.value = e as Error
    } finally {
      loading.value = false
    }
  }

  if (options.immediate !== false) {
    execute()
  }

  return { data, error, loading, execute }
}

// Usage
<script setup lang="ts">
interface User {
  id: number
  name: string
}

const { data, error, loading } = useFetch<User>('/api/user')
</script>
```

## Generic Components

```vue
<!-- GenericList.vue -->
<script setup lang="ts" generic="T extends { id: number }">
interface Props {
  items: T[]
  selected?: T
}

interface Emits {
  (e: 'select', item: T): void
}

const props = defineProps<Props>()
const emit = defineEmits<Emits>()

function handleSelect(item: T) {
  emit('select', item)
}
</script>

<template>
  <div>
    <div
      v-for="item in items"
      :key="item.id"
      @click="handleSelect(item)"
    >
      <slot :item="item"></slot>
    </div>
  </div>
</template>

<!-- Usage -->
<script setup lang="ts">
interface User {
  id: number
  name: string
  email: string
}

const users: User[] = [
  { id: 1, name: 'John', email: 'john@example.com' }
]

function handleUserSelect(user: User) {
  console.log('Selected user:', user.name)
}
</script>

<template>
  <GenericList :items="users" @select="handleUserSelect">
    <template #default="{ item }">
      <div>{{ item.name }} - {{ item.email }}</div>
    </template>
  </GenericList>
</template>
```

## Event Handlers Typing

```vue
<script setup lang="ts">
// DOM events
function handleClick(event: MouseEvent) {
  console.log(event.clientX, event.clientY)
}

function handleInput(event: Event) {
  const target = event.target as HTMLInputElement
  console.log(target.value)
}

function handleKeydown(event: KeyboardEvent) {
  if (event.key === 'Enter') {
    console.log('Enter pressed')
  }
}

// Custom events from child components
interface CustomPayload {
  id: number
  value: string
}

function handleCustomEvent(payload: CustomPayload) {
  console.log(payload.id, payload.value)
}
</script>

<template>
  <button @click="handleClick">Click me</button>
  <input @input="handleInput" @keydown="handleKeydown" />
  <ChildComponent @custom="handleCustomEvent" />
</template>
```

## Provide/Inject Typing

```vue
<!-- Parent.vue -->
<script setup lang="ts">
import { provide, InjectionKey, Ref, ref } from 'vue'

interface UserContext {
  user: Ref<User>
  updateUser: (user: User) => void
}

// Create typed injection key
export const userContextKey = Symbol() as InjectionKey<UserContext>

const user = ref<User>({ id: 1, name: 'John', email: 'john@example.com' })

function updateUser(newUser: User) {
  user.value = newUser
}

// Provide with type safety
provide(userContextKey, {
  user,
  updateUser
})
</script>

<!-- Child.vue -->
<script setup lang="ts">
import { inject } from 'vue'
import { userContextKey } from './Parent.vue'

// Inject with type safety
const userContext = inject(userContextKey)

// With default value
const defaultContext: UserContext = {
  user: ref({ id: 0, name: '', email: '' }),
  updateUser: () => {}
}

const contextWithDefault = inject(userContextKey, defaultContext)

// Or throw if not provided
const requiredContext = inject(userContextKey)
if (!requiredContext) {
  throw new Error('User context not provided')
}
</script>
```

## Store Typing (Pinia)

```typescript
// stores/user.ts
import { defineStore } from 'pinia'
import { ref, computed } from 'vue'

interface User {
  id: number
  name: string
  email: string
  role: 'admin' | 'user'
}

export const useUserStore = defineStore('user', () => {
  // State
  const user = ref<User | null>(null)
  const users = ref<User[]>([])

  // Getters
  const isAdmin = computed(() => user.value?.role === 'admin')
  const userCount = computed(() => users.value.length)

  // Actions
  async function fetchUser(id: number): Promise<User> {
    const response = await fetch(`/api/users/${id}`)
    const data = await response.json()
    user.value = data
    return data
  }

  function logout() {
    user.value = null
  }

  return {
    user,
    users,
    isAdmin,
    userCount,
    fetchUser,
    logout
  }
})

// Typed store instance
export type UserStore = ReturnType<typeof useUserStore>
```

## Global Properties Typing

```typescript
// plugins/api.ts
export default defineNuxtPlugin(() => {
  const api = {
    async get<T>(url: string): Promise<T> {
      const response = await fetch(url)
      return response.json()
    },
    async post<T>(url: string, data: unknown): Promise<T> {
      const response = await fetch(url, {
        method: 'POST',
        body: JSON.stringify(data)
      })
      return response.json()
    }
  }

  return {
    provide: {
      api
    }
  }
})

// types/nuxt.d.ts - Augment types
declare module '#app' {
  interface NuxtApp {
    $api: {
      get<T>(url: string): Promise<T>
      post<T>(url: string, data: unknown): Promise<T>
    }
  }
}

declare module 'vue' {
  interface ComponentCustomProperties {
    $api: {
      get<T>(url: string): Promise<T>
      post<T>(url: string, data: unknown): Promise<T>
    }
  }
}

// Usage
<script setup lang="ts">
interface User {
  id: number
  name: string
}

const { $api } = useNuxtApp()
const user = await $api.get<User>('/api/user')
</script>
```

## Utility Types for Vue

```typescript
import type {
  PropType,
  ExtractPropTypes,
  ExtractPublicPropTypes,
  ComponentPublicInstance,
  ComponentCustomProperties
} from 'vue'

// Extract props type from runtime props
const props = {
  name: { type: String, required: true },
  age: { type: Number, default: 0 },
  items: { type: Array as PropType<string[]>, default: () => [] }
}

type Props = ExtractPropTypes<typeof props>
// { name: string; age: number; items: string[] }

// For public props (what parent sees)
type PublicProps = ExtractPublicPropTypes<typeof props>
// { name: string; age?: number; items?: string[] }
```

## MaybeRef & MaybeRefOrGetter

```typescript
import type { MaybeRef, MaybeRefOrGetter } from 'vue'
import { toValue } from 'vue'

// Composable accepting flexible input types
function useDouble(value: MaybeRefOrGetter<number>) {
  return computed(() => toValue(value) * 2)
}

// All these work:
useDouble(5)           // plain value
useDouble(ref(5))      // ref
useDouble(() => 5)     // getter

// Type definition
type MaybeRef<T> = T | Ref<T>
type MaybeRefOrGetter<T> = MaybeRef<T> | (() => T)
```

## Strict Component Instance Type

```typescript
import type { ComponentPublicInstance } from 'vue'

// Get component instance type
import MyComponent from './MyComponent.vue'

type MyComponentInstance = InstanceType<typeof MyComponent>

// For refs
const componentRef = ref<MyComponentInstance | null>(null)

// Access exposed methods with type safety
componentRef.value?.exposedMethod()

// Generic component ref type
type ComponentRef<T> = T extends new (...args: any[]) => infer R
  ? R extends { $props: infer P }
    ? ComponentPublicInstance<P>
    : never
  : never
```

## defineModel Type Patterns

```typescript
// Basic typed model
const model = defineModel<string>()

// Required model
const required = defineModel<number>({ required: true })

// With default
const withDefault = defineModel<string[]>({ default: () => [] })

// Named models with different types
const firstName = defineModel<string>('firstName')
const age = defineModel<number>('age', { default: 0 })

// Model with transform
const [value, modifiers] = defineModel<string, 'trim' | 'uppercase'>({
  set(val) {
    if (modifiers.trim) val = val.trim()
    if (modifiers.uppercase) val = val.toUpperCase()
    return val
  }
})
```

## Type-Safe Event Bus (Alternative to mitt)

```typescript
// types/events.ts
import type { InjectionKey } from 'vue'

interface EventMap {
  'user:login': { userId: number; timestamp: Date }
  'user:logout': void
  'notification': { message: string; type: 'success' | 'error' }
}

type EventCallback<T> = T extends void ? () => void : (payload: T) => void

interface EventBus {
  on<K extends keyof EventMap>(event: K, callback: EventCallback<EventMap[K]>): void
  off<K extends keyof EventMap>(event: K, callback: EventCallback<EventMap[K]>): void
  emit<K extends keyof EventMap>(event: K, ...args: EventMap[K] extends void ? [] : [EventMap[K]]): void
}

export const eventBusKey: InjectionKey<EventBus> = Symbol('eventBus')

// Usage in component
const bus = inject(eventBusKey)!
bus.on('user:login', ({ userId, timestamp }) => {
  // Fully typed!
})
bus.emit('notification', { message: 'Hello', type: 'success' })
```

## Discriminated Unions for State

```typescript
// Type-safe loading states
type AsyncState<T> =
  | { status: 'idle' }
  | { status: 'loading' }
  | { status: 'success'; data: T }
  | { status: 'error'; error: Error }

const userState = ref<AsyncState<User>>({ status: 'idle' })

// Type narrowing in template
watch(userState, (state) => {
  if (state.status === 'success') {
    console.log(state.data.name) // TypeScript knows data exists
  }
  if (state.status === 'error') {
    console.log(state.error.message) // TypeScript knows error exists
  }
})

// In template
<template>
  <div v-if="userState.status === 'loading'">Loading...</div>
  <div v-else-if="userState.status === 'error'">{{ userState.error.message }}</div>
  <div v-else-if="userState.status === 'success'">{{ userState.data.name }}</div>
</template>
```

## satisfies Operator (TypeScript 4.9+)

```typescript
// Validate object shape without widening type
const routes = {
  home: '/',
  about: '/about',
  user: '/user/:id'
} satisfies Record<string, string>

// routes.home is still '/' (literal type), not string

// Useful for config objects
const config = {
  api: {
    baseUrl: 'https://api.example.com',
    timeout: 5000
  },
  features: {
    darkMode: true,
    notifications: false
  }
} satisfies {
  api: { baseUrl: string; timeout: number }
  features: Record<string, boolean>
}
```

## Type-Only Imports

```typescript
// Always use type imports for types (better tree-shaking)
import type { Ref, ComputedRef, PropType } from 'vue'
import type { RouteLocationNormalized } from 'vue-router'
import type { User, Post } from '@/types'

// Mixed import
import { ref, computed, type Ref, type ComputedRef } from 'vue'
```

## Quick Reference

| Pattern | Type |
|---------|------|
| `defineProps<T>()` | Interface for props |
| `defineEmits<T>()` | Interface for emits |
| `defineModel<T>()` | Typed v-model (Vue 3.4+) |
| `ref<T>()` | Typed ref |
| `reactive<T>()` | Typed reactive object |
| `computed<T>()` | Typed computed |
| `ref<HTMLElement \| null>` | Template refs |
| `InstanceType<typeof Component>` | Component instance type |
| `generic="T"` | Generic components |
| `InjectionKey<T>` | Typed provide/inject |
| `MaybeRefOrGetter<T>` | Flexible composable inputs |
| `satisfies` | Type validation without widening |
| `import type` | Type-only imports |
| Type guards | Runtime type checking |
| Discriminated unions | Type-safe state patterns |
