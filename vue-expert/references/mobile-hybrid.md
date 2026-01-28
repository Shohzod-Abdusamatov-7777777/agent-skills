# Mobile & Hybrid Apps

> Reference for: Vue Expert
> Load when: Quasar, Capacitor, PWA, service worker, mobile, hybrid app, offline

---

## Quasar Framework

### Project Setup

```bash
# Create new Quasar project
npm init quasar

# Add Quasar to existing Vue project
npm install quasar @quasar/extras
npm install -D @quasar/vite-plugin
```

```typescript
// vite.config.ts - Quasar plugin setup
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'
import { quasar, transformAssetUrls } from '@quasar/vite-plugin'

export default defineConfig({
  plugins: [
    vue({
      template: { transformAssetUrls }
    }),
    quasar({
      sassVariables: 'src/quasar-variables.scss'
    })
  ]
})
```

### Quasar Configuration

```javascript
// quasar.config.js
export default configure((ctx) => ({
  // Build modes: spa, pwa, ssr, capacitor, electron, bex
  boot: ['axios', 'i18n'],

  css: ['app.scss'],

  extras: [
    'roboto-font',
    'material-icons'
  ],

  framework: {
    plugins: ['Notify', 'Dialog', 'Loading', 'LocalStorage'],
    config: {
      notify: { position: 'top-right' },
      loading: { spinnerColor: 'primary' }
    }
  },

  build: {
    target: { browser: ['es2022', 'firefox115', 'chrome115', 'safari14'] },
    vueRouterMode: 'history'
  }
}))
```

### Quasar Components with Composition API

```vue
<script setup lang="ts">
import { useQuasar } from 'quasar'

const $q = useQuasar()

function showNotification() {
  $q.notify({
    message: 'Action completed successfully',
    type: 'positive',
    position: 'top',
    timeout: 3000
  })
}

function showConfirmDialog() {
  $q.dialog({
    title: 'Confirm',
    message: 'Are you sure you want to proceed?',
    cancel: true,
    persistent: true
  }).onOk(() => {
    // User confirmed
  })
}

async function showLoading() {
  $q.loading.show({ message: 'Processing...' })
  await doAsyncWork()
  $q.loading.hide()
}
</script>
```

### Layout System

```vue
<template>
  <q-layout view="lHh Lpr lFf">
    <q-header elevated>
      <q-toolbar>
        <q-btn flat dense round icon="menu" @click="toggleLeftDrawer" />
        <q-toolbar-title>My App</q-toolbar-title>
        <q-btn flat round icon="person" />
      </q-toolbar>
    </q-header>

    <q-drawer v-model="leftDrawerOpen" show-if-above bordered>
      <q-list>
        <q-item clickable v-ripple to="/dashboard">
          <q-item-section avatar>
            <q-icon name="dashboard" />
          </q-item-section>
          <q-item-section>Dashboard</q-item-section>
        </q-item>
      </q-list>
    </q-drawer>

    <q-page-container>
      <router-view />
    </q-page-container>
  </q-layout>
</template>

<script setup lang="ts">
import { ref } from 'vue'

const leftDrawerOpen = ref(false)

function toggleLeftDrawer() {
  leftDrawerOpen.value = !leftDrawerOpen.value
}
</script>
```

### Platform Detection

```vue
<script setup lang="ts">
import { useQuasar } from 'quasar'

const $q = useQuasar()

// Platform detection
const isMobile = $q.platform.is.mobile
const isIOS = $q.platform.is.ios
const isAndroid = $q.platform.is.android
const isDesktop = $q.platform.is.desktop
const isCapacitor = $q.platform.is.capacitor

// Screen utilities
const isSmallScreen = $q.screen.lt.md
const screenWidth = $q.screen.width
</script>

<template>
  <div>
    <MobileNav v-if="isMobile" />
    <DesktopNav v-else />
  </div>
</template>
```

---

## Capacitor Integration

### Setup

```bash
# Add Capacitor to Quasar
quasar mode add capacitor

# Initialize Capacitor
cd src-capacitor
npx cap init "App Name" "com.example.app"

# Add platforms
npx cap add android
npx cap add ios

# Sync and run
npx cap sync
npx cap open android
```

### Capacitor Configuration

