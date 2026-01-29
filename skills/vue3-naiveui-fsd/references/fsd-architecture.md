# FSD Architecture

Feature-Sliced Design (FSD) directory structure and layer rules for Vue 3 projects.

## Directory Structure

```
src/
├── app/                      # Application layer
│   ├── layouts/              # App layouts
│   │   └── main/
│   │       ├── MainLayout.vue
│   │       └── ui/
│   │           ├── Header.vue
│   │           ├── Sidebar.vue
│   │           └── Breadcrumbs.vue
│   ├── providers/            # App-wide providers
│   │   └── i18n.ts
│   ├── router/               # Router configuration
│   │   └── index.ts
│   ├── store/                # Global stores (auth, operation)
│   │   ├── auth.ts
│   │   └── operation.ts
│   ├── styles/               # Global styles
│   │   └── main.css
│   ├── App.vue
│   └── main.ts
│
├── pages/                    # Pages layer (routes)
│   ├── auth/
│   │   └── LoginPage.vue
│   ├── user/
│   │   ├── UserListPage.vue
│   │   └── UserDetailPage.vue
│   ├── client/
│   │   ├── ClientListPage.vue
│   │   └── ClientDetailPage.vue
│   └── index.ts
│
├── features/                 # User interactions/actions
│   ├── user/
│   │   ├── ui/
│   │   │   ├── UserForm.vue
│   │   │   └── UserFilters.vue
│   │   ├── model/
│   │   │   └── useUserForm.ts
│   │   └── index.ts
│   ├── auth/
│   │   ├── ui/
│   │   │   └── LoginForm.vue
│   │   ├── model/
│   │   │   └── useLoginForm.ts
│   │   └── index.ts
│   └── client/
│       ├── ui/
│       │   └── ClientForm.vue
│       ├── model/
│       │   └── useClientForm.ts
│       └── index.ts
│
├── entities/                 # Business entities
│   ├── user/
│   │   ├── ui/
│   │   │   ├── UserCard.vue
│   │   │   └── UserSelect.vue
│   │   ├── model/
│   │   │   └── types.ts
│   │   └── index.ts
│   ├── catalog/
│   │   ├── ui/
│   │   │   ├── RoleSelect.vue
│   │   │   ├── StatusSelect.vue
│   │   │   └── CurrencySelect.vue
│   │   ├── model/
│   │   │   └── types.ts
│   │   └── index.ts
│   └── transaction/
│       ├── model/
│       │   └── types.ts
│       └── index.ts
│
└── shared/                   # Reusable infrastructure
    ├── api/
    │   ├── base.ts           # Axios instance & interceptors
    │   ├── user.ts           # User API service
    │   ├── client.ts         # Client API service
    │   └── index.ts
    ├── composables/
    │   ├── usePagination.ts
    │   ├── useValidationRules.ts
    │   ├── useTheme.ts
    │   ├── useDebouncedRef.ts
    │   └── index.ts
    ├── lib/
    │   ├── formatters.ts
    │   ├── utils.ts
    │   └── index.ts
    ├── ui/
    │   ├── BaseTable.vue
    │   ├── BaseSearch.vue
    │   ├── BaseModal.vue
    │   ├── BasePagination.vue
    │   ├── BaseDescriptions.vue
    │   ├── StateBadge.vue
    │   ├── CopyText.vue
    │   ├── AddBtn.vue
    │   ├── EditBtn.vue
    │   ├── DeleteBtn.vue
    │   ├── SaveBtn.vue
    │   └── index.ts
    └── types/
        ├── common.ts
        ├── auth.ts
        └── index.ts
```

## Layer Rules

### Import Hierarchy

```
┌─────────────────────────────────────────────────────────┐
│  APP                                                     │
│  Can import from: pages, features, entities, shared      │
└─────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────┐
│  PAGES                                                   │
│  Can import from: features, entities, shared             │
└─────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────┐
│  FEATURES                                                │
│  Can import from: entities, shared                       │
└─────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────┐
│  ENTITIES                                                │
│  Can import from: shared only                            │
└─────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────┐
│  SHARED                                                  │
│  Cannot import from any layer above                      │
└─────────────────────────────────────────────────────────┘
```

### Layer Descriptions

| Layer | Purpose | Examples |
|-------|---------|----------|
| **app** | Application initialization, providers, global config | router, stores, layouts, i18n |
| **pages** | Route-level components, page composition | UserListPage, ClientDetailPage |
| **features** | User interactions, business actions | UserForm, LoginForm, Filters |
| **entities** | Business entities, domain models | UserCard, RoleSelect, types |
| **shared** | Reusable infrastructure | API, composables, UI components |

## Slice Structure

Each slice (feature/entity) contains these segments:

```
feature-name/
├── ui/           # Vue components
│   ├── FeatureForm.vue
│   └── FeatureList.vue
├── model/        # Business logic, composables, types
│   ├── useFeatureForm.ts
│   └── types.ts
└── index.ts      # Public API (re-exports)
```

### Index File Pattern

```typescript
// features/user/index.ts
export { default as UserForm } from './ui/UserForm.vue'
export { default as UserFilters } from './ui/UserFilters.vue'
export { useUserForm } from './model/useUserForm'

// entities/user/index.ts
export type { IUser, IUserForm, IUserList } from './model/types'
export { default as UserCard } from './ui/UserCard.vue'
export { default as UserSelect } from './ui/UserSelect.vue'
```

## Forbidden Imports

```typescript
// ❌ WRONG: Entity importing from feature
// entities/user/model/userStore.ts
import { authApi } from '@/features/auth'

// ❌ WRONG: Feature importing from page
// features/user/ui/UserForm.vue
import { UserListPage } from '@/pages/user'

// ❌ WRONG: Shared importing from entity
// shared/api/base.ts
import { useUserStore } from '@/entities/user'

// ❌ WRONG: Cross-slice import at same level
// features/user/model/useUserForm.ts
import { useClientForm } from '@/features/client'
```

## Correct Import Examples

```typescript
// ✅ Page importing from feature and entity
// pages/user/UserListPage.vue
import { UserForm } from '@/features/user'
import type { IUserList } from '@/entities/user'
import { BaseTable } from '@/shared/ui'

// ✅ Feature importing from entity and shared
// features/user/model/useUserForm.ts
import type { IUserForm } from '@/entities/user'
import { useValidationRules } from '@/shared/composables'
import { UserService } from '@/shared/api'

// ✅ Entity importing from shared only
// entities/catalog/ui/RoleSelect.vue
import { get } from '@/shared/api/base'
import type { ISelectOption } from '@/shared/types'
```

## Vite Aliases Configuration

```typescript
// vite.config.ts
import { fileURLToPath, URL } from 'node:url'
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'

export default defineConfig({
  plugins: [vue()],
  resolve: {
    alias: {
      '@': fileURLToPath(new URL('./src', import.meta.url)),
    },
  },
})
```

```json
// tsconfig.app.json
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"]
    }
  }
}
```
