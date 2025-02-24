<script setup>
import { ref, watch, computed, onMounted } from 'vue'
import { ElMessage } from 'element-plus'
import axios from 'axios'
import mytest from './components/mytest.vue'
const API_URL = 'http://localhost:3000/api/todos'

// 数据状态
const list = ref([])
const str = ref({
  text: '',
})
const activeNames = ref([])
const loading = ref(false)

// 计算属性：对列表进行排序，未完成的在前
const sortedList = computed(() => {
  return [...list.value].sort((a, b) => {
    if (a.completed === b.completed) return 0;
    return a.completed ? 1 : -1;
  });
});

// 拖拽相关方法
let draggedIndex = null

function dragStart(event, index) {
  draggedIndex = index
  event.dataTransfer.effectAllowed = 'move'
}

function dragOver(event) {
  event.preventDefault()
}

async function drop(event, index) {
  event.preventDefault()
  if (draggedIndex === null || draggedIndex === index) return

  // 获取拖拽的项目和目标位置的项目
  const draggedItem = list.value[draggedIndex]
  const targetItem = list.value[index]

  // 更新数组顺序
  list.value.splice(draggedIndex, 1)
  list.value.splice(index, 0, draggedItem)

  // 重置拖拽索引
  draggedIndex = null

  try {
    // 这里可以添加与后端同步顺序的逻辑，如果需要的话
    ElMessage.success('排序成功')
  } catch (error) {
    ElMessage.error('排序失败')
    console.error('Error reordering todos:', error)
  }
}

// 获取所有待办事项
async function fetchTodos() {
  try {
    loading.value = true
    const response = await axios.get(API_URL)
    list.value = response.data
  } catch (error) {
    ElMessage.error('获取待办事项失败')
    console.error('Error fetching todos:', error)
  } finally {
    loading.value = false
  }
}

// 添加待办事项
async function add() {
  if (str.value.text.trim()) {
    try {
      const newTodo = {
        content: str.value.text,
        completed: false,
        completionNote: '',
        timestamp: new Date().toISOString()
      }
      const response = await axios.post(API_URL, newTodo)
      list.value.push(response.data)
      str.value.text = ''
      ElMessage.success('添加成功')
    } catch (error) {
      ElMessage.error('添加失败')
      console.error('Error adding todo:', error)
    }
  }
}

// 删除待办事项
async function deleteItem(index) {
  const todo = list.value[index]
  try {
    await axios.delete(`${API_URL}/${todo.id}`)
    list.value.splice(index, 1)
    ElMessage.success('删除成功')
  } catch (error) {
    ElMessage.error('删除失败')
    console.error('Error deleting todo:', error)
  }
}

// 更新待办事项
async function updateTodo(todo) {
  try {
    await axios.put(`${API_URL}/${todo.id}`, todo)
    return true
  } catch (error) {
    ElMessage.error('更新失败')
    console.error('Error updating todo:', error)
    return false
  }
}

// 切换完成状态
async function toggleComplete(item) {
  if (!item.completed) {
    item.showNoteInput = true
    item.timestamp = new Date().toISOString()
  }
}

// 保存完成说明
async function saveNote(item) {
  if (item.completionNote.trim()) {
    item.completed = true
    if (await updateTodo(item)) {
      item.showNoteInput = false
      activeNames.value = activeNames.value.filter(name => name !== item.id)
      ElMessage.success('保存成功')
    }
  } else {
    ElMessage.warning('请输入完成说明')
  }
}

// 取消完成说明
function cancelNote(item) {
  item.showNoteInput = false
  item.completionNote = ''
  activeNames.value = activeNames.value.filter(name => name !== item.id)
}

// 编辑说明
function editNote(item) {
  item.showNoteInput = true
}

// 格式化时间
function formatDate(timestamp) {
  const date = new Date(timestamp)
  return `${date.getFullYear()}-${String(date.getMonth() + 1).padStart(2, '0')}-${String(date.getDate()).padStart(2, '0')} ${String(date.getHours()).padStart(2, '0')}:${String(date.getMinutes()).padStart(2, '0')}`
}

// 初始化加载数据
onMounted(() => {
  fetchTodos()
})
</script>

