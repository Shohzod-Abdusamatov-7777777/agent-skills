# Entities & Features

> Reference for: Vue FSD Expert
> Load when: Creating entities, features, distinguishing nouns vs verbs, business logic

## Entity vs Feature: The Key Distinction

| Aspect | Entity | Feature |
|--------|--------|---------|
| Represents | **Nouns** - things, data | **Verbs** - actions, interactions |
| Examples | User, Product, Order, Article | Login, AddToCart, CreateOrder |
| Contains | Data types, stores, display components | Action handlers, forms, buttons |
| Side effects | Minimal (CRUD operations) | Significant (business logic) |
| Imports from | shared only | entities, shared |

## Entity Structure

### Complete Entity Example: User

```
src/entities/user/
├── api/
│   └── user.api.ts       # API calls for user data
├── model/
│   ├── user.types.ts     # TypeScript interfaces
│   ├── user.store.ts     # Pinia store
│   └── user.selectors.ts # Computed getters (optional)
├── ui/
│   ├── UserCard.vue      # Display component
│   ├── UserAvatar.vue    # Avatar component
│   └── UserBadge.vue     # Status badge
├── lib/
│   └── format-user.ts    # Entity-specific utils
└── index.ts              # Public API
```

### Entity Types

```typescript
// entities/user/model/user.types.ts
export interface User {
  id: number
  email: string
  name: string
  avatar: string | null
  role: UserRole
  createdAt: string
  updatedAt: string
}

export type UserRole = 'admin' | 'user' | 'guest'

export interface UserPreview {
  id: number
  name: string
  avatar: string | null
}

// For API responses
export interface UserListResponse {
  users: User[]
  total: number
  page: number
  limit: number
}
```

### Entity API

```typescript
// entities/user/api/user.api.ts
import { apiClient } from '@/shared/api'
import type { User, UserListResponse } from '../model/user.types'

export const userApi = {
  async getById(id: number): Promise<User> {
    const { data } = await apiClient.get<User>(`/users/${id}`)
    return data
  },

  async getList(params: { page: number; limit: number }): Promise<UserListResponse> {
    const { data } = await apiClient.get<UserListResponse>('/users', { params })
    return data
  },

  async update(id: number, payload: Partial<User>): Promise<User> {
    const { data } = await apiClient.patch<User>(`/users/${id}`, payload)
    return data
  }
}
```

### Entity Store (Pinia)

```typescript
// entities/user/model/user.store.ts
import { defineStore } from 'pinia'
import { ref, computed } from 'vue'
import type { User } from './user.types'
import { userApi } from '../api/user.api'

export const useUserStore = defineStore('user', () => {
  // State
  const users = ref<Map<number, User>>(new Map())
  const loading = ref(false)
  const error = ref<string | null>(null)

  // Getters
  const userById = computed(() => {
    return (id: number) => users.value.get(id)
  })

  const userList = computed(() => Array.from(users.value.values()))

  // Actions
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

  async function fetchUsers(params: { page: number; limit: number }) {
    loading.value = true
    error.value = null

    try {
      const response = await userApi.getList(params)
      response.users.forEach(user => users.value.set(user.id, user))
      return response
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

  return {
    users,
    loading,
    error,
    userById,
    userList,
    fetchUser,
    fetchUsers,
    setUser
  }
})
```

### Entity UI Component

```vue
<!-- entities/user/ui/UserCard.vue -->
<script setup lang="ts">
import type { User } from '../model/user.types'
import { Avatar } from '@/shared/ui'
import { formatDate } from '@/shared/lib'

interface Props {
  user: User
  showRole?: boolean
}

const props = withDefaults(defineProps<Props>(), {
  showRole: false
})
</script>

<template>
  <div class="user-card">
    <Avatar :src="user.avatar" :alt="user.name" />
    <div class="user-card__info">
      <h3 class="user-card__name">{{ user.name }}</h3>
      <p class="user-card__email">{{ user.email }}</p>
      <span v-if="showRole" class="user-card__role">{{ user.role }}</span>
      <time class="user-card__date">{{ formatDate(user.createdAt) }}</time>
    </div>
  </div>
</template>

<style scoped>
.user-card {
  display: flex;
  gap: 1rem;
  padding: 1rem;
  border-radius: 8px;
  background: var(--color-surface);
}

.user-card__info {
  display: flex;
  flex-direction: column;
  gap: 0.25rem;
}

.user-card__name {
  font-weight: 600;
  margin: 0;
}

.user-card__email {
  color: var(--color-text-secondary);
  margin: 0;
}
</style>
```

