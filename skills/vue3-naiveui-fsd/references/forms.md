# Form Patterns

Feature-based form composable pattern for Vue 3 + Naive UI.

## Form Composable Pattern

Each form has a dedicated composable in the feature's `model/` folder.

```typescript
// src/features/user/model/useUserForm.ts
import { ref, onMounted, useTemplateRef } from 'vue'
import { useMessage, type FormInst, type FormRules } from 'naive-ui'
import { useI18n } from 'vue-i18n'
import { useValidationRules } from '@/shared/composables'
import { UserService } from '@/shared/api'
import type { IUserForm } from '@/entities/user'

// Default form values
const defaultForm: IUserForm = {
  name: '',
  email: '',
  phone: '',
  role_id: null,
  password: '',
  password_confirmation: '',
}

// Emit type for the form
interface UseUserFormEmit {
  (e: 'success'): void
  (e: 'cancel'): void
}

export function useUserForm(emit: UseUserFormEmit, userId?: number) {
  const { t } = useI18n()
  const message = useMessage()
  const { requiredRule, emailRule, minLengthRule, phoneRules } = useValidationRules()

  // Form ref using useTemplateRef (Vue 3.5+)
  const formRef = useTemplateRef<FormInst>('formRef')

  // Form state
  const form = ref<IUserForm>({ ...defaultForm })
  const loading = ref(false)
  const isEditMode = ref(!!userId)

  // Validation rules
  const rules: FormRules = {
    name: [requiredRule(t('user.name')), minLengthRule(2)],
    email: [requiredRule(t('user.email')), emailRule],
    phone: phoneRules,
    role_id: [
      {
        required: true,
        type: 'number',
        message: t('validation.required', { field: t('user.role') }),
        trigger: ['blur', 'change'],
      },
    ],
    // Password required only on create
    password: isEditMode.value
      ? []
      : [requiredRule(t('user.password')), minLengthRule(8)],
  }

  // Load existing user data
  async function loadUser(id: number): Promise<void> {
    loading.value = true
    try {
      const user = await UserService.show(id)
      form.value = {
        name: user.name,
        email: user.email,
        phone: user.phone,
        role_id: user.role_id,
        password: '',
        password_confirmation: '',
      }
    } catch {
      message.error(t('error.loadFailed'))
    } finally {
      loading.value = false
    }
  }

  // Submit handler
  async function onSubmit(e: Event): Promise<void> {
    e.preventDefault()

    // Validate form
    try {
      await formRef.value?.validate()
    } catch {
      return
    }

    loading.value = true
    try {
      const payload = { ...form.value }

      // Remove empty password on update
      if (isEditMode.value && !payload.password) {
        delete payload.password
        delete payload.password_confirmation
      }

      if (userId) {
        await UserService.update(userId, payload)
        message.success(t('user.updateSuccess'))
      } else {
        await UserService.create(payload)
        message.success(t('user.createSuccess'))
      }

      emit('success')
    } catch (error: any) {
      handleApiError(error)
    } finally {
      loading.value = false
    }
  }

  // Handle API validation errors
  function handleApiError(error: any): void {
    if (error.errors) {
      // 422 validation errors
      Object.values(error.errors).flat().forEach((msg: any) => {
        message.error(msg)
      })
    } else if (error.message) {
      message.error(error.message)
    } else {
      message.error(t('error.saveFailed'))
    }
  }

  // Reset form to default
  function resetForm(): void {
    form.value = JSON.parse(JSON.stringify(defaultForm))
    formRef.value?.restoreValidation()
  }

  // Load user on mount if editing
  onMounted(() => {
    if (userId) {
      loadUser(userId)
    }
  })

  return {
    form,
    formRef,
    rules,
    loading,
    isEditMode,
    onSubmit,
    resetForm,
    loadUser,
  }
}
```

## Form Component