```typescript
// capacitor.config.ts
import type { CapacitorConfig } from '@capacitor/cli'

const config: CapacitorConfig = {
  appId: 'com.example.myapp',
  appName: 'My App',
  webDir: 'dist/spa',
  server: {
    androidScheme: 'https',
    // For development
    url: 'http://192.168.1.100:9000',
    cleartext: true
  },
  plugins: {
    SplashScreen: {
      launchAutoHide: false,
      showSpinner: true
    },
    PushNotifications: {
      presentationOptions: ['badge', 'sound', 'alert']
    }
  }
}

export default config
```

### Native Plugins with TypeScript

```typescript
// composables/useCamera.ts
import { ref } from 'vue'
import { Camera, CameraResultType, CameraSource } from '@capacitor/camera'

export function useCamera() {
  const photo = ref<string | null>(null)
  const error = ref<string | null>(null)

  async function takePhoto() {
    try {
      const image = await Camera.getPhoto({
        resultType: CameraResultType.Uri,
        source: CameraSource.Camera,
        quality: 90
      })
      photo.value = image.webPath ?? null
    } catch (e) {
      error.value = (e as Error).message
    }
  }

  async function pickFromGallery() {
    try {
      const image = await Camera.getPhoto({
        resultType: CameraResultType.Uri,
        source: CameraSource.Photos,
        quality: 90
      })
      photo.value = image.webPath ?? null
    } catch (e) {
      error.value = (e as Error).message
    }
  }

  return { photo, error, takePhoto, pickFromGallery }
}
```

```typescript
// composables/useGeolocation.ts
import { ref, onMounted, onUnmounted } from 'vue'
import { Geolocation, Position } from '@capacitor/geolocation'

export function useGeolocation() {
  const position = ref<Position | null>(null)
  const error = ref<string | null>(null)
  let watchId: string | null = null

  async function getCurrentPosition() {
    try {
      position.value = await Geolocation.getCurrentPosition({
        enableHighAccuracy: true
      })
    } catch (e) {
      error.value = (e as Error).message
    }
  }

  async function watchPosition() {
    watchId = await Geolocation.watchPosition(
      { enableHighAccuracy: true },
      (pos, err) => {
        if (err) {
          error.value = err.message
        } else if (pos) {
          position.value = pos
        }
      }
    )
  }

  function stopWatching() {
    if (watchId) {
      Geolocation.clearWatch({ id: watchId })
      watchId = null
    }
  }

  onUnmounted(stopWatching)

  return { position, error, getCurrentPosition, watchPosition, stopWatching }
}
```

### Push Notifications

```typescript
// composables/usePushNotifications.ts
import { ref, onMounted } from 'vue'
import { PushNotifications, Token, PushNotificationSchema } from '@capacitor/push-notifications'
import { Capacitor } from '@capacitor/core'

export function usePushNotifications() {
  const token = ref<string | null>(null)
  const notifications = ref<PushNotificationSchema[]>([])

  async function register() {
    if (!Capacitor.isNativePlatform()) return

    const permission = await PushNotifications.requestPermissions()
    if (permission.receive !== 'granted') return

    await PushNotifications.register()
  }

  onMounted(() => {
    if (!Capacitor.isNativePlatform()) return

    PushNotifications.addListener('registration', (t: Token) => {
      token.value = t.value
    })

    PushNotifications.addListener('pushNotificationReceived', (notification) => {
      notifications.value.push(notification)
    })

    PushNotifications.addListener('pushNotificationActionPerformed', (action) => {
      // Handle notification tap
      console.log('Action:', action.actionId)
    })
  })

  return { token, notifications, register }
}
```

### App Lifecycle

```typescript
// composables/useAppLifecycle.ts
import { onMounted, onUnmounted } from 'vue'
import { App } from '@capacitor/app'
import { Capacitor } from '@capacitor/core'

export function useAppLifecycle() {
  onMounted(() => {
    if (!Capacitor.isNativePlatform()) return

    App.addListener('appStateChange', ({ isActive }) => {
      if (isActive) {
        // App came to foreground
        refreshData()
      } else {
        // App went to background
        saveState()
      }
    })

    App.addListener('backButton', ({ canGoBack }) => {
      if (!canGoBack) {
        App.exitApp()
      } else {
        window.history.back()
      }
    })
  })

  onUnmounted(() => {
    App.removeAllListeners()
  })
}
```

---

## PWA & Service Workers

### Workbox Configuration

