# Pinia Stores

Global store patterns for Vue 3 + Pinia in FSD architecture.

## Store Location

Global stores live in `app/store/`:

```
src/app/store/
├── auth.ts       # Authentication & user state
├── operation.ts  # Business operation state
└── index.ts      # Store exports
```

## Auth Store

```typescript
// src/app/store/auth.ts
import { defineStore } from 'pinia'
import { ref, computed } from 'vue'
import Cookies from 'js-cookie'
import type { IUser, LoginDto, LoginResponseDto } from '@/shared/types/auth'
import { post, get } from '@/shared/api/base'
import router from '@/app/router'

export const useAuthStore = defineStore('auth', () => {
  // ============================================
  // State
  // ============================================
  const user = ref<IUser | null>(null)
  const isLoading = ref(false)

  // ============================================
  // Getters
  // ============================================
  const isAuthenticated = computed(() => !!Cookies.get('token'))

  const isAdmin = computed(() => user.value?.role?.name === 'admin')

  const userName = computed(() => user.value?.name ?? '')

  const userPermissions = computed(() => user.value?.permissions ?? [])

  // Permission check function
  function can(permission: string): boolean {
    if (!user.value) return false
    if (isAdmin.value) return true // Admin has all permissions
    return userPermissions.value.includes(permission)
  }

  // Multiple permissions check
  function canAny(permissions: string[]): boolean {
    return permissions.some((p) => can(p))
  }

  function canAll(permissions: string[]): boolean {
    return permissions.every((p) => can(p))
  }

  // ============================================
  // Actions
  // ============================================
  async function login(credentials: LoginDto): Promise<void> {
    isLoading.value = true
    try {
      const response = await post<LoginResponseDto>('/auth/login', credentials)
      Cookies.set('token', response.token, { expires: 7 })
      user.value = response.user
      await router.push('/')
    } finally {
      isLoading.value = false
    }
  }

  async function logout(): Promise<void> {
    try {
      await post('/auth/logout')
    } catch {
      // Ignore logout errors
    } finally {
      Cookies.remove('token')
      user.value = null
      await router.push('/auth/login')
    }
  }

  async function fetchUser(): Promise<void> {
    if (!isAuthenticated.value) return
    try {
      user.value = await get<IUser>('/auth/me')
    } catch {
      await logout()
    }
  }

  async function updateProfile(data: Partial<IUser>): Promise<void> {
    isLoading.value = true
    try {
      user.value = await post<IUser>('/auth/profile', data)
    } finally {
      isLoading.value = false
    }
  }

  // ============================================
  // Return
  // ============================================
  return {
    // State
    user,
    isLoading,
    // Getters
    isAuthenticated,
    isAdmin,
    userName,
    userPermissions,
    // Methods
    can,
    canAny,
    canAll,
    // Actions
    login,
    logout,
    fetchUser,
    updateProfile,
  }
})
```

## Operation Store

```typescript
// src/app/store/operation.ts
import { defineStore } from 'pinia'
import { ref, computed, onMounted } from 'vue'
import { get, post } from '@/shared/api/base'

interface IOperationDay {
  id: number
  date: string
  status: OperationStatus
  opened_at: string | null
  closed_at: string | null
}

type OperationStatus = 'open' | 'closed' | 'calculating'

export const useOperationStore = defineStore('operation', () => {
  // ============================================
  // State
  // ============================================
  const currentDay = ref<IOperationDay | null>(null)
  const isLoading = ref(false)

  // ============================================
  // Getters
  // ============================================
  const isOpen = computed(() => currentDay.value?.status === 'open')

  const isClosed = computed(() => currentDay.value?.status === 'closed')

  const isCalculating = computed(() => currentDay.value?.status === 'calculating')

  const canPerformOperations = computed(() => isOpen.value)

  const statusLabel = computed(() => {
    if (!currentDay.value) return 'N/A'
    const labels: Record<OperationStatus, string> = {
      open: 'Ochiq',
      closed: 'Yopiq',
      calculating: 'Hisoblanyapti',
    }
    return labels[currentDay.value.status]
  })

  // ============================================
  // Actions
  // ============================================
  async function fetchCurrentDay(): Promise<void> {
    isLoading.value = true
    try {
      currentDay.value = await get<IOperationDay>('/operation/current')
    } catch {
      currentDay.value = null
    } finally {
      isLoading.value = false
    }
  }

  async function openDay(): Promise<void> {
    isLoading.value = true
    try {
      currentDay.value = await post<IOperationDay>('/operation/open')
    } finally {
      isLoading.value = false
    }
  }

  async function closeDay(): Promise<void> {
    isLoading.value = true
    try {
      currentDay.value = await post<IOperationDay>('/operation/close')
    } finally {
      isLoading.value = false
    }
  }

  // Auto-fetch on store creation
  fetchCurrentDay()

  // ============================================
  // Return
  // ============================================
  return {
    // State
    currentDay,
    isLoading,
    // Getters
    isOpen,
    isClosed,
    isCalculating,
    canPerformOperations,
    statusLabel,
    // Actions
    fetchCurrentDay,
    openDay,
    closeDay,
  }
})
```

