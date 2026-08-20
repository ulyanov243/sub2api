# 布局组件集成指南

## 快速开始

### 1. 导入布局组件

```typescript
// In your view files
import { AppLayout, AuthLayout } from '@/components/layout'
```

### 2. 在路由中使用

```typescript
// src/router/index.ts
import { createRouter, createWebHistory } from 'vue-router'
import type { RouteRecordRaw } from 'vue-router'

// Views
import DashboardView from '@/views/DashboardView.vue'
import LoginView from '@/views/auth/LoginView.vue'
import RegisterView from '@/views/auth/RegisterView.vue'

const routes: RouteRecordRaw[] = [
  // Auth routes (no layout needed - views use AuthLayout internally)
  {
    path: '/login',
    name: 'Login',
    component: LoginView,
    meta: { requiresAuth: false }
  },
  {
    path: '/register',
    name: 'Register',
    component: RegisterView,
    meta: { requiresAuth: false }
  },

  // User routes (use AppLayout)
  {
    path: '/dashboard',
    name: 'Dashboard',
    component: DashboardView,
    meta: { requiresAuth: true, title: 'Dashboard' }
  },
  {
    path: '/api-keys',
    name: 'ApiKeys',
    component: () => import('@/views/ApiKeysView.vue'),
    meta: { requiresAuth: true, title: 'API Keys' }
  },
  {
    path: '/usage',
    name: 'Usage',
    component: () => import('@/views/UsageView.vue'),
    meta: { requiresAuth: true, title: 'Usage Statistics' }
  },
  {
    path: '/redeem',
    name: 'Redeem',
    component: () => import('@/views/RedeemView.vue'),
    meta: { requiresAuth: true, title: 'Redeem Code' }
  },
  {
    path: '/profile',
    name: 'Profile',
    component: () => import('@/views/ProfileView.vue'),
    meta: { requiresAuth: true, title: 'Profile Settings' }
  },

  // Admin routes (use AppLayout, admin only)
  {
    path: '/admin/dashboard',
    name: 'AdminDashboard',
    component: () => import('@/views/admin/DashboardView.vue'),
    meta: { requiresAuth: true, requiresAdmin: true, title: 'Admin Dashboard' }
  },
  {
    path: '/admin/users',
    name: 'AdminUsers',
    component: () => import('@/views/admin/UsersView.vue'),
    meta: { requiresAuth: true, requiresAdmin: true, title: 'User Management' }
  },
  {
    path: '/admin/groups',
    name: 'AdminGroups',
    component: () => import('@/views/admin/GroupsView.vue'),
    meta: { requiresAuth: true, requiresAdmin: true, title: 'Groups' }
  },
  {
    path: '/admin/accounts',
    name: 'AdminAccounts',
    component: () => import('@/views/admin/AccountsView.vue'),
    meta: { requiresAuth: true, requiresAdmin: true, title: 'Accounts' }
  },
  {
    path: '/admin/proxies',
    name: 'AdminProxies',
    component: () => import('@/views/admin/ProxiesView.vue'),
    meta: { requiresAuth: true, requiresAdmin: true, title: 'Proxies' }
  },
  {
    path: '/admin/redeem-codes',
    name: 'AdminRedeemCodes',
    component: () => import('@/views/admin/RedeemCodesView.vue'),
    meta: { requiresAuth: true, requiresAdmin: true, title: 'Redeem Codes' }
  },

  // Default redirect
  {
    path: '/',
    redirect: '/dashboard'
  }
]

const router = createRouter({
  history: createWebHistory(),
  routes
})

// Navigation guards
router.beforeEach((to, from, next) => {
  const authStore = useAuthStore()

  if (to.meta.requiresAuth && !authStore.isAuthenticated) {
    // Redirect to login if not authenticated
    next('/login')
  } else if (to.meta.requiresAdmin && !authStore.isAdmin) {
    // Redirect to dashboard if not admin
    next('/dashboard')
  } else {
    next()
  }
})

export default router
```

