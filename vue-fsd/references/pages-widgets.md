# Pages & Widgets

> Reference for: Vue FSD Expert
> Load when: Route composition, complex UI blocks, page structure, widget patterns

## Pages Layer

Pages are **full screens** that correspond to application routes. Each page composes widgets, features, and entities.

### Page Structure

```
src/pages/
├── home/
│   ├── ui/
│   │   └── HomePage.vue
│   └── index.ts
├── catalog/
│   ├── ui/
│   │   └── CatalogPage.vue
│   ├── model/
│   │   └── catalog.store.ts    # Page-specific state (optional)
│   └── index.ts
├── product/
│   ├── ui/
│   │   └── ProductPage.vue
│   └── index.ts
├── cart/
│   ├── ui/
│   │   └── CartPage.vue
│   └── index.ts
├── checkout/
│   ├── ui/
│   │   └── CheckoutPage.vue
│   └── index.ts
└── profile/
    ├── ui/
    │   └── ProfilePage.vue
    └── index.ts
```

### Page Example: Home Page

```vue
<!-- pages/home/ui/HomePage.vue -->
<script setup lang="ts">
import { Header } from '@/widgets/header'
import { Footer } from '@/widgets/footer'
import { FeaturedProducts } from '@/widgets/featured-products'
import { CategoryList } from '@/widgets/category-list'
import { HeroBanner } from '@/widgets/hero-banner'
</script>

<template>
  <div class="home-page">
    <Header />

    <main class="home-page__content">
      <HeroBanner />

      <section class="home-page__section">
        <h2>Featured Products</h2>
        <FeaturedProducts />
      </section>

      <section class="home-page__section">
        <h2>Browse Categories</h2>
        <CategoryList />
      </section>
    </main>

    <Footer />
  </div>
</template>

<style scoped>
.home-page {
  min-height: 100vh;
  display: flex;
  flex-direction: column;
}

.home-page__content {
  flex: 1;
  max-width: 1200px;
  margin: 0 auto;
  padding: 2rem;
}

.home-page__section {
  margin-bottom: 3rem;
}
</style>
```

### Page Example: Catalog Page

```vue
<!-- pages/catalog/ui/CatalogPage.vue -->
<script setup lang="ts">
import { ref, computed, watch } from 'vue'
import { useRoute, useRouter } from 'vue-router'
import { Header } from '@/widgets/header'
import { ProductList } from '@/widgets/product-list'
import { SearchFilters } from '@/features/search-products'
import { useProductStore } from '@/entities/product'

const route = useRoute()
const router = useRouter()
const productStore = useProductStore()

// Page-level state from URL
const page = computed(() => Number(route.query.page) || 1)
const category = computed(() => route.query.category as string | undefined)
const search = computed(() => route.query.q as string | undefined)

// Fetch products based on filters
watch(
  [page, category, search],
  async () => {
    await productStore.fetchProducts({
      page: page.value,
      category: category.value,
      search: search.value
    })
  },
  { immediate: true }
)

function handlePageChange(newPage: number) {
  router.push({ query: { ...route.query, page: newPage } })
}

function handleFilterChange(filters: { category?: string; search?: string }) {
  router.push({ query: { ...filters, page: 1 } })
}
</script>

<template>
  <div class="catalog-page">
    <Header />

    <main class="catalog-page__content">
      <aside class="catalog-page__sidebar">
        <SearchFilters
          :category="category"
          :search="search"
          @change="handleFilterChange"
        />
      </aside>

      <section class="catalog-page__products">
        <ProductList
          :products="productStore.products"
          :loading="productStore.loading"
          :page="page"
          :total-pages="productStore.totalPages"
          @page-change="handlePageChange"
        />
      </section>
    </main>
  </div>
</template>
```

### Page Example: Product Detail Page

