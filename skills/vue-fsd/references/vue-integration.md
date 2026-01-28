# Vue Integration

> Reference for: Vue FSD Expert
> Load when: Vue-specific patterns, composables, Pinia stores, Vue Router integration

## Vue 3 + FSD Best Practices

### Composition API in FSD

FSD works naturally with Vue 3 Composition API:

| FSD Concept | Vue Implementation |
|-------------|-------------------|
| Entity/Feature Store | Pinia store |
| Slice composables | Vue composables |
| Public API | Named exports from index.ts |
| Segments | Folders (ui/, model/, api/, lib/) |

---

## Pinia Store Patterns

### Entity Store Template

```typescript
// entities/user/model/user.store.ts
import { defineStore } from 'pinia'
import { ref, computed } from 'vue'
import type { User } from './user.types'
import { userApi } from '../api/user.api'

export const useUserStore = defineStore('entities/user', () => {
  // ===== STATE =====
  const users = ref<Map<number, User>>(new Map())
  const loading = ref(false)
  const error = ref<string | null>(null)

  // ===== GETTERS =====
  const userById = computed(() => (id: number) => users.value.get(id))
  const userList = computed(() => Array.from(users.value.values()))

  // ===== ACTIONS =====
  async function fetchUser(id: number) {
    if (users.value.has(id)) return users.value.get(id)

    loading.value = true
    error.value = null

    try {
      const user = await userApi.getById(id)
      users.value.set(user.id, user)
      return user
    } catch (e) {
      error.value = (e as Error).message
      throw e
    } finally {
      loading.value = false
    }
  }

  function setUser(user: User) {
    users.value.set(user.id, user)
  }

  function removeUser(id: number) {
    users.value.delete(id)
  }

  function $reset() {
    users.value.clear()
    loading.value = false
    error.value = null
  }

  return {
    // State
    users,
    loading,
    error,
    // Getters
    userById,
    userList,
    // Actions
    fetchUser,
    setUser,
    removeUser,
    $reset
  }
})
```

### Feature Store Template

```typescript
// features/auth/model/auth.store.ts
import { defineStore } from 'pinia'
import { ref, computed } from 'vue'
import { useRouter } from 'vue-router'
import { useUserStore, type User } from '@/entities/user'
import type { LoginCredentials, AuthTokens } from './auth.types'
import { authApi } from '../api/auth.api'
import { tokenStorage } from '../lib/token'

export const useAuthStore = defineStore('features/auth', () => {
  const router = useRouter()
  const userStore = useUserStore()

  // ===== STATE =====
  const currentUserId = ref<number | null>(null)
  const tokens = ref<AuthTokens | null>(tokenStorage.get())
  const loading = ref(false)
  const error = ref<string | null>(null)

  // ===== GETTERS =====
  const isAuthenticated = computed(() => !!tokens.value?.accessToken)
  const currentUser = computed(() =>
    currentUserId.value ? userStore.userById(currentUserId.value) : null
  )

  // ===== ACTIONS =====
  async function login(credentials: LoginCredentials) {
    loading.value = true
    error.value = null

    try {
      const response = await authApi.login(credentials)

      tokens.value = response.tokens
      tokenStorage.set(response.tokens)

      userStore.setUser(response.user)
      currentUserId.value = response.user.id

      const redirect = router.currentRoute.value.query.redirect as string
      router.push(redirect || '/dashboard')
    } catch (e) {
      error.value = (e as Error).message
      throw e
    } finally {
      loading.value = false
    }
  }

  async function logout() {
    try {
      await authApi.logout()
    } finally {
      tokens.value = null
      currentUserId.value = null
      tokenStorage.clear()
      router.push('/login')
    }
  }

  return {
    currentUserId,
    tokens,
    loading,
    error,
    isAuthenticated,
    currentUser,
    login,
    logout
  }
})
```

### Store Naming Convention

Use namespaced store IDs to avoid conflicts:

