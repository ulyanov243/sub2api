# 认证视图视觉指南

本文说明认证视图的视觉设计与布局。

## 布局结构

LoginView 和 RegisterView 均使用 AuthLayout 组件，该组件提供：

```
┌─────────────────────────────────────────────┐
│                                             │
│  ┌─────────────────────────────────────┐   │
│  │                                     │   │
│  │         Sub2API Logo                │   │
│  │  "Subscription to API Conversion"   │   │
│  │                                     │   │
│  └─────────────────────────────────────┘   │
│                                             │
│  ┌─────────────────────────────────────┐   │
│  │                                     │   │
│  │    [Form Content - White Card]     │   │
│  │                                     │   │
│  └─────────────────────────────────────┘   │
│                                             │
│         [Footer Links]                      │
│                                             │
└─────────────────────────────────────────────┘

Background: Gradient (Indigo → White → Purple)
Card: White with rounded corners and shadow
Max Width: 28rem (448px)
Centered: Both horizontally and vertically
```

## LoginView 视觉设计

### 默认状态

```
┌─────────────────────────────────────────────┐
│                                             │
│         🔷 Sub2API                          │
│    Subscription to API Conversion Platform  │
│                                             │
│  ┌─────────────────────────────────────┐   │
│  │                                     │   │
│  │        Welcome Back                 │   │
│  │  Sign in to your account to continue│  │
│  │                                     │   │
│  │  Username                           │   │
│  │  ┌────────────────────────────────┐ │   │
│  │  │ Enter your username          │ │   │
│  │  └────────────────────────────────┘ │   │
│  │                                     │   │
│  │  Password                           │   │
│  │  ┌────────────────────────────────┐ │   │
│  │  │ ••••••••••••••                 │ │   │
│  │  └────────────────────────────────┘ │   │
│  │                                     │   │
│  │  ☐ Remember me                      │   │
│  │                                     │   │
│  │  ┌────────────────────────────────┐ │   │
│  │  │       Sign In                  │ │   │
│  │  └────────────────────────────────┘ │   │
│  │                                     │   │
│  └─────────────────────────────────────┘   │
│                                             │
│    Don't have an account? Sign up          │
│                                             │
└─────────────────────────────────────────────┘
```

### 加载状态

```
┌─────────────────────────────────────────────┐
│  ┌────────────────────────────────┐         │
│  │  Username                      │         │
│  │  ┌──────────────────────────┐  │         │
│  │  │ john_doe                 │  │         │
│  │  └──────────────────────────┘  │         │
│  │                                │         │
│  │  Password                      │         │
│  │  ┌──────────────────────────┐  │         │
│  │  │ ••••••••••••            │  │         │
│  │  └──────────────────────────┘  │         │
│  │                                │         │
│  │  ☑ Remember me                 │         │
│  │                                │         │
│  │  ┌──────────────────────────┐  │         │
│  │  │ ⟳ Signing in...         │  │ ← Spinner
│  │  └──────────────────────────┘  │         │
│  │      (Button disabled)         │         │
│  └────────────────────────────────┘         │
└─────────────────────────────────────────────┘
```

### 错误状态

```
┌─────────────────────────────────────────────┐
│  ┌────────────────────────────────┐         │
│  │  Username                      │         │
│  │  ┌──────────────────────────┐  │         │
│  │  │ jo                       │  │ ← Red border
│  │  └──────────────────────────┘  │         │
│  │  ⚠ Username must be at least 3 │ ← Red text
│  │     characters                 │         │
│  │                                │         │
│  │  Password                      │         │
│  │  ┌──────────────────────────┐  │         │
│  │  │                          │  │ ← Red border
│  │  └──────────────────────────┘  │         │
│  │  ⚠ Password is required        │ ← Red text
│  │                                │         │
│  │  ┌──────────────────────────┐  │         │
│  │  │ ⚠ Invalid username or    │  │ ← Error banner
│  │  │   password. Please try   │  │
│  │  │   again.                 │  │
│  │  └──────────────────────────┘  │         │
│  │                                │         │
│  │  ┌──────────────────────────┐  │         │
│  │  │       Sign In            │  │         │
│  │  └──────────────────────────┘  │         │
│  └────────────────────────────────┘         │
└─────────────────────────────────────────────┘
```

## RegisterView 视觉设计

### 默认状态