```vue
<!-- src/features/user/ui/UserForm.vue -->
<script setup lang="ts">
import { NForm, NFormItem, NInput, NButton, NSpace, NGrid, NGi } from 'naive-ui'
import { useUserForm } from '../model/useUserForm'
import { RoleSelect } from '@/entities/catalog'
import { PhoneInput } from '@/shared/ui'

const props = defineProps<{
  userId?: number
}>()

const emit = defineEmits<{
  success: []
  cancel: []
}>()

const { form, rules, loading, isEditMode, onSubmit } = useUserForm(
  emit,
  props.userId
)
</script>

<template>
  <NForm
    ref="formRef"
    :model="form"
    :rules="rules"
    label-placement="top"
    require-mark-placement="right-hanging"
    @submit.prevent="onSubmit"
  >
    <NGrid :cols="2" :x-gap="16">
      <NGi>
        <NFormItem :label="$t('user.name')" path="name">
          <NInput
            v-model:value="form.name"
            :placeholder="$t('user.namePlaceholder')"
            :disabled="loading"
          />
        </NFormItem>
      </NGi>

      <NGi>
        <NFormItem :label="$t('user.email')" path="email">
          <NInput
            v-model:value="form.email"
            type="email"
            :placeholder="$t('user.emailPlaceholder')"
            :disabled="loading"
          />
        </NFormItem>
      </NGi>

      <NGi>
        <NFormItem :label="$t('user.phone')" path="phone">
          <PhoneInput v-model="form.phone" :disabled="loading" />
        </NFormItem>
      </NGi>

      <NGi>
        <NFormItem :label="$t('user.role')" path="role_id">
          <RoleSelect v-model:value="form.role_id" :disabled="loading" />
        </NFormItem>
      </NGi>

      <NGi v-if="!isEditMode">
        <NFormItem :label="$t('user.password')" path="password">
          <NInput
            v-model:value="form.password"
            type="password"
            show-password-on="click"
            :placeholder="$t('user.passwordPlaceholder')"
            :disabled="loading"
          />
        </NFormItem>
      </NGi>

      <NGi v-if="!isEditMode">
        <NFormItem
          :label="$t('user.passwordConfirmation')"
          path="password_confirmation"
        >
          <NInput
            v-model:value="form.password_confirmation"
            type="password"
            show-password-on="click"
            :placeholder="$t('user.passwordConfirmationPlaceholder')"
            :disabled="loading"
          />
        </NFormItem>
      </NGi>
    </NGrid>

    <NSpace justify="end" class="mt-4">
      <NButton :disabled="loading" @click="emit('cancel')">
        {{ $t('common.cancel') }}
      </NButton>
      <NButton type="primary" :loading="loading" attr-type="submit">
        {{ isEditMode ? $t('common.update') : $t('common.create') }}
      </NButton>
    </NSpace>
  </NForm>
</template>
```

## Complex Form with Tabs

```vue
<!-- src/features/client/ui/ClientForm.vue -->
<script setup lang="ts">
import { ref } from 'vue'
import { NForm, NFormItem, NInput, NTabs, NTabPane, NButton, NSpace } from 'naive-ui'
import { useClientForm } from '../model/useClientForm'
import { ClientTypeSelect, CurrencySelect } from '@/entities/catalog'
import { AddressForm } from './AddressForm.vue'
import { ContactsForm } from './ContactsForm.vue'

const props = defineProps<{ clientId?: number }>()
const emit = defineEmits<{ success: []; cancel: [] }>()

const { form, rules, loading, isEditMode, onSubmit } = useClientForm(
  emit,
  props.clientId
)

const activeTab = ref('basic')
</script>

<template>
  <NForm
    ref="formRef"
    :model="form"
    :rules="rules"
    label-placement="top"
    @submit.prevent="onSubmit"
  >
    <NTabs v-model:value="activeTab" type="line">
      <NTabPane name="basic" :tab="$t('client.basicInfo')">
        <NFormItem :label="$t('client.name')" path="name">
          <NInput v-model:value="form.name" :disabled="loading" />
        </NFormItem>

        <NFormItem :label="$t('client.type')" path="type">
          <ClientTypeSelect v-model:value="form.type" :disabled="loading" />
        </NFormItem>

        <NFormItem :label="$t('client.inn')" path="inn">
          <NInput v-model:value="form.inn" :disabled="loading" />
        </NFormItem>
      </NTabPane>

      <NTabPane name="address" :tab="$t('client.address')">
        <AddressForm v-model="form.address" :disabled="loading" />
      </NTabPane>

      <NTabPane name="contacts" :tab="$t('client.contacts')">
        <ContactsForm v-model="form.contacts" :disabled="loading" />
      </NTabPane>
    </NTabs>

    <NSpace justify="end" class="mt-4">
      <NButton :disabled="loading" @click="emit('cancel')">
        {{ $t('common.cancel') }}
      </NButton>
      <NButton type="primary" :loading="loading" attr-type="submit">
        {{ isEditMode ? $t('common.update') : $t('common.create') }}
      </NButton>
    </NSpace>
  </NForm>
</template>
```

