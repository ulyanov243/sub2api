# 布局组件

用于 Sub2API 前端的 Vue 3 布局组件，使用 Composition API、TypeScript 和 TailwindCSS 构建。

## 组件

### 1. AppLayout.vue

包含侧边栏和页头的主应用布局。

**用法：**

```vue
<template>
  <AppLayout>
    <!-- Your page content here -->
    <h1>Dashboard</h1>
    <p>Welcome to your dashboard!</p>
  </AppLayout>
</template>

<script setup lang="ts">
import { AppLayout } from '@/components/layout'
</script>
```

**特性：**

- 响应式侧边栏（可折叠）。
- 顶部固定页头。
- 带 slot 的主内容区域。
- 根据侧边栏状态自动调整边距。

---

### 2. AppSidebar.vue

包含用户和管理区块的导航侧边栏。

**特性：**

- 顶部 Logo/品牌。
- 用户导航链接：
  - Dashboard
  - API Keys
  - Usage
  - Redeem
  - Profile
- 管理导航链接（仅管理员显示）：
  - Admin Dashboard
  - Users
  - Groups
  - Accounts
  - Proxies
  - Redeem Codes
- 带切换按钮的可折叠侧边栏。
- 活动路由高亮。
- 使用 HTML 实体图标。
- 响应式设计（移动端友好）。

**由 AppLayout 自动使用**，无需单独导入。

---

### 3. AppHeader.vue

带有用户信息和操作的顶部页头。

**特性：**

- 移动端菜单切换按钮。
- 页面标题（来自 route meta 或 slot）。
- 用户余额显示（仅桌面端）。
- 包含以下内容的用户下拉菜单：
  - Profile link
  - Logout button
- 显示姓名首字母的用户头像。
- 下拉菜单的点击外部关闭处理。
- 响应式设计。

**使用自定义标题：**

```vue
<template>
  <AppLayout>
    <template #title> Custom Page Title </template>

    <!-- Your content -->
  </AppLayout>
</template>
```

**由 AppLayout 自动使用**，无需单独导入。

---

### 4. AuthLayout.vue

用于认证页面（登录/注册）的简单居中布局。

**用法：**

```vue
<template>
  <AuthLayout>
    <!-- Login/Register form content -->
    <h2 class="mb-6 text-2xl font-bold">Login</h2>

    <form @submit.prevent="handleLogin">
      <!-- Form fields -->
    </form>

    <!-- Optional footer slot -->
    <template #footer>
      <p>
        Don't have an account?
        <router-link to="/register" class="text-indigo-600 hover:underline"> Sign up </router-link>
      </p>
    </template>
  </AuthLayout>
</template>

<script setup lang="ts">
import { AuthLayout } from '@/components/layout'

function handleLogin() {
  // Login logic
}
</script>
```

**特性：**

- 居中的卡片容器。
- 渐变背景。
- 顶部 Logo/品牌。
- 主内容 slot。
- 可选的页脚链接 slot。
- 完整响应式支持。

---

## 路由配置

要设置页头中的页面标题，请向路由添加 meta：

```typescript
// router/index.ts
const routes = [
  {
    path: '/dashboard',
    component: DashboardView,
    meta: { title: 'Dashboard' }
  },
  {
    path: '/api-keys',
    component: ApiKeysView,
    meta: { title: 'API Keys' }
  }
  // ...
]
```

---

## Store 依赖

这些组件使用以下 Pinia store：

- **useAuthStore**：用于用户认证状态、角色检查和登出。
- **useAppStore**：用于侧边栏状态管理和 Toast 通知。

请确保在应用中正确初始化这些 store。

---

## 样式

所有组件使用 TailwindCSS utility class。请确保 `tailwind.config.js` 包含组件路径：

```js
module.exports = {
  content: ['./index.html', './src/**/*.{vue,js,ts,jsx,tsx}']
  // ...
}
```

---

## 图标

组件为简便起见使用 HTML 实体图标：

- &#128200; 图表（Dashboard）
- &#128273; 密钥（API Keys）
- &#128202; 柱状图（Usage）
- &#127873; 礼物（Redeem）
- &#128100; 用户（Profile）
- &#128268; 管理（Admin）
- &#128101; 用户（Users）
- &#128193; 文件夹（Groups）
- &#127760; 地球（Accounts）
- &#128260; 网络（Proxies）
- &#127991; 票券（Redeem Codes）

如有需要，可将这些替换为您偏好的图标库（如 Heroicons、Font Awesome）。

---

## 移动端响应式

所有组件均完整响应式：

- **AppSidebar**：桌面端固定定位，移动端默认隐藏。
- **AppHeader**：小屏幕显示移动菜单切换，隐藏余额显示。
- **AuthLayout**：会针对移动设备适配 padding 和卡片尺寸。

侧边栏使用 Tailwind 的响应式 breakpoint（`md:`）调整行为。