```typescript
// entities/user/model/user.store.ts
defineStore('entities/user', () => { ... })

// entities/product/model/product.store.ts
defineStore('entities/product', () => { ... })

// features/auth/model/auth.store.ts
defineStore('features/auth', () => { ... })

// features/cart/model/cart.store.ts
defineStore('features/cart', () => { ... })
```

---

## Composables in FSD

### Slice-Specific Composables

```typescript
// features/auth/lib/use-auth-guard.ts
import { computed } from 'vue'
import { useRouter, useRoute } from 'vue-router'
import { useAuthStore } from '../model/auth.store'

export function useAuthGuard() {
  const router = useRouter()
  const route = useRoute()
  const authStore = useAuthStore()

  const isAuthenticated = computed(() => authStore.isAuthenticated)

  function requireAuth() {
    if (!isAuthenticated.value) {
      router.push({
        path: '/login',
        query: { redirect: route.fullPath }
      })
      return false
    }
    return true
  }

  function requireGuest() {
    if (isAuthenticated.value) {
      router.push('/dashboard')
      return false
    }
    return true
  }

  return {
    isAuthenticated,
    requireAuth,
    requireGuest
  }
}
```

### Entity-Specific Composables

```typescript
// entities/product/lib/use-product.ts
import { ref, computed, watch } from 'vue'
import { useProductStore } from '../model/product.store'
import type { Product } from '../model/product.types'

export function useProduct(productId: number | (() => number)) {
  const productStore = useProductStore()

  const id = computed(() =>
    typeof productId === 'function' ? productId() : productId
  )

  const product = computed(() => productStore.productById(id.value))
  const loading = ref(false)
  const error = ref<string | null>(null)

  async function fetch() {
    if (product.value) return product.value

    loading.value = true
    error.value = null

    try {
      await productStore.fetchProduct(id.value)
      return product.value
    } catch (e) {
      error.value = (e as Error).message
      throw e
    } finally {
      loading.value = false
    }
  }

  // Auto-fetch when ID changes
  watch(id, () => fetch(), { immediate: true })

  return {
    product,
    loading,
    error,
    refetch: fetch
  }
}
```

### Shared Composables

```typescript
// shared/composables/use-async-state.ts
import { ref, shallowRef } from 'vue'

interface UseAsyncStateOptions<T> {
  immediate?: boolean
  initialState?: T
  onSuccess?: (data: T) => void
  onError?: (error: Error) => void
}

export function useAsyncState<T>(
  asyncFn: () => Promise<T>,
  options: UseAsyncStateOptions<T> = {}
) {
  const { immediate = true, initialState, onSuccess, onError } = options

  const data = shallowRef<T | undefined>(initialState)
  const loading = ref(false)
  const error = ref<Error | null>(null)

  async function execute() {
    loading.value = true
    error.value = null

    try {
      const result = await asyncFn()
      data.value = result
      onSuccess?.(result)
      return result
    } catch (e) {
      const err = e as Error
      error.value = err
      onError?.(err)
      throw err
    } finally {
      loading.value = false
    }
  }

  if (immediate) {
    execute()
  }

  return {
    data,
    loading,
    error,
    execute
  }
}
```

---

## Vue Router Integration

### Route Configuration with FSD

```typescript
// app/router/index.ts
import { createRouter, createWebHistory, type RouteRecordRaw } from 'vue-router'

// Lazy load pages
const routes: RouteRecordRaw[] = [
  {
    path: '/',
    name: 'home',
    component: () => import('@/pages/home').then(m => m.HomePage)
  },
  {
    path: '/catalog',
    name: 'catalog',
    component: () => import('@/pages/catalog').then(m => m.CatalogPage)
  },
  {
    path: '/product/:id',
    name: 'product',
    component: () => import('@/pages/product').then(m => m.ProductPage),
    props: route => ({ id: Number(route.params.id) })
  },
  {
    path: '/login',
    name: 'login',
    component: () => import('@/pages/auth').then(m => m.LoginPage),
    meta: { requiresGuest: true }
  },
  {
    path: '/dashboard',
    name: 'dashboard',
    component: () => import('@/pages/dashboard').then(m => m.DashboardPage),
    meta: { requiresAuth: true }
  }
]

export const router = createRouter({
  history: createWebHistory(),
  routes
})
```

