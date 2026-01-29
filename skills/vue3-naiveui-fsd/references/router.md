# Router Configuration

Vue Router setup with auth guards and loading bar.

## Router Setup

```typescript
// src/app/router/index.ts
import { createRouter, createWebHistory, type RouteRecordRaw } from 'vue-router'
import { createDiscreteApi } from 'naive-ui'
import Cookies from 'js-cookie'
import { useAuthStore } from '@/app/store/auth'

// Create discrete API for loading bar outside of Vue context
const { loadingBar } = createDiscreteApi(['loadingBar'])

const routes: RouteRecordRaw[] = [
  // Public routes
  {
    path: '/auth/login',
    name: 'login',
    component: () => import('@/pages/auth/LoginPage.vue'),
    meta: { guest: true },
  },

  // Protected routes
  {
    path: '/',
    component: () => import('@/app/layouts/main/MainLayout.vue'),
    meta: { requiresAuth: true },
    children: [
      {
        path: '',
        name: 'dashboard',
        component: () => import('@/pages/DashboardPage.vue'),
      },
      {
        path: 'user',
        name: 'user-list',
        component: () => import('@/pages/user/UserListPage.vue'),
        meta: { permission: 'users.view' },
      },
      {
        path: 'user/:id',
        name: 'user-detail',
        component: () => import('@/pages/user/UserDetailPage.vue'),
        meta: { permission: 'users.view' },
      },
      {
        path: 'client',
        name: 'client-list',
        component: () => import('@/pages/client/ClientListPage.vue'),
        meta: { permission: 'clients.view' },
      },
      {
        path: 'client/:id',
        name: 'client-detail',
        component: () => import('@/pages/client/ClientDetailPage.vue'),
        meta: { permission: 'clients.view' },
      },
      {
        path: 'contract',
        name: 'contract-list',
        component: () => import('@/pages/contract/ContractListPage.vue'),
        meta: { permission: 'contracts.view' },
      },
      {
        path: 'settings',
        name: 'settings',
        component: () => import('@/pages/settings/SettingsPage.vue'),
      },
    ],
  },

  // 404
  {
    path: '/:pathMatch(.*)*',
    name: 'not-found',
    component: () => import('@/pages/NotFoundPage.vue'),
  },
]

const router = createRouter({
  history: createWebHistory(import.meta.env.BASE_URL),
  routes,
  scrollBehavior(to, from, savedPosition) {
    if (savedPosition) {
      return savedPosition
    }
    return { top: 0 }
  },
})

// Track first load for initial user fetch
let isFirstLoad = true

// Navigation guards
router.beforeEach(async (to, from, next) => {
  // Start loading bar
  loadingBar.start()

  const token = Cookies.get('token')
  const requiresAuth = to.matched.some((record) => record.meta.requiresAuth)
  const isGuestRoute = to.matched.some((record) => record.meta.guest)

  // Redirect to login if auth required but no token
  if (requiresAuth && !token) {
    loadingBar.finish()
    next({ name: 'login', query: { redirect: to.fullPath } })
    return
  }

  // Redirect to dashboard if logged in and trying to access guest route
  if (isGuestRoute && token) {
    loadingBar.finish()
    next({ name: 'dashboard' })
    return
  }

  // Fetch user on first authenticated navigation
  if (token && isFirstLoad) {
    isFirstLoad = false
    const authStore = useAuthStore()
    await authStore.fetchUser()
  }

  // Check permission
  const requiredPermission = to.meta.permission as string | undefined
  if (requiredPermission && token) {
    const authStore = useAuthStore()
    if (!authStore.can(requiredPermission)) {
      loadingBar.error()
      next({ name: 'dashboard' })
      return
    }
  }

  next()
})

router.afterEach((to, from, failure) => {
  if (failure) {
    loadingBar.error()
  } else {
    loadingBar.finish()
  }
})

// Handle navigation errors
router.onError((error) => {
  loadingBar.error()
  console.error('Router error:', error)
})

export default router
```

## Route Meta Types

```typescript
// src/app/router/types.ts
import 'vue-router'

declare module 'vue-router' {
  interface RouteMeta {
    requiresAuth?: boolean
    guest?: boolean
    permission?: string
    permissions?: string[]
    title?: string
    breadcrumb?: string | ((route: RouteLocationNormalized) => string)
  }
}
```

## Route Guards Helper

