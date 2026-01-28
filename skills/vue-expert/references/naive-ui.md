# Naive UI

> Reference for: Vue Expert
> Load when: Using Naive UI components, theming, form validation, data tables

## Installation & Setup

```bash
npm install naive-ui
# Optional: vfonts for icons
npm install vfonts
```

### Global Registration (main.ts)

```typescript
import { createApp } from 'vue'
import naive from 'naive-ui'
import App from './App.vue'

const app = createApp(App)
app.use(naive)
app.mount('#app')
```

### On-Demand Import (Recommended)

```vue
<script setup lang="ts">
import { NButton, NInput, NForm, NFormItem } from 'naive-ui'
</script>

<template>
  <n-button type="primary">Click me</n-button>
</template>
```

### Auto Import with unplugin

```typescript
// vite.config.ts
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'
import Components from 'unplugin-vue-components/vite'
import { NaiveUiResolver } from 'unplugin-vue-components/resolvers'
import AutoImport from 'unplugin-auto-import/vite'

export default defineConfig({
  plugins: [
    vue(),
    AutoImport({
      imports: [
        'vue',
        {
          'naive-ui': ['useDialog', 'useMessage', 'useNotification', 'useLoadingBar']
        }
      ]
    }),
    Components({
      resolvers: [NaiveUiResolver()]
    })
  ]
})
```

## Theming

### Theme Provider

```vue
<script setup lang="ts">
import { darkTheme, type GlobalTheme } from 'naive-ui'
import { ref, computed } from 'vue'

const isDark = ref(false)
const theme = computed<GlobalTheme | null>(() => isDark.value ? darkTheme : null)
</script>

<template>
  <n-config-provider :theme="theme">
    <n-button @click="isDark = !isDark">Toggle Theme</n-button>
    <router-view />
  </n-config-provider>
</template>
```

### Custom Theme Overrides

```vue
<script setup lang="ts">
import { type GlobalThemeOverrides } from 'naive-ui'

const themeOverrides: GlobalThemeOverrides = {
  common: {
    primaryColor: '#18a058',
    primaryColorHover: '#36ad6a',
    primaryColorPressed: '#0c7a43',
    borderRadius: '8px'
  },
  Button: {
    textColor: '#333'
  },
  Card: {
    borderRadius: '12px'
  }
}
</script>

<template>
  <n-config-provider :theme-overrides="themeOverrides">
    <app-content />
  </n-config-provider>
</template>
```

## Form Handling

### Basic Form with Validation

```vue
<script setup lang="ts">
import { ref } from 'vue'
import {
  NForm,
  NFormItem,
  NInput,
  NButton,
  type FormInst,
  type FormRules,
  type FormItemRule
} from 'naive-ui'

interface FormModel {
  username: string
  email: string
  password: string
}

const formRef = ref<FormInst | null>(null)
const formValue = ref<FormModel>({
  username: '',
  email: '',
  password: ''
})

const rules: FormRules = {
  username: [
    { required: true, message: 'Username is required', trigger: 'blur' },
    { min: 3, max: 20, message: 'Length should be 3-20', trigger: 'blur' }
  ],
  email: [
    { required: true, message: 'Email is required', trigger: 'blur' },
    { type: 'email', message: 'Invalid email format', trigger: 'blur' }
  ],
  password: [
    { required: true, message: 'Password is required', trigger: 'blur' },
    { min: 8, message: 'Minimum 8 characters', trigger: 'blur' }
  ]
}

async function handleSubmit() {
  try {
    await formRef.value?.validate()
    console.log('Form is valid:', formValue.value)
  } catch (errors) {
    console.log('Validation failed:', errors)
  }
}

function handleReset() {
  formRef.value?.restoreValidation()
  formValue.value = { username: '', email: '', password: '' }
}
</script>

<template>
  <n-form ref="formRef" :model="formValue" :rules="rules">
    <n-form-item label="Username" path="username">
      <n-input v-model:value="formValue.username" placeholder="Enter username" />
    </n-form-item>

    <n-form-item label="Email" path="email">
      <n-input v-model:value="formValue.email" placeholder="Enter email" />
    </n-form-item>

    <n-form-item label="Password" path="password">
      <n-input
        v-model:value="formValue.password"
        type="password"
        placeholder="Enter password"
        show-password-on="click"
      />
    </n-form-item>

    <n-form-item>
      <n-button type="primary" @click="handleSubmit">Submit</n-button>
      <n-button style="margin-left: 12px" @click="handleReset">Reset</n-button>
    </n-form-item>
  </n-form>
</template>
```

### Dynamic Validation Rules

```typescript
const passwordConfirmRule: FormItemRule = {
  required: true,
  validator: (rule, value) => {
    if (value !== formValue.value.password) {
      return new Error('Passwords do not match')
    }
    return true
  },
  trigger: ['blur', 'password-input']
}
```

## Data Table

### Basic Table with Pagination & Sorting