## Store Index

```typescript
// src/app/store/index.ts
export { useAuthStore } from './auth'
export { useOperationStore } from './operation'
```

## Usage in Components

```vue
<script setup lang="ts">
import { useAuthStore, useOperationStore } from '@/app/store'

const authStore = useAuthStore()
const operationStore = useOperationStore()

// Access state
const userName = authStore.userName
const isOpen = operationStore.isOpen

// Check permissions
const canEdit = authStore.can('users.edit')
const canDelete = authStore.can('users.delete')

// Perform actions
async function handleLogout() {
  await authStore.logout()
}

async function handleOpenDay() {
  await operationStore.openDay()
}
</script>

<template>
  <div>
    <p>Welcome, {{ authStore.userName }}</p>
    <p>Operation day: {{ operationStore.statusLabel }}</p>

    <NButton v-if="canEdit" @click="handleEdit">
      Edit
    </NButton>

    <NButton @click="handleLogout">
      Logout
    </NButton>
  </div>
</template>
```

## Permission-Based Rendering

```vue
<script setup lang="ts">
import { useAuthStore } from '@/app/store'

const authStore = useAuthStore()
</script>

<template>
  <!-- Single permission -->
  <NButton v-if="authStore.can('users.create')" @click="handleAdd">
    Add User
  </NButton>

  <!-- Any of permissions -->
  <div v-if="authStore.canAny(['users.edit', 'users.delete'])">
    <EditBtn v-if="authStore.can('users.edit')" />
    <DeleteBtn v-if="authStore.can('users.delete')" />
  </div>

  <!-- Admin only -->
  <AdminPanel v-if="authStore.isAdmin" />
</template>
```

## Store with Entity (if needed)

For entity-specific stores, place them in the entity's model folder:

```typescript
// src/entities/notification/model/notificationStore.ts
import { defineStore } from 'pinia'
import { ref, computed } from 'vue'
import { get, post } from '@/shared/api/base'
import type { INotification } from './types'

export const useNotificationStore = defineStore('notification', () => {
  const notifications = ref<INotification[]>([])
  const isLoading = ref(false)

  const unreadCount = computed(() =>
    notifications.value.filter((n) => !n.read_at).length
  )

  const hasUnread = computed(() => unreadCount.value > 0)

  async function fetchNotifications(): Promise<void> {
    isLoading.value = true
    try {
      notifications.value = await get<INotification[]>('/notifications')
    } finally {
      isLoading.value = false
    }
  }

  async function markAsRead(id: number): Promise<void> {
    await post(`/notifications/${id}/read`)
    const notification = notifications.value.find((n) => n.id === id)
    if (notification) {
      notification.read_at = new Date().toISOString()
    }
  }

  async function markAllAsRead(): Promise<void> {
    await post('/notifications/read-all')
    notifications.value.forEach((n) => {
      n.read_at = new Date().toISOString()
    })
  }

  return {
    notifications,
    isLoading,
    unreadCount,
    hasUnread,
    fetchNotifications,
    markAsRead,
    markAllAsRead,
  }
})
```

## Testing Stores

```typescript
// src/app/store/__tests__/auth.spec.ts
import { describe, it, expect, beforeEach, vi } from 'vitest'
import { setActivePinia, createPinia } from 'pinia'
import { useAuthStore } from '../auth'
import * as api from '@/shared/api/base'

vi.mock('@/shared/api/base')

describe('authStore', () => {
  beforeEach(() => {
    setActivePinia(createPinia())
    vi.clearAllMocks()
  })

  it('login sets user and token', async () => {
    const mockResponse = {
      token: 'test-token',
      user: { id: 1, name: 'Test User' },
    }
    vi.mocked(api.post).mockResolvedValue(mockResponse)

    const store = useAuthStore()
    await store.login({ phone: '+998901234567', password: 'password' })

    expect(store.user).toEqual(mockResponse.user)
    expect(store.isAuthenticated).toBe(true)
  })

  it('can() returns true for admin', () => {
    const store = useAuthStore()
    store.user = { id: 1, name: 'Admin', role: { name: 'admin' } } as any

    expect(store.can('any.permission')).toBe(true)
  })

  it('can() checks user permissions', () => {
    const store = useAuthStore()
    store.user = {
      id: 1,
      name: 'User',
      role: { name: 'user' },
      permissions: ['users.view', 'users.edit'],
    } as any

    expect(store.can('users.view')).toBe(true)
    expect(store.can('users.delete')).toBe(false)
  })
})
```