```vue
<!-- pages/product/ui/ProductPage.vue -->
<script setup lang="ts">
import { computed, onMounted } from 'vue'
import { useRoute } from 'vue-router'
import { Header } from '@/widgets/header'
import { ProductGallery } from '@/widgets/product-gallery'
import { RelatedProducts } from '@/widgets/related-products'
import { AddToCartButton } from '@/features/add-to-cart'
import { ProductCard, useProductStore, type Product } from '@/entities/product'
import { formatCurrency } from '@/shared/lib'
import { Spinner } from '@/shared/ui'

const route = useRoute()
const productStore = useProductStore()

const productId = computed(() => Number(route.params.id))
const product = computed(() => productStore.productById(productId.value))

onMounted(async () => {
  await productStore.fetchProduct(productId.value)
})
</script>

<template>
  <div class="product-page">
    <Header />

    <main class="product-page__content">
      <div v-if="productStore.loading" class="product-page__loading">
        <Spinner size="lg" />
      </div>

      <template v-else-if="product">
        <div class="product-page__main">
          <ProductGallery :images="product.images" />

          <div class="product-page__info">
            <h1 class="product-page__title">{{ product.name }}</h1>
            <p class="product-page__price">{{ formatCurrency(product.price) }}</p>
            <p class="product-page__description">{{ product.description }}</p>

            <AddToCartButton :product="product" />
          </div>
        </div>

        <section class="product-page__related">
          <h2>Related Products</h2>
          <RelatedProducts :category="product.category" :exclude-id="product.id" />
        </section>
      </template>

      <div v-else class="product-page__not-found">
        Product not found
      </div>
    </main>
  </div>
</template>
```

### Page Public API

```typescript
// pages/home/index.ts
export { default as HomePage } from './ui/HomePage.vue'

// pages/catalog/index.ts
export { default as CatalogPage } from './ui/CatalogPage.vue'

// pages/product/index.ts
export { default as ProductPage } from './ui/ProductPage.vue'
```

---

## Widgets Layer

Widgets are **complex reusable UI blocks** that combine multiple features and entities.

### Widget Structure

```
src/widgets/
├── header/
│   ├── ui/
│   │   ├── Header.vue
│   │   ├── HeaderNav.vue
│   │   └── HeaderActions.vue
│   └── index.ts
├── footer/
│   ├── ui/
│   │   └── Footer.vue
│   └── index.ts
├── sidebar/
│   ├── ui/
│   │   └── Sidebar.vue
│   │   └── SidebarMenu.vue
│   └── index.ts
├── product-list/
│   ├── ui/
│   │   └── ProductList.vue
│   └── index.ts
├── cart-preview/
│   ├── ui/
│   │   └── CartPreview.vue
│   └── index.ts
└── user-menu/
    ├── ui/
    │   └── UserMenu.vue
    └── index.ts
```

### Widget Example: Header

```vue
<!-- widgets/header/ui/Header.vue -->
<script setup lang="ts">
import { computed } from 'vue'
import { RouterLink } from 'vue-router'
import { useAuthStore } from '@/features/auth'
import { CartPreview } from '@/widgets/cart-preview'
import { UserMenu } from '@/widgets/user-menu'
import { SearchInput } from '@/features/search-products'
import { Button } from '@/shared/ui'
import { ROUTES } from '@/shared/config'

const authStore = useAuthStore()
const isAuthenticated = computed(() => authStore.isAuthenticated)
</script>

<template>
  <header class="header">
    <div class="header__container">
      <!-- Logo -->
      <RouterLink to="/" class="header__logo">
        MyShop
      </RouterLink>

      <!-- Navigation -->
      <nav class="header__nav">
        <RouterLink to="/catalog">Catalog</RouterLink>
        <RouterLink to="/deals">Deals</RouterLink>
        <RouterLink to="/about">About</RouterLink>
      </nav>

      <!-- Search -->
      <div class="header__search">
        <SearchInput />
      </div>

      <!-- Actions -->
      <div class="header__actions">
        <CartPreview />

        <template v-if="isAuthenticated">
          <UserMenu />
        </template>
        <template v-else>
          <Button variant="ghost" :to="ROUTES.LOGIN">Login</Button>
          <Button :to="ROUTES.REGISTER">Sign Up</Button>
        </template>
      </div>
    </div>
  </header>
</template>

<style scoped>
.header {
  background: white;
  border-bottom: 1px solid var(--color-border);
  padding: 1rem 0;
  position: sticky;
  top: 0;
  z-index: 100;
}

.header__container {
  max-width: 1200px;
  margin: 0 auto;
  padding: 0 1rem;
  display: flex;
  align-items: center;
  gap: 2rem;
}

.header__logo {
  font-size: 1.5rem;
  font-weight: 700;
  color: var(--color-primary);
  text-decoration: none;
}

.header__nav {
  display: flex;
  gap: 1.5rem;
}

.header__nav a {
  color: var(--color-text);
  text-decoration: none;
  transition: color 0.2s;
}

.header__nav a:hover,
.header__nav a.router-link-active {
  color: var(--color-primary);
}

.header__search {
  flex: 1;
  max-width: 400px;
}

.header__actions {
  display: flex;
  align-items: center;
  gap: 1rem;
}
</style>
```