### 3. 在 main.ts 中初始化 Store

```typescript
// src/main.ts
import { createApp } from 'vue'
import { createPinia } from 'pinia'
import App from './App.vue'
import router from './router'
import './style.css'

const app = createApp(App)
const pinia = createPinia()

app.use(pinia)
app.use(router)

// Initialize auth state on app startup
import { useAuthStore } from '@/stores'
const authStore = useAuthStore()
authStore.checkAuth()

app.mount('#app')
```

### 4. 更新 App.vue

```vue
<!-- src/App.vue -->
<template>
  <router-view />
</template>

<script setup lang="ts">
// App.vue just renders the router view
// Layouts are handled by individual views
</script>
```

---

## View 组件模板

### 已认证页面模板

```vue
<!-- src/views/DashboardView.vue -->
<template>
  <AppLayout>
    <div class="space-y-6">
      <h1 class="text-3xl font-bold text-gray-900">Dashboard</h1>

      <!-- Your content here -->
    </div>
  </AppLayout>
</template>

<script setup lang="ts">
import { AppLayout } from '@/components/layout'

// Your component logic here
</script>
```

### 认证页面模板

```vue
<!-- src/views/auth/LoginView.vue -->
<template>
  <AuthLayout>
    <h2 class="mb-6 text-2xl font-bold text-gray-900">Login</h2>

    <!-- Your login form here -->

    <template #footer>
      <p class="text-gray-600">
        Don't have an account?
        <router-link to="/register" class="text-indigo-600 hover:underline"> Sign up </router-link>
      </p>
    </template>
  </AuthLayout>
</template>

<script setup lang="ts">
import { AuthLayout } from '@/components/layout'

// Your login logic here
</script>
```

---

## 自定义

### 更改颜色

组件默认使用 Tailwind 的 indigo 配色。要更改：

```vue
<!-- Change all instances of indigo-* to your preferred color -->
<div class="bg-blue-600">   <!-- Instead of bg-indigo-600 -->
<div class="text-blue-600">  <!-- Instead of text-indigo-600 -->
```

### 添加自定义图标

将 HTML 实体图标替换为您偏好的图标库：

```vue
<!-- Before (HTML entities) -->
<span class="text-lg">&#128200;</span>

<!-- After (Heroicons example) -->
<ChartBarIcon class="h-5 w-5" />
```

### 自定义侧边栏

在 `AppSidebar.vue` 中修改导航项：

```typescript
// Add/remove/modify navigation items
const userNavItems = [
  { path: '/dashboard', label: 'Dashboard', icon: '&#128200;' },
  { path: '/new-page', label: 'New Page', icon: '&#128196;' } // Add new item
  // ...
]
```

### 自定义页头

在 `AppHeader.vue` 中修改用户下拉菜单：

```vue
<!-- Add new dropdown items -->
<router-link
  to="/settings"
  @click="closeDropdown"
  class="flex items-center px-4 py-2 text-sm text-gray-700 hover:bg-gray-100"
>
  <span class="mr-2">&#9881;</span>
  Settings
</router-link>
```

---

## 移动端响应式行为

### 侧边栏

- **桌面端（md+）**：始终可见，可折叠为仅图标视图。
- **移动端**：默认隐藏，通过页头菜单切换显示。

### 页头

- **桌面端**：显示完整用户信息和余额。
- **移动端**：显示带汉堡菜单的紧凑视图。

要改善移动端体验，可添加遮罩和过渡：

```vue
<!-- AppSidebar.vue enhancement for mobile -->
<aside
  class="fixed left-0 top-0 z-40 h-screen transition-transform duration-300"
  :class="[
    sidebarCollapsed ? 'w-16' : 'w-64',
    // Hide on mobile when collapsed
    'md:translate-x-0',
    sidebarCollapsed ? '-translate-x-full md:translate-x-0' : 'translate-x-0'
  ]"
>
  <!-- ... -->
</aside>

<!-- Add overlay for mobile -->
<div
  v-if="!sidebarCollapsed"
  @click="toggleSidebar"
  class="fixed inset-0 z-30 bg-black bg-opacity-50 md:hidden"
></div>
```