```
┌─────────────────────────────────────────────┐
│                                             │
│         🔷 Sub2API                          │
│    Subscription to API Conversion Platform  │
│                                             │
│  ┌─────────────────────────────────────┐   │
│  │                                     │   │
│  │        Create Account               │   │
│  │     Sign up to start using Sub2API  │   │
│  │                                     │   │
│  │  Username                           │   │
│  │  ┌────────────────────────────────┐ │   │
│  │  │ Choose a username            │ │   │
│  │  └────────────────────────────────┘ │   │
│  │                                     │   │
│  │  Email                              │   │
│  │  ┌────────────────────────────────┐ │   │
│  │  │ your.email@example.com       │ │   │
│  │  └────────────────────────────────┘ │   │
│  │                                     │   │
│  │  Password                           │   │
│  │  ┌────────────────────────────────┐ │   │
│  │  │ Create a strong password     │ │   │
│  │  └────────────────────────────────┘ │   │
│  │  At least 8 characters with letters │  │
│  │  and numbers                        │   │
│  │                                     │   │
│  │  Confirm Password                   │   │
│  │  ┌────────────────────────────────┐ │   │
│  │  │ Confirm your password        │ │   │
│  │  └────────────────────────────────┘ │   │
│  │                                     │   │
│  │  ┌────────────────────────────────┐ │   │
│  │  │     Create Account             │ │   │
│  │  └────────────────────────────────┘ │   │
│  │                                     │   │
│  │  By signing up, you agree to our   │   │
│  │  Terms of Service and Privacy Policy│  │
│  │                                     │   │
│  └─────────────────────────────────────┘   │
│                                             │
│   Already have an account? Sign in         │
│                                             │
└─────────────────────────────────────────────┘
```

### 校验错误

```
┌─────────────────────────────────────────────┐
│  ┌────────────────────────────────┐         │
│  │  Username                      │         │
│  │  ┌──────────────────────────┐  │         │
│  │  │ jane@smith               │  │ ← Red border
│  │  └──────────────────────────┘  │         │
│  │  ⚠ Username can only contain   │ ← Red text
│  │     letters, numbers, _, and - │         │
│  │                                │         │
│  │  Email                         │         │
│  │  ┌──────────────────────────┐  │         │
│  │  │ invalid-email            │  │ ← Red border
│  │  └──────────────────────────┘  │         │
│  │  ⚠ Please enter a valid email  │ ← Red text
│  │     address                    │         │
│  │                                │         │
│  │  Password                      │         │
│  │  ┌──────────────────────────┐  │         │
│  │  │ short                    │  │ ← Red border
│  │  └──────────────────────────┘  │         │
│  │  ⚠ Password must be at least 8 │ ← Red text
│  │     characters with letters    │         │
│  │     and numbers                │         │
│  │                                │         │
│  │  Confirm Password              │         │
│  │  ┌──────────────────────────┐  │         │
│  │  │ different                │  │ ← Red border
│  │  └──────────────────────────┘  │         │
│  │  ⚠ Passwords do not match      │ ← Red text
│  │                                │         │
│  └────────────────────────────────┘         │
└─────────────────────────────────────────────┘
```

### 加载状态

```
┌─────────────────────────────────────────────┐
│  ┌────────────────────────────────┐         │
│  │  Username: jane_smith          │         │
│  │  Email: jane@example.com       │         │
│  │  Password: ••••••••••••        │         │
│  │  Confirm: ••••••••••••         │         │
│  │                                │         │
│  │  ┌──────────────────────────┐  │         │
│  │  │ ⟳ Creating account...   │  │ ← Spinner
│  │  └──────────────────────────┘  │         │
│  │      (All inputs disabled)     │         │
│  └────────────────────────────────┘         │
└─────────────────────────────────────────────┘
```

## 调色板

### 主色

- **Indigo-600**：`#4F46E5` - 主按钮、链接和品牌色。
- **Indigo-700**：`#4338CA` - 按钮悬停状态。
- **Indigo-500**：`#6366F1` - 焦点环。

### 中性色

- **Gray-900**：`#111827` - 标题。
- **Gray-700**：`#374151` - 标签。
- **Gray-600**：`#4B5563` - 正文。
- **Gray-500**：`#6B7280` - 辅助文字。
- **Gray-300**：`#D1D5DB` - 边框。
- **Gray-100**：`#F3F4F6` - 禁用背景。
- **White**：`#FFFFFF` - 卡片背景。

### 错误颜色

- **Red-600**：`#DC2626` - 错误文字。
- **Red-500**：`#EF4444` - 错误边框、焦点环。
- **Red-50**：`#FEF2F2` - 错误横幅背景。
- **Red-200**：`#FECACA` - 错误横幅边框。

### 成功颜色

