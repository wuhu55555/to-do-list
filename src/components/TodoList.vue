<template>
  <div class="container">
    <div class="header">
      <h1>To Do List</h1>
    </div>
    
    <!-- 添加任务 -->
    <div class="add-task">
      <input 
        v-model="newTask" 
        type="text" 
        placeholder="请输入新任务" 
        class="task-input"
        @keyup.enter="addTask"
      />
      <button @click="addTask" class="add-btn">添加</button>
    </div>
    
    <!-- 任务列表 -->
    <div class="task-list" v-if="tasks.length > 0">
      <div 
        v-for="task in tasks" 
        :key="task.id" 
        class="task-item"
        :class="{ 'completed': task.completed }"
      >
        <div class="task-content">
          <input 
            type="checkbox" 
            :checked="task.completed" 
            @change="toggleTask(task.id)"
            class="task-checkbox"
          />
          <span 
            v-if="editingId !== task.id"
            :class="{ 'task-text-completed': task.completed }"
            @click="startEdit(task)"
          >{{ task.content }}</span>
          <input 
            v-else
            v-model="editContent"
            @keyup.enter="saveEdit"
            class="edit-input"
            ref="editInput"
          />
        </div>
        <div class="task-actions">
          <button 
            @click="startEdit(task)" 
            class="edit-btn" 
            :disabled="task.completed"
            v-if="editingId !== task.id"
            :class="{ 'disabled': task.completed }"
          >编辑</button>
          <button 
            @click="saveEdit" 
            class="save-btn"
            v-if="editingId === task.id"
          >保存</button>
          <button @click="deleteTask(task.id)" class="delete-btn">删除</button>
        </div>
      </div>
    </div>
    
    <!-- 空状态 -->
    <div v-else class="empty-state">
      <p>暂无任务，添加一个新任务吧！</p>
    </div>
    
    <!-- 任务统计 -->
    <div class="task-stats">
      <span>总任务: {{ tasks.length }}</span>
      <span>已完成: {{ completedCount }}</span>
      <span>未完成: {{ pendingCount }}</span>
      <button @click="clearCompleted" class="clear-btn" v-if="completedCount > 0">清除已完成</button>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, onMounted, nextTick } from 'vue'
import type { Todo } from '../types/todo'

// 响应式数据
const tasks = ref<Todo[]>([])
const newTask = ref<string>('')
const editingId = ref<number | null>(null)
const editContent = ref<string>('')
const editInput = ref<HTMLInputElement>()

// 计算属性
const completedCount = computed(() => {
  return tasks.value.filter(task => task.completed).length
})

const pendingCount = computed(() => {
  return tasks.value.filter(task => !task.completed).length
})

// 初始化数据
const initData = (): void => {
  const storedTasks = localStorage.getItem('todoList')
  if (storedTasks) {
    tasks.value = JSON.parse(storedTasks)
  } else {
    // 设置初始模拟数据
    tasks.value = [
      { id: Date.now() + 1, content: '学习Vue 3开发', completed: false },
      { id: Date.now() + 2, content: '完成待办事项应用', completed: true },
      { id: Date.now() + 3, content: '练习TypeScript', completed: false }
    ]
    saveToStorage()
  }
}

// 保存数据到本地存储
const saveToStorage = (): void => {
  localStorage.setItem('todoList', JSON.stringify(tasks.value))
}

// 添加任务
const addTask = (): void => {
  if (!newTask.value.trim()) {
    alert('请输入任务内容')
    return
  }
  
  const task: Todo = {
    id: Date.now(),
    content: newTask.value.trim(),
    completed: false
  }
  
  tasks.value.unshift(task)
  newTask.value = ''
  saveToStorage()
  // 添加新任务时退出编辑状态
  exitEditState()
}

// 切换任务状态
const toggleTask = (id: number): void => {
  const task = tasks.value.find(t => t.id === id)
  if (task) {
    task.completed = !task.completed
    saveToStorage()
    // 如果切换的是正在编辑的任务，退出编辑状态
    if (editingId.value === id) {
      exitEditState()
    }
  }
}

// 退出编辑状态
const exitEditState = (): void => {
  editingId.value = null
  editContent.value = ''
}

// 开始编辑
const startEdit = (task: Todo): void => {
  // 如果任务已完成，不允许编辑
  if (task.completed) {
    return
  }
  
  editingId.value = task.id
  editContent.value = task.content
  
  // 等待DOM更新后聚焦输入框
  nextTick(() => {
    if (editInput.value) {
      editInput.value.focus()
    }
  })
}

// 保存编辑
const saveEdit = (): void => {
  const task = tasks.value.find(t => t.id === editingId.value)
  if (task) {
    const newContent = editContent.value.trim()
    // 如果内容为空，提示用户不能保存空内容
    if (!newContent) {
      alert('任务内容不能为空')
      return
    }
    // 如果内容有变化，保存新内容
    if (newContent !== task.content) {
      task.content = newContent
      saveToStorage()
    }
  }
  
  exitEditState()
}

// 删除任务
const deleteTask = (id: number): void => {
  // 如果删除的是正在编辑的任务，先退出编辑状态
  if (editingId.value === id) {
    exitEditState()
  }
  
  if (confirm('确定要删除这个任务吗？')) {
    tasks.value = tasks.value.filter(task => task.id !== id)
    saveToStorage()
  }
}

// 清除已完成任务
const clearCompleted = (): void => {
  if (confirm('确定要清除所有已完成的任务吗？')) {
    tasks.value = tasks.value.filter(task => !task.completed)
    saveToStorage()
    // 清除完成后退出编辑状态
    exitEditState()
  }
}