---

## 状态管理集成

### Auth Store 用法

```typescript
import { useAuthStore } from '@/stores'

const authStore = useAuthStore()

// Check if user is authenticated
if (authStore.isAuthenticated) {
  // User is logged in
}

// Check if user is admin
if (authStore.isAdmin) {
  // User has admin role
}

// Get current user
const user = authStore.user
```

### App Store 用法

```typescript
import { useAppStore } from '@/stores'

const appStore = useAppStore()

// Toggle sidebar
appStore.toggleSidebar()

// Show notifications
appStore.showSuccess('Operation completed!')
appStore.showError('Something went wrong')
appStore.showInfo('Did you know...')
appStore.showWarning('Be careful!')

// Loading state
appStore.setLoading(true)
// ... perform operation
appStore.setLoading(false)

// Or use helper
await appStore.withLoading(async () => {
  // Your async operation
})
```

---

## 无障碍特性

所有布局组件均包含：

- **语义 HTML**：正确使用 `<nav>`、`<header>`、`<main>`、`<aside>`。
- **ARIA 标签**：按钮具有描述性标签。
- **键盘导航**：所有交互元素均可通过键盘访问。
- **焦点管理**：通过 Tailwind 的 `focus:` utility 提供正确焦点状态。
- **颜色对比度**：符合 WCAG AA 的颜色组合。

要进一步增强：

```vue
<!-- Add skip to main content link -->
<a
  href="#main-content"
  class="sr-only rounded bg-white px-4 py-2 focus:not-sr-only focus:absolute focus:left-4 focus:top-4"
>
  Skip to main content
</a>

<main id="main-content">
  <!-- Content -->
</main>
```

---

## 测试

### 布局组件单元测试

```typescript
// AppHeader.test.ts
import { describe, it, expect, beforeEach } from 'vitest'
import { mount } from '@vue/test-utils'
import { createPinia, setActivePinia } from 'pinia'
import AppHeader from '@/components/layout/AppHeader.vue'

describe('AppHeader', () => {
  beforeEach(() => {
    setActivePinia(createPinia())
  })

  it('renders user info when authenticated', () => {
    const wrapper = mount(AppHeader)
    // Add assertions
  })

  it('shows dropdown when clicked', async () => {
    const wrapper = mount(AppHeader)
    await wrapper.find('button').trigger('click')
    expect(wrapper.find('.dropdown').exists()).toBe(true)
  })
})
```

---

## 性能优化

### 懒加载

上方 Router 示例中，使用布局的 view 已经进行了懒加载。

### 代码拆分

导入布局组件时会自动进行代码拆分：

```typescript
// This creates a separate chunk for layout components
import { AppLayout } from '@/components/layout'
```

### 减少重复渲染

布局组件使用 `computed` ref 来避免不必要的重复渲染：

```typescript
const sidebarCollapsed = computed(() => appStore.sidebarCollapsed)
// This only re-renders when sidebarCollapsed changes
```

---

## 故障排除

### 侧边栏未显示

- 检查 `useAppStore` 是否正确初始化。
- 确认 Tailwind class 已被处理。
- 检查与其他组件的 z-index 冲突。

### 路由未在侧边栏高亮

- 确保路由路径完全匹配。
- 检查 `isActive()` 函数逻辑。
- 确认 `useRoute()` 正常工作。

### 未显示用户信息

- 确保 auth store 已通过 `checkAuth()` 初始化。
- 确认用户已登录。
- 检查 localStorage 中的认证数据。

### 移动菜单不工作

- 确认正确调用 `toggleSidebar()`。
- 检查响应式 breakpoint（`md:`）。
- 在实际移动设备或浏览器开发者工具中测试。