```javascript
// quasar.config.js
export default configure((ctx) => ({
  pwa: {
    workboxMode: 'GenerateSW', // or 'InjectManifest'

    workboxOptions: {
      skipWaiting: true,
      clientsClaim: true,
      cleanupOutdatedCaches: true,

      // Cache strategies
      runtimeCaching: [
        {
          // Cache API responses
          urlPattern: /^https:\/\/api\./,
          handler: 'NetworkFirst',
          options: {
            cacheName: 'api-cache',
            networkTimeoutSeconds: 10,
            expiration: {
              maxEntries: 100,
              maxAgeSeconds: 60 * 60 * 24 // 24 hours
            }
          }
        },
        {
          // Cache images
          urlPattern: /\.(?:png|jpg|jpeg|svg|gif|webp)$/,
          handler: 'CacheFirst',
          options: {
            cacheName: 'image-cache',
            expiration: {
              maxEntries: 50,
              maxAgeSeconds: 60 * 60 * 24 * 30 // 30 days
            }
          }
        },
        {
          // Cache fonts
          urlPattern: /\.(?:woff|woff2|ttf|eot)$/,
          handler: 'CacheFirst',
          options: {
            cacheName: 'font-cache',
            expiration: {
              maxAgeSeconds: 60 * 60 * 24 * 365 // 1 year
            }
          }
        }
      ]
    }
  }
}))
```

### Web App Manifest

```javascript
// quasar.config.js
export default configure((ctx) => ({
  pwa: {
    manifest: {
      name: 'My Progressive App',
      short_name: 'MyApp',
      description: 'A Progressive Web Application',
      display: 'standalone',
      orientation: 'portrait',
      background_color: '#ffffff',
      theme_color: '#1976D2',
      start_url: '/',
      icons: [
        {
          src: 'icons/icon-128x128.png',
          sizes: '128x128',
          type: 'image/png'
        },
        {
          src: 'icons/icon-512x512.png',
          sizes: '512x512',
          type: 'image/png'
        }
      ]
    }
  }
}))
```

### Install Prompt Handling

```typescript
// composables/usePWAInstall.ts
import { ref, onMounted } from 'vue'

interface BeforeInstallPromptEvent extends Event {
  prompt(): Promise<void>
  userChoice: Promise<{ outcome: 'accepted' | 'dismissed' }>
}

export function usePWAInstall() {
  const canInstall = ref(false)
  const isInstalled = ref(false)
  let deferredPrompt: BeforeInstallPromptEvent | null = null

  onMounted(() => {
    // Check if already installed
    isInstalled.value = window.matchMedia('(display-mode: standalone)').matches

    window.addEventListener('beforeinstallprompt', (e) => {
      e.preventDefault()
      deferredPrompt = e as BeforeInstallPromptEvent
      canInstall.value = true
    })

    window.addEventListener('appinstalled', () => {
      isInstalled.value = true
      canInstall.value = false
      deferredPrompt = null
    })
  })

  async function install() {
    if (!deferredPrompt) return false

    await deferredPrompt.prompt()
    const { outcome } = await deferredPrompt.userChoice

    deferredPrompt = null
    canInstall.value = false

    return outcome === 'accepted'
  }

  return { canInstall, isInstalled, install }
}
```

### PWA Update Flow

```typescript
// composables/usePWAUpdate.ts
import { ref, onMounted } from 'vue'
import { useQuasar } from 'quasar'

export function usePWAUpdate() {
  const $q = useQuasar()
  const needsUpdate = ref(false)
  let registration: ServiceWorkerRegistration | null = null

  onMounted(() => {
    if (!('serviceWorker' in navigator)) return

    navigator.serviceWorker.ready.then((reg) => {
      registration = reg

      reg.addEventListener('updatefound', () => {
        const newWorker = reg.installing
        if (!newWorker) return

        newWorker.addEventListener('statechange', () => {
          if (newWorker.state === 'installed' && navigator.serviceWorker.controller) {
            needsUpdate.value = true
            promptUpdate()
          }
        })
      })
    })
  })

  function promptUpdate() {
    $q.notify({
      message: 'A new version is available',
      timeout: 0,
      actions: [
        {
          label: 'Update',
          color: 'white',
          handler: updateApp
        },
        {
          label: 'Later',
          color: 'white'
        }
      ]
    })
  }

  function updateApp() {
    if (registration?.waiting) {
      registration.waiting.postMessage({ type: 'SKIP_WAITING' })
    }
    window.location.reload()
  }

  return { needsUpdate, updateApp }
}
```

