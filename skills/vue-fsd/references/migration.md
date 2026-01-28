# Migration Guide

> Reference for: Vue FSD Expert
> Load when: Migrating existing projects to FSD, refactoring, gradual adoption

## When to Use FSD

### Good Fit
- Medium to large projects (10+ pages)
- Multiple developers/teams
- Long-term maintenance expected
- Complex business domain
- Need clear code ownership

### Not Recommended
- Small projects (< 5 pages)
- Prototypes or MVPs
- Single developer projects
- Simple CRUD apps

---

## Migration Strategies

### Strategy 1: Greenfield (New Project)

Start fresh with FSD structure:

```bash
# Create new Vue project
npm create vue@latest my-app -- --typescript

# Create FSD structure
cd my-app
mkdir -p src/{app,pages,widgets,features,entities,shared}
mkdir -p src/shared/{api,config,lib,ui,types}
```

### Strategy 2: Gradual Migration (Recommended)

Migrate piece by piece while keeping the app working.

**Phase 1: Setup shared layer**
```
src/
├── components/      # Old components
├── views/           # Old views
├── store/           # Old Vuex/Pinia
├── shared/          # NEW - Start here
│   ├── api/
│   ├── config/
│   ├── lib/
│   └── ui/
└── ...
```

**Phase 2: Extract entities**
```
src/
├── components/
├── views/
├── store/
├── shared/
├── entities/        # NEW - Extract domain models
│   ├── user/
│   └── product/
└── ...
```

**Phase 3: Extract features**
```
src/
├── components/      # Getting smaller
├── views/
├── shared/
├── entities/
├── features/        # NEW - Extract user actions
│   ├── auth/
│   └── cart/
└── ...
```

**Phase 4: Convert pages**
```
src/
├── app/             # NEW - App initialization
├── pages/           # NEW - Converted from views
├── widgets/         # NEW - Extracted from components
├── features/
├── entities/
└── shared/
```

### Strategy 3: Module-by-Module

Migrate one business domain at a time:

1. Start with `auth` feature (login/logout)
2. Then `user` entity
3. Then related pages
4. Repeat for next domain

---

## Step-by-Step Migration

### Step 1: Create shared/ui

Move generic UI components:

```
# Before
src/components/
├── BaseButton.vue
├── BaseInput.vue
├── BaseModal.vue
└── ...

# After
src/shared/ui/
├── Button/
│   ├── Button.vue
│   └── index.ts
├── Input/
│   ├── Input.vue
│   └── index.ts
├── Modal/
│   ├── Modal.vue
│   └── index.ts
└── index.ts
```

Update imports:
```typescript
// Before
import BaseButton from '@/components/BaseButton.vue'

// After
import { Button } from '@/shared/ui'
```

### Step 2: Create shared/lib

Move utility functions:

```
# Before
src/utils/
├── formatDate.ts
├── formatCurrency.ts
└── debounce.ts

# After
src/shared/lib/
├── date.ts        # formatDate
├── number.ts      # formatCurrency
├── async.ts       # debounce
└── index.ts
```

### Step 3: Create shared/api

Move API client:

```
# Before
src/api/
├── client.ts
└── interceptors.ts

# After
src/shared/api/
├── client.ts
├── types.ts
└── index.ts
```

### Step 4: Extract First Entity

Choose a simple entity to start:

```
# Before (Vuex)
src/store/modules/user.ts

# After
src/entities/user/
├── api/
│   └── user.api.ts
├── model/
│   ├── user.types.ts
│   └── user.store.ts
├── ui/
│   ├── UserCard.vue
│   └── UserAvatar.vue
└── index.ts
```

Migration example:

```typescript
// Before: src/store/modules/user.ts (Vuex)
export default {
  state: () => ({
    users: [],
    currentUser: null
  }),
  mutations: {
    SET_USERS(state, users) {
      state.users = users
    }
  },
  actions: {
    async fetchUsers({ commit }) {
      const users = await api.getUsers()
      commit('SET_USERS', users)
    }
  }
}

// After: src/entities/user/model/user.store.ts (Pinia + FSD)
import { defineStore } from 'pinia'
import { ref, computed } from 'vue'
import type { User } from './user.types'
import { userApi } from '../api/user.api'

export const useUserStore = defineStore('entities/user', () => {
  const users = ref<Map<number, User>>(new Map())
  const loading = ref(false)

  const userList = computed(() => Array.from(users.value.values()))

  async function fetchUsers() {
    loading.value = true
    try {
      const data = await userApi.getList()
      data.forEach(user => users.value.set(user.id, user))
    } finally {
      loading.value = false
    }
  }

  return { users, loading, userList, fetchUsers }
})
```

### Step 5: Extract First Feature

Choose a feature that uses the entity:

