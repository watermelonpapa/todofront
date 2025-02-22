<script setup>
import { ref, watch, computed } from 'vue'
import { ElMessage } from 'element-plus'

// 重新设计的数据结构
const list = ref([
  {
    id: 1,
    content: '吃饭',
    completed: false,
    completionNote: '',
    showNoteInput: false,
    timestamp: Date.now()
  },
  {
    id: 2,
    content: 'sleep',
    completed: false,
    completionNote: '',
    showNoteInput: false,
    timestamp: Date.now()
  },
  {
    id: 3,
    content: 'beat',
    completed: false,
    completionNote: '',
    showNoteInput: false,
    timestamp: Date.now()
  }
])

const str = ref({
  text: '',
})

const activeNames = ref([]) // 用于控制展开面板

// 计算属性：对列表进行排序，未完成的在前
const sortedList = computed(() => {
  return [...list.value].sort((a, b) => {
    if (a.completed === b.completed) return 0;
    return a.completed ? 1 : -1;
  });
});

function add(tempdata) {
  if (str.value.text.trim()) {
    list.value.push({
      id: Date.now(),
      content: str.value.text,
      completed: false,
      completionNote: '',
      showNoteInput: false,
      timestamp: Date.now()
    })
    str.value.text = ''
    ElMessage.success('添加成功')
  }
}

// 拖拽相关函数保持不变
function dragStart(event, index) {
  event.dataTransfer.setData('index', index)
}

function dragOver(event) {
  event.preventDefault()
}

function drop(event, dropIndex) {
  const dragIndex = event.dataTransfer.getData('index')
  const items = [...list.value]
  const temp = items[dragIndex]
  items[dragIndex] = items[dropIndex]
  items[dropIndex] = temp
  list.value = items
}

// 添加删除函数
function deleteItem(index) {
  ElMessage.success('删除成功')
  list.value.splice(index, 1)
}

// 修改 toggleComplete 函数
function toggleComplete(item) {
  if (!item.completed) {
    // 未完成项目点击时，显示输入框
    item.showNoteInput = true
    item.timestamp = Date.now()
  }
  // 已完成项目点击时不做任何处理，保持展开状态即可查看完成说明
}

// 修改 saveNote 函数
function saveNote(item) {
  if (item.completionNote.trim()) {
    item.showNoteInput = false
    item.completed = true  // 在保存说明时才标记为完成
    // 保存说明后收起面板
    activeNames.value = activeNames.value.filter(name => name !== item.id)
    ElMessage.success('保存成功')
  } else {
    ElMessage.warning('请输入完成说明')
  }
}

// 新增编辑说明的函数
function editNote(item) {
  item.showNoteInput = true
}

// 添加取消完成说明的函数
function cancelNote(item) {
  item.showNoteInput = false
  item.completionNote = '' // 清空输入的内容
  // 从展开面板中移除
  activeNames.value = activeNames.value.filter(name => name !== item.id)
}

// 格式化时间
function formatDate(timestamp) {
  const date = new Date(timestamp)
  return `${date.getFullYear()}-${String(date.getMonth() + 1).padStart(2, '0')}-${String(date.getDate()).padStart(2, '0')} ${String(date.getHours()).padStart(2, '0')}:${String(date.getMinutes()).padStart(2, '0')}`
}
</script>

<template>
  <div class="todo-app">
    <el-card class="box-card">
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