- **Green-600**：`#16A34A` - 成功文字。
- **Green-50**：`#F0FDF4` - 成功横幅背景。

### 背景渐变

- **起点**：Indigo-100（`#E0E7FF`）。
- **中间**：White（`#FFFFFF`）。
- **终点**：Purple-100（`#F3E8FF`）。

## 排版

### 字体族

- **默认**：系统字体栈（`ui-sans-serif, system-ui, -apple-system, ...`）。

### 字号

- **标题（h2）**：`1.5rem`（24px）、`font-bold`。
- **正文**：`0.875rem`（14px）、`font-normal`。
- **标签**：`0.875rem`（14px）、`font-medium`。
- **辅助文字**：`0.75rem`（12px）、`font-normal`。
- **错误文字**：`0.875rem`（14px）、`font-normal`。

### 行高

- **标题**：`1.5`。
- **正文**：`1.5`。
- **辅助文字**：`1.25`。

## 间距

### 卡片间距

- **内边距**：四边均为 `2rem`（32px）。
- **区块间距**：`1.5rem`（24px）。
- **字段间距**：`1rem`（16px）。

### 输入框间距

- **内边距**：`0.5rem 1rem`（8px 16px）。
- **标签 margin-bottom**：`0.25rem`（4px）。
- **错误文字 margin-top**：`0.25rem`（4px）。

### 按钮间距

- **内边距**：`0.5rem 1rem`（8px 16px）。
- **margin-top**：`1rem`（16px）。

## 交互状态

### 输入框状态

**默认：**

```css
border: 1px solid #D1D5DB (gray-300)
focus: 2px ring #6366F1 (indigo-500)
```

**错误：**

```css
border: 1px solid #EF4444 (red-500)
focus: 2px ring #EF4444 (red-500)
```

**禁用：**

```css
background: #F3F4F6 (gray-100)
cursor: not-allowed
opacity: 0.6
```

### 按钮状态

**默认：**

```css
background: #4F46E5 (indigo-600)
text: #FFFFFF (white)
shadow: shadow-sm
```

**悬停：**

```css
background: #4338CA (indigo-700)
transition: colors 150ms
```

**焦点：**

```css
outline: none
ring: 2px offset-2 #6366F1 (indigo-500)
```

**禁用：**

```css
opacity: 0.5
cursor: not-allowed
```

**加载：**

```css
opacity: 0.5
cursor: not-allowed
+ spinning icon
```

### 链接状态

**默认：**

```css
color: #4F46E5 (indigo-600)
font-weight: 500 (medium)
```

**悬停：**

```css
color: #6366F1 (indigo-500)
transition: colors 150ms
```

## 响应式设计

### 断点

**移动端（< 640px）：**

```
- Full width container
- Padding: 1rem (16px)
- Smaller text sizes
```

**平板端（640px - 768px）：**

```
- Max width: 28rem (448px)
- Centered layout
- Standard spacing
```

**桌面端（> 768px）：**

```
- Max width: 28rem (448px)
- Centered layout
- Standard spacing
```

### 移动端优化

1. 触摸友好的点击目标（至少 44px）。
2. 移动端正确的键盘处理。
3. 防止聚焦输入框时缩放。
4. 响应式字号。
5. 全宽输入框。
6. 为拇指操作提供足够间距。

## 动画

### 过渡

- 颜色变化：`150ms ease-in-out`。
- 透明度变化：`150ms ease-in-out`。
- Transform：`150ms ease-in-out`。

### 加载 Spinner

```css
@keyframes spin {
  from {
    transform: rotate(0deg);
  }
  to {
    transform: rotate(360deg);
  }
}
animation: spin 1s linear infinite;
```

### Toast 动画

- 进入：从右侧滑入 + 淡入。
- 退出：向右侧滑出 + 淡出。
- 时长：300ms。

## 无障碍特性

### 视觉指示器

- 清晰的焦点状态（2px ring）。
- 错误状态（红色边框 + 红色文字）。
- 加载状态（spinner + 文字）。
- 成功状态（绿色 Toast）。

### 颜色对比度

- 白底文字：> 7:1（AAA）。
- 白底标签：> 4.5:1（AA）。
- 按钮：> 4.5:1（AA）。
- 错误文字：> 4.5:1（AA）。

### 交互元素

- 最小尺寸：44x44px（移动端）。
- 清晰的悬停状态。
- 明显的禁用状态。
- 可通过键盘访问。

### 屏幕阅读器支持

- 所有输入框使用正确 label。
- 需要时使用 ARIA 属性。
- 错误提示。
- 加载状态提示。

