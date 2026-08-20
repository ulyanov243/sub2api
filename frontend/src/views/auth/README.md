# 认证视图

此目录包含 Sub2API 前端应用的 Vue 3 认证视图。

## 组件

### LoginView.vue

供现有用户认证的登录页面。

**特性：**

- 带校验的用户名和密码输入。
- 用于持久会话的“记住我”复选框。
- 带实时错误显示的表单校验。
- 认证过程中的加载状态。
- 登录失败时显示错误消息。
- 登录成功后重定向至仪表盘。
- 为新用户提供注册链接。

**用法：**

```vue
<template>
  <LoginView />
</template>

<script setup lang="ts">
import { LoginView } from '@/views/auth'
</script>
```

**路由：**

- 路径：`/login`
- 名称：`Login`
- Meta：`{ requiresAuth: false }`

**校验规则：**

- 用户名：必填，至少 3 个字符。
- 密码：必填，至少 6 个字符。

**行为：**

- 使用凭据调用 `authStore.login()`。
- 登录成功时显示成功 Toast。
- 失败时显示错误 Toast 和内联错误消息。
- 重定向至 `/dashboard` 或 query 参数中的目标路由。
- 将已认证用户从登录页重定向离开。

### RegisterView.vue

供新用户创建账户的注册页面。

**特性：**

- 用户名、邮箱、密码和确认密码输入。
- 完整表单校验。
- 密码强度要求（8+ 个字符、字母 + 数字）。
- 使用正则表达式的邮箱格式校验。
- 密码匹配校验。
- 注册过程中的加载状态。
- 注册失败时显示错误消息。
- 成功后重定向至仪表盘。
- 为现有用户提供登录页链接。

**用法：**

```vue
<template>
  <RegisterView />
</template>

<script setup lang="ts">
import { RegisterView } from '@/views/auth'
</script>
```

**路由：**

- 路径：`/register`
- 名称：`Register`
- Meta：`{ requiresAuth: false }`

**校验规则：**

- 用户名：
  - 必填。
  - 3-50 个字符。
  - 仅限字母、数字、下划线和连字符。
- 邮箱：
  - 必填。
  - 合法邮箱格式（RFC 5322 正则表达式）。
- 密码：
  - 必填。
  - 至少 8 个字符。
  - 必须至少包含一个字母和一个数字。
- 确认密码：
  - 必填。
  - 必须与密码匹配。

**行为：**

- 使用用户数据调用 `authStore.register()`。
- 注册成功时显示成功 Toast。
- 失败时显示错误 Toast 和内联错误消息。
- 注册成功后重定向至 `/dashboard`。
- 将已认证用户从注册页重定向离开。

## 架构

### 组件结构

两个视图遵循一致的结构：

```
<template>
  <AuthLayout>
    <div class="space-y-6">
      <!-- Title -->
      <!-- Form -->
      <!-- Error Message -->
      <!-- Submit Button -->
    </div>

    <template #footer>
      <!-- Footer Links -->
    </template>
  </AuthLayout>
</template>

<script setup lang="ts">
// Imports
// State
// Validation
// Form Handlers
</script>
```

### 状态管理

两个视图都使用：

- `useAuthStore()` - 用于认证操作（登录、注册）。
- `useAppStore()` - 用于 Toast 通知和 UI 反馈。
- `useRouter()` - 用于导航和重定向。

### 校验策略

**客户端校验：**

- 提交表单时进行实时校验。
- 字段级错误消息。
- 完整校验规则。
- TypeScript 类型安全。

**服务端校验：**

- Backend API 校验所有输入。
- 正确处理错误响应。
- 显示用户友好的错误消息。

### 样式

**设计系统：**

- TailwindCSS utility class。
- 一致的配色方案（indigo 为主色）。
- 响应式设计。
- 无障碍表单控件。
- 带 spinner 动画的加载状态。

**视觉反馈：**

- 无效字段显示红色边框。
- 输入框下方显示错误消息。
- API 错误显示全局错误横幅。
- 完成时显示成功 Toast。
- 提交按钮显示加载 spinner。

