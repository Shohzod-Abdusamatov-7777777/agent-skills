# Shared UI Components

Reusable UI components in the shared layer.

## BaseTable

Data table wrapper with common defaults.

```vue
<!-- src/shared/ui/BaseTable.vue -->
<script setup lang="ts">
import { NDataTable } from 'naive-ui'
import type { DataTableColumns } from 'naive-ui'

interface Props {
  columns: DataTableColumns<any>
  data: any[]
  loading?: boolean
  rowKey?: string
  maxHeight?: number | string
  scrollX?: number | string
}

const props = withDefaults(defineProps<Props>(), {
  loading: false,
  rowKey: 'id',
  maxHeight: undefined,
  scrollX: undefined,
})
</script>

<template>
  <NDataTable
    :columns="columns"
    :data="data"
    :loading="loading"
    :row-key="(row: any) => row[rowKey]"
    :max-height="maxHeight"
    :scroll-x="scrollX"
    :bordered="false"
    striped
  >
    <template #empty>
      <div class="py-8 text-center text-gray-500">
        {{ $t('common.noData') }}
      </div>
    </template>
  </NDataTable>
</template>
```

## BaseModal

Modal dialog wrapper.

```vue
<!-- src/shared/ui/BaseModal.vue -->
<script setup lang="ts">
import { NModal, NCard } from 'naive-ui'

interface Props {
  show: boolean
  title: string
  width?: string | number
  closable?: boolean
  maskClosable?: boolean
}

const props = withDefaults(defineProps<Props>(), {
  width: 500,
  closable: true,
  maskClosable: false,
})

const emit = defineEmits<{
  'update:show': [value: boolean]
}>()

function handleClose(): void {
  emit('update:show', false)
}
</script>

<template>
  <NModal
    :show="show"
    :mask-closable="maskClosable"
    @update:show="emit('update:show', $event)"
  >
    <NCard
      :title="title"
      :style="{ width: typeof width === 'number' ? `${width}px` : width }"
      :closable="closable"
      @close="handleClose"
    >
      <slot />

      <template v-if="$slots.footer" #footer>
        <slot name="footer" />
      </template>
    </NCard>
  </NModal>
</template>
```

## BaseSearch

Search input with debounce.

```vue
<!-- src/shared/ui/BaseSearch.vue -->
<script setup lang="ts">
import { NInput } from 'naive-ui'
import { Search } from 'lucide-vue-next'

interface Props {
  modelValue: string
  placeholder?: string
  disabled?: boolean
}

const props = withDefaults(defineProps<Props>(), {
  placeholder: 'Search...',
  disabled: false,
})

const emit = defineEmits<{
  'update:modelValue': [value: string]
}>()
</script>

<template>
  <NInput
    :value="modelValue"
    :placeholder="placeholder"
    :disabled="disabled"
    clearable
    style="width: 250px"
    @update:value="emit('update:modelValue', $event)"
  >
    <template #prefix>
      <Search class="w-4 h-4 text-gray-400" />
    </template>
  </NInput>
</template>
```

## BasePagination

Pagination wrapper.

```vue
<!-- src/shared/ui/BasePagination.vue -->
<script setup lang="ts">
import { NPagination, NSpace, NSelect } from 'naive-ui'

interface Props {
  page: number
  perPage: number
  total: number
  pageSizes?: number[]
  showSizePicker?: boolean
}

const props = withDefaults(defineProps<Props>(), {
  pageSizes: () => [10, 15, 25, 50],
  showSizePicker: true,
})

const emit = defineEmits<{
  'update:page': [value: number]
  'update:perPage': [value: number]
}>()

const pageSizeOptions = props.pageSizes.map((size) => ({
  label: `${size} / page`,
  value: size,
}))
</script>

<template>
  <div class="flex items-center justify-between">
    <div class="text-sm text-gray-500">
      {{ $t('common.total') }}: {{ total }}
    </div>

    <NSpace>
      <NSelect
        v-if="showSizePicker"
        :value="perPage"
        :options="pageSizeOptions"
        size="small"
        style="width: 120px"
        @update:value="emit('update:perPage', $event)"
      />

      <NPagination
        :page="page"
        :page-size="perPage"
        :item-count="total"
        :page-slot="7"
        @update:page="emit('update:page', $event)"
      />
    </NSpace>
  </div>
</template>
```

