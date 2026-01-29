# Utility Functions

Helper functions for formatting and common operations.

## Formatters

```typescript
// src/shared/lib/formatters.ts
import dayjs from 'dayjs'

/**
 * Format date to DD.MM.YYYY
 */
export function formatDate(date: string | Date | null | undefined): string {
  if (!date) return '-'
  return dayjs(date).format('DD.MM.YYYY')
}

/**
 * Format date to DD.MM.YYYY HH:mm
 */
export function formatDateTime(date: string | Date | null | undefined): string {
  if (!date) return '-'
  return dayjs(date).format('DD.MM.YYYY HH:mm')
}

/**
 * Format date to relative time (e.g., "2 hours ago")
 */
export function formatRelativeTime(date: string | Date | null | undefined): string {
  if (!date) return '-'
  return dayjs(date).fromNow()
}

/**
 * Format phone number to +998 XX XXX XX XX
 */
export function formatPhone(phone: string | null | undefined): string {
  if (!phone) return '-'
  const cleaned = phone.replace(/\D/g, '')
  if (cleaned.length !== 12) return phone
  return `+${cleaned.slice(0, 3)} ${cleaned.slice(3, 5)} ${cleaned.slice(5, 8)} ${cleaned.slice(8, 10)} ${cleaned.slice(10)}`
}

/**
 * Format currency with locale
 */
export function formatCurrency(
  amount: number | null | undefined,
  currency = 'UZS'
): string {
  if (amount === null || amount === undefined) return '-'
  return new Intl.NumberFormat('uz-UZ', {
    style: 'currency',
    currency,
    minimumFractionDigits: 0,
    maximumFractionDigits: 0,
  }).format(amount)
}

/**
 * Format number with thousand separators
 */
export function formatNumber(value: number | null | undefined): string {
  if (value === null || value === undefined) return '-'
  return new Intl.NumberFormat('uz-UZ').format(value)
}

/**
 * Format percentage
 */
export function formatPercent(value: number | null | undefined, decimals = 1): string {
  if (value === null || value === undefined) return '-'
  return `${value.toFixed(decimals)}%`
}

/**
 * Format file size
 */
export function formatFileSize(bytes: number | null | undefined): string {
  if (bytes === null || bytes === undefined) return '-'
  const units = ['B', 'KB', 'MB', 'GB', 'TB']
  let unitIndex = 0
  let size = bytes

  while (size >= 1024 && unitIndex < units.length - 1) {
    size /= 1024
    unitIndex++
  }

  return `${size.toFixed(1)} ${units[unitIndex]}`
}

/**
 * Format INN (9 digits)
 */
export function formatInn(inn: string | null | undefined): string {
  if (!inn) return '-'
  const cleaned = inn.replace(/\D/g, '')
  if (cleaned.length !== 9) return inn
  return `${cleaned.slice(0, 3)} ${cleaned.slice(3, 6)} ${cleaned.slice(6)}`
}

/**
 * Truncate text with ellipsis
 */
export function truncateText(text: string | null | undefined, maxLength = 50): string {
  if (!text) return '-'
  if (text.length <= maxLength) return text
  return `${text.slice(0, maxLength)}...`
}

/**
 * Format full name (first + last)
 */
export function formatFullName(
  firstName: string | null | undefined,
  lastName: string | null | undefined
): string {
  const parts = [firstName, lastName].filter(Boolean)
  return parts.length > 0 ? parts.join(' ') : '-'
}
```

## Date Utilities

```typescript
// src/shared/lib/dateUtils.ts
import dayjs from 'dayjs'

/**
 * Get start of today
 */
export function startOfToday(): string {
  return dayjs().startOf('day').toISOString()
}

/**
 * Get end of today
 */
export function endOfToday(): string {
  return dayjs().endOf('day').toISOString()
}

/**
 * Get start of current month
 */
export function startOfMonth(): string {
  return dayjs().startOf('month').toISOString()
}

/**
 * Get end of current month
 */
export function endOfMonth(): string {
  return dayjs().endOf('month').toISOString()
}

/**
 * Calculate remaining days until date
 */
export function calculateRemainingDays(endDate: string | Date): number {
  const end = dayjs(endDate)
  const now = dayjs()
  return end.diff(now, 'day')
}

/**
 * Check if date is past
 */
export function isPastDate(date: string | Date): boolean {
  return dayjs(date).isBefore(dayjs(), 'day')
}

/**
 * Check if date is today
 */
export function isToday(date: string | Date): boolean {
  return dayjs(date).isSame(dayjs(), 'day')
}

/**
 * Get date range for filter
 */
export function getDateRange(period: 'today' | 'week' | 'month' | 'year'): [string, string] {
  const now = dayjs()

  switch (period) {
    case 'today':
      return [now.startOf('day').toISOString(), now.endOf('day').toISOString()]
    case 'week':
      return [now.startOf('week').toISOString(), now.endOf('week').toISOString()]
    case 'month':
      return [now.startOf('month').toISOString(), now.endOf('month').toISOString()]
    case 'year':
      return [now.startOf('year').toISOString(), now.endOf('year').toISOString()]
  }
}
```