## 依赖

### 组件

- `AuthLayout` - 来自 `@/components/layout` 的认证页面布局包装器。

### Store

- `authStore` - 来自 `@/stores/auth` 的认证状态管理。
- `appStore` - 来自 `@/stores/app` 的应用状态和 Toast。

### 库

- Vue 3 Composition API。
- 用于导航的 Vue Router。
- 用于状态管理的 Pinia。
- 用于类型安全的 TypeScript。

## 使用示例

### 基本登录流程

```typescript
// User enters credentials
formData.username = 'john_doe'
formData.password = 'SecurePass123'

// Submit form
await handleLogin()

// On success:
// - authStore.login() called
// - Token and user stored
// - Success toast shown
// - Redirected to /dashboard

// On error:
// - Error message displayed
// - Error toast shown
// - Form remains editable
```

### 基本注册流程

```typescript
// User enters registration data
formData.username = 'jane_smith'
formData.email = 'jane@example.com'
formData.password = 'SecurePass123'
formData.confirmPassword = 'SecurePass123'

// Submit form
await handleRegister()

// On success:
// - authStore.register() called
// - Token and user stored
// - Success toast shown
// - Redirected to /dashboard

// On error:
// - Error message displayed
// - Error toast shown
// - Form remains editable
```

## 错误处理

### 客户端错误

```typescript
// Validation errors
errors.username = 'Username must be at least 3 characters'
errors.email = 'Please enter a valid email address'
errors.password = 'Password must be at least 8 characters with letters and numbers'
errors.confirmPassword = 'Passwords do not match'
```

### 服务端错误

```typescript
// API error responses
{
  response: {
    data: {
      detail: 'Username already exists'
    }
  }
}

// Displayed as:
errorMessage.value = 'Username already exists'
appStore.showError('Username already exists')
```

## 无障碍

- 语义化 HTML 元素（`<label>`、`<input>`、`<button>`）。
- label 上正确的 `for` 属性。
- 用于加载状态的 ARIA 属性。
- 键盘导航支持。
- 焦点管理。
- 错误提示。
- 足够的颜色对比度。

## 测试注意事项

### 单元测试

- 表单校验逻辑。
- 错误处理。
- 状态管理。
- Router 导航。

### 集成测试

- 完整登录流程。
- 完整注册流程。
- 错误场景。
- 重定向行为。

### E2E 测试

- 完整用户旅程。
- 表单交互。
- API 集成。
- 成功/错误状态。

## 后续增强

可能的改进：

- OAuth/SSO 集成（Google、GitHub）。
- 双因素认证（2FA）。
- 密码强度计。
- 邮箱验证流程。
- 忘记密码功能。
- 社交登录按钮。
- CAPTCHA 集成。
- 会话超时警告。
- 密码可见性切换。
- 自动填充支持增强。

## 安全注意事项

- 绝不记录或显示密码。
- 生产环境必须使用 HTTPS。
- JWT token 安全存储在 localStorage 中。
- API 的 CORS 防护。
- 使用 Vue 自动转义防护 XSS。
- 使用基于 token 的认证防护 CSRF。
- Backend API 限流。
- 输入净化。
- 安全的密码要求。

## 性能

- 懒加载路由。
- 最小 bundle 大小。
- 快速首次渲染。
- 通过 reactive ref 优化重复渲染。
- 没有不必要的 watcher。
- 高效的表单校验。

## 浏览器支持

- 现代浏览器（Chrome、Firefox、Safari、Edge）。
- 需要 ES2015+。
- Flexbox 和 CSS Grid。
- Tailwind CSS utility。
- Vue 3 runtime。

## 相关文档

- [Auth Store 文档](/src/stores/README.md#auth-store)
- [AuthLayout 组件](/src/components/layout/README.md#authlayout)
- [Router 配置](/src/router/index.ts)
- [API 文档](/src/api/README.md#authentication)
- [类型定义](/src/types/index.ts)