### Entity Public API

```typescript
// entities/user/index.ts
// Types
export type { User, UserRole, UserPreview, UserListResponse } from './model/user.types'

// Store
export { useUserStore } from './model/user.store'

// API (optional - expose if other slices need direct access)
export { userApi } from './api/user.api'

// UI Components
export { default as UserCard } from './ui/UserCard.vue'
export { default as UserAvatar } from './ui/UserAvatar.vue'
export { default as UserBadge } from './ui/UserBadge.vue'
```

---

## Feature Structure

### Complete Feature Example: Auth

```
src/features/auth/
├── api/
│   └── auth.api.ts       # Auth-specific API calls
├── model/
│   ├── auth.types.ts     # Feature types
│   └── auth.store.ts     # Auth state and logic
├── ui/
│   ├── LoginForm.vue     # Login form component
│   ├── RegisterForm.vue  # Register form
│   └── LogoutButton.vue  # Logout action button
├── lib/
│   └── token.ts          # Token utilities
└── index.ts              # Public API
```

### Feature Types

```typescript
// features/auth/model/auth.types.ts
export interface LoginCredentials {
  email: string
  password: string
}

export interface RegisterData {
  email: string
  password: string
  name: string
}

export interface AuthTokens {
  accessToken: string
  refreshToken: string
}

export interface AuthState {
  isAuthenticated: boolean
  tokens: AuthTokens | null
}
```

### Feature API

```typescript
// features/auth/api/auth.api.ts
import { apiClient } from '@/shared/api'
import type { User } from '@/entities/user'
import type { LoginCredentials, RegisterData, AuthTokens } from '../model/auth.types'

export const authApi = {
  async login(credentials: LoginCredentials): Promise<{ user: User; tokens: AuthTokens }> {
    const { data } = await apiClient.post('/auth/login', credentials)
    return data
  },

  async register(data: RegisterData): Promise<{ user: User; tokens: AuthTokens }> {
    const { data: response } = await apiClient.post('/auth/register', data)
    return response
  },

  async logout(): Promise<void> {
    await apiClient.post('/auth/logout')
  },

  async refreshTokens(refreshToken: string): Promise<AuthTokens> {
    const { data } = await apiClient.post('/auth/refresh', { refreshToken })
    return data
  },

  async getCurrentUser(): Promise<User> {
    const { data } = await apiClient.get<User>('/auth/me')
    return data
  }
}
```

### Feature Store

```typescript
// features/auth/model/auth.store.ts
import { defineStore } from 'pinia'
import { ref, computed } from 'vue'
import { useRouter } from 'vue-router'
import { useUserStore, type User } from '@/entities/user'
import type { LoginCredentials, RegisterData, AuthTokens } from './auth.types'
import { authApi } from '../api/auth.api'
import { tokenStorage } from '../lib/token'

export const useAuthStore = defineStore('auth', () => {
  const router = useRouter()
  const userStore = useUserStore()

  // State
  const currentUserId = ref<number | null>(null)
  const tokens = ref<AuthTokens | null>(tokenStorage.get())
  const loading = ref(false)
  const error = ref<string | null>(null)

  // Getters
  const isAuthenticated = computed(() => !!tokens.value?.accessToken)

  const currentUser = computed(() => {
    if (!currentUserId.value) return null
    return userStore.userById(currentUserId.value)
  })

  // Actions
  async function login(credentials: LoginCredentials) {
    loading.value = true
    error.value = null

    try {
      const { user, tokens: newTokens } = await authApi.login(credentials)

      tokens.value = newTokens
      tokenStorage.set(newTokens)

      userStore.setUser(user)
      currentUserId.value = user.id

      router.push('/dashboard')
    } catch (e) {
      error.value = (e as Error).message
      throw e
    } finally {
      loading.value = false
    }
  }

  async function register(data: RegisterData) {
    loading.value = true
    error.value = null

    try {
      const { user, tokens: newTokens } = await authApi.register(data)

      tokens.value = newTokens
      tokenStorage.set(newTokens)

      userStore.setUser(user)
      currentUserId.value = user.id

      router.push('/dashboard')
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

  async function checkAuth() {
    if (!tokens.value) return

    try {
      const user = await authApi.getCurrentUser()
      userStore.setUser(user)
      currentUserId.value = user.id
    } catch {
      logout()
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
    register,
    logout,
    checkAuth
  }
})
```