### Navigation Guards

```typescript
// app/router/guards.ts
import type { Router } from 'vue-router'
import { useAuthStore } from '@/features/auth'

export function setupGuards(router: Router) {
  router.beforeEach((to, from, next) => {
    const authStore = useAuthStore()

    // Check auth requirement
    if (to.meta.requiresAuth && !authStore.isAuthenticated) {
      return next({
        name: 'login',
        query: { redirect: to.fullPath }
      })
    }

    // Check guest requirement
    if (to.meta.requiresGuest && authStore.isAuthenticated) {
      return next({ name: 'dashboard' })
    }

    next()
  })
}
```

---

## Component Patterns

### Smart vs Dumb Components

**Dumb Components** (entities/shared ui) - No store access, props only:

```vue
<!-- entities/product/ui/ProductCard.vue -->
<script setup lang="ts">
import type { Product } from '../model/product.types'
import { formatCurrency } from '@/shared/lib'

interface Props {
  product: Product
  showActions?: boolean
}

defineProps<Props>()
defineEmits<{
  click: [product: Product]
}>()
</script>

<template>
  <div class="product-card" @click="$emit('click', product)">
    <img :src="product.image" :alt="product.name" />
    <h3>{{ product.name }}</h3>
    <p>{{ formatCurrency(product.price) }}</p>
    <slot v-if="showActions" name="actions" />
  </div>
</template>
```

**Smart Components** (features/widgets/pages) - Can access stores:

```vue
<!-- features/add-to-cart/ui/AddToCartButton.vue -->
<script setup lang="ts">
import { computed } from 'vue'
import type { Product } from '@/entities/product'
import { useCartStore } from '../model/cart.store'
import { Button } from '@/shared/ui'

interface Props {
  product: Product
  size?: 'sm' | 'md' | 'lg'
}

const props = withDefaults(defineProps<Props>(), {
  size: 'md'
})

const cartStore = useCartStore()

const isInCart = computed(() => cartStore.hasItem(props.product.id))
const quantity = computed(() => cartStore.getItemQuantity(props.product.id))

function handleAdd() {
  cartStore.addItem(props.product)
}

function handleRemove() {
  cartStore.removeItem(props.product.id)
}
</script>

<template>
  <div class="add-to-cart">
    <template v-if="isInCart">
      <Button :size="size" variant="outline" @click="handleRemove">-</Button>
      <span>{{ quantity }}</span>
      <Button :size="size" @click="handleAdd">+</Button>
    </template>
    <Button v-else :size="size" @click="handleAdd">
      Add to Cart
    </Button>
  </div>
</template>
```

---

## App Layer Setup

### Main Entry Point

```typescript
// app/main.ts
import { createApp } from 'vue'
import { createPinia } from 'pinia'
import App from './App.vue'
import { router } from './router'
import { setupGuards } from './router/guards'

// Styles
import './styles/global.css'

const app = createApp(App)
const pinia = createPinia()

app.use(pinia)
app.use(router)

// Setup navigation guards after Pinia
setupGuards(router)

app.mount('#app')
```

### App Component

```vue
<!-- app/App.vue -->
<script setup lang="ts">
import { onMounted } from 'vue'
import { useAuthStore } from '@/features/auth'

const authStore = useAuthStore()

// Check auth on app mount
onMounted(async () => {
  await authStore.checkAuth()
})
</script>

<template>
  <RouterView />
</template>
```

---

## Quick Reference

| Pattern | Location | Purpose |
|---------|----------|---------|
| Entity Store | `entities/*/model/*.store.ts` | CRUD for domain data |
| Feature Store | `features/*/model/*.store.ts` | Business logic and actions |
| Slice Composable | `*/lib/use-*.ts` | Reusable slice logic |
| Shared Composable | `shared/composables/` | Generic utilities |
| Smart Component | features, widgets, pages | Store access allowed |
| Dumb Component | entities, shared | Props only, no stores |
| Router Config | `app/router/` | Route definitions |
| Guards | `app/router/guards.ts` | Navigation protection |
