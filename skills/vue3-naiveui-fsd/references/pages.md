# Page Patterns

Page component patterns for Vue 3 + Naive UI with FSD architecture.

## List Page Pattern

```vue
<!-- src/pages/user/UserListPage.vue -->
<script setup lang="ts">
import { ref, onMounted, watch } from 'vue'
import { NCard, NSpace, useMessage } from 'naive-ui'
import { usePagination } from '@/shared/composables'
import { UserService } from '@/shared/api'
import { BaseTable, BaseSearch, BasePagination, AddBtn, BaseModal } from '@/shared/ui'
import { UserForm, UserFilters } from '@/features/user'
import type { IUserList, IUserListParams } from '@/entities/user'

const message = useMessage()

// Pagination
const { page, perPage, total, setTotal, resetPagination } = usePagination()

// State
const users = ref<IUserList[]>([])
const loading = ref(false)
const search = ref('')
const showFormModal = ref(false)
const editingUserId = ref<number | undefined>()

// Filters
const filters = ref<IUserListParams>({
  role_id: undefined,
  status: undefined,
})

// Table columns
const columns = [
  { title: 'ID', key: 'id', width: 80 },
  { title: 'Name', key: 'name', sorter: true },
  { title: 'Email', key: 'email', ellipsis: { tooltip: true } },
  { title: 'Phone', key: 'phone' },
  { title: 'Role', key: 'role_name' },
  {
    title: 'Status',
    key: 'status',
    render: (row: IUserList) => h(StateBadge, { status: row.status }),
  },
  {
    title: 'Actions',
    key: 'actions',
    width: 120,
    fixed: 'right',
    render: (row: IUserList) =>
      h(NSpace, null, () => [
        h(EditBtn, { onClick: () => handleEdit(row) }),
        h(DeleteBtn, { onConfirm: () => handleDelete(row) }),
      ]),
  },
]

// Fetch data
async function fetchUsers(): Promise<void> {
  loading.value = true
  try {
    const params: IUserListParams = {
      page: page.value,
      per_page: perPage.value,
      search: search.value || undefined,
      ...filters.value,
    }
    const response = await UserService.getList(params)
    users.value = response.data
    setTotal(response.meta.total)
  } catch {
    message.error('Failed to load users')
  } finally {
    loading.value = false
  }
}

// Handlers
function handleAdd(): void {
  editingUserId.value = undefined
  showFormModal.value = true
}

function handleEdit(user: IUserList): void {
  editingUserId.value = user.id
  showFormModal.value = true
}

async function handleDelete(user: IUserList): Promise<void> {
  try {
    await UserService.delete(user.id)
    message.success('User deleted')
    await fetchUsers()
  } catch {
    message.error('Failed to delete user')
  }
}

function handleFormSuccess(): void {
  showFormModal.value = false
  fetchUsers()
}

function handleFiltersChange(newFilters: IUserListParams): void {
  filters.value = newFilters
  resetPagination()
  fetchUsers()
}

// Watchers
watch([page, perPage], fetchUsers)
watch(search, () => {
  resetPagination()
  fetchUsers()
})

onMounted(fetchUsers)
</script>

<template>
  <div class="space-y-4">
    <!-- Filters -->
    <UserFilters
      v-model="filters"
      @change="handleFiltersChange"
    />

    <!-- Main card -->
    <NCard :title="$t('user.list')">
      <template #header-extra>
        <NSpace>
          <BaseSearch
            v-model="search"
            :placeholder="$t('user.search')"
          />
          <AddBtn @click="handleAdd" />
        </NSpace>
      </template>

      <BaseTable
        :columns="columns"
        :data="users"
        :loading="loading"
      />

      <template #footer>
        <BasePagination
          v-model:page="page"
          v-model:per-page="perPage"
          :total="total"
        />
      </template>
    </NCard>

    <!-- Form modal -->
    <BaseModal
      v-model:show="showFormModal"
      :title="editingUserId ? $t('user.edit') : $t('user.create')"
      :width="600"
    >
      <UserForm
        :user-id="editingUserId"
        @success="handleFormSuccess"
        @cancel="showFormModal = false"
      />
    </BaseModal>
  </div>
</template>
```