## BaseDescriptions

Description list for detail views.

```vue
<!-- src/shared/ui/BaseDescriptions.vue -->
<script setup lang="ts">
import { NDescriptions, NDescriptionsItem } from 'naive-ui'

interface DescriptionItem {
  label: string
  value: any
  span?: number
}

interface Props {
  items: DescriptionItem[]
  column?: number
  labelPlacement?: 'left' | 'top'
  bordered?: boolean
}

const props = withDefaults(defineProps<Props>(), {
  column: 2,
  labelPlacement: 'left',
  bordered: false,
})
</script>

<template>
  <NDescriptions
    :column="column"
    :label-placement="labelPlacement"
    :bordered="bordered"
  >
    <NDescriptionsItem
      v-for="item in items"
      :key="item.label"
      :label="item.label"
      :span="item.span"
    >
      {{ item.value ?? '-' }}
    </NDescriptionsItem>
  </NDescriptions>
</template>
```

## StateBadge

Status badge component.

```vue
<!-- src/shared/ui/StateBadge.vue -->
<script setup lang="ts">
import { computed } from 'vue'
import { NTag } from 'naive-ui'

interface Props {
  status: string
  size?: 'small' | 'medium' | 'large'
}

const props = withDefaults(defineProps<Props>(), {
  size: 'small',
})

const statusConfig: Record<string, { type: 'success' | 'warning' | 'error' | 'info' | 'default'; label: string }> = {
  active: { type: 'success', label: 'Active' },
  inactive: { type: 'default', label: 'Inactive' },
  pending: { type: 'warning', label: 'Pending' },
  approved: { type: 'success', label: 'Approved' },
  rejected: { type: 'error', label: 'Rejected' },
  open: { type: 'success', label: 'Open' },
  closed: { type: 'default', label: 'Closed' },
  processing: { type: 'info', label: 'Processing' },
}

const config = computed(() => statusConfig[props.status] || { type: 'default', label: props.status })
</script>

<template>
  <NTag :type="config.type" :size="size" round>
    {{ config.label }}
  </NTag>
</template>
```

## Action Buttons

### AddBtn

```vue
<!-- src/shared/ui/AddBtn.vue -->
<script setup lang="ts">
import { NButton } from 'naive-ui'
import { Plus } from 'lucide-vue-next'

interface Props {
  text?: string
  size?: 'tiny' | 'small' | 'medium' | 'large'
  disabled?: boolean
}

const props = withDefaults(defineProps<Props>(), {
  text: undefined,
  size: 'medium',
  disabled: false,
})

const emit = defineEmits<{
  click: []
}>()
</script>

<template>
  <NButton
    type="primary"
    :size="size"
    :disabled="disabled"
    @click="emit('click')"
  >
    <template #icon>
      <Plus class="w-4 h-4" />
    </template>
    {{ text ?? $t('common.add') }}
  </NButton>
</template>
```

### EditBtn

```vue
<!-- src/shared/ui/EditBtn.vue -->
<script setup lang="ts">
import { NButton, NTooltip } from 'naive-ui'
import { Edit } from 'lucide-vue-next'

interface Props {
  size?: 'tiny' | 'small' | 'medium' | 'large'
  disabled?: boolean
  tooltip?: string
}

const props = withDefaults(defineProps<Props>(), {
  size: 'small',
  disabled: false,
  tooltip: undefined,
})

const emit = defineEmits<{
  click: []
}>()
</script>

<template>
  <NTooltip v-if="tooltip">
    <template #trigger>
      <NButton
        text
        :size="size"
        :disabled="disabled"
        @click="emit('click')"
      >
        <Edit class="w-4 h-4" />
      </NButton>
    </template>
    {{ tooltip }}
  </NTooltip>

  <NButton
    v-else
    text
    :size="size"
    :disabled="disabled"
    @click="emit('click')"
  >
    <Edit class="w-4 h-4" />
  </NButton>
</template>
```

### DeleteBtn

