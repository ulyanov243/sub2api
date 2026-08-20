# Pinia Store 文档

此目录包含 Sub2API 前端应用的全部 Pinia store。

## Store 概览

### 1. Auth Store（`auth.ts`）

管理用户认证状态、登录/登出及 token 持久化。

**状态：**

- `user: User | null` - 当前已认证用户。
- `token: string | null` - JWT 认证 token。

**计算属性：**

- `isAuthenticated: boolean` - 用户当前是否已认证。

**操作：**

- `login(credentials)` - 使用用户名/密码认证用户。
- `register(userData)` - 注册新用户账户。
- `logout()` - 清除认证状态并登出。
- `checkAuth()` - 从 localStorage 恢复会话。
- `refreshUser()` - 从服务端获取最新用户数据。

### 2. App Store（`app.ts`）

管理全局 UI 状态，包括侧边栏、加载指示器和 Toast 通知。

**状态：**

- `sidebarCollapsed: boolean` - 侧边栏折叠状态。
- `loading: boolean` - 全局加载状态。
- `toasts: Toast[]` - 活动 Toast 通知。

**计算属性：**

- `hasActiveToasts: boolean` - 是否有活动的 Toast。

**操作：**

- `toggleSidebar()` - 切换侧边栏状态。
- `setSidebarCollapsed(collapsed)` - 显式设置侧边栏状态。
- `setLoading(isLoading)` - 设置加载状态。
- `showToast(type, message, duration?)` - 显示 Toast 通知。
- `showSuccess(message, duration?)` - 显示成功 Toast。
- `showError(message, duration?)` - 显示错误 Toast。
- `showInfo(message, duration?)` - 显示信息 Toast。
- `showWarning(message, duration?)` - 显示警告 Toast。
- `hideToast(id)` - 隐藏指定 Toast。
- `clearAllToasts()` - 清除所有 Toast。
- `withLoading(operation)` - 在加载状态下执行异步操作。
- `withLoadingAndError(operation, errorMessage?)` - 执行带加载和错误处理的操作。
- `reset()` - 将 store 重置为默认值。

## 使用示例

### Auth Store

```typescript
import { useAuthStore } from '@/stores'

// In component setup
const authStore = useAuthStore()

// Initialize on app startup
authStore.checkAuth()

// Login
try {
  await authStore.login({ username: 'user', password: 'pass' })
  console.log('Logged in:', authStore.user)
} catch (error) {
  console.error('Login failed:', error)
}

// Check authentication
if (authStore.isAuthenticated) {
  console.log('User is logged in:', authStore.user?.username)
}

// Logout
authStore.logout()
```

### App Store

```typescript
import { useAppStore } from '@/stores'

// In component setup
const appStore = useAppStore()

// Sidebar control
appStore.toggleSidebar()
appStore.setSidebarCollapsed(true)

// Loading state
appStore.setLoading(true)
// ... do work
appStore.setLoading(false)

// Or use helper
await appStore.withLoading(async () => {
  const data = await fetchData()
  return data
})

// Toast notifications
appStore.showSuccess('Operation completed!')
appStore.showError('Something went wrong!', 5000)
appStore.showInfo('FYI: This is informational')
appStore.showWarning('Be careful!')

// Custom toast
const toastId = appStore.showToast('info', 'Custom message', undefined) // No auto-dismiss
// Later...
appStore.hideToast(toastId)
```

### 在 Vue 组件中组合使用

```vue
<script setup lang="ts">
import { useAuthStore, useAppStore } from '@/stores'
import { onMounted } from 'vue'

const authStore = useAuthStore()
const appStore = useAppStore()

onMounted(() => {
  // Check for existing session
  authStore.checkAuth()
})

async function handleLogin(username: string, password: string) {
  try {
    await appStore.withLoading(async () => {
      await authStore.login({ username, password })
    })
    appStore.showSuccess('Welcome back!')
  } catch (error) {
    appStore.showError('Login failed. Please check your credentials.')
  }
}

async function handleLogout() {
  authStore.logout()
  appStore.showInfo('You have been logged out.')
}
</script>

<template>
  <div>
    <button @click="appStore.toggleSidebar">Toggle Sidebar</button>

    <div v-if="appStore.loading">Loading...</div>

    <div v-if="authStore.isAuthenticated">
      Welcome, {{ authStore.user?.username }}!
      <button @click="handleLogout">Logout</button>
    </div>
    <div v-else>
      <button @click="handleLogin('user', 'pass')">Login</button>
    </div>
  </div>
</template>
```

## 持久化

- **Auth Store**：Token 和用户数据会自动持久化到 `localStorage`。
  - 键：`auth_token`、`auth_user`。
  - 调用 `checkAuth()` 时恢复。
- **App Store**：不持久化（页面刷新时 UI 状态重置）。

## TypeScript 支持

所有 store 均具有完整的 TypeScript 类型。请从 `@/types` 导入类型：

```typescript
import type { User, Toast, ToastType } from '@/types'
```

## 测试

Store 可重置为初始状态：

```typescript
// Auth store
authStore.logout() // Clears all auth state

// App store
appStore.reset() // Resets to defaults
```