### Feature UI Component

```vue
<!-- features/auth/ui/LoginForm.vue -->
<script setup lang="ts">
import { ref } from 'vue'
import { Button, Input } from '@/shared/ui'
import { useAuthStore } from '../model/auth.store'
import type { LoginCredentials } from '../model/auth.types'

const authStore = useAuthStore()

const form = ref<LoginCredentials>({
  email: '',
  password: ''
})

async function handleSubmit() {
  try {
    await authStore.login(form.value)
  } catch {
    // Error is handled by store
  }
}
</script>

<template>
  <form class="login-form" @submit.prevent="handleSubmit">
    <h2 class="login-form__title">Login</h2>

    <div v-if="authStore.error" class="login-form__error">
      {{ authStore.error }}
    </div>

    <Input
      v-model="form.email"
      type="email"
      label="Email"
      placeholder="Enter your email"
      required
    />

    <Input
      v-model="form.password"
      type="password"
      label="Password"
      placeholder="Enter your password"
      required
    />

    <Button
      type="submit"
      :loading="authStore.loading"
      :disabled="authStore.loading"
      block
    >
      Sign In
    </Button>
  </form>
</template>
```

### Feature Public API

```typescript
// features/auth/index.ts
// Types
export type { LoginCredentials, RegisterData, AuthTokens } from './model/auth.types'

// Store
export { useAuthStore } from './model/auth.store'

// UI Components
export { default as LoginForm } from './ui/LoginForm.vue'
export { default as RegisterForm } from './ui/RegisterForm.vue'
export { default as LogoutButton } from './ui/LogoutButton.vue'
```

---

## More Entity Examples

### Product Entity

```typescript
// entities/product/index.ts
export type { Product, ProductCategory, ProductListResponse } from './model/product.types'
export { useProductStore } from './model/product.store'
export { productApi } from './api/product.api'
export { default as ProductCard } from './ui/ProductCard.vue'
export { default as ProductPrice } from './ui/ProductPrice.vue'
export { default as ProductRating } from './ui/ProductRating.vue'
```

### Order Entity

```typescript
// entities/order/index.ts
export type { Order, OrderStatus, OrderItem } from './model/order.types'
export { useOrderStore } from './model/order.store'
export { default as OrderCard } from './ui/OrderCard.vue'
export { default as OrderStatusBadge } from './ui/OrderStatusBadge.vue'
```

---

## More Feature Examples

### Add to Cart Feature

```typescript
// features/add-to-cart/index.ts
export { useCartStore } from './model/cart.store'
export { default as AddToCartButton } from './ui/AddToCartButton.vue'
export { default as CartQuantitySelector } from './ui/CartQuantitySelector.vue'
```

### Search Products Feature

```typescript
// features/search-products/index.ts
export { useSearchStore } from './model/search.store'
export { default as SearchInput } from './ui/SearchInput.vue'
export { default as SearchFilters } from './ui/SearchFilters.vue'
export { default as SearchResults } from './ui/SearchResults.vue'
```

### Create Order Feature

```typescript
// features/create-order/index.ts
export { useCheckoutStore } from './model/checkout.store'
export { default as CheckoutForm } from './ui/CheckoutForm.vue'
export { default as OrderSummary } from './ui/OrderSummary.vue'
```

---

## Entity vs Feature Decision Tree

```
Is it data/model that the app works with?
├── YES → Entity (User, Product, Order)
└── NO → Is it a user action/interaction?
    ├── YES → Feature (Login, AddToCart, Search)
    └── NO → Is it a composed UI block?
        ├── YES → Widget (Header, Sidebar, ProductList)
        └── NO → Shared (utilities, UI kit)
```

## Quick Reference

| Pattern | Entity | Feature |
|---------|--------|---------|
| Type | Noun | Verb |
| Focus | Data representation | User interaction |
| Store | CRUD operations | Business logic |
| UI | Display components | Action components |
| Imports | shared only | entities, shared |
| Examples | User, Product, Order | Auth, AddToCart, Search |