## Form with Dynamic Fields

```vue
<script setup lang="ts">
import { NForm, NFormItem, NInput, NButton, NSpace, NDynamicInput } from 'naive-ui'

const form = ref({
  name: '',
  phones: [''],
  emails: [''],
})

const rules: FormRules = {
  name: [{ required: true, message: 'Required' }],
  phones: {
    type: 'array',
    required: true,
    min: 1,
    message: 'At least one phone required',
  },
}
</script>

<template>
  <NForm :model="form" :rules="rules">
    <NFormItem label="Name" path="name">
      <NInput v-model:value="form.name" />
    </NFormItem>

    <NFormItem label="Phone Numbers" path="phones">
      <NDynamicInput
        v-model:value="form.phones"
        :min="1"
        :max="5"
        placeholder="Enter phone"
      />
    </NFormItem>

    <NFormItem label="Emails" path="emails">
      <NDynamicInput
        v-model:value="form.emails"
        placeholder="Enter email"
      >
        <template #create-button-default>
          Add Email
        </template>
      </NDynamicInput>
    </NFormItem>
  </NForm>
</template>
```

## Form with File Upload

```typescript
// useDocumentForm.ts
export function useDocumentForm(emit: UseFormEmit) {
  const form = ref({
    title: '',
    file: null as File | null,
  })

  async function onSubmit(): Promise<void> {
    if (!form.value.file) {
      message.error('Please select a file')
      return
    }

    const formData = new FormData()
    formData.append('title', form.value.title)
    formData.append('file', form.value.file)

    loading.value = true
    try {
      await DocumentService.upload(formData)
      emit('success')
    } finally {
      loading.value = false
    }
  }

  function handleFileChange(options: { file: { file: File } }): void {
    form.value.file = options.file.file
  }

  return { form, onSubmit, handleFileChange }
}
```

```vue
<template>
  <NForm :model="form">
    <NFormItem label="Title" path="title">
      <NInput v-model:value="form.title" />
    </NFormItem>

    <NFormItem label="File" path="file">
      <NUpload
        :max="1"
        :default-upload="false"
        @change="handleFileChange"
      >
        <NButton>Select File</NButton>
      </NUpload>
    </NFormItem>

    <NButton type="primary" @click="onSubmit">
      Upload
    </NButton>
  </NForm>
</template>
```

## Custom Validation

```typescript
const rules: FormRules = {
  // Async validation
  email: [
    { required: true, message: 'Email required' },
    {
      async asyncValidator(rule, value) {
        const exists = await UserService.checkEmail(value)
        if (exists) {
          return new Error('Email already exists')
        }
      },
      trigger: 'blur',
    },
  ],

  // Custom validator
  password_confirmation: [
    {
      validator(rule, value) {
        if (value !== form.value.password) {
          return new Error('Passwords do not match')
        }
        return true
      },
      trigger: ['blur', 'input'],
    },
  ],

  // Conditional validation
  inn: [
    {
      validator(rule, value) {
        if (form.value.type === 'legal_entity' && !value) {
          return new Error('INN required for legal entities')
        }
        return true
      },
      trigger: 'blur',
    },
  ],
}
```
