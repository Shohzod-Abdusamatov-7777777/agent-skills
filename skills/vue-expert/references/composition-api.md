# Composition API

> Reference for: Vue Expert
> Load when: Using ref, reactive, computed, watch, lifecycle hooks

## Script Setup Syntax

```vue
<script setup lang="ts">
import { ref, reactive, computed, watch, onMounted } from 'vue'

// Automatic component registration - no need to register in components option
import UserCard from './UserCard.vue'

// Props with TypeScript
interface Props {
  userId: number
  optional?: string
}
const props = withDefaults(defineProps<Props>(), {
  optional: 'default value'
})

// Emits with TypeScript
interface Emits {
  (e: 'update', value: string): void
  (e: 'delete', id: number): void
}
const emit = defineEmits<Emits>()

// Reactive state
const count = ref(0)
const user = reactive({
  name: 'John',
  age: 30
})

// Computed
const doubled = computed(() => count.value * 2)

// Methods
function increment() {
  count.value++
  emit('update', count.value.toString())
}

// Lifecycle
onMounted(() => {
  console.log('Component mounted')
})
</script>
```

## Ref vs Reactive

```typescript
import { ref, reactive, toRefs } from 'vue'

// Use ref() for primitives
const count = ref(0)
const message = ref('hello')
const isActive = ref(true)

// Access/modify with .value
count.value++
console.log(message.value)

// Use reactive() for objects
const state = reactive({
  count: 0,
  user: {
    name: 'John',
    email: 'john@example.com'
  }
})

// No .value needed for reactive
state.count++
state.user.name = 'Jane'

// Convert reactive to refs for destructuring
const { count: refCount, user } = toRefs(state)
// Now refCount.value works
```

## Computed Properties

```typescript
import { ref, computed } from 'vue'

const firstName = ref('John')
const lastName = ref('Doe')

// Read-only computed
const fullName = computed(() => {
  return `${firstName.value} ${lastName.value}`
})

// Writable computed
const fullNameWritable = computed({
  get() {
    return `${firstName.value} ${lastName.value}`
  },
  set(value: string) {
    const [first, last] = value.split(' ')
    firstName.value = first
    lastName.value = last
  }
})

// Computed with complex logic (cached until dependencies change)
const filteredItems = computed(() => {
  return items.value.filter(item =>
    item.name.toLowerCase().includes(searchQuery.value.toLowerCase())
  )
})
```

## Watchers

```typescript
import { ref, watch, watchEffect } from 'vue'

const count = ref(0)
const user = ref({ name: 'John', age: 30 })

// Watch single source
watch(count, (newValue, oldValue) => {
  console.log(`Count changed from ${oldValue} to ${newValue}`)
})

// Watch multiple sources
watch([count, user], ([newCount, newUser], [oldCount, oldUser]) => {
  console.log('Count or user changed')
})

// Watch with options
watch(
  () => user.value.name, // Getter function
  (newName) => {
    console.log(`Name changed to ${newName}`)
  },
  {
    immediate: true, // Run immediately
    deep: true // Deep watch for objects
  }
)

// watchEffect - automatically tracks dependencies
watchEffect(() => {
  console.log(`Count is ${count.value}`)
  // Automatically re-runs when count changes
})

// Cleanup and stop watching
const stop = watchEffect((onCleanup) => {
  const timer = setInterval(() => console.log('tick'), 1000)

  onCleanup(() => {
    clearInterval(timer)
  })
})

// Stop watching when needed
stop()
```

## Lifecycle Hooks

```typescript
import {
  onBeforeMount,
  onMounted,
  onBeforeUpdate,
  onUpdated,
  onBeforeUnmount,
  onUnmounted,
  onErrorCaptured
} from 'vue'

// Before component is mounted
onBeforeMount(() => {
  console.log('Before mount')
})

// After component is mounted (DOM is ready)
onMounted(() => {
  console.log('Mounted - DOM is ready')
  // Fetch data, setup event listeners, etc.
})

// Before component updates
onBeforeUpdate(() => {
  console.log('Before update')
})

// After component updates
onUpdated(() => {
  console.log('Updated')
})

// Before component is unmounted
onBeforeUnmount(() => {
  console.log('Before unmount - cleanup here')
})

// After component is unmounted
onUnmounted(() => {
  console.log('Unmounted')
  // Cleanup: remove event listeners, cancel timers, etc.
})

// Error handling
onErrorCaptured((err, instance, info) => {
  console.error('Error captured:', err, info)
  return false // Prevent error from propagating
})
```

## Composables Pattern

```typescript
// composables/useCounter.ts
import { ref, computed } from 'vue'

export function useCounter(initialValue = 0) {
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

// Usage in component
<script setup lang="ts">
import { useCounter } from './composables/useCounter'

const { count, doubled, increment, decrement } = useCounter(10)
</script>
```

## Advanced Composable with Cleanup

```typescript
// composables/useEventListener.ts
import { onMounted, onUnmounted } from 'vue'

export function useEventListener(
  target: EventTarget,
  event: string,
  handler: EventListener
) {
  onMounted(() => {
    target.addEventListener(event, handler)
  })

  onUnmounted(() => {
    target.removeEventListener(event, handler)
  })
}

// Usage
<script setup lang="ts">
import { useEventListener } from './composables/useEventListener'

function handleClick(e: MouseEvent) {
  console.log('Clicked at:', e.clientX, e.clientY)
}

useEventListener(window, 'click', handleClick)
</script>
```