// 组件挂载时初始化数据
onMounted(() => {
  initData()
})
</script>

<style lang="scss" scoped>
// SCSS 变量定义
$primary-color: #4A90E2;
$primary-hover: #357ABD;
$success-color: #4CAF50;
$success-hover: #45a049;
$danger-color: #f44336;
$danger-hover: #da190b;
$warning-color: #ff9800;
$warning-hover: #e68900;
$disabled-color: #cccccc;
$disabled-text: #666666;
$text-muted: #999;
$text-secondary: #666;
$border-color: #e0e0e0;
$background-light: #f5f5f5;
$white: #fff;
$border-radius: 8px;
$border-radius-sm: 4px;
$shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
$transition: all 0.3s;

// 响应式断点
$mobile-breakpoint: 768px;
$tablet-breakpoint: 1024px;

// 混入定义
@mixin button-style($bg-color, $hover-color: null) {
  background-color: $bg-color;
  color: $white;
  border: none;
  border-radius: $border-radius-sm;
  font-size: 14px;
  padding: 8px 15px;
  cursor: pointer;
  transition: background-color 0.3s;

  @if $hover-color {
    &:hover {
      background-color: $hover-color;
    }
  }
}

@mixin card-style {
  background: $white;
  border-radius: $border-radius;
  box-shadow: $shadow;
}

// 响应式混入
@mixin mobile {
  @media (max-width: $mobile-breakpoint) {
    @content;
  }
}

@mixin tablet {
  @media (min-width: $mobile-breakpoint + 1) and (max-width: $tablet-breakpoint) {
    @content;
  }
}

.container {
  padding: 20px;
  max-width: 600px;
  margin: 0 auto;
  min-height: 100vh;
  background-color: $background-light;

  @include mobile {
    padding: 15px;
    max-width: 100%;
  }
}

.header {
  text-align: center;
  margin-bottom: 30px;

  h1 {
    font-size: 32px;
    font-weight: bold;
    color: $primary-color;
  }
}

.add-task {
  display: flex;
  margin-bottom: 30px;
  padding: 20px;
  @include card-style;

  @include mobile {
    flex-direction: column;
    gap: 15px;
    padding: 15px;
  }
}

.task-input {
  flex: 1;
  height: 40px;
  font-size: 16px;
  padding: 0 15px;
  border: 1px solid $border-color;
  border-radius: $border-radius-sm;
  margin-right: 15px;
  outline: none;

  &:focus {
    border-color: $primary-color;
  }

  @include mobile {
    margin-right: 0;
    height: 44px;
    font-size: 16px; // 防止iOS缩放
  }
}

.add-btn {
  @include button-style($primary-color, $primary-hover);
  font-size: 16px;
  padding: 0 20px;

  @include mobile {
    height: 44px;
    width: 100%;
    padding: 0;
  }
}

.task-list {
  margin-bottom: 30px;
}

.task-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 20px;
  margin-bottom: 15px;
  @include card-style;
  transition: $transition;

  @include mobile {
    flex-direction: column;
    align-items: stretch;
    padding: 15px;
    gap: 15px;
  }
}

.task-content {
  flex: 1;
  display: flex;
  align-items: center;

  @include mobile {
    width: 100%;
    justify-content: space-between;
  }
}

.task-checkbox {
  margin-right: 15px;
  transform: scale(1.3);
}

.task-text-completed {
  text-decoration: line-through;
  color: $text-muted;
}

.edit-input {
  flex: 1;
  height: 35px;
  font-size: 16px;
  padding: 0 15px;
  border: 1px solid $primary-color;
  border-radius: $border-radius-sm;
  margin: 0 15px;
  outline: none;

  @include mobile {
    height: 40px;
    margin: 0;
    width: 100%;
  }
}

.task-actions {
  display: flex;
  gap: 10px;
  margin-left: 15px;

  @include mobile {
    margin-left: 0;
    justify-content: flex-end;
    gap: 8px;
  }
}

.edit-btn {
  @include button-style($success-color, $success-hover);

  @include mobile {
    min-width: 60px;
    height: 36px;
    font-size: 13px;
    padding: 6px 12px;
  }

  &.disabled,
  &:disabled {
    background-color: $disabled-color;
    color: $disabled-text;
    cursor: not-allowed;

    &:hover {
      background-color: $disabled-color;
    }
  }
}

.save-btn {
  @include button-style($primary-color, $primary-hover);

  @include mobile {
    min-width: 60px;
    height: 36px;
    font-size: 13px;
    padding: 6px 12px;
  }
}

.delete-btn {
  @include button-style($danger-color, $danger-hover);

  @include mobile {
    min-width: 60px;
    height: 36px;
    font-size: 13px;
    padding: 6px 12px;
  }
}

.empty-state {
  text-align: center;
  padding: 50px 0;
  color: $text-muted;
  font-size: 18px;
}

.task-stats {
  display: flex;
  justify-content: space-between;
  align-items: center;
  flex-wrap: wrap;
  gap: 15px;
  padding: 20px;
  font-size: 14px;
  color: $text-secondary;
  @include card-style;

  @include mobile {
    flex-direction: column;
    align-items: flex-start;
    gap: 10px;
    padding: 15px;
    font-size: 13px;
  }
}

.clear-btn {
  @include button-style($warning-color, $warning-hover);
  padding: 8px 20px;

  @include mobile {
    width: 100%;
    height: 40px;
    padding: 0;
    font-size: 14px;
  }
}
</style>