## General Utilities

```typescript
// src/shared/lib/utils.ts

/**
 * Debounce function
 */
export function debounce<T extends (...args: any[]) => any>(
  fn: T,
  delay: number
): (...args: Parameters<T>) => void {
  let timeoutId: ReturnType<typeof setTimeout>

  return (...args: Parameters<T>) => {
    clearTimeout(timeoutId)
    timeoutId = setTimeout(() => fn(...args), delay)
  }
}

/**
 * Throttle function
 */
export function throttle<T extends (...args: any[]) => any>(
  fn: T,
  limit: number
): (...args: Parameters<T>) => void {
  let inThrottle = false

  return (...args: Parameters<T>) => {
    if (!inThrottle) {
      fn(...args)
      inThrottle = true
      setTimeout(() => {
        inThrottle = false
      }, limit)
    }
  }
}

/**
 * Deep clone object
 */
export function deepClone<T>(obj: T): T {
  return JSON.parse(JSON.stringify(obj))
}

/**
 * Check if object is empty
 */
export function isEmpty(obj: object): boolean {
  return Object.keys(obj).length === 0
}

/**
 * Remove undefined/null values from object
 */
export function cleanObject<T extends object>(obj: T): Partial<T> {
  return Object.fromEntries(
    Object.entries(obj).filter(([_, v]) => v !== undefined && v !== null)
  ) as Partial<T>
}

/**
 * Generate random ID
 */
export function generateId(): string {
  return Math.random().toString(36).substring(2, 9)
}

/**
 * Sleep utility
 */
export function sleep(ms: number): Promise<void> {
  return new Promise((resolve) => setTimeout(resolve, ms))
}

/**
 * Download blob as file
 */
export function downloadBlob(blob: Blob, filename: string): void {
  const url = window.URL.createObjectURL(blob)
  const link = document.createElement('a')
  link.href = url
  link.download = filename
  document.body.appendChild(link)
  link.click()
  document.body.removeChild(link)
  window.URL.revokeObjectURL(url)
}

/**
 * Copy text to clipboard
 */
export async function copyToClipboard(text: string): Promise<boolean> {
  try {
    await navigator.clipboard.writeText(text)
    return true
  } catch {
    return false
  }
}

/**
 * Parse query string to object
 */
export function parseQueryString(queryString: string): Record<string, string> {
  const params = new URLSearchParams(queryString)
  const result: Record<string, string> = {}

  params.forEach((value, key) => {
    result[key] = value
  })

  return result
}

/**
 * Build query string from object
 */
export function buildQueryString(params: Record<string, any>): string {
  const cleaned = cleanObject(params)
  return new URLSearchParams(cleaned as Record<string, string>).toString()
}
```

## Validators

```typescript
// src/shared/lib/validators.ts

/**
 * Validate email format
 */
export function isValidEmail(email: string): boolean {
  const pattern = /^[^\s@]+@[^\s@]+\.[^\s@]+$/
  return pattern.test(email)
}

/**
 * Validate phone format (+998XXXXXXXXX)
 */
export function isValidPhone(phone: string): boolean {
  const cleaned = phone.replace(/\D/g, '')
  return cleaned.length === 12 && cleaned.startsWith('998')
}

/**
 * Validate INN format (9 digits)
 */
export function isValidInn(inn: string): boolean {
  const cleaned = inn.replace(/\D/g, '')
  return cleaned.length === 9
}

/**
 * Validate password strength
 */
export function isStrongPassword(password: string): boolean {
  // At least 8 characters, 1 uppercase, 1 lowercase, 1 number
  const pattern = /^(?=.*[a-z])(?=.*[A-Z])(?=.*\d).{8,}$/
  return pattern.test(password)
}

/**
 * Validate URL format
 */
export function isValidUrl(url: string): boolean {
  try {
    new URL(url)
    return true
  } catch {
    return false
  }
}
```

## Index Export

```typescript
// src/shared/lib/index.ts
export {
  formatDate,
  formatDateTime,
  formatRelativeTime,
  formatPhone,
  formatCurrency,
  formatNumber,
  formatPercent,
  formatFileSize,
  formatInn,
  truncateText,
  formatFullName,
} from './formatters'

export {
  startOfToday,
  endOfToday,
  startOfMonth,
  endOfMonth,
  calculateRemainingDays,
  isPastDate,
  isToday,
  getDateRange,
} from './dateUtils'

export {
  debounce,
  throttle,
  deepClone,
  isEmpty,
  cleanObject,
  generateId,
  sleep,
  downloadBlob,
  copyToClipboard,
  parseQueryString,
  buildQueryString,
} from './utils'

export {
  isValidEmail,
  isValidPhone,
  isValidInn,
  isStrongPassword,
  isValidUrl,
} from './validators'
```