## defineModel (Vue 3.4+)

```vue
<!-- ChildInput.vue -->
<script setup lang="ts">
// Simple v-model - replaces defineProps + defineEmits pattern
const model = defineModel<string>()

// With default value
const modelWithDefault = defineModel<string>({ default: '' })

// Named v-model
const firstName = defineModel<string>('firstName')
const lastName = defineModel<string>('lastName')

// With modifiers
const [modelValue, modifiers] = defineModel<string>({
  set(value) {
    if (modifiers.capitalize) {
      return value.charAt(0).toUpperCase() + value.slice(1)
    }
    return value
  }
})
</script>

<template>
  <input v-model="model" />
</template>

<!-- Parent.vue -->
<template>
  <ChildInput v-model="name" />
  <ChildInput v-model:firstName="first" v-model:lastName="last" />
  <ChildInput v-model.capitalize="text" />
</template>
```

## defineExpose

```vue
<script setup lang="ts">
import { ref } from 'vue'

const count = ref(0)
const internalState = ref('private')

function increment() {
  count.value++
}

function reset() {
  count.value = 0
}

// Only expose specific properties/methods to parent
defineExpose({
  count,
  increment,
  reset
  // internalState is NOT exposed
})
</script>

<!-- Parent component accessing child -->
<script setup lang="ts">
import { ref } from 'vue'
import ChildComponent from './ChildComponent.vue'

const childRef = ref<InstanceType<typeof ChildComponent> | null>(null)

function handleClick() {
  childRef.value?.increment()
  console.log(childRef.value?.count) // Works
  // childRef.value?.internalState // TypeScript error - not exposed
}
</script>

<template>
  <ChildComponent ref="childRef" />
</template>
```

## effectScope

```typescript
import { effectScope, ref, computed, watch, onScopeDispose } from 'vue'

// Create isolated effect scope
const scope = effectScope()

scope.run(() => {
  const count = ref(0)
  const doubled = computed(() => count.value * 2)

  watch(count, () => {
    console.log('Count changed')
  })

  // Cleanup when scope is disposed
  onScopeDispose(() => {
    console.log('Scope disposed')
  })
})

// Stop all effects in scope
scope.stop()

// Useful for composables with cleanup
function useFeature() {
  const scope = effectScope()

  const state = scope.run(() => {
    const data = ref(null)
    const loading = ref(false)

    // Effects are contained in this scope
    watchEffect(() => {
      // ...
    })

    return { data, loading }
  })

  // Return cleanup function
  const dispose = () => scope.stop()

  return { ...state, dispose }
}
```

## shallowRef & shallowReactive

```typescript
import { shallowRef, shallowReactive, triggerRef } from 'vue'

// shallowRef - only .value is reactive (not deep)
const state = shallowRef({ count: 0, nested: { value: 1 } })

state.value.count = 1 // Does NOT trigger update
state.value = { count: 1, nested: { value: 1 } } // Triggers update

// Manual trigger
state.value.count = 2
triggerRef(state) // Force trigger update

// shallowReactive - only root level is reactive
const shallowState = shallowReactive({
  count: 0,
  nested: { value: 1 }
})

shallowState.count = 1 // Reactive
shallowState.nested.value = 2 // NOT reactive

// Use for large objects where deep reactivity isn't needed
const largeDataset = shallowRef<DataItem[]>([])
```

## toValue & toRef (Vue 3.3+)

```typescript
import { ref, toRef, toValue, MaybeRefOrGetter } from 'vue'

// toValue - unwrap refs/getters to plain value
const count = ref(5)
const getter = () => 10

toValue(count) // 5
toValue(getter) // 10
toValue(15) // 15

// Useful in composables that accept flexible inputs
function useDouble(input: MaybeRefOrGetter<number>) {
  return computed(() => toValue(input) * 2)
}

useDouble(5)         // Works with plain value
useDouble(ref(5))    // Works with ref
useDouble(() => 5)   // Works with getter

// toRef - create ref from reactive property
const state = reactive({ count: 0 })
const countRef = toRef(state, 'count')

countRef.value++ // Updates state.count

// toRef with getter (Vue 3.3+)
const doubled = toRef(() => state.count * 2)
```

## Quick Reference

| Pattern | Use Case |
|---------|----------|
| `ref()` | Primitives (string, number, boolean) |
| `reactive()` | Objects and arrays |
| `computed()` | Derived state (cached) |
| `watch()` | Side effects on specific changes |
| `watchEffect()` | Auto-tracked side effects |
| `onMounted()` | DOM-dependent operations |
| `onUnmounted()` | Cleanup (timers, listeners) |
| `defineModel()` | Simplified v-model (Vue 3.4+) |
| `defineExpose()` | Expose child methods to parent |
| `effectScope()` | Group and dispose effects together |
| `shallowRef()` | Shallow reactivity for performance |
| `toValue()` | Unwrap refs/getters (Vue 3.3+) |
| Composables | Reusable stateful logic |