### Offline Detection

```typescript
// composables/useOnlineStatus.ts
import { ref, onMounted, onUnmounted } from 'vue'

export function useOnlineStatus() {
  const isOnline = ref(navigator.onLine)

  function updateOnlineStatus() {
    isOnline.value = navigator.onLine
  }

  onMounted(() => {
    window.addEventListener('online', updateOnlineStatus)
    window.addEventListener('offline', updateOnlineStatus)
  })

  onUnmounted(() => {
    window.removeEventListener('online', updateOnlineStatus)
    window.removeEventListener('offline', updateOnlineStatus)
  })

  return { isOnline }
}
```

---

## Additional Capacitor Plugins

### Biometric Authentication

```typescript
// composables/useBiometrics.ts
import { ref } from 'vue'
import { NativeBiometric, BiometryType } from 'capacitor-native-biometric'

export function useBiometrics() {
  const isAvailable = ref(false)
  const biometryType = ref<BiometryType>(BiometryType.NONE)

  async function checkAvailability() {
    try {
      const result = await NativeBiometric.isAvailable()
      isAvailable.value = result.isAvailable
      biometryType.value = result.biometryType
    } catch {
      isAvailable.value = false
    }
  }

  async function authenticate(reason = 'Authenticate to continue') {
    try {
      await NativeBiometric.verifyIdentity({
        reason,
        title: 'Authentication',
        subtitle: 'Use biometrics to login',
        maxAttempts: 3
      })
      return true
    } catch {
      return false
    }
  }

  // Store credentials securely
  async function saveCredentials(server: string, username: string, password: string) {
    await NativeBiometric.setCredentials({
      server,
      username,
      password
    })
  }

  async function getCredentials(server: string) {
    return await NativeBiometric.getCredentials({ server })
  }

  return { isAvailable, biometryType, checkAvailability, authenticate, saveCredentials, getCredentials }
}
```

### Local Notifications

```typescript
// composables/useLocalNotifications.ts
import { LocalNotifications, LocalNotificationSchema } from '@capacitor/local-notifications'

export function useLocalNotifications() {
  async function requestPermission() {
    const permission = await LocalNotifications.requestPermissions()
    return permission.display === 'granted'
  }

  async function schedule(options: Partial<LocalNotificationSchema>) {
    await LocalNotifications.schedule({
      notifications: [{
        id: Date.now(),
        title: options.title || 'Notification',
        body: options.body || '',
        schedule: options.schedule || { at: new Date(Date.now() + 1000) },
        sound: 'default',
        smallIcon: 'ic_notification',
        ...options
      }]
    })
  }

  async function scheduleReminder(title: string, body: string, date: Date) {
    await schedule({
      title,
      body,
      schedule: { at: date }
    })
  }

  async function cancelAll() {
    const pending = await LocalNotifications.getPending()
    if (pending.notifications.length > 0) {
      await LocalNotifications.cancel({ notifications: pending.notifications })
    }
  }

  return { requestPermission, schedule, scheduleReminder, cancelAll }
}
```

### File System

```typescript
// composables/useFileSystem.ts
import { Filesystem, Directory, Encoding } from '@capacitor/filesystem'

export function useFileSystem() {
  async function writeFile(path: string, data: string) {
    await Filesystem.writeFile({
      path,
      data,
      directory: Directory.Documents,
      encoding: Encoding.UTF8
    })
  }

  async function readFile(path: string) {
    const result = await Filesystem.readFile({
      path,
      directory: Directory.Documents,
      encoding: Encoding.UTF8
    })
    return result.data as string
  }

  async function deleteFile(path: string) {
    await Filesystem.deleteFile({
      path,
      directory: Directory.Documents
    })
  }

  async function listFiles(path = '') {
    const result = await Filesystem.readdir({
      path,
      directory: Directory.Documents
    })
    return result.files
  }

  async function downloadFile(url: string, filename: string) {
    const response = await fetch(url)
    const blob = await response.blob()
    const base64 = await blobToBase64(blob)

    await Filesystem.writeFile({
      path: filename,
      data: base64,
      directory: Directory.Documents
    })
  }

  return { writeFile, readFile, deleteFile, listFiles, downloadFile }
}
```

### Haptics

