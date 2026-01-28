# Public API Pattern

> Reference for: Vue FSD Expert
> Load when: Slice exports, index.ts patterns, encapsulation, cross-slice communication

## The Public API Rule

**Every slice MUST expose its functionality through a single `index.ts` file.**

This is the most critical pattern in FSD - it ensures:
- Encapsulation of internal implementation
- Clear contract for consumers
- Easy refactoring without breaking imports
- Better tree-shaking

---

## Basic Structure

```
src/entities/user/
├── api/
│   └── user.api.ts       # Internal
├── model/
│   ├── user.types.ts     # Internal
│   └── user.store.ts     # Internal
├── ui/
│   ├── UserCard.vue      # Internal
│   └── UserAvatar.vue    # Internal
└── index.ts              # ✅ PUBLIC API - Only this is exported
```

---

## Export Patterns

### Named Exports (Recommended)

```typescript
// entities/user/index.ts

// ✅ GOOD - Named exports
export type { User, UserRole, UserPreview } from './model/user.types'
export { useUserStore } from './model/user.store'
export { userApi } from './api/user.api'
export { default as UserCard } from './ui/UserCard.vue'
export { default as UserAvatar } from './ui/UserAvatar.vue'
```

### Avoid Wildcard Re-exports

```typescript
// ❌ BAD - Wildcard exports break tree-shaking
export * from './model/user.types'
export * from './model/user.store'
export * from './ui'
```

### Avoid Default Exports for Slices

```typescript
// ❌ BAD - Default export from slice
export default {
  UserCard,
  UserAvatar,
  useUserStore
}

// ✅ GOOD - Named exports
export { UserCard, UserAvatar, useUserStore }
```

---

## Import Examples

### Correct Imports (Through Public API)

```typescript
// In pages/profile/ui/ProfilePage.vue

// ✅ GOOD - Import from slice root
import { UserCard, UserAvatar, useUserStore, type User } from '@/entities/user'
import { LoginForm, useAuthStore } from '@/features/auth'
import { Button, Input } from '@/shared/ui'
```

### Incorrect Imports (Bypassing Public API)

```typescript
// ❌ BAD - Direct import from internal module
import { User } from '@/entities/user/model/user.types'
import UserCard from '@/entities/user/ui/UserCard.vue'
import { useUserStore } from '@/entities/user/model/user.store'

// ❌ BAD - Importing internal utilities
import { formatUserName } from '@/entities/user/lib/format'
```

---

## Organizing Exports

### Types First, Then Code

```typescript
// entities/product/index.ts

// 1. Types (always first)
export type {
  Product,
  ProductCategory,
  ProductVariant,
  ProductListResponse
} from './model/product.types'

// 2. Store/Model
export { useProductStore } from './model/product.store'

// 3. API (optional - only if needed externally)
export { productApi } from './api/product.api'

// 4. UI Components
export { default as ProductCard } from './ui/ProductCard.vue'
export { default as ProductPrice } from './ui/ProductPrice.vue'
export { default as ProductRating } from './ui/ProductRating.vue'
export { default as ProductBadge } from './ui/ProductBadge.vue'
```

### Grouping Related Exports

```typescript
// features/auth/index.ts

// Types
export type {
  LoginCredentials,
  RegisterData,
  AuthTokens,
  AuthState
} from './model/auth.types'

// Store
export { useAuthStore } from './model/auth.store'

// UI - Forms
export { default as LoginForm } from './ui/LoginForm.vue'
export { default as RegisterForm } from './ui/RegisterForm.vue'
export { default as ForgotPasswordForm } from './ui/ForgotPasswordForm.vue'

// UI - Actions
export { default as LogoutButton } from './ui/LogoutButton.vue'
export { default as SocialLoginButtons } from './ui/SocialLoginButtons.vue'

// Composables (optional)
export { useAuthGuard } from './lib/use-auth-guard'
```

---

## Selective Exports

### Don't Export Everything

Not all internal code needs to be exported. Only export what other slices need:

```typescript
// entities/order/index.ts

// ✅ Export - Other slices need this type
export type { Order, OrderStatus, OrderItem } from './model/order.types'

// ✅ Export - Other slices use this store
export { useOrderStore } from './model/order.store'

// ✅ Export - Reusable display components
export { default as OrderCard } from './ui/OrderCard.vue'
export { default as OrderStatusBadge } from './ui/OrderStatusBadge.vue'

// ❌ Don't export - Internal helper only used within slice
// export { calculateOrderTotal } from './lib/calculations'

// ❌ Don't export - Internal validation schema
// export { orderSchema } from './model/order.schema'

// ❌ Don't export - Internal component only used by OrderCard
// export { default as OrderItemRow } from './ui/OrderItemRow.vue'
```

