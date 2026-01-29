# API Layer

Axios-based HTTP client with interceptors and service pattern.

## Base API Setup

```typescript
// src/shared/api/base.ts
import axios, { type AxiosError, type AxiosResponse } from 'axios'
import Cookies from 'js-cookie'
import router from '@/app/router'

const api = axios.create({
  baseURL: import.meta.env.VITE_API_URL,
  headers: {
    'Content-Type': 'application/json',
    Accept: 'application/json',
  },
})

// Request interceptor - add auth token
api.interceptors.request.use((config) => {
  const token = Cookies.get('token')
  if (token) {
    config.headers.Authorization = `Bearer ${token}`
  }
  return config
})

// Response interceptor - handle errors
api.interceptors.response.use(
  (response: AxiosResponse) => response,
  async (error: AxiosError) => {
    const status = error.response?.status

    if (status === 401) {
      Cookies.remove('token')
      await router.push('/auth/login')
    }

    if (status === 422) {
      // Validation error - return for form handling
      return Promise.reject(error.response?.data)
    }

    if (status === 400) {
      // Handle blob responses (PDF errors)
      if (error.response?.data instanceof Blob) {
        const text = await error.response.data.text()
        const json = JSON.parse(text)
        return Promise.reject(json)
      }
      return Promise.reject(error.response?.data)
    }

    if (status === 429) {
      console.error('Rate limit exceeded')
    }

    if (status === 500) {
      console.error('Server error')
    }

    return Promise.reject(error)
  }
)

export default api
```

## API Methods

```typescript
// src/shared/api/base.ts (continued)

export async function get<T>(url: string, params?: object): Promise<T> {
  const response = await api.get<T>(url, { params })
  return response.data
}

export async function post<T>(url: string, data?: object): Promise<T> {
  const response = await api.post<T>(url, data)
  return response.data
}

export async function put<T>(url: string, data?: object): Promise<T> {
  const response = await api.put<T>(url, data)
  return response.data
}

export async function del<T>(url: string): Promise<T> {
  const response = await api.delete<T>(url)
  return response.data
}

// FormData upload
export async function formData<T>(url: string, data: FormData): Promise<T> {
  const response = await api.post<T>(url, data, {
    headers: { 'Content-Type': 'multipart/form-data' },
  })
  return response.data
}

// Blob response (PDF, Excel)
export async function print<T>(url: string, params?: object): Promise<T> {
  const response = await api.get<T>(url, {
    params,
    responseType: 'blob',
  })
  return response.data
}

// With credentials
export async function getWithCredentials<T>(url: string, params?: object): Promise<T> {
  const response = await api.get<T>(url, {
    params,
    withCredentials: true,
  })
  return response.data
}

export async function postWithCredentials<T>(url: string, data?: object): Promise<T> {
  const response = await api.post<T>(url, data, {
    withCredentials: true,
  })
  return response.data
}
```

## Service Pattern

```typescript
// src/shared/api/user.ts
import { get, post, put, del } from './base'
import type { IUser, IUserList, IUserForm, IUserListParams } from '@/entities/user'
import type { IPaginatedResponse } from '@/shared/types'

export const UserService = {
  getList(params?: IUserListParams) {
    return get<IPaginatedResponse<IUserList>>('/users', params)
  },

  show(id: number | string) {
    return get<IUser>(`/users/${id}`)
  },

  create(payload: IUserForm) {
    return post<IUser>('/users', payload)
  },

  update(id: number | string, payload: IUserForm) {
    return put<IUser>(`/users/${id}`, payload)
  },

  delete(id: number | string) {
    return del<void>(`/users/${id}`)
  },
}
```

## Service with Special Methods

```typescript
// src/shared/api/contract.ts
import { get, post, put, del, print, formData } from './base'
import type { IContract, IContractForm, IContractListParams } from '@/entities/contract'
import type { IPaginatedResponse } from '@/shared/types'

export const ContractService = {
  getList(params?: IContractListParams) {
    return get<IPaginatedResponse<IContract>>('/contracts', params)
  },

  show(id: number | string) {
    return get<IContract>(`/contracts/${id}`)
  },

  create(payload: IContractForm) {
    return post<IContract>('/contracts', payload)
  },

  update(id: number | string, payload: IContractForm) {
    return put<IContract>(`/contracts/${id}`, payload)
  },

  delete(id: number | string) {
    return del<void>(`/contracts/${id}`)
  },

  // Print PDF
  print(id: number | string) {
    return print<Blob>(`/contracts/${id}/print`)
  },

  // Upload with file
  uploadDocument(id: number | string, file: File) {
    const data = new FormData()
    data.append('document', file)
    return formData<IContract>(`/contracts/${id}/upload`, data)
  },

  // Action methods
  approve(id: number | string) {
    return post<IContract>(`/contracts/${id}/approve`)
  },

  reject(id: number | string, reason: string) {
    return post<IContract>(`/contracts/${id}/reject`, { reason })
  },
}
```

## API Index Export

```typescript
// src/shared/api/index.ts
export { default as api, get, post, put, del, formData, print } from './base'
export { UserService } from './user'
export { ClientService } from './client'
export { ContractService } from './contract'
export { RoleService } from './role'
export { TariffService } from './tariff'
// ... other services
```

## Usage in Components

```typescript
// In feature composable
import { UserService } from '@/shared/api'

async function loadUsers(): Promise<void> {
  loading.value = true
  try {
    const response = await UserService.getList({
      page: page.value,
      per_page: perPage.value,
      search: search.value,
    })
    users.value = response.data
    total.value = response.meta.total
  } catch (error) {
    message.error('Failed to load users')
  } finally {
    loading.value = false
  }
}
```

## Handling Validation Errors

```typescript
async function onSubmit(): Promise<void> {
  loading.value = true
  try {
    await UserService.create(form.value)
    message.success('User created')
    emit('success')
  } catch (error: any) {
    // 422 validation errors from API
    if (error.errors) {
      Object.values(error.errors).flat().forEach((msg: any) => {
        message.error(msg)
      })
    } else if (error.message) {
      message.error(error.message)
    } else {
      message.error('Failed to save')
    }
  } finally {
    loading.value = false
  }
}
```

## Download PDF/Blob

```typescript
async function handlePrint(id: number): Promise<void> {
  try {
    const blob = await ContractService.print(id)
    const url = window.URL.createObjectURL(blob)
    const link = document.createElement('a')
    link.href = url
    link.download = `contract-${id}.pdf`
    link.click()
    window.URL.revokeObjectURL(url)
  } catch (error) {
    message.error('Failed to download PDF')
  }
}
```

## Environment Configuration

```env
# .env
VITE_API_URL=http://localhost:8000/api

# .env.production
VITE_API_URL=https://api.example.com/api
```