## Detail Page Pattern

```vue
<!-- src/pages/user/UserDetailPage.vue -->
<script setup lang="ts">
import { ref, onMounted } from 'vue'
import { useRoute, useRouter } from 'vue-router'
import { NCard, NSpace, NButton, NDescriptions, NDescriptionsItem, useMessage } from 'naive-ui'
import { UserService } from '@/shared/api'
import { BaseModal, StateBadge, EditBtn, DeleteBtn } from '@/shared/ui'
import { UserForm } from '@/features/user'
import type { IUserDetail } from '@/entities/user'
import { formatDate, formatPhone } from '@/shared/lib'

const route = useRoute()
const router = useRouter()
const message = useMessage()

const userId = Number(route.params.id)
const user = ref<IUserDetail | null>(null)
const loading = ref(false)
const showEditModal = ref(false)

async function fetchUser(): Promise<void> {
  loading.value = true
  try {
    user.value = await UserService.show(userId)
  } catch {
    message.error('User not found')
    router.push('/user')
  } finally {
    loading.value = false
  }
}

async function handleDelete(): Promise<void> {
  try {
    await UserService.delete(userId)
    message.success('User deleted')
    router.push('/user')
  } catch {
    message.error('Failed to delete')
  }
}

function handleEditSuccess(): void {
  showEditModal.value = false
  fetchUser()
}

onMounted(fetchUser)
</script>

<template>
  <NSpin :show="loading">
    <NCard v-if="user" :title="user.name">
      <template #header-extra>
        <NSpace>
          <EditBtn @click="showEditModal = true" />
          <DeleteBtn @confirm="handleDelete" />
        </NSpace>
      </template>

      <NDescriptions label-placement="left" :column="2">
        <NDescriptionsItem :label="$t('user.email')">
          {{ user.email }}
        </NDescriptionsItem>

        <NDescriptionsItem :label="$t('user.phone')">
          {{ formatPhone(user.phone) }}
        </NDescriptionsItem>

        <NDescriptionsItem :label="$t('user.role')">
          {{ user.role?.name }}
        </NDescriptionsItem>

        <NDescriptionsItem :label="$t('user.status')">
          <StateBadge :status="user.status" />
        </NDescriptionsItem>

        <NDescriptionsItem :label="$t('common.createdAt')">
          {{ formatDate(user.created_at) }}
        </NDescriptionsItem>

        <NDescriptionsItem :label="$t('user.lastLogin')">
          {{ formatDate(user.last_login_at) }}
        </NDescriptionsItem>
      </NDescriptions>

      <!-- Related data sections -->
      <NDivider />

      <NTabs type="line">
        <NTabPane name="permissions" :tab="$t('user.permissions')">
          <NTag
            v-for="permission in user.permissions"
            :key="permission"
            class="mr-2 mb-2"
          >
            {{ permission }}
          </NTag>
        </NTabPane>

        <NTabPane name="activity" :tab="$t('user.activity')">
          <!-- Activity log -->
        </NTabPane>
      </NTabs>
    </NCard>

    <BaseModal
      v-model:show="showEditModal"
      :title="$t('user.edit')"
      :width="600"
    >
      <UserForm
        :user-id="userId"
        @success="handleEditSuccess"
        @cancel="showEditModal = false"
      />
    </BaseModal>
  </NSpin>
</template>
```

## Dashboard Page Pattern

