# Composables

Reusable composition functions for Vue 3 projects.

## usePagination

URL query synced pagination state.

```typescript
// src/shared/composables/usePagination.ts
import { ref, watch } from 'vue'
import { useRoute, useRouter } from 'vue-router'

interface UsePaginationOptions {
  defaultPerPage?: number
}

export function usePagination(options: UsePaginationOptions = {}) {
  const { defaultPerPage = 15 } = options

  const route = useRoute()
  const router = useRouter()

  // Initialize from URL query
  const page = ref(Number(route.query.page) || 1)
  const perPage = ref(Number(route.query.per_page) || defaultPerPage)
  const total = ref(0)
  const sortBy = ref<string | undefined>(route.query.sortBy as string)
  const descending = ref(route.query.descending === 'true')

  // Sync state to URL
  watch([page, perPage, sortBy, descending], () => {
    router.replace({
      query: {
        ...route.query,
        page: page.value.toString(),
        per_page: perPage.value.toString(),
        sortBy: sortBy.value || undefined,
        descending: descending.value ? 'true' : undefined,
      },
    })
  })

  // Reset page when perPage changes
  watch(perPage, () => {
    page.value = 1
  })

  function setTotal(value: number): void {
    total.value = value
  }

  function resetPagination(): void {
    page.value = 1
  }

  // Computed pagination for NDataTable
  const pagination = computed(() => ({
    page: page.value,
    pageSize: perPage.value,
    itemCount: total.value,
    showSizePicker: true,
    pageSizes: [10, 15, 25, 50],
  }))

  return {
    page,
    perPage,
    total,
    sortBy,
    descending,
    pagination,
    setTotal,
    resetPagination,
  }
}
```

### Usage

```vue
<script setup lang="ts">
import { watch, onMounted } from 'vue'
import { usePagination } from '@/shared/composables'

const { page, perPage, total, setTotal, resetPagination } = usePagination()

async function fetchData() {
  const response = await UserService.getList({
    page: page.value,
    per_page: perPage.value,
  })
  users.value = response.data
  setTotal(response.meta.total)
}

// Refetch on pagination change
watch([page, perPage], fetchData)

// Reset page on search
watch(search, () => {
  resetPagination()
  fetchData()
})

onMounted(fetchData)
</script>
```

## useValidationRules

i18n integrated form validation rules.

```typescript
// src/shared/composables/useValidationRules.ts
import { useI18n } from 'vue-i18n'
import type { FormItemRule } from 'naive-ui'

export function useValidationRules() {
  const { t } = useI18n()

  const requiredRule = (field: string): FormItemRule => ({
    required: true,
    message: t('validation.required', { field }),
    trigger: ['blur', 'input'],
  })

  const emailRule: FormItemRule = {
    type: 'email',
    message: t('validation.email'),
    trigger: ['blur', 'input'],
  }

  const minLengthRule = (min: number): FormItemRule => ({
    min,
    message: t('validation.minLength', { min }),
    trigger: ['blur', 'input'],
  })

  const maxLengthRule = (max: number): FormItemRule => ({
    max,
    message: t('validation.maxLength', { max }),
    trigger: ['blur', 'input'],
  })

  const phoneRules: FormItemRule[] = [
    {
      required: true,
      message: t('validation.required', { field: t('common.phone') }),
      trigger: ['blur', 'input'],
    },
    {
      pattern: /^\+998\d{9}$/,
      message: t('validation.phone'),
      trigger: ['blur', 'input'],
    },
  ]

  const passwordRules: FormItemRule[] = [
    {
      required: true,
      message: t('validation.required', { field: t('common.password') }),
      trigger: ['blur', 'input'],
    },
    {
      min: 8,
      message: t('validation.minLength', { min: 8 }),
      trigger: ['blur', 'input'],
    },
  ]

  const innRules: FormItemRule[] = [
    {
      required: true,
      message: t('validation.required', { field: 'INN' }),
      trigger: ['blur', 'input'],
    },
    {
      pattern: /^\d{9}$/,
      message: t('validation.inn'),
      trigger: ['blur', 'input'],
    },
  ]

  const numberRule = (field: string): FormItemRule => ({
    type: 'number',
    required: true,
    message: t('validation.number', { field }),
    trigger: ['blur', 'change'],
  })

  const positiveNumberRule = (field: string): FormItemRule => ({
    type: 'number',
    required: true,
    validator: (rule, value) => value > 0,
    message: t('validation.positive', { field }),
    trigger: ['blur', 'change'],
  })

  return {
    requiredRule,
    emailRule,
    minLengthRule,
    maxLengthRule,
    phoneRules,
    passwordRules,
    innRules,
    numberRule,
    positiveNumberRule,
  }
}
```

### Usage

```typescript
import { useValidationRules } from '@/shared/composables'
import type { FormRules } from 'naive-ui'

const { requiredRule, emailRule, minLengthRule } = useValidationRules()

const rules: FormRules = {
  name: [requiredRule('Name'), minLengthRule(2)],
  email: [requiredRule('Email'), emailRule],
  phone: phoneRules,
  amount: [positiveNumberRule('Amount')],
}
```

## useTheme

Dark/Light theme management.

