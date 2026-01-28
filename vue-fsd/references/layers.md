# FSD Layer Architecture

> Reference for: Vue FSD Expert
> Load when: Understanding FSD layers, hierarchy, import rules, project structure

## The Six Layers

Feature-Sliced Design organizes code into six layers, each with specific responsibilities:

```
src/
├── app/           # Layer 6: Application initialization
├── pages/         # Layer 5: Route-based screens
├── widgets/       # Layer 4: Complex UI blocks (optional)
├── features/      # Layer 3: User interactions (optional)
├── entities/      # Layer 2: Business domain models (optional)
└── shared/        # Layer 1: Reusable infrastructure
```

## The Import Rule (Critical)

**Modules can ONLY import from layers BELOW them.**

```
app      → can import from: pages, widgets, features, entities, shared
pages    → can import from: widgets, features, entities, shared
widgets  → can import from: features, entities, shared
features → can import from: entities, shared
entities → can import from: shared
shared   → can import from: nothing (external packages only)
```

**Never import:**
- Upward (features cannot import from pages)
- Sideways within same layer (user entity cannot import product entity)

## Layer Details

### 1. shared/ - Infrastructure Layer

Project-agnostic code that could be used in any project.

```
src/shared/
├── api/              # API client, interceptors
│   ├── client.ts
│   └── index.ts
├── config/           # Environment, constants
│   ├── env.ts
│   └── index.ts
├── lib/              # Utility functions
│   ├── format-date.ts
│   ├── debounce.ts
│   └── index.ts
├── ui/               # UI kit components
│   ├── Button/
│   │   ├── Button.vue
│   │   └── index.ts
│   ├── Input/
│   ├── Modal/
│   └── index.ts
└── types/            # Global TypeScript types
    └── index.ts
```

**No slices** - shared is organized by technical concern, not business domain.

### 2. entities/ - Business Domain Layer

Business domain models representing core data structures.

```
src/entities/
├── user/
│   ├── api/
│   │   └── user.api.ts
│   ├── model/
│   │   ├── user.types.ts
│   │   └── user.store.ts
│   ├── ui/
│   │   ├── UserCard.vue
│   │   └── UserAvatar.vue
│   └── index.ts          # Public API
├── product/
│   ├── api/
│   ├── model/
│   ├── ui/
│   └── index.ts
└── order/
    └── ...
```

**Entities are nouns** - User, Product, Order, Article, Comment

### 3. features/ - User Interaction Layer

User actions and interactions with business value.

```
src/features/
├── auth/
│   ├── api/
│   │   └── auth.api.ts
│   ├── model/
│   │   └── auth.store.ts
│   ├── ui/
│   │   ├── LoginForm.vue
│   │   └── LogoutButton.vue
│   └── index.ts
├── add-to-cart/
│   ├── model/
│   ├── ui/
│   │   └── AddToCartButton.vue
│   └── index.ts
└── search-products/
    └── ...
```

**Features are verbs** - Login, Logout, AddToCart, SearchProducts, CreateOrder

### 4. widgets/ - Composite UI Layer

Complex reusable UI blocks composed from features and entities.

```
src/widgets/
├── header/
│   ├── ui/
│   │   └── Header.vue
│   └── index.ts
├── sidebar/
│   ├── ui/
│   │   └── Sidebar.vue
│   └── index.ts
├── product-list/
│   ├── ui/
│   │   └── ProductList.vue
│   └── index.ts
└── user-profile-card/
    └── ...
```

**Widgets compose** features and entities into meaningful UI blocks.

### 5. pages/ - Route Layer

Full screens corresponding to routes.

```
src/pages/
├── home/
│   ├── ui/
│   │   └── HomePage.vue
│   └── index.ts
├── catalog/
│   ├── ui/
│   │   └── CatalogPage.vue
│   └── index.ts
├── product/
│   ├── ui/
│   │   └── ProductPage.vue
│   └── index.ts
├── cart/
│   └── ...
└── profile/
    └── ...
```

**One slice per route** - Pages compose widgets, features, and entities.

### 6. app/ - Application Layer

Application initialization and global setup.

```
src/app/
├── providers/
│   ├── RouterProvider.vue
│   ├── StoreProvider.vue
│   └── index.ts
├── styles/
│   ├── global.css
│   └── variables.css
├── router/
│   └── index.ts
├── App.vue
└── main.ts
```

**No slices** - app layer handles initialization, routing, providers.