```vue
<!-- src/pages/DashboardPage.vue -->
<script setup lang="ts">
import { ref, onMounted } from 'vue'
import { NGrid, NGi, NCard, NStatistic } from 'naive-ui'
import { DashboardService } from '@/shared/api'
import { formatCurrency, formatNumber } from '@/shared/lib'

interface IDashboardStats {
  total_users: number
  total_clients: number
  total_contracts: number
  total_revenue: number
}

const stats = ref<IDashboardStats | null>(null)
const loading = ref(false)

async function fetchStats(): Promise<void> {
  loading.value = true
  try {
    stats.value = await DashboardService.getStats()
  } finally {
    loading.value = false
  }
}

onMounted(fetchStats)
</script>

<template>
  <div class="space-y-6">
    <h1 class="text-2xl font-bold">{{ $t('dashboard.title') }}</h1>

    <NGrid :cols="4" :x-gap="16" :y-gap="16">
      <NGi>
        <NCard>
          <NStatistic
            :label="$t('dashboard.totalUsers')"
            :value="formatNumber(stats?.total_users)"
          />
        </NCard>
      </NGi>

      <NGi>
        <NCard>
          <NStatistic
            :label="$t('dashboard.totalClients')"
            :value="formatNumber(stats?.total_clients)"
          />
        </NCard>
      </NGi>

      <NGi>
        <NCard>
          <NStatistic
            :label="$t('dashboard.totalContracts')"
            :value="formatNumber(stats?.total_contracts)"
          />
        </NCard>
      </NGi>

      <NGi>
        <NCard>
          <NStatistic
            :label="$t('dashboard.totalRevenue')"
            :value="formatCurrency(stats?.total_revenue)"
          />
        </NCard>
      </NGi>
    </NGrid>

    <!-- Charts section -->
    <NGrid :cols="2" :x-gap="16">
      <NGi>
        <NCard :title="$t('dashboard.revenueChart')">
          <RevenueChart />
        </NCard>
      </NGi>

      <NGi>
        <NCard :title="$t('dashboard.clientsChart')">
          <ClientsChart />
        </NCard>
      </NGi>
    </NGrid>

    <!-- Recent activity -->
    <NCard :title="$t('dashboard.recentActivity')">
      <RecentActivityList />
    </NCard>
  </div>
</template>
```

## Login Page Pattern

```vue
<!-- src/pages/auth/LoginPage.vue -->
<script setup lang="ts">
import { NCard } from 'naive-ui'
import { LoginForm } from '@/features/auth'
</script>

<template>
  <div class="min-h-screen flex items-center justify-center bg-gray-100 dark:bg-gray-900">
    <NCard class="w-full max-w-md" :title="$t('auth.login')">
      <LoginForm />
    </NCard>
  </div>
</template>
```

## Page with Tabs

```vue
<!-- src/pages/settings/SettingsPage.vue -->
<script setup lang="ts">
import { ref } from 'vue'
import { useRoute, useRouter } from 'vue-router'
import { NCard, NTabs, NTabPane } from 'naive-ui'
import { ProfileSettings, SecuritySettings, NotificationSettings } from '@/features/settings'

const route = useRoute()
const router = useRouter()

const activeTab = ref(route.query.tab as string || 'profile')

function handleTabChange(tab: string): void {
  activeTab.value = tab
  router.replace({ query: { tab } })
}
</script>

<template>
  <NCard :title="$t('settings.title')">
    <NTabs
      :value="activeTab"
      type="line"
      @update:value="handleTabChange"
    >
      <NTabPane name="profile" :tab="$t('settings.profile')">
        <ProfileSettings />
      </NTabPane>

      <NTabPane name="security" :tab="$t('settings.security')">
        <SecuritySettings />
      </NTabPane>

      <NTabPane name="notifications" :tab="$t('settings.notifications')">
        <NotificationSettings />
      </NTabPane>
    </NTabs>
  </NCard>
</template>
```

## Page with Sidebar Filters

```vue
<!-- src/pages/report/ReportPage.vue -->
<script setup lang="ts">
import { NLayout, NLayoutSider, NLayoutContent, NCard } from 'naive-ui'
import { ReportFilters, ReportTable } from '@/features/report'
</script>

<template>
  <NLayout has-sider>
    <NLayoutSider
      :width="280"
      :collapsed-width="0"
      show-trigger
      bordered
    >
      <NCard title="Filters" :bordered="false">
        <ReportFilters @change="handleFiltersChange" />
      </NCard>
    </NLayoutSider>

    <NLayoutContent>
      <NCard :title="$t('report.title')">
        <template #header-extra>
          <NButton @click="handleExport">
            Export
          </NButton>
        </template>

        <ReportTable :filters="filters" />
      </NCard>
    </NLayoutContent>
  </NLayout>
</template>
```