## 图标

### 加载 Spinner

```svg
<svg class="animate-spin h-4 w-4" fill="none" viewBox="0 0 24 24">
  <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"/>
  <path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"/>
</svg>
```

### 错误图标

```svg
<svg class="h-5 w-5 text-red-400" fill="currentColor" viewBox="0 0 20 20">
  <path fill-rule="evenodd" d="M10 18a8 8 0 100-16 8 8 0 000 16zM8.707 7.293a1 1 0 00-1.414 1.414L8.586 10l-1.293 1.293a1 1 0 101.414 1.414L10 11.414l1.293 1.293a1 1 0 001.414-1.414L11.414 10l1.293-1.293a1 1 0 00-1.414-1.414L10 8.586 8.707 7.293z" clip-rule="evenodd"/>
</svg>
```

## 浏览器兼容性

### 支持的浏览器

- Chrome/Edge：最近 2 个版本。
- Firefox：最近 2 个版本。
- Safari：最近 2 个版本。
- Mobile Safari：iOS 14+。
- Chrome Mobile：最近 2 个版本。

### 使用的 CSS 特性

- Flexbox（完全支持）。
- CSS Grid（完全支持）。
- CSS Transitions（完全支持）。
- CSS Custom Properties（完全支持）。
- 渐变背景（完全支持）。

### 使用的 JavaScript 特性

- ES2015+ 语法。
- Async/await。
- Optional chaining。
- Nullish coalescing。
- Modules。

## 打印样式

（不适用于认证页面，用户不应打印登录表单。）

## 深色模式注意事项

**后续增强：**

- 用户偏好中的深色模式切换。
- 系统偏好检测。
- 持久化深色模式设置。
- 针对深色背景调整调色板。

```css
/* Example dark mode colors (not implemented yet) */
dark:bg-gray-900
dark:text-white
dark:border-gray-700
```

## 性能指标

### 目标指标

- First Contentful Paint（FCP）：< 1s。
- Largest Contentful Paint（LCP）：< 2.5s。
- Time to Interactive（TTI）：< 3s。
- Cumulative Layout Shift（CLS）：< 0.1。
- First Input Delay（FID）：< 100ms。

### 优化策略

- 懒加载非关键资源。
- 最小化初始 bundle 大小。
- 使用高效动画（transform、opacity）。
- 优化图像（Logo、图标）。
- 预连接 API 域名。
- 缓存静态资源。

## 组件大小

### Bundle 影响

- LoginView.vue：约 4 KB（minified）。
- RegisterView.vue：约 6 KB（minified）。
- AuthLayout.vue：约 1 KB（minified）。
- 合计：约 11 KB（不含依赖）。

### 依赖

- Vue 3：约 40 KB（runtime）。
- Vue Router：约 15 KB。
- Pinia：约 10 KB。
- 框架开销合计：约 65 KB（gzipped）。

## 测试检查清单

### 视觉回归测试

- [ ] 默认状态（登录）。
- [ ] 默认状态（注册）。
- [ ] 加载状态。
- [ ] 错误状态（校验）。
- [ ] 错误状态（API）。
- [ ] 成功状态。
- [ ] 移动端视图。
- [ ] 平板端视图。
- [ ] 桌面端视图。
- [ ] 焦点状态。
- [ ] 悬停状态。

### 跨浏览器测试

- [ ] Chrome（Windows、Mac、Linux）。
- [ ] Firefox（Windows、Mac、Linux）。
- [ ] Safari（Mac、iOS）。
- [ ] Edge（Windows）。
- [ ] Chrome Mobile（Android）。
- [ ] Safari Mobile（iOS）。

### 无障碍测试

- [ ] 键盘导航。
- [ ] 屏幕阅读器（NVDA）。
- [ ] 屏幕阅读器（JAWS）。
- [ ] 屏幕阅读器（VoiceOver）。
- [ ] 颜色对比度。
- [ ] 焦点指示器。
- [ ] 错误提示。

## 设计资产

### Figma/Sketch 文件

（不适用：使用 Tailwind 直接在代码中设计。）

### 设计 Token

- 在 Tailwind config 中定义。
- 与设计系统一致。
- 可在所有组件中复用。

### 图标设计

- 内联 SVG 图标。
- Heroicons（outline 和 solid）。
- 一致的 stroke width。
- 具有正确 ARIA 标签的无障碍支持。

---

**注意：** 本视觉指南仅供参考和文档说明。实际实现位于使用 TailwindCSS class 的 Vue 组件中。