```vue
<script setup lang="ts">
import { ref, h } from 'vue'
import { NDataTable, NButton, NTag, type DataTableColumns } from 'naive-ui'

interface User {
  id: number
  name: string
  email: string
  status: 'active' | 'inactive'
}

const loading = ref(false)
const data = ref<User[]>([
  { id: 1, name: 'John Doe', email: 'john@example.com', status: 'active' },
  { id: 2, name: 'Jane Smith', email: 'jane@example.com', status: 'inactive' }
])

const columns: DataTableColumns<User> = [
  { title: 'ID', key: 'id', sorter: 'default', width: 80 },
  { title: 'Name', key: 'name', sorter: 'default' },
  { title: 'Email', key: 'email' },
  {
    title: 'Status',
    key: 'status',
    render(row) {
      return h(NTag, {
        type: row.status === 'active' ? 'success' : 'error'
      }, { default: () => row.status })
    },
    filterOptions: [
      { label: 'Active', value: 'active' },
      { label: 'Inactive', value: 'inactive' }
    ],
    filter: (value, row) => row.status === value
  },
  {
    title: 'Actions',
    key: 'actions',
    render(row) {
      return h(NButton, {
        size: 'small',
        onClick: () => handleEdit(row)
      }, { default: () => 'Edit' })
    }
  }
]

const pagination = ref({
  page: 1,
  pageSize: 10,
  showSizePicker: true,
  pageSizes: [10, 20, 50],
  onChange: (page: number) => {
    pagination.value.page = page
    fetchData()
  },
  onUpdatePageSize: (pageSize: number) => {
    pagination.value.pageSize = pageSize
    pagination.value.page = 1
    fetchData()
  }
})

function handleEdit(row: User) {
  console.log('Edit:', row)
}

async function fetchData() {
  loading.value = true
  // API call here
  loading.value = false
}
</script>

<template>
  <n-data-table
    :columns="columns"
    :data="data"
    :pagination="pagination"
    :loading="loading"
    :bordered="false"
    striped
  />
</template>
```

### Server-Side Pagination

```vue
<script setup lang="ts">
import { ref, onMounted } from 'vue'

const data = ref([])
const loading = ref(false)
const pagination = ref({
  page: 1,
  pageSize: 10,
  itemCount: 0,
  prefix: ({ itemCount }) => `Total: ${itemCount}`
})

async function fetchData(page: number, pageSize: number) {
  loading.value = true
  const response = await api.getUsers({ page, pageSize })
  data.value = response.data
  pagination.value.itemCount = response.total
  loading.value = false
}

function handlePageChange(page: number) {
  pagination.value.page = page
  fetchData(page, pagination.value.pageSize)
}

onMounted(() => fetchData(1, 10))
</script>

<template>
  <n-data-table
    remote
    :columns="columns"
    :data="data"
    :pagination="pagination"
    :loading="loading"
    @update:page="handlePageChange"
  />
</template>
```

## Feedback Components

### Message & Notification

```vue
<script setup lang="ts">
import { useMessage, useNotification, useDialog, useLoadingBar } from 'naive-ui'

const message = useMessage()
const notification = useNotification()
const dialog = useDialog()
const loadingBar = useLoadingBar()

function showMessage() {
  message.success('Operation successful')
  message.error('Something went wrong')
  message.warning('Warning message')
  message.info('Info message')
  message.loading('Loading...', { duration: 3000 })
}

function showNotification() {
  notification.success({
    title: 'Success',
    content: 'Your changes have been saved',
    duration: 5000,
    meta: 'Just now'
  })
}

function showDialog() {
  dialog.warning({
    title: 'Confirm Delete',
    content: 'Are you sure you want to delete this item?',
    positiveText: 'Delete',
    negativeText: 'Cancel',
    onPositiveClick: () => {
      message.success('Deleted')
    }
  })
}

async function withLoadingBar() {
  loadingBar.start()
  try {
    await api.saveData()
    loadingBar.finish()
  } catch {
    loadingBar.error()
  }
}
</script>
```

### Provider Setup for Composables

```vue
<!-- App.vue -->
<script setup lang="ts">
import { NMessageProvider, NNotificationProvider, NDialogProvider, NLoadingBarProvider } from 'naive-ui'
</script>

<template>
  <n-loading-bar-provider>
    <n-message-provider>
      <n-notification-provider>
        <n-dialog-provider>
          <router-view />
        </n-dialog-provider>
      </n-notification-provider>
    </n-message-provider>
  </n-loading-bar-provider>
</template>
```

## Common Components

### Select with Search

```vue
<script setup lang="ts">
import { ref } from 'vue'
import { NSelect, type SelectOption } from 'naive-ui'

const value = ref<string | null>(null)
const options: SelectOption[] = [
  { label: 'Vue', value: 'vue' },
  { label: 'React', value: 'react' },
  { label: 'Angular', value: 'angular' }
]

// Remote search
const loading = ref(false)
const remoteOptions = ref<SelectOption[]>([])

async function handleSearch(query: string) {
  if (!query) return
  loading.value = true
  const results = await api.search(query)
  remoteOptions.value = results.map(item => ({
    label: item.name,
    value: item.id
  }))
  loading.value = false
}
</script>

<template>
  <n-select
    v-model:value="value"
    :options="options"
    filterable
    clearable
    placeholder="Select framework"
  />

  <!-- Remote search -->
  <n-select
    v-model:value="value"
    :options="remoteOptions"
    :loading="loading"
    filterable
    remote
    clearable
    @search="handleSearch"
  />
</template>
```

