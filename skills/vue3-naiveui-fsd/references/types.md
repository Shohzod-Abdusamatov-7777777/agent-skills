# TypeScript Patterns

Type definitions and naming conventions for Vue 3 + FSD projects.

## Type Naming Convention (I-prefix)

All interfaces use the `I` prefix for clear identification:

```typescript
// src/entities/user/model/types.ts

// Base entity type
export interface IUser {
  id: number
  name: string
  email: string
  phone: string
  role_id: number
  role: IRole
  status: UserStatus
  created_at: string
  updated_at: string
}

// List response type (simplified for tables)
export interface IUserList {
  id: number
  name: string
  email: string
  phone: string
  role_name: string
  status: UserStatus
}

// Detail response type (full data with relations)
export interface IUserDetail extends IUser {
  permissions: string[]
  last_login_at: string | null
  created_by: IUser | null
}

// Form submission type
export interface IUserForm {
  name: string
  email: string
  phone: string
  role_id: number | null
  password?: string
  password_confirmation?: string
}

// Query parameters type
export interface IUserListParams {
  page?: number
  per_page?: number
  search?: string
  role_id?: number
  status?: UserStatus
  sortBy?: string
  descending?: boolean
}

// Status as union type (not interface)
export type UserStatus = 'active' | 'inactive' | 'pending'
```

## Common Types

```typescript
// src/shared/types/common.ts

// Paginated API response
export interface IPaginatedResponse<T> {
  data: T[]
  meta: {
    current_page: number
    last_page: number
    per_page: number
    total: number
  }
}

// Pagination state
export interface IPagination {
  page: number
  per_page: number
  total: number
  sortBy?: string
  descending?: boolean
}

// Select option for dropdowns
export interface ISelectOption {
  label: string
  value: number | string
}

// Nullable utility type
export type Nullable<T> = T | null

// API error response
export interface IApiError {
  message: string
  errors?: Record<string, string[]>
}
```

## Auth Types

```typescript
// src/shared/types/auth.ts

export interface LoginDto {
  phone: string
  password: string
}

export interface LoginResponseDto {
  token: string
  user: IUser
}

export interface IUser {
  id: number
  name: string
  email: string
  phone: string
  role: IRole
  permissions: string[]
}

export interface IRole {
  id: number
  name: string
  permissions: string[]
}
```

## Entity Type Pattern

For each entity, create these type variations:

```typescript
// Pattern for any entity
export interface IEntity {
  id: number
  // ... base fields
  created_at: string
  updated_at: string
}

export interface IEntityList {
  id: number
  // ... simplified fields for table display
}

export interface IEntityDetail extends IEntity {
  // ... additional fields with relations
}

export interface IEntityForm {
  // ... fields for create/update form
  // Note: no id, created_at, updated_at
}

export interface IEntityListParams {
  page?: number
  per_page?: number
  search?: string
  // ... filter fields
  sortBy?: string
  descending?: boolean
}
```

## Example: Client Entity

```typescript
// src/entities/client/model/types.ts

export interface IClient {
  id: number
  name: string
  phone: string
  inn: string
  address: string
  type: ClientType
  status: ClientStatus
  created_at: string
  updated_at: string
}

export interface IClientList {
  id: number
  name: string
  phone: string
  inn: string
  type_name: string
  status: ClientStatus
}

export interface IClientDetail extends IClient {
  contracts: IContract[]
  transactions: ITransaction[]
  total_debt: number
  manager: IUser | null
}

export interface IClientForm {
  name: string
  phone: string
  inn: string
  address: string
  type: ClientType | null
  manager_id: number | null
}

export interface IClientListParams {
  page?: number
  per_page?: number
  search?: string
  type?: ClientType
  status?: ClientStatus
  manager_id?: number
  sortBy?: string
  descending?: boolean
}

export type ClientType = 'individual' | 'legal_entity' | 'entrepreneur'
export type ClientStatus = 'active' | 'inactive' | 'blocked'
```

## Enum Pattern

```typescript
// Use union types for simple enums
export type UserStatus = 'active' | 'inactive' | 'pending'

// Use TypeScript enum for complex cases with values
export enum SubjectType {
  LEGAL_ENTITY = 'legal_entity',
  INDIVIDUAL = 'individual',
  ENTREPRENEUR = 'entrepreneur',
}

// Enum with display labels
export const SubjectTypeLabels: Record<SubjectType, string> = {
  [SubjectType.LEGAL_ENTITY]: 'Yuridik shaxs',
  [SubjectType.INDIVIDUAL]: 'Jismoniy shaxs',
  [SubjectType.ENTREPRENEUR]: 'Yakka tartibdagi tadbirkor',
}
```

## Generic Service Response Types

```typescript
// src/shared/types/api.ts

// Single item response
export interface IResponse<T> {
  data: T
  message?: string
}

// List response with pagination
export interface IPaginatedResponse<T> {
  data: T[]
  meta: {
    current_page: number
    last_page: number
    per_page: number
    total: number
  }
}

// Success response
export interface ISuccessResponse {
  success: boolean
  message: string
}

// Error response
export interface IErrorResponse {
  message: string
  errors?: Record<string, string[]>
}
```

## Component Props/Emits Types

```typescript
// In component
interface Props {
  user: IUser
  loading?: boolean
  editable?: boolean
}

interface Emits {
  (e: 'success'): void
  (e: 'cancel'): void
  (e: 'update', user: IUser): void
  (e: 'delete', id: number): void
}

const props = withDefaults(defineProps<Props>(), {
  loading: false,
  editable: true,
})

const emit = defineEmits<Emits>()

// Vue 3.3+ shorthand for emits
const emit = defineEmits<{
  success: []
  cancel: []
  update: [user: IUser]
  delete: [id: number]
}>()
```

## Type Guards

```typescript
// Type guard function
function isUser(value: unknown): value is IUser {
  return (
    typeof value === 'object' &&
    value !== null &&
    'id' in value &&
    'email' in value
  )
}

// Usage
if (isUser(data)) {
  console.log(data.email) // TypeScript knows data is IUser
}
```

## Utility Types

```typescript
// Make specific fields required
type RequireFields<T, K extends keyof T> = T & Required<Pick<T, K>>

// Make specific fields optional
type OptionalFields<T, K extends keyof T> = Omit<T, K> & Partial<Pick<T, K>>

// Extract keys by value type
type KeysOfType<T, V> = {
  [K in keyof T]: T[K] extends V ? K : never
}[keyof T]

// Deep partial
type DeepPartial<T> = {
  [P in keyof T]?: T[P] extends object ? DeepPartial<T[P]> : T[P]
}
```

## Index Exports

```typescript
// src/shared/types/index.ts
export type { IPaginatedResponse, IPagination, ISelectOption, Nullable } from './common'
export type { LoginDto, LoginResponseDto, IUser, IRole } from './auth'

// src/entities/user/index.ts
export type {
  IUser,
  IUserList,
  IUserDetail,
  IUserForm,
  IUserListParams,
  UserStatus,
} from './model/types'
```
