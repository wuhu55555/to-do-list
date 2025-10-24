<template>
  <view class="container">
    <view class="header">
      <text class="title">To Do List</text>
    </view>
    
    <!-- 添加任务 -->
    <view class="add-task">
      <input v-model="newTask" type="text" placeholder="请输入新任务" class="task-input" />
      <button @click="addTask" class="add-btn">添加</button>
    </view>
    
    <!-- 任务列表 -->
    <view class="task-list" v-if="tasks.length > 0">
      <view 
        v-for="task in tasks" 
        :key="task.id" 
        class="task-item"
        :class="{ 'completed': task.completed }"
      >
        <view class="task-content">
          <checkbox 
            :checked="task.completed" 
            @change="toggleTask(task.id)"
            class="task-checkbox"
          />
          <text 
            v-if="editingId !== task.id"
            :class="{ 'task-text-completed': task.completed }"
            @click="startEdit(task)"
          >{{ task.content }}</text>
          <input 
            v-else
            v-model="editContent"
            @blur="saveEdit"
            @keyup.enter="saveEdit"
            class="edit-input"
            ref="editInput"
            :data-task-id="task.id"
          />
        </view>
        <view class="task-actions">
          <button @click="startEdit(task)" class="edit-btn" v-if="editingId !== task.id">编辑</button>
          <button @click="deleteTask(task.id)" class="delete-btn">删除</button>
        </view>
      </view>
    </view>
    
    <!-- 空状态 -->
    <view v-else class="empty-state">
      <text>暂无任务，添加一个新任务吧！</text>
    </view>
    
    <!-- 任务统计 -->
    <view class="task-stats">
      <text>总任务: {{ tasks.length }}</text>
      <text>已完成: {{ completedCount }}</text>
      <text>未完成: {{ pendingCount }}</text>
      <button @click="clearCompleted" class="clear-btn" v-if="completedCount > 0">清除已完成</button>
    </view>
  </view>
</template>

<script>
import { ref, computed, onMounted, nextTick } from 'vue'
import { showToast } from '@dcloudio/uni-app'

export default {
  name: 'IndexPage',
  setup() {
    // 响应式数据
    const tasks = ref([])
    const newTask = ref('')
    const editingId = ref(null)
    const editContent = ref('')
    const editInput = ref(null)
    
    // 计算属性
    const completedCount = computed(() => {
      return tasks.value.filter(task => task.completed).length
    })
    
    const pendingCount = computed(() => {
      return tasks.value.filter(task => !task.completed).length
    })
    
    // 初始化数据
    const initData = () => {
      // 从本地存储加载数据
      const storedTasks = uni.getStorageSync('todoList')
      if (storedTasks) {
        tasks.value = storedTasks
      } else {
        // 设置初始模拟数据
        tasks.value = [
          { id: Date.now() + 1, content: '学习UniApp开发', completed: false },
          { id: Date.now() + 2, content: '完成待办事项小程序', completed: true },
          { id: Date.now() + 3, content: '练习Vue3 Composition API', completed: false }
        ]
        saveToStorage()
      }
    }
    
    // 保存数据到本地存储
    const saveToStorage = () => {
      uni.setStorageSync('todoList', tasks.value)
    }
    
    // 添加任务
    const addTask = () => {
      if (!newTask.value.trim()) {
        showToast({
          title: '请输入任务内容',
          icon: 'none'
        })
        return
      }
      
      const task = {
        id: Date.now(),
        content: newTask.value.trim(),
        completed: false
      }
      
      tasks.value.unshift(task)
      newTask.value = ''
      saveToStorage()
      
      showToast({
        title: '任务添加成功',
        icon: 'success'
      })
    }
    
    // 切换任务状态
    const toggleTask = (id) => {
      const task = tasks.value.find(t => t.id === id)
      if (task) {
        task.completed = !task.completed
        saveToStorage()
      }
    }
    
    // 开始编辑
    const startEdit = async (task) => {
      editingId.value = task.id
      editContent.value = task.content
      
      // 等待DOM更新后聚焦输入框
      await nextTick()
      const input = document.querySelector(`.edit-input[data-task-id="${task.id}"]`)
      if (input) {
        input.focus()
      }
    }
    
    // 保存编辑
    const saveEdit = () => {
      if (!editContent.value.trim()) {
        showToast({
          title: '任务内容不能为空',
          icon: 'none'
        })
        return
      }
      
      const task = tasks.value.find(t => t.id === editingId.value)
      if (task) {
        task.content = editContent.value.trim()
        saveToStorage()
        
        showToast({
          title: '任务更新成功',
          icon: 'success'
        })
      }
      
      editingId.value = null
      editContent.value = ''
    }
    
    // 删除任务
    const deleteTask = (id) => {
      uni.showModal({
        title: '确认删除',
        content: '确定要删除这个任务吗？',
        success: (res) => {
          if (res.confirm) {
            tasks.value = tasks.value.filter(task => task.id !== id)
            saveToStorage()
            
            showToast({
              title: '任务删除成功',
              icon: 'success'
            })
          }
        }
      })
    }
    
    // 清除已完成任务
    const clearCompleted = () => {
      uni.showModal({
        title: '确认清除',
        content: '确定要清除所有已完成的任务吗？',
        success: (res) => {
          if (res.confirm) {
            tasks.value = tasks.value.filter(task => !task.completed)
            saveToStorage()
            
            showToast({
              title: '已完成任务已清除',
              icon: 'success'
            })
          }
        }
      })
    }
    
    // 生命周期钩子
    onMounted(() => {
      initData()
    })
    
    return {
      tasks,
      newTask,
      editingId,
      editContent,
      editInput,
      completedCount,
      pendingCount,
      addTask,
      toggleTask,
      startEdit,
      saveEdit,
      deleteTask,
      clearCompleted
    }
  }
}
</script>