---

## Cross-Slice Communication

### Using @x Notation for Cross-Entity References

When entities legitimately need to reference each other:

```typescript
// entities/user/@x/product.ts
// This file explicitly marks cross-entity dependency

import { type Product } from '@/entities/product'

export interface UserFavoriteProduct {
  userId: number
  product: Product
  addedAt: string
}
```

```typescript
// entities/user/index.ts
export type { UserFavoriteProduct } from './@x/product'
```

### Prefer Features for Cross-Entity Logic

Instead of entities importing each other, create a feature:

```typescript
// ❌ BAD - Entity importing another entity
// entities/user/model/user.store.ts
import { useProductStore } from '@/entities/product' // Violates FSD rules

// ✅ GOOD - Create a feature for cross-entity logic
// features/user-favorites/model/favorites.store.ts
import { useUserStore, type User } from '@/entities/user'
import { useProductStore, type Product } from '@/entities/product'

export const useFavoritesStore = defineStore('favorites', () => {
  const userStore = useUserStore()
  const productStore = useProductStore()

  // Cross-entity logic here
})
```

---

## Shared Layer Exports

The shared layer doesn't use slices, but still needs organized exports:

```typescript
// shared/ui/index.ts
export { Button } from './Button'
export { Input } from './Input'
export { Select } from './Select'
export { Modal } from './Modal'
export { Card } from './Card'
export { Avatar } from './Avatar'
export { Badge } from './Badge'
export { Spinner } from './Spinner'
export { Pagination } from './Pagination'
export { Alert } from './Alert'
export { Tooltip } from './Tooltip'

// shared/lib/index.ts
export { formatDate, formatDateTime, formatRelative } from './date'
export { formatCurrency, formatNumber } from './number'
export { capitalize, truncate, slugify } from './string'
export { debounce, throttle, sleep } from './async'
export { storage } from './storage'

// shared/api/index.ts
export { apiClient } from './client'
export type { ApiResponse, ApiError, PaginatedResponse } from './types'
```

---

## ESLint Rules for Public API

```javascript
// .eslintrc.js
module.exports = {
  rules: {
    'no-restricted-imports': [
      'error',
      {
        patterns: [
          {
            group: ['@/entities/*/model/*', '@/entities/*/api/*', '@/entities/*/ui/*', '@/entities/*/lib/*'],
            message: 'Import from slice root instead: @/entities/slice-name'
          },
          {
            group: ['@/features/*/model/*', '@/features/*/api/*', '@/features/*/ui/*', '@/features/*/lib/*'],
            message: 'Import from slice root instead: @/features/slice-name'
          },
          {
            group: ['@/widgets/*/ui/*'],
            message: 'Import from slice root instead: @/widgets/slice-name'
          },
          {
            group: ['@/pages/*/ui/*'],
            message: 'Import from slice root instead: @/pages/slice-name'
          }
        ]
      }
    ]
  }
}
```

---

## Barrel File Template

```typescript
// Template for any slice index.ts

// =============================================================================
// TYPES
// =============================================================================
export type {
  // Main types
  MainType,
  // Supporting types
  SupportingType,
  // Response types (if API exposed)
  ResponseType
} from './model/types'

// =============================================================================
// STORE / MODEL
// =============================================================================
export { useSliceStore } from './model/store'

// =============================================================================
// API (optional - only if external access needed)
// =============================================================================
export { sliceApi } from './api/api'

// =============================================================================
// UI COMPONENTS
// =============================================================================
export { default as MainComponent } from './ui/MainComponent.vue'
export { default as SecondaryComponent } from './ui/SecondaryComponent.vue'

// =============================================================================
// COMPOSABLES (optional)
// =============================================================================
export { useSliceComposable } from './lib/use-composable'
```

---

## Quick Reference

| Rule | Description |
|------|-------------|
| Single entry point | Every slice has one `index.ts` |
| Named exports only | No wildcards, no default exports |
| No direct imports | Never bypass public API |
| Types first | Export types before code |
| Selective exports | Only export what's needed externally |
| @x notation | Mark cross-entity dependencies explicitly |
