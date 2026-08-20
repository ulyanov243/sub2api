# Vue Router 配置

## 概览

此目录包含 Sub2API 前端应用的 Vue Router 配置。Router 实现了包含认证守卫、基于角色的访问控制和懒加载的完整导航系统。

## 文件

- **index.ts**：包含路由定义和导航守卫的主 Router 配置。
- **meta.d.ts**：路由 meta 字段的 TypeScript 类型定义。

## 路由结构

### 公开路由（无需认证）

| 路径 | 组件 | 说明 |
| ----------- | ------------ | ---------------------- |
| `/login` | LoginView | 用户登录页面 |
| `/register` | RegisterView | 用户注册页面 |

### 用户路由（需要认证）

| 路径 | 组件 | 说明 |
| ------------ | ------------- | ---------------------------- |
| `/` | - | 重定向至 `/dashboard` |
| `/dashboard` | DashboardView | 带统计的用户仪表盘 |
| `/keys` | KeysView | API Key 管理 |
| `/usage` | UsageView | 用量记录与统计 |
| `/redeem` | RedeemView | 兑换码界面 |
| `/profile` | ProfileView | 用户资料设置 |

### 管理路由（需要管理员角色）

| 路径 | 组件 | 说明 |
| ------------------ | ------------------ | ------------------------------- |
| `/admin` | - | 重定向至 `/admin/dashboard` |
| `/admin/dashboard` | AdminDashboardView | 管理仪表盘 |
| `/admin/users` | AdminUsersView | 用户管理 |
| `/admin/groups` | AdminGroupsView | 分组管理 |
| `/admin/accounts` | AdminAccountsView | 账户管理 |
| `/admin/proxies` | AdminProxiesView | 代理管理 |
| `/admin/redeem` | AdminRedeemView | 兑换码管理 |

### 特殊路由

| 路径 | 组件 | 说明 |
| ----------------- | ------------ | -------------- |
| `/:pathMatch(.*)` | NotFoundView | 404 错误页面 |

## 导航守卫

### 认证守卫（beforeEach）

Router 实现以下完整导航守卫：

1. **设置页面标题**：根据路由 meta 更新文档标题。
2. **检查认证**：
   - 公开路由（`requiresAuth: false`）无需登录即可访问。
   - 受保护路由需要认证。
   - 未认证时重定向至 `/login`。
3. **防止重复登录**：
   - 已认证用户会从登录/注册页面重定向离开。
4. **基于角色的访问控制**：
   - 管理路由（`requiresAdmin: true`）需要管理员角色。
   - 非管理员用户会重定向至 `/dashboard`。
5. **保留目标位置**：
   - 在 query 参数中保存原始 URL，以便登录后重定向。

### 流程图

```
User navigates to route
        ↓
Set page title from meta
        ↓
Is route public? ──Yes──→ Already authenticated? ──Yes──→ Redirect to /dashboard
        ↓ No                                        ↓ No
        ↓                                      Allow access
        ↓
Is user authenticated? ──No──→ Redirect to /login with redirect query
        ↓ Yes
        ↓
Requires admin role? ──Yes──→ Is user admin? ──No──→ Redirect to /dashboard
        ↓ No                                  ↓ Yes
        ↓                                     ↓
Allow access ←────────────────────────────────┘
```

## 路由 Meta 字段

每个路由可定义以下 meta 字段：

```typescript
interface RouteMeta {
  requiresAuth?: boolean // Default: true (requires authentication)
  requiresAdmin?: boolean // Default: false (admin access only)
  title?: string // Page title
  breadcrumbs?: Array<{
    // Breadcrumb navigation
    label: string
    to?: string
  }>
  icon?: string // Icon for navigation menu
  hideInMenu?: boolean // Hide from navigation menu
}
```

## 懒加载

所有路由组件均使用动态导入来拆分代码：

```typescript
component: () => import('@/views/user/DashboardView.vue')
```

优势：

- 减少初始 bundle 大小。
- 加快首次页面加载。
- 按需加载组件。
- 由 Vite 自动进行代码拆分。