```
# Before
src/components/LoginForm.vue
src/store/modules/auth.ts

# After
src/features/auth/
├── api/
│   └── auth.api.ts
├── model/
│   ├── auth.types.ts
│   └── auth.store.ts
├── ui/
│   ├── LoginForm.vue
│   └── LogoutButton.vue
└── index.ts
```

### Step 6: Convert Pages

Convert views to pages:

```
# Before
src/views/
├── Home.vue
├── Catalog.vue
└── Product.vue

# After
src/pages/
├── home/
│   ├── ui/HomePage.vue
│   └── index.ts
├── catalog/
│   ├── ui/CatalogPage.vue
│   └── index.ts
└── product/
    ├── ui/ProductPage.vue
    └── index.ts
```

Update router:

```typescript
// Before
const routes = [
  { path: '/', component: () => import('@/views/Home.vue') }
]

// After
const routes = [
  { path: '/', component: () => import('@/pages/home').then(m => m.HomePage) }
]
```

### Step 7: Extract Widgets

Identify composed UI blocks:

```
# Before
src/components/
├── TheHeader.vue      # Uses auth, cart, search
├── TheSidebar.vue     # Uses navigation, user
└── ProductList.vue    # Uses product cards, cart

# After
src/widgets/
├── header/
│   ├── ui/Header.vue
│   └── index.ts
├── sidebar/
│   ├── ui/Sidebar.vue
│   └── index.ts
└── product-list/
    ├── ui/ProductList.vue
    └── index.ts
```

### Step 8: Setup App Layer

Finalize app initialization:

```
src/app/
├── providers/
│   └── index.ts
├── router/
│   ├── index.ts
│   └── guards.ts
├── styles/
│   └── global.css
├── App.vue
└── main.ts
```

---

## Common Migration Challenges

### Challenge 1: Circular Dependencies

**Problem:** Entity A imports Entity B, and B imports A.

**Solution:** Create a feature for cross-entity logic.

```typescript
// ❌ Before: Circular dependency
// entities/user/model/user.store.ts
import { useOrderStore } from '@/entities/order' // User imports Order

// entities/order/model/order.store.ts
import { useUserStore } from '@/entities/user' // Order imports User

// ✅ After: Extract to feature
// features/user-orders/model/user-orders.store.ts
import { useUserStore } from '@/entities/user'
import { useOrderStore } from '@/entities/order'

// Feature handles the cross-entity logic
```

### Challenge 2: Large Components

**Problem:** Component does too much (display + logic + API).

**Solution:** Split into entity UI + feature logic.

```vue
<!-- ❌ Before: Everything in one component -->
<script setup>
const user = ref(null)
const loading = ref(false)

async function login(credentials) {
  loading.value = true
  const response = await api.login(credentials)
  user.value = response.user
  localStorage.setItem('token', response.token)
  router.push('/dashboard')
}
</script>

<!-- ✅ After: Split responsibilities -->
<!-- features/auth/ui/LoginForm.vue - Just the form -->
<script setup>
import { useAuthStore } from '../model/auth.store'
import { Input, Button } from '@/shared/ui'

const authStore = useAuthStore()
const form = ref({ email: '', password: '' })

async function handleSubmit() {
  await authStore.login(form.value)
}
</script>
```

### Challenge 3: Global State

**Problem:** App-wide state doesn't fit in entities/features.

**Solution:** Keep in app layer or create `processes` layer.

```typescript
// app/providers/global-state.ts
import { ref, provide, inject } from 'vue'

const GLOBAL_STATE_KEY = Symbol('global-state')

export function provideGlobalState() {
  const theme = ref<'light' | 'dark'>('light')
  const locale = ref('en')

  provide(GLOBAL_STATE_KEY, { theme, locale })
}

export function useGlobalState() {
  return inject(GLOBAL_STATE_KEY)!
}
```

---

## Migration Checklist

### Before Starting
- [ ] Document current architecture
- [ ] Identify all business domains (entities)
- [ ] Identify all user actions (features)
- [ ] Get team buy-in

### During Migration
- [ ] Setup path aliases in vite.config.ts
- [ ] Setup ESLint rules for imports
- [ ] Create shared/ui with basic components
- [ ] Create shared/lib with utilities
- [ ] Create shared/api with HTTP client
- [ ] Migrate entities one by one
- [ ] Migrate features one by one
- [ ] Convert pages
- [ ] Extract widgets

### After Migration
- [ ] Remove old folders (components/, views/, store/)
- [ ] Update documentation
- [ ] Train team on FSD principles
- [ ] Setup CI checks for import rules

---

## Quick Reference

| Stage | Action |
|-------|--------|
| 1 | Create shared layer (ui, lib, api) |
| 2 | Extract entities (domain models) |
| 3 | Extract features (user actions) |
| 4 | Convert pages |
| 5 | Extract widgets |
| 6 | Setup app layer |
| 7 | Remove legacy folders |