### Modal Dialog

```vue
<script setup lang="ts">
import { ref } from 'vue'
import { NModal, NButton, NCard } from 'naive-ui'

const showModal = ref(false)
</script>

<template>
  <n-button @click="showModal = true">Open Modal</n-button>

  <n-modal v-model:show="showModal">
    <n-card
      style="width: 600px"
      title="Modal Title"
      :bordered="false"
      closable
      @close="showModal = false"
    >
      <p>Modal content here</p>
      <template #footer>
        <n-button @click="showModal = false">Cancel</n-button>
        <n-button type="primary" style="margin-left: 12px">Confirm</n-button>
      </template>
    </n-card>
  </n-modal>
</template>
```

### Upload Component

```vue
<script setup lang="ts">
import { ref } from 'vue'
import { NUpload, type UploadFileInfo, useMessage } from 'naive-ui'

const message = useMessage()
const fileList = ref<UploadFileInfo[]>([])

function handleChange({ file, fileList: newFileList }: { file: UploadFileInfo, fileList: UploadFileInfo[] }) {
  fileList.value = newFileList
}

function handleFinish({ file, event }: { file: UploadFileInfo, event?: ProgressEvent }) {
  const response = JSON.parse((event?.target as XMLHttpRequest).response)
  file.url = response.url
  message.success('Upload successful')
}

function handleError({ file }: { file: UploadFileInfo }) {
  message.error(`Failed to upload ${file.name}`)
}
</script>

<template>
  <n-upload
    action="/api/upload"
    :file-list="fileList"
    :max="5"
    accept="image/*"
    list-type="image-card"
    @change="handleChange"
    @finish="handleFinish"
    @error="handleError"
  >
    Click to Upload
  </n-upload>
</template>
```

## Layout Components

### Basic Layout

```vue
<script setup lang="ts">
import { NLayout, NLayoutHeader, NLayoutSider, NLayoutContent, NMenu } from 'naive-ui'
import { ref, h } from 'vue'
import { RouterLink } from 'vue-router'

const collapsed = ref(false)

const menuOptions = [
  {
    label: () => h(RouterLink, { to: '/' }, { default: () => 'Dashboard' }),
    key: 'dashboard',
    icon: renderIcon(DashboardIcon)
  },
  {
    label: 'Users',
    key: 'users',
    children: [
      { label: () => h(RouterLink, { to: '/users' }, { default: () => 'List' }), key: 'user-list' },
      { label: () => h(RouterLink, { to: '/users/new' }, { default: () => 'Add' }), key: 'user-add' }
    ]
  }
]
</script>

<template>
  <n-layout has-sider style="height: 100vh">
    <n-layout-sider
      bordered
      collapse-mode="width"
      :collapsed-width="64"
      :width="240"
      :collapsed="collapsed"
      show-trigger
      @collapse="collapsed = true"
      @expand="collapsed = false"
    >
      <n-menu
        :collapsed="collapsed"
        :collapsed-width="64"
        :collapsed-icon-size="22"
        :options="menuOptions"
      />
    </n-layout-sider>

    <n-layout>
      <n-layout-header bordered style="height: 64px; padding: 0 24px">
        Header
      </n-layout-header>
      <n-layout-content style="padding: 24px">
        <router-view />
      </n-layout-content>
    </n-layout>
  </n-layout>
</template>
```

## Best Practices

### DO
- Use `n-config-provider` at app root for theming
- Set up message/notification providers once at root
- Use TypeScript for component props and events
- Leverage built-in form validation
- Use `remote` prop for server-side data in tables/selects

### DON'T
- Don't mix Naive UI with other UI libraries
- Don't override styles directly, use theme overrides
- Don't forget to handle loading states
- Don't ignore TypeScript types from naive-ui

## Common Imports Cheatsheet

```typescript
// Layout
import { NLayout, NLayoutHeader, NLayoutSider, NLayoutContent, NLayoutFooter } from 'naive-ui'

// Form
import { NForm, NFormItem, NInput, NInputNumber, NSelect, NCheckbox, NRadio, NSwitch, NDatePicker } from 'naive-ui'

// Data Display
import { NDataTable, NTable, NList, NCard, NDescriptions, NTag, NAvatar, NBadge } from 'naive-ui'

// Feedback
import { NButton, NModal, NDrawer, NSpin, NProgress, NAlert } from 'naive-ui'

// Navigation
import { NMenu, NBreadcrumb, NTabs, NDropdown, NPagination } from 'naive-ui'

// Composables
import { useMessage, useNotification, useDialog, useLoadingBar } from 'naive-ui'

// Types
import type {
  FormInst, FormRules, FormItemRule,
  DataTableColumns, SelectOption,
  GlobalTheme, GlobalThemeOverrides
} from 'naive-ui'
```