## Complete Project Structure

```
src/
├── app/
│   ├── providers/
│   │   └── index.ts
│   ├── router/
│   │   └── index.ts
│   ├── styles/
│   │   └── global.css
│   ├── App.vue
│   └── main.ts
│
├── pages/
│   ├── home/
│   │   ├── ui/HomePage.vue
│   │   └── index.ts
│   ├── catalog/
│   │   ├── ui/CatalogPage.vue
│   │   └── index.ts
│   └── product/
│       ├── ui/ProductPage.vue
│       └── index.ts
│
├── widgets/
│   ├── header/
│   │   ├── ui/Header.vue
│   │   └── index.ts
│   └── product-list/
│       ├── ui/ProductList.vue
│       └── index.ts
│
├── features/
│   ├── auth/
│   │   ├── api/auth.api.ts
│   │   ├── model/auth.store.ts
│   │   ├── ui/LoginForm.vue
│   │   └── index.ts
│   └── add-to-cart/
│       ├── ui/AddToCartButton.vue
│       └── index.ts
│
├── entities/
│   ├── user/
│   │   ├── api/user.api.ts
│   │   ├── model/
│   │   │   ├── user.types.ts
│   │   │   └── user.store.ts
│   │   ├── ui/UserCard.vue
│   │   └── index.ts
│   └── product/
│       ├── api/product.api.ts
│       ├── model/
│       │   ├── product.types.ts
│       │   └── product.store.ts
│       ├── ui/ProductCard.vue
│       └── index.ts
│
└── shared/
    ├── api/
    │   ├── client.ts
    │   └── index.ts
    ├── config/
    │   └── index.ts
    ├── lib/
    │   └── index.ts
    ├── ui/
    │   ├── Button/
    │   ├── Input/
    │   └── index.ts
    └── types/
        └── index.ts
```

## Import Examples

```typescript
// ✅ CORRECT - Importing from lower layers

// In pages/catalog/ui/CatalogPage.vue
import { ProductList } from '@/widgets/product-list'
import { SearchProducts } from '@/features/search-products'
import { ProductCard } from '@/entities/product'
import { Button } from '@/shared/ui'

// In features/auth/ui/LoginForm.vue
import { useUserStore } from '@/entities/user'
import { Input, Button } from '@/shared/ui'
import { apiClient } from '@/shared/api'

// In entities/user/ui/UserCard.vue
import { formatDate } from '@/shared/lib'
import { Avatar } from '@/shared/ui'
```

```typescript
// ❌ WRONG - Violates import rules

// In entities/user/model/user.store.ts
import { useAuthStore } from '@/features/auth'  // ❌ Entity importing from feature

// In features/auth/ui/LoginForm.vue
import { HomePage } from '@/pages/home'  // ❌ Feature importing from page

// In entities/user/ui/UserCard.vue
import { ProductCard } from '@/entities/product'  // ❌ Sideways import in same layer
```

## Path Aliases Configuration

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
      '@app': fileURLToPath(new URL('./src/app', import.meta.url)),
      '@pages': fileURLToPath(new URL('./src/pages', import.meta.url)),
      '@widgets': fileURLToPath(new URL('./src/widgets', import.meta.url)),
      '@features': fileURLToPath(new URL('./src/features', import.meta.url)),
      '@entities': fileURLToPath(new URL('./src/entities', import.meta.url)),
      '@shared': fileURLToPath(new URL('./src/shared', import.meta.url))
    }
  }
})
```

```json
// tsconfig.json
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"],
      "@app/*": ["src/app/*"],
      "@pages/*": ["src/pages/*"],
      "@widgets/*": ["src/widgets/*"],
      "@features/*": ["src/features/*"],
      "@entities/*": ["src/entities/*"],
      "@shared/*": ["src/shared/*"]
    }
  }
}
```

## When to Use Each Layer

| Layer | Use When |
|-------|----------|
| app | Entry point, global providers, routing setup |
| pages | Full screen corresponding to a route |
| widgets | Reusable UI block composed of features/entities |
| features | User action with business value |
| entities | Core business domain model |
| shared | Generic utilities, UI kit, API client |

## Quick Reference

| Rule | Description |
|------|-------------|
| Import direction | Only downward, never upward or sideways |
| Public API | Every slice exports through index.ts |
| Entities | Nouns - data models with identity |
| Features | Verbs - user actions with side effects |
| Widgets | Composed UI blocks |
| Pages | Route screens, one per route |