### Widget Example: Product List

```vue
<!-- widgets/product-list/ui/ProductList.vue -->
<script setup lang="ts">
import { ProductCard, type Product } from '@/entities/product'
import { AddToCartButton } from '@/features/add-to-cart'
import { Spinner, Pagination } from '@/shared/ui'

interface Props {
  products: Product[]
  loading?: boolean
  page?: number
  totalPages?: number
}

const props = withDefaults(defineProps<Props>(), {
  loading: false,
  page: 1,
  totalPages: 1
})

const emit = defineEmits<{
  'page-change': [page: number]
}>()
</script>

<template>
  <div class="product-list">
    <div v-if="loading" class="product-list__loading">
      <Spinner size="lg" />
    </div>

    <template v-else>
      <div v-if="products.length === 0" class="product-list__empty">
        No products found
      </div>

      <div v-else class="product-list__grid">
        <div v-for="product in products" :key="product.id" class="product-list__item">
          <ProductCard :product="product">
            <template #actions>
              <AddToCartButton :product="product" size="sm" />
            </template>
          </ProductCard>
        </div>
      </div>

      <Pagination
        v-if="totalPages > 1"
        :current-page="page"
        :total-pages="totalPages"
        @change="emit('page-change', $event)"
      />
    </template>
  </div>
</template>

<style scoped>
.product-list__grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(280px, 1fr));
  gap: 1.5rem;
}

.product-list__loading,
.product-list__empty {
  display: flex;
  justify-content: center;
  align-items: center;
  min-height: 200px;
  color: var(--color-text-secondary);
}
</style>
```

### Widget Example: Cart Preview

```vue
<!-- widgets/cart-preview/ui/CartPreview.vue -->
<script setup lang="ts">
import { ref, computed } from 'vue'
import { RouterLink } from 'vue-router'
import { useCartStore } from '@/features/add-to-cart'
import { ProductCard, type Product } from '@/entities/product'
import { Button, Badge, Popover } from '@/shared/ui'
import { formatCurrency } from '@/shared/lib'
import { ROUTES } from '@/shared/config'

const cartStore = useCartStore()
const isOpen = ref(false)

const itemCount = computed(() => cartStore.itemCount)
const totalPrice = computed(() => cartStore.totalPrice)
const items = computed(() => cartStore.items)
</script>

<template>
  <Popover v-model:open="isOpen">
    <template #trigger>
      <button class="cart-preview__trigger">
        <span class="cart-preview__icon">🛒</span>
        <Badge v-if="itemCount > 0" :value="itemCount" />
      </button>
    </template>

    <template #content>
      <div class="cart-preview__dropdown">
        <h3 class="cart-preview__title">Your Cart</h3>

        <div v-if="items.length === 0" class="cart-preview__empty">
          Your cart is empty
        </div>

        <template v-else>
          <ul class="cart-preview__items">
            <li v-for="item in items.slice(0, 3)" :key="item.product.id">
              <img :src="item.product.image" :alt="item.product.name" />
              <div>
                <p>{{ item.product.name }}</p>
                <p>{{ item.quantity }} × {{ formatCurrency(item.product.price) }}</p>
              </div>
            </li>
          </ul>

          <p v-if="items.length > 3" class="cart-preview__more">
            +{{ items.length - 3 }} more items
          </p>

          <div class="cart-preview__footer">
            <p class="cart-preview__total">
              Total: {{ formatCurrency(totalPrice) }}
            </p>
            <Button :to="ROUTES.CART" block @click="isOpen = false">
              View Cart
            </Button>
          </div>
        </template>
      </div>
    </template>
  </Popover>
</template>
```

