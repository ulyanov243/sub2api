# 通用组件

此目录包含使用 Composition API、TypeScript 和 TailwindCSS 构建的可复用 Vue 3 组件。

## 组件

### DataTable.vue

支持排序、加载状态与自定义单元格渲染的通用数据表格组件。

**Props：**

- `columns: Column[]` - 包含 key、label、sortable 和 formatter 的列定义数组。
- `data: any[]` - 要显示的数据对象数组。
- `loading?: boolean` - 是否显示加载骨架屏。
- `defaultSortKey?: string` - 默认排序键（仅在不存在持久化排序状态时使用）。
- `defaultSortOrder?: 'asc' | 'desc'` - 默认排序顺序（默认：`asc`）。
- `sortStorageKey?: string` - 将排序状态（键 + 顺序）持久化至 localStorage。
- `rowKey?: string | (row: any) => string | number` - 行键字段或解析器（默认 `row.id`，回退到索引）。

**Slots：**

- `empty` - 自定义空状态内容。
- `cell-{key}` - 特定列的自定义单元格渲染器（接收 `row` 和 `value`）。

**用法：**

```vue
<DataTable
  :columns="[
    { key: 'name', label: 'Name', sortable: true },
    { key: 'email', label: 'Email' },
    { key: 'status', label: 'Status', formatter: (val) => val.toUpperCase() }
  ]"
  :data="users"
  :loading="isLoading"
>
  <template #cell-actions="{ row }">
    <button @click="editUser(row)">Edit</button>
  </template>
</DataTable>
```

---

### Pagination.vue

具有页码、导航和每页数量选择器的分页组件。

**Props：**

- `total: number` - item 总数。
- `page: number` - 当前页（从 1 开始）。
- `pageSize: number` - 每页 item 数。
- `pageSizeOptions?: number[]` - 可选的每页数量（默认：[10, 20, 50, 100]）。

**Events：**

- `update:page` - 页面变更时触发。
- `update:pageSize` - 每页数量变更时触发。

**用法：**

```vue
<Pagination
  :total="totalUsers"
  :page="currentPage"
  :pageSize="pageSize"
  @update:page="currentPage = $event"
  @update:pageSize="pageSize = $event"
/>
```

---

### Modal.vue

可自定义尺寸和关闭行为的 Modal 对话框。

**Props：**

- `show: boolean` - 控制 Modal 可见性。
- `title: string` - Modal 标题。
- `size?: 'sm' | 'md' | 'lg' | 'xl' | 'full'` - Modal 尺寸（默认：`md`）。
- `closeOnEscape?: boolean` - 按 Escape 键时关闭（默认：true）。
- `closeOnClickOutside?: boolean` - 点击背景时关闭（默认：true）。

**Events：**

- `close` - Modal 应关闭时触发。

**Slots：**

- `default` - Modal 主体内容。
- `footer` - Modal 页脚内容。

**用法：**

```vue
<Modal :show="showModal" title="Edit User" size="lg" @close="showModal = false">
  <form @submit.prevent="saveUser">
    <!-- Form content -->
  </form>

  <template #footer>
    <button @click="showModal = false">Cancel</button>
    <button @click="saveUser">Save</button>
  </template>
</Modal>
```

---

### ConfirmDialog.vue

基于 Modal 组件构建的确认对话框。

**Props：**

- `show: boolean` - 控制对话框可见性。
- `title: string` - 对话框标题。
- `message: string` - 确认消息。
- `confirmText?: string` - 确认按钮文字（默认：`Confirm`）。
- `cancelText?: string` - 取消按钮文字（默认：`Cancel`）。
- `danger?: boolean` - 是否使用危险/红色样式（默认：false）。

**Events：**

- `confirm` - 用户确认时触发。
- `cancel` - 用户取消时触发。

**用法：**

```vue
<ConfirmDialog
  :show="showDeleteConfirm"
  title="Delete User"
  message="Are you sure you want to delete this user? This action cannot be undone."
  confirm-text="Delete"
  cancel-text="Cancel"
  danger
  @confirm="deleteUser"
  @cancel="showDeleteConfirm = false"
/>
```

---

### StatCard.vue

用于展示指标的统计卡片组件，可选显示变化指示器。

**Props：**

- `title: string` - 卡片标题。
- `value: number | string` - 要显示的主值。
- `icon?: Component` - 图标组件。
- `change?: number` - 百分比变化值。
- `changeType?: 'up' | 'down' | 'neutral'` - 变化方向（默认：`neutral`）。
- `formatValue?: (value) => string` - 自定义值格式化器。

**用法：**

```vue
<StatCard title="Total Users" :value="1234" :icon="UserIcon" :change="12.5" change-type="up" />
```

---

### Toast.vue

自动显示 app store 中 Toast 的通知组件。

**用法：**

```vue
<!-- Add once in App.vue or layout -->
<Toast />
```

```typescript
// Trigger toasts from anywhere using the app store
import { useAppStore } from '@/stores/app'

const appStore = useAppStore()

appStore.addToast({
  type: 'success',
  title: 'Success!',
  message: 'User created successfully',
  duration: 3000
})

appStore.addToast({
  type: 'error',
  message: 'Failed to delete user'
})
```

---

### LoadingSpinner.vue

简单的动画加载指示器。

**Props：**

- `size?: 'sm' | 'md' | 'lg' | 'xl'` - 指示器尺寸（默认：`md`）。
- `color?: 'primary' | 'secondary' | 'white' | 'gray'` - 指示器颜色（默认：`primary`）。

**用法：**

```vue
<LoadingSpinner size="lg" color="primary" />
```

---

### EmptyState.vue

具有图标、消息和可选操作按钮的空状态占位组件。

**Props：**

- `icon?: Component` - 图标组件。
- `title: string` - 空状态标题。
- `description: string` - 空状态描述。
- `actionText?: string` - 操作按钮文字。
- `actionTo?: string | object` - Router 链接目标。
- `actionIcon?: boolean` - 是否在按钮中显示加号图标（默认：true）。

**Slots：**

- `icon` - 自定义图标内容。
- `action` - 自定义操作按钮/链接。

**用法：**

```vue
<EmptyState
  title="No users found"
  description="Get started by creating your first user account."
  action-text="Add User"
  :action-to="{ name: 'users-create' }"
/>
```

## 导入

可单独导入组件：

```typescript
import { DataTable, Pagination, Modal } from '@/components/common'
```

或导入指定组件：

```typescript
import DataTable from '@/components/common/DataTable.vue'
```

## 特性

所有组件均包含：

- 带有完善类型定义的 **TypeScript 支持**。
- 包含 ARIA 属性和键盘导航的 **无障碍支持**。
- 适配移动端布局的 **响应式设计**。
- 保持设计一致性的 **TailwindCSS 样式**。
- 使用 `<script setup>` 的 **Vue 3 Composition API**。
- 用于自定义的 **Slot 支持**。
