# 开发指南

## 开发环境设置

### 1. 环境要求
- Node.js >= 16.0.0
- npm >= 8.0.0
- 现代浏览器（Chrome、Firefox、Safari、Edge）

### 2. 项目初始化
```bash
# 克隆项目
git clone [repository-url]

# 进入项目目录
cd cursor-vue-todolist

# 安装依赖
npm install

# 启动开发服务器
npm run dev
```

## 开发规范

### 1. 代码风格

#### 1.1 Vue 组件规范
```vue
<template>
  <!-- 模板内容 -->
</template>

<script setup lang="ts">
// 导入
import { ref, computed, onMounted } from 'vue'
import type { Todo } from '../types/todo'

// 响应式数据
const data = ref<string>('')

// 计算属性
const computedValue = computed(() => {
  return data.value.toUpperCase()
})

// 方法
const handleClick = () => {
  // 处理逻辑
}

// 生命周期
onMounted(() => {
  // 初始化逻辑
})
</script>

<style lang="scss" scoped>
// 样式内容
</style>
```

#### 1.2 TypeScript 规范
```typescript
// 接口定义
interface Todo {
  id: number
  content: string
  completed: boolean
}

// 类型注解
const addTask = (content: string): void => {
  // 实现逻辑
}

// 泛型使用
const tasks = ref<Todo[]>([])
```

#### 1.3 SCSS 规范
```scss
// 变量定义
$primary-color: #4A90E2;
$border-radius: 8px;

// 混入定义
@mixin button-style($bg-color, $hover-color: null) {
  background-color: $bg-color;
  // 样式内容
}

// 嵌套使用
.container {
  padding: 20px;
  
  .header {
    text-align: center;
  }
  
  @include mobile {
    padding: 15px;
  }
}
```

### 2. 命名规范

#### 2.1 文件命名
- 组件文件：PascalCase（如 `TodoList.vue`）
- 工具文件：camelCase（如 `utils.ts`）
- 样式文件：kebab-case（如 `main-style.scss`）

#### 2.2 变量命名
- 常量：UPPER_SNAKE_CASE
- 变量：camelCase
- 组件：PascalCase

#### 2.3 CSS 类命名
- 使用 BEM 规范
- 类名：kebab-case

## 组件开发

### 1. 组件结构

#### 1.1 单文件组件结构
```vue
<template>
  <!-- 1. 模板部分 -->
</template>

<script setup lang="ts">
// 2. 脚本部分
</script>

<style lang="scss" scoped>
// 3. 样式部分
</style>
```

#### 1.2 组件职责
- **单一职责**：每个组件只负责一个功能
- **高内聚**：组件内部逻辑紧密相关
- **低耦合**：组件间依赖最小化

### 2. 状态管理

#### 2.1 响应式数据
```typescript
// 使用 ref
const count = ref<number>(0)

// 使用 reactive
const state = reactive({
  tasks: [] as Todo[],
  loading: false
})

// 使用 computed
const completedCount = computed(() => {
  return tasks.value.filter(task => task.completed).length
})
```

#### 2.2 数据持久化
```typescript
// 保存到本地存储
const saveToStorage = (): void => {
  localStorage.setItem('todoList', JSON.stringify(tasks.value))
}

// 从本地存储加载
const loadFromStorage = (): void => {
  const stored = localStorage.getItem('todoList')
  if (stored) {
    tasks.value = JSON.parse(stored)
  }
}
```

## 样式开发

### 1. SCSS 最佳实践

#### 1.1 变量管理
```scss
// 颜色变量
$primary-color: #4A90E2;
$success-color: #4CAF50;
$danger-color: #f44336;

// 尺寸变量
$border-radius: 8px;
$spacing-unit: 16px;

// 响应式断点
$mobile-breakpoint: 768px;
```

#### 1.2 混入使用
```scss
// 按钮样式混入
@mixin button-style($bg-color, $hover-color: null) {
  background-color: $bg-color;
  border: none;
  border-radius: $border-radius;
  padding: 8px 16px;
  cursor: pointer;
  
  @if $hover-color {
    &:hover {
      background-color: $hover-color;
    }
  }
}

// 使用混入
.primary-button {
  @include button-style($primary-color, $primary-hover);
}
```

#### 1.3 响应式设计
```scss
// 响应式混入
@mixin mobile {
  @media (max-width: $mobile-breakpoint) {
    @content;
  }
}

// 使用响应式混入
.container {
  padding: 20px;
  
  @include mobile {
    padding: 15px;
  }
}
```

### 2. 移动端适配

#### 2.1 触摸优化
```scss
// 触摸友好的按钮尺寸
.touch-button {
  min-height: 44px;
  min-width: 44px;
  padding: 12px 16px;
}

// 防止iOS缩放
input {
  font-size: 16px;
}
```

#### 2.2 布局适配
```scss
// 移动端垂直布局
.task-item {
  display: flex;
  justify-content: space-between;
  
  @include mobile {
    flex-direction: column;
    gap: 15px;
  }
}
```

## 调试技巧

### 1. Vue DevTools
- 安装 Vue DevTools 浏览器扩展
- 查看组件状态和props
- 监控响应式数据变化

### 2. 控制台调试
```typescript
// 调试响应式数据
console.log('Tasks:', tasks.value)

// 调试计算属性
console.log('Completed count:', completedCount.value)
```

### 3. 类型检查
```bash
# 运行类型检查
npm run type-check

# 开发时自动类型检查
npm run dev
```

## 测试指南

### 1. 单元测试
```typescript
// 测试示例
import { mount } from '@vue/test-utils'
import TodoList from '@/components/TodoList.vue'

describe('TodoList', () => {
  test('renders correctly', () => {
    const wrapper = mount(TodoList)
    expect(wrapper.exists()).toBe(true)
  })
})
```

### 2. 集成测试
- 测试组件间交互
- 测试数据流
- 测试用户操作

## 性能优化

### 1. 组件优化
```typescript
// 使用 shallowRef 优化大对象
const largeData = shallowRef<LargeObject>({})

// 使用 computed 缓存计算结果
const expensiveValue = computed(() => {
  return heavyCalculation(data.value)
})
```

### 2. 样式优化
```scss
// 使用 transform 代替 position 变化
.animated-element {
  transform: translateX(0);
  transition: transform 0.3s ease;
  
  &.moved {
    transform: translateX(100px);
  }
}
```

## 部署指南

### 1. 构建优化
```bash
# 构建生产版本
npm run build

# 预览构建结果
npm run preview
```

### 2. 环境配置
```javascript
// vite.config.js
export default defineConfig({
  build: {
    outDir: 'dist',
    sourcemap: false,
    minify: 'terser'
  }
})
```

## 常见问题

### 1. TypeScript 错误
- 检查类型定义是否正确
- 确保导入路径正确
- 使用类型断言解决类型问题

### 2. SCSS 编译错误
- 检查语法是否正确
- 确保变量和混入已定义
- 检查嵌套层级是否过深

### 3. 移动端问题
- 检查触摸区域是否足够大
- 确保字体大小不小于16px
- 测试不同设备的兼容性

## 最佳实践

### 1. 代码组织
- 按功能模块组织代码
- 保持组件的单一职责
- 使用 TypeScript 类型检查

### 2. 性能考虑
- 合理使用计算属性
- 避免不必要的重新渲染
- 优化图片和资源加载

### 3. 用户体验
- 提供即时反馈
- 处理错误状态
- 优化加载性能

---

遵循这些开发指南，可以确保代码质量和项目的可维护性。