```typescript
// src/shared/composables/useTheme.ts
import { ref, computed, watchEffect } from 'vue'
import { darkTheme, type GlobalThemeOverrides } from 'naive-ui'

type ThemeMode = 'light' | 'dark'

const STORAGE_KEY = 'app-theme'

// Singleton state
const mode = ref<ThemeMode>(
  (localStorage.getItem(STORAGE_KEY) as ThemeMode) || 'light'
)

export function useTheme() {
  const isDark = computed(() => mode.value === 'dark')

  const theme = computed(() => (isDark.value ? darkTheme : null))

  const themeOverrides = computed<GlobalThemeOverrides>(() => ({
    common: {
      primaryColor: '#6366f1',
      primaryColorHover: '#818cf8',
      primaryColorPressed: '#4f46e5',
      primaryColorSuppl: '#6366f1',
      borderRadius: '8px',
      fontFamily: 'Inter, -apple-system, BlinkMacSystemFont, sans-serif',
    },
    Button: {
      fontWeight: '500',
    },
    Card: {
      borderRadius: '12px',
    },
    Input: {
      borderRadius: '8px',
    },
  }))

  function toggleTheme(): void {
    mode.value = isDark.value ? 'light' : 'dark'
  }

  function setTheme(newMode: ThemeMode): void {
    mode.value = newMode
  }

  // Persist and apply theme
  watchEffect(() => {
    localStorage.setItem(STORAGE_KEY, mode.value)
    document.documentElement.classList.toggle('dark', isDark.value)
  })

  return {
    mode,
    isDark,
    theme,
    themeOverrides,
    toggleTheme,
    setTheme,
  }
}
```

### Usage in App.vue

```vue
<script setup lang="ts">
import { NConfigProvider } from 'naive-ui'
import { useTheme } from '@/shared/composables'

const { theme, themeOverrides } = useTheme()
</script>

<template>
  <NConfigProvider :theme="theme" :theme-overrides="themeOverrides">
    <RouterView />
  </NConfigProvider>
</template>
```

## useDebouncedRef

Debounced reactive value.

```typescript
// src/shared/composables/useDebouncedRef.ts
import { ref, watch, type Ref } from 'vue'

export function useDebouncedRef<T>(
  initialValue: T,
  delay = 300
): [Ref<T>, Ref<T>] {
  const value = ref(initialValue) as Ref<T>
  const debouncedValue = ref(initialValue) as Ref<T>

  let timeout: ReturnType<typeof setTimeout>

  watch(value, (newValue) => {
    clearTimeout(timeout)
    timeout = setTimeout(() => {
      debouncedValue.value = newValue
    }, delay)
  })

  return [value, debouncedValue]
}
```

### Usage

```vue
<script setup lang="ts">
import { watch } from 'vue'
import { useDebouncedRef } from '@/shared/composables'

const [search, debouncedSearch] = useDebouncedRef('', 500)

// Fetch when debounced value changes
watch(debouncedSearch, (value) => {
  fetchUsers({ search: value })
})
</script>

<template>
  <NInput v-model:value="search" placeholder="Search..." />
</template>
```

## useUnsavedChanges

Detect unsaved form changes.

```typescript
// src/shared/composables/useUnsavedChanges.ts
import { ref, watch, onBeforeUnmount } from 'vue'
import { onBeforeRouteLeave } from 'vue-router'

export function useUnsavedChanges<T extends object>(initialData: T) {
  const originalData = ref(JSON.stringify(initialData))
  const hasChanges = ref(false)

  function checkChanges(currentData: T): void {
    hasChanges.value = JSON.stringify(currentData) !== originalData.value
  }

  function resetChanges(newData: T): void {
    originalData.value = JSON.stringify(newData)
    hasChanges.value = false
  }

  // Warn before leaving page
  onBeforeRouteLeave((to, from, next) => {
    if (hasChanges.value) {
      const answer = window.confirm(
        'You have unsaved changes. Are you sure you want to leave?'
      )
      if (!answer) {
        next(false)
        return
      }
    }
    next()
  })

  // Warn before closing tab
  const handleBeforeUnload = (e: BeforeUnloadEvent) => {
    if (hasChanges.value) {
      e.preventDefault()
      e.returnValue = ''
    }
  }

  window.addEventListener('beforeunload', handleBeforeUnload)

  onBeforeUnmount(() => {
    window.removeEventListener('beforeunload', handleBeforeUnload)
  })

  return {
    hasChanges,
    checkChanges,
    resetChanges,
  }
}
```

## useSharedBreakpoints

Responsive breakpoint helpers.

```typescript
// src/shared/composables/useSharedBreakpoints.ts
import { computed } from 'vue'
import { useBreakpoints } from '@vueuse/core'

const breakpoints = useBreakpoints({
  sm: 640,
  md: 768,
  lg: 1024,
  xl: 1280,
  '2xl': 1536,
})

export function useSharedBreakpoints() {
  const isMobile = computed(() => breakpoints.smaller('md').value)
  const isTablet = computed(() => breakpoints.between('md', 'lg').value)
  const isDesktop = computed(() => breakpoints.greater('lg').value)

  const currentBreakpoint = computed(() => {
    if (breakpoints.smaller('sm').value) return 'xs'
    if (breakpoints.between('sm', 'md').value) return 'sm'
    if (breakpoints.between('md', 'lg').value) return 'md'
    if (breakpoints.between('lg', 'xl').value) return 'lg'
    if (breakpoints.between('xl', '2xl').value) return 'xl'
    return '2xl'
  })

  return {
    isMobile,
    isTablet,
    isDesktop,
    currentBreakpoint,
    breakpoints,
  }
}
```

## Composables Index

```typescript
// src/shared/composables/index.ts
export { usePagination } from './usePagination'
export { useValidationRules } from './useValidationRules'
export { useTheme } from './useTheme'
export { useDebouncedRef } from './useDebouncedRef'
export { useUnsavedChanges } from './useUnsavedChanges'
export { useSharedBreakpoints } from './useSharedBreakpoints'
```