```typescript
// composables/useHaptics.ts
import { Haptics, ImpactStyle, NotificationType } from '@capacitor/haptics'

export function useHaptics() {
  async function impact(style: ImpactStyle = ImpactStyle.Medium) {
    await Haptics.impact({ style })
  }

  async function notification(type: NotificationType = NotificationType.Success) {
    await Haptics.notification({ type })
  }

  async function vibrate(duration = 300) {
    await Haptics.vibrate({ duration })
  }

  async function selectionStart() {
    await Haptics.selectionStart()
  }

  async function selectionChanged() {
    await Haptics.selectionChanged()
  }

  async function selectionEnd() {
    await Haptics.selectionEnd()
  }

  return { impact, notification, vibrate, selectionStart, selectionChanged, selectionEnd }
}

// Usage
const { impact, notification } = useHaptics()
impact(ImpactStyle.Heavy) // Button press
notification(NotificationType.Success) // Action completed
```

### Share & Social

```typescript
// composables/useShare.ts
import { Share } from '@capacitor/share'

export function useShare() {
  async function shareText(title: string, text: string) {
    await Share.share({ title, text })
  }

  async function shareUrl(title: string, url: string) {
    await Share.share({ title, url })
  }

  async function shareFile(title: string, files: string[]) {
    await Share.share({ title, files })
  }

  async function canShare() {
    const result = await Share.canShare()
    return result.value
  }

  return { shareText, shareUrl, shareFile, canShare }
}
```

### Status Bar & Navigation Bar

```typescript
// composables/useStatusBar.ts
import { StatusBar, Style } from '@capacitor/status-bar'
import { NavigationBar } from '@capgo/capacitor-navigation-bar'

export function useStatusBar() {
  async function setDarkContent() {
    await StatusBar.setStyle({ style: Style.Dark })
  }

  async function setLightContent() {
    await StatusBar.setStyle({ style: Style.Light })
  }

  async function setBackgroundColor(color: string) {
    await StatusBar.setBackgroundColor({ color })
  }

  async function hide() {
    await StatusBar.hide()
  }

  async function show() {
    await StatusBar.show()
  }

  return { setDarkContent, setLightContent, setBackgroundColor, hide, show }
}

export function useNavigationBar() {
  async function setColor(color: string, darkButtons = false) {
    await NavigationBar.setColor({ color, darkButtons })
  }

  async function hide() {
    await NavigationBar.hide()
  }

  async function show() {
    await NavigationBar.show()
  }

  return { setColor, hide, show }
}
```

---

## Deep Linking

```typescript
// composables/useDeepLinks.ts
import { App } from '@capacitor/app'
import { useRouter } from 'vue-router'

export function useDeepLinks() {
  const router = useRouter()

  function init() {
    App.addListener('appUrlOpen', ({ url }) => {
      const path = new URL(url).pathname
      router.push(path)
    })
  }

  return { init }
}

// capacitor.config.ts
const config: CapacitorConfig = {
  plugins: {
    App: {
      androidScheme: 'https',
      android: {
        intentFilters: [
          {
            action: { intent: 'android.intent.action.VIEW' },
            autoVerify: true,
            data: [
              { scheme: 'https', host: 'myapp.com', pathPrefix: '/' }
            ],
            categories: ['android.intent.category.DEFAULT', 'android.intent.category.BROWSABLE']
          }
        ]
      }
    }
  }
}
```

---

## Quick Reference

| Pattern | Use Case |
|---------|----------|
| `useQuasar()` | Access Quasar plugins ($q) |
| `$q.platform.is.*` | Platform detection |
| `$q.notify()` | Toast notifications |
| `$q.dialog()` | Modal dialogs |
| `@capacitor/camera` | Native camera access |
| `@capacitor/geolocation` | GPS location |
| `@capacitor/push-notifications` | Push notifications |
| `@capacitor/local-notifications` | Local scheduled notifications |
| `@capacitor/filesystem` | File read/write |
| `@capacitor/haptics` | Vibration feedback |
| `@capacitor/share` | Native share sheet |
| `@capacitor/status-bar` | Status bar styling |
| `capacitor-native-biometric` | Face ID / Fingerprint |
| `workboxMode: 'GenerateSW'` | Auto-generate service worker |
| `runtimeCaching` | Workbox cache strategies |
| `beforeinstallprompt` | PWA install prompt |
| `navigator.serviceWorker.ready` | Service worker lifecycle |