<template>
  <div class="todo-app">
    <el-card class="box-card" v-loading="loading">
      <template #header>
        <div class="card-header">
          <span class="title">Todo App</span>
        </div>
      </template>

      <div class="todo-form">
        <el-input
          v-model="str.text"
          placeholder="添加新的待办事项"
          class="todo-input"
          @keyup.enter="add"
        >
          <template #append>
            <el-button type="primary" @click="add">添加</el-button>
          </template>
        </el-input>
      </div>
      <mytest />
      <el-collapse v-model="activeNames">
        <div
          v-for="(item, index) in sortedList"
          :key="item.id"
          class="item"
          draggable="true"
          @dragstart="dragStart($event, index)"
          @dragover="dragOver"
          @drop="drop($event, index)"
        >
          <el-collapse-item :name="item.id">
            <template #title>
              <div class="item-header" 
                :class="{ 'completed-item': item.completed }"
                @click="toggleComplete(item)"
              >
                <div class="left-section">
                  <span class="item-content">{{ item.content }}</span>
                </div>
                <div class="right-section">
                  <el-tag 
                    :type="item.completed ? 'success' : 'info'"
                    size="small"
                  >
                    {{ item.completed ? '已完成' : '进行中' }}
                  </el-tag>
                  <el-button 
                    type="danger" 
                    link
                    @click.stop="deleteItem(index)"
                  >
                    删除
                  </el-button>
                </div>
              </div>
            </template>

            <!-- 只在需要时显示内容 -->
            <div v-if="item.showNoteInput || item.completionNote" class="note-section">
              <div v-if="item.showNoteInput" class="input-section">
                <el-input
                  v-model="item.completionNote"
                  type="textarea"
                  :rows="3"
                  placeholder="请输入完成说明..."
                />
                <div class="button-group">
                  <el-button type="primary" @click="saveNote(item)">保存说明</el-button>
                  <el-button @click="cancelNote(item)">取消</el-button>
                </div>
              </div>
              <div v-else-if="item.completed" class="completion-detail">
                <div class="note-header">完成说明：</div>
                <div class="note-content">{{ item.completionNote || '暂无说明' }}</div>
                <div class="note-time">完成时间：{{ formatDate(item.timestamp) }}</div>
                <div class="button-group">
                  <el-button 
                    type="primary" 
                    link
                    @click="editNote(item)"
                  >
                    编辑说明
                  </el-button>
                </div>
              </div>
            </div>
          </el-collapse-item>
        </div>
      </el-collapse>
    </el-card>
  </div>
</template>

<style scoped>
.todo-app {
  max-width: 800px;
  margin: 40px auto;
  padding: 0 20px;
}

.card-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.title {
  font-size: 24px;
  font-weight: bold;
}

.todo-form {
  margin-bottom: 20px;
}

.item {
  margin-bottom: 10px;
  border-radius: 4px;
  transition: all 0.3s;
}

.item-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  width: 100%;
  padding: 10px;
  cursor: pointer;
  transition: background-color 0.3s;
}

.item-header:hover {
  background-color: #f5f7fa;
}

.left-section {
  display: flex;
  align-items: center;
  gap: 10px;
}

.completed-item .item-content {
  text-decoration: line-through;
  color: #999;
}

.input-section {
  padding: 10px;
  background-color: #fff;
  border-radius: 4px;
}

.note-section {
  padding: 10px;
  background-color: #f8f9fa;
  border-radius: 4px;
  margin-top: 10px;
}

.button-group {
  margin-top: 10px;
  display: flex;
  justify-content: flex-end;
  gap: 10px;  /* 添加按钮之间的间距 */
}

.completion-detail {
  padding: 10px;
}

.note-header {
  font-weight: bold;
  margin-bottom: 8px;
  color: #666;
}

.note-content {
  white-space: pre-wrap;
  color: #333;
  line-height: 1.5;
  margin-bottom: 8px;
}

.note-time {
  font-size: 12px;
  color: #999;
  text-align: right;
}

:deep(.el-collapse-item__header) {
  background-color: transparent;
}

:deep(.el-collapse-item__content) {
  padding-bottom: 0;
}

.right-section {
  display: flex;
  align-items: center;
  gap: 10px;
}

/* 确保删除按钮的颜色正确 */
:deep(.el-button--danger.el-button--link) {
  color: #f56c6c;
}

:deep(.el-button--danger.el-button--link:hover) {
  color: #ff7875;
}
</style>