```typescript
// src/app/router/guards.ts
import type { NavigationGuardWithThis } from 'vue-router'
import { useAuthStore } from '@/app/store/auth'

export const requireAuth: NavigationGuardWithThis<undefined> = (to, from, next) => {
  const authStore = useAuthStore()

  if (!authStore.isAuthenticated) {
    next({ name: 'login', query: { redirect: to.fullPath } })
    return
  }

  next()
}

export const requireGuest: NavigationGuardWithThis<undefined> = (to, from, next) => {
  const authStore = useAuthStore()

  if (authStore.isAuthenticated) {
    next({ name: 'dashboard' })
    return
  }

  next()
}

export const requirePermission = (permission: string): NavigationGuardWithThis<undefined> => {
  return (to, from, next) => {
    const authStore = useAuthStore()

    if (!authStore.can(permission)) {
      next({ name: 'dashboard' })
      return
    }

    next()
  }
}
```

## Breadcrumbs

```vue
<!-- src/app/layouts/main/ui/Breadcrumbs.vue -->
<script setup lang="ts">
import { computed } from 'vue'
import { useRoute, useRouter } from 'vue-router'
import { NBreadcrumb, NBreadcrumbItem } from 'naive-ui'

interface BreadcrumbItem {
  label: string
  path?: string
}

const route = useRoute()
const router = useRouter()

const breadcrumbs = computed<BreadcrumbItem[]>(() => {
  const items: BreadcrumbItem[] = [
    { label: 'Home', path: '/' },
  ]

  for (const matched of route.matched) {
    if (matched.meta.breadcrumb) {
      const label = typeof matched.meta.breadcrumb === 'function'
        ? matched.meta.breadcrumb(route)
        : matched.meta.breadcrumb

      items.push({
        label,
        path: matched.path === route.path ? undefined : matched.path,
      })
    }
  }

  return items
})

function handleClick(path?: string): void {
  if (path) {
    router.push(path)
  }
}
</script>

<template>
  <NBreadcrumb>
    <NBreadcrumbItem
      v-for="(item, index) in breadcrumbs"
      :key="index"
      :clickable="!!item.path"
      @click="handleClick(item.path)"
    >
      {{ item.label }}
    </NBreadcrumbItem>
  </NBreadcrumb>
</template>
```

## Navigation Utilities

```typescript
// src/shared/lib/navigation.ts
import router from '@/app/router'

/**
 * Navigate to route with query params
 */
export function navigateTo(
  name: string,
  params?: Record<string, string | number>,
  query?: Record<string, string>
): void {
  router.push({ name, params, query })
}

/**
 * Navigate back or to fallback
 */
export function goBack(fallback = '/'): void {
  if (window.history.length > 1) {
    router.back()
  } else {
    router.push(fallback)
  }
}

/**
 * Redirect after login
 */
export function redirectAfterLogin(): void {
  const redirect = router.currentRoute.value.query.redirect as string
  router.push(redirect || '/')
}

/**
 * Open in new tab
 */
export function openInNewTab(name: string, params?: Record<string, string | number>): void {
  const resolved = router.resolve({ name, params })
  window.open(resolved.href, '_blank')
}
```

## Route-Based Page Title

```typescript
// src/app/router/pageTitle.ts
import { watch } from 'vue'
import { useRoute } from 'vue-router'
import { useI18n } from 'vue-i18n'

export function usePageTitle() {
  const route = useRoute()
  const { t } = useI18n()

  watch(
    () => route.meta.title,
    (title) => {
      const appName = 'MKO'
      if (title) {
        document.title = `${t(title as string)} | ${appName}`
      } else {
        document.title = appName
      }
    },
    { immediate: true }
  )
}
```

Usage in App.vue:

```vue
<script setup lang="ts">
import { usePageTitle } from '@/app/router/pageTitle'

usePageTitle()
</script>
```

## Lazy Loading with Loading State

```typescript
// For routes with heavy components
{
  path: 'report',
  name: 'report',
  component: () => import('@/pages/report/ReportPage.vue'),
  meta: {
    requiresAuth: true,
    title: 'pages.report',
  },
}
```

## Route Transitions

```vue
<!-- In MainLayout.vue -->
<template>
  <RouterView v-slot="{ Component, route }">
    <Transition name="fade" mode="out-in">
      <component :is="Component" :key="route.path" />
    </Transition>
  </RouterView>
</template>

<style>
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.2s ease;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}
</style>
```