```vue
<!-- src/shared/ui/DeleteBtn.vue -->
<script setup lang="ts">
import { NButton, NPopconfirm } from 'naive-ui'
import { Trash2 } from 'lucide-vue-next'

interface Props {
  size?: 'tiny' | 'small' | 'medium' | 'large'
  disabled?: boolean
  confirmText?: string
}

const props = withDefaults(defineProps<Props>(), {
  size: 'small',
  disabled: false,
  confirmText: undefined,
})

const emit = defineEmits<{
  confirm: []
}>()
</script>

<template>
  <NPopconfirm @positive-click="emit('confirm')">
    <template #trigger>
      <NButton text type="error" :size="size" :disabled="disabled">
        <Trash2 class="w-4 h-4" />
      </NButton>
    </template>
    {{ confirmText ?? $t('common.deleteConfirm') }}
  </NPopconfirm>
</template>
```

### SaveBtn

```vue
<!-- src/shared/ui/SaveBtn.vue -->
<script setup lang="ts">
import { NButton } from 'naive-ui'
import { Save } from 'lucide-vue-next'

interface Props {
  text?: string
  size?: 'tiny' | 'small' | 'medium' | 'large'
  loading?: boolean
  disabled?: boolean
}

const props = withDefaults(defineProps<Props>(), {
  text: undefined,
  size: 'medium',
  loading: false,
  disabled: false,
})

const emit = defineEmits<{
  click: []
}>()
</script>

<template>
  <NButton
    type="primary"
    :size="size"
    :loading="loading"
    :disabled="disabled"
    @click="emit('click')"
  >
    <template v-if="!loading" #icon>
      <Save class="w-4 h-4" />
    </template>
    {{ text ?? $t('common.save') }}
  </NButton>
</template>
```

## CopyText

Copy to clipboard component.

```vue
<!-- src/shared/ui/CopyText.vue -->
<script setup lang="ts">
import { ref } from 'vue'
import { NButton, NTooltip, useMessage } from 'naive-ui'
import { Copy, Check } from 'lucide-vue-next'

interface Props {
  text: string
  successMessage?: string
}

const props = withDefaults(defineProps<Props>(), {
  successMessage: 'Copied!',
})

const message = useMessage()
const copied = ref(false)

async function handleCopy(): Promise<void> {
  try {
    await navigator.clipboard.writeText(props.text)
    copied.value = true
    message.success(props.successMessage)
    setTimeout(() => {
      copied.value = false
    }, 2000)
  } catch {
    message.error('Failed to copy')
  }
}
</script>

<template>
  <span class="inline-flex items-center gap-1">
    <slot>{{ text }}</slot>
    <NTooltip>
      <template #trigger>
        <NButton text size="tiny" @click="handleCopy">
          <Check v-if="copied" class="w-3 h-3 text-green-500" />
          <Copy v-else class="w-3 h-3" />
        </NButton>
      </template>
      {{ copied ? 'Copied!' : 'Copy' }}
    </NTooltip>
  </span>
</template>
```

## PhoneInput

Phone number input with mask.

```vue
<!-- src/shared/ui/PhoneInput.vue -->
<script setup lang="ts">
import { NInput } from 'naive-ui'
import { vMaska } from 'maska'

interface Props {
  modelValue: string
  disabled?: boolean
  placeholder?: string
}

const props = withDefaults(defineProps<Props>(), {
  disabled: false,
  placeholder: '+998 __ ___ __ __',
})

const emit = defineEmits<{
  'update:modelValue': [value: string]
}>()
</script>

<template>
  <NInput
    :value="modelValue"
    v-maska
    data-maska="+998 ## ### ## ##"
    :placeholder="placeholder"
    :disabled="disabled"
    @update:value="emit('update:modelValue', $event)"
  />
</template>
```

## Index Export

```typescript
// src/shared/ui/index.ts
export { default as BaseTable } from './BaseTable.vue'
export { default as BaseModal } from './BaseModal.vue'
export { default as BaseSearch } from './BaseSearch.vue'
export { default as BasePagination } from './BasePagination.vue'
export { default as BaseDescriptions } from './BaseDescriptions.vue'
export { default as StateBadge } from './StateBadge.vue'
export { default as CopyText } from './CopyText.vue'
export { default as AddBtn } from './AddBtn.vue'
export { default as EditBtn } from './EditBtn.vue'
export { default as DeleteBtn } from './DeleteBtn.vue'
export { default as SaveBtn } from './SaveBtn.vue'
export { default as PhoneInput } from './PhoneInput.vue'
```