### Widget Example: User Menu

```vue
<!-- widgets/user-menu/ui/UserMenu.vue -->
<script setup lang="ts">
import { ref, computed } from 'vue'
import { RouterLink } from 'vue-router'
import { useAuthStore, LogoutButton } from '@/features/auth'
import { UserAvatar } from '@/entities/user'
import { Popover } from '@/shared/ui'
import { ROUTES } from '@/shared/config'

const authStore = useAuthStore()
const isOpen = ref(false)

const user = computed(() => authStore.currentUser)
</script>

<template>
  <Popover v-model:open="isOpen">
    <template #trigger>
      <button class="user-menu__trigger">
        <UserAvatar v-if="user" :user="user" size="sm" />
      </button>
    </template>

    <template #content>
      <div class="user-menu__dropdown">
        <div class="user-menu__header">
          <UserAvatar v-if="user" :user="user" size="md" />
          <div>
            <p class="user-menu__name">{{ user?.name }}</p>
            <p class="user-menu__email">{{ user?.email }}</p>
          </div>
        </div>

        <nav class="user-menu__nav">
          <RouterLink :to="ROUTES.PROFILE" @click="isOpen = false">
            Profile
          </RouterLink>
          <RouterLink to="/orders" @click="isOpen = false">
            My Orders
          </RouterLink>
          <RouterLink to="/settings" @click="isOpen = false">
            Settings
          </RouterLink>
        </nav>

        <div class="user-menu__footer">
          <LogoutButton />
        </div>
      </div>
    </template>
  </Popover>
</template>
```

### Widget Public API

```typescript
// widgets/header/index.ts
export { default as Header } from './ui/Header.vue'

// widgets/product-list/index.ts
export { default as ProductList } from './ui/ProductList.vue'

// widgets/cart-preview/index.ts
export { default as CartPreview } from './ui/CartPreview.vue'

// widgets/user-menu/index.ts
export { default as UserMenu } from './ui/UserMenu.vue'
```

---

## Router Configuration

```typescript
// app/router/index.ts
import { createRouter, createWebHistory } from 'vue-router'
import { HomePage } from '@/pages/home'
import { useAuthStore } from '@/features/auth'

const router = createRouter({
  history: createWebHistory(),
  routes: [
    {
      path: '/',
      name: 'home',
      component: HomePage
    },
    {
      path: '/catalog',
      name: 'catalog',
      component: () => import('@/pages/catalog').then(m => m.CatalogPage)
    },
    {
      path: '/product/:id',
      name: 'product',
      component: () => import('@/pages/product').then(m => m.ProductPage)
    },
    {
      path: '/cart',
      name: 'cart',
      component: () => import('@/pages/cart').then(m => m.CartPage)
    },
    {
      path: '/checkout',
      name: 'checkout',
      component: () => import('@/pages/checkout').then(m => m.CheckoutPage),
      meta: { requiresAuth: true }
    },
    {
      path: '/login',
      name: 'login',
      component: () => import('@/pages/auth').then(m => m.LoginPage)
    },
    {
      path: '/profile',
      name: 'profile',
      component: () => import('@/pages/profile').then(m => m.ProfilePage),
      meta: { requiresAuth: true }
    }
  ]
})

// Navigation guard
router.beforeEach((to, from, next) => {
  const authStore = useAuthStore()

  if (to.meta.requiresAuth && !authStore.isAuthenticated) {
    next({ name: 'login', query: { redirect: to.fullPath } })
  } else {
    next()
  }
})

export { router }
```

---

## Quick Reference

| Layer | Purpose | Contains |
|-------|---------|----------|
| Pages | Route screens | Full page composition |
| Widgets | UI blocks | Feature + entity combinations |

| Pattern | When to Use |
|---------|-------------|
| Page | One per route, composes widgets/features/entities |
| Widget | Reusable complex UI block (Header, Sidebar, ProductList) |
| Feature in Page | User actions specific to that route |
| Entity in Widget | Display data within composed block |