<style>
.container {
  padding: 20rpx;
  box-sizing: border-box;
  min-height: 100vh;
  background-color: #f5f5f5;
}

.header {
  text-align: center;
  margin-bottom: 30rpx;
}

.title {
  font-size: 48rpx;
  font-weight: bold;
  color: #4A90E2;
}

.add-task {
  display: flex;
  margin-bottom: 30rpx;
  background: #fff;
  border-radius: 8rpx;
  padding: 20rpx;
  box-shadow: 0 2rpx 10rpx rgba(0, 0, 0, 0.1);
}

.task-input {
  flex: 1;
  height: 80rpx;
  font-size: 28rpx;
  padding: 0 20rpx;
  border: 1rpx solid #e0e0e0;
  border-radius: 8rpx;
  margin-right: 20rpx;
}

.add-btn {
  background-color: #4A90E2;
  color: #fff;
  border: none;
  border-radius: 8rpx;
  font-size: 28rpx;
  padding: 0 40rpx;
}

.task-list {
  margin-bottom: 30rpx;
}

.task-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  background: #fff;
  padding: 30rpx;
  margin-bottom: 20rpx;
  border-radius: 8rpx;
  box-shadow: 0 2rpx 10rpx rgba(0, 0, 0, 0.1);
}

.task-content {
  flex: 1;
  display: flex;
  align-items: center;
}

.task-checkbox {
  margin-right: 20rpx;
  transform: scale(1.5);
}

.task-text-completed {
  text-decoration: line-through;
  color: #999;
}

.edit-input {
  flex: 1;
  height: 60rpx;
  font-size: 28rpx;
  padding: 0 20rpx;
  border: 1rpx solid #4A90E2;
  border-radius: 8rpx;
  margin-left: 20rpx;
}

.task-actions {
  display: flex;
  gap: 10rpx;
}

.edit-btn {
  background-color: #4CAF50;
  color: #fff;
  border: none;
  border-radius: 8rpx;
  font-size: 24rpx;
  padding: 0 20rpx;
  height: 60rpx;
  line-height: 60rpx;
}

.delete-btn {
  background-color: #f44336;
  color: #fff;
  border: none;
  border-radius: 8rpx;
  font-size: 24rpx;
  padding: 0 20rpx;
  height: 60rpx;
  line-height: 60rpx;
}

.empty-state {
  text-align: center;
  padding: 100rpx 0;
  color: #999;
  font-size: 28rpx;
}

.task-stats {
  display: flex;
  justify-content: space-between;
  align-items: center;
  flex-wrap: wrap;
  gap: 20rpx;
  background: #fff;
  padding: 20rpx;
  border-radius: 8rpx;
  box-shadow: 0 2rpx 10rpx rgba(0, 0, 0, 0.1);
  font-size: 24rpx;
  color: #666;
}

.clear-btn {
  background-color: #ff9800;
  color: #fff;
  border: none;
  border-radius: 8rpx;
  font-size: 24rpx;
  padding: 0 30rpx;
  height: 60rpx;
  line-height: 60rpx;
}
</style>