## 认证 Store 集成

Router 集成了 Pinia auth store（`@/stores/auth`）：

```typescript
const authStore = useAuthStore()

// Check authentication status
authStore.isAuthenticated

// Check admin role
authStore.isAdmin
```

## 使用示例

### 编程式导航

```typescript
import { useRouter } from 'vue-router'

const router = useRouter()

// Navigate to a route
router.push('/dashboard')

// Navigate with query parameters
router.push({
  path: '/usage',
  query: { filter: 'today' }
})

// Navigate to admin route (will be blocked if not admin)
router.push('/admin/users')
```

### 路由链接

```vue
<template>
  <!-- Simple link -->
  <router-link to="/dashboard">Dashboard</router-link>

  <!-- Named route -->
  <router-link :to="{ name: 'Keys' }">API Keys</router-link>

  <!-- With query parameters -->
  <router-link :to="{ path: '/usage', query: { page: 1 } }"> Usage </router-link>
</template>
```

### 检查当前路由

```typescript
import { useRoute } from 'vue-router'

const route = useRoute()

// Check if on admin page
const isAdminPage = route.path.startsWith('/admin')

// Get route meta
const requiresAdmin = route.meta.requiresAdmin
```

## 滚动行为

Router 实现自动滚动管理：

- **浏览器导航**：恢复保存的滚动位置。
- **新路由**：滚动至页面顶部。
- **Hash 链接**：滚动至锚点（实现后）。

## 错误处理

Router 包含导航失败的错误处理：

```typescript
router.onError((error) => {
  console.error('Router error:', error)
})
```

## 路由测试

测试导航守卫和路由访问：

1. **公开路由访问**：未认证时访问 `/login`。
2. **受保护路由**：未登录时尝试访问 `/dashboard`（应重定向）。
3. **管理员访问**：以普通用户登录并尝试 `/admin/users`（应重定向至仪表盘）。
4. **管理员成功访问**：以管理员登录后访问 `/admin/users`（应成功）。
5. **404 处理**：访问不存在的路由（应显示 404 页面）。

## 开发提示

### 添加新路由

1. 在 `routes` 数组中添加路由定义。
2. 创建对应 view 组件。
3. 设置合适的 meta 字段（`requiresAuth`、`requiresAdmin`）。
4. 使用 `() => import()` 进行懒加载。
5. 更新本 README 的路由文档。

### 调试导航

启用 Vue Router 调试模式：

```typescript
// In browser console
window.__VUE_ROUTER__ = router

// Check current route
router.currentRoute.value
```

### 常见问题

**问题：** 刷新页面时出现 404

- **原因：** 服务器未配置 SPA。
- **解决方法：** 配置服务器为所有路由提供 `index.html`。

**问题：** 导航守卫运行两次

- **原因：** 多次调用 `next()`。
- **解决方法：** 确保每个代码路径仅调用一次 `next()`。

**问题：** 用户数据未加载

- **原因：** Auth store 未初始化。
- **解决方法：** 在 App.vue 或 main.ts 中调用 `authStore.checkAuth()`。

## 安全注意事项

1. **仅客户端**：导航守卫在客户端执行；服务端也必须验证。
2. **Token 验证**：API 应在每个请求中验证 JWT token。
3. **角色检查**：Backend 必须验证管理员角色，不能仅依赖前端。
4. **XSS 防护**：Vue 会自动转义模板内容。
5. **CSRF 防护**：状态变更操作应使用 CSRF token。

## 性能优化

1. **懒加载**：所有路由均使用动态导入。
2. **代码拆分**：Vite 自动拆分路由 chunk。
3. **预取**：可为常用路径添加路由预取。
4. **路由缓存**：Vue Router 缓存组件实例。

## 后续增强

- [ ] 添加面包屑导航系统。
- [ ] 实现超出 admin/user 的基于路由权限。
- [ ] 添加路由过渡动画。
- [ ] 为预期导航实现路由预取。
- [ ] 添加导航分析跟踪。
