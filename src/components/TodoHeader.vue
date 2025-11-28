<script setup>
import { ref, computed, onMounted } from 'vue'

const WORD_COUNT_LIMIT = 50

const newTodo = ref('')
const countLimit = ref(WORD_COUNT_LIMIT)
const dueDate = ref('') // 截止时间
const minDate = ref('') // 最小可选时间（当前时间）
const emit = defineEmits(['addTodo'])

// 获取当前时间，格式化为datetime-local输入框需要的格式
const getCurrentDateTime = () => {
  const now = new Date()
  // 确保分钟是向下取整的，这样用户可以选择当前分钟
  now.setSeconds(0, 0)
  return now.toISOString().slice(0, 16) // 格式：YYYY-MM-DDTHH:MM
}

// 组件挂载时设置最小时间
onMounted(() => {
  minDate.value = getCurrentDateTime()
})

// 验证时间是否有效
const isDueDateValid = computed(() => {
  if (!dueDate.value) return false
  return dueDate.value >= minDate.value
})

// 验证表单是否有效
const isFormValid = computed(() => {
  return newTodo.value.length > 0 && newTodo.value.length <= WORD_COUNT_LIMIT && isDueDateValid.value
})

const isShowMsg = computed(() => {
  return newTodo.value.length > WORD_COUNT_LIMIT
})

const addNewTodo = () => {
  if (!isFormValid.value) {
    return
  }
  emit('addTodo', {
    txt: newTodo.value,
    dueDate: dueDate.value || null
  })
  console.log('添加待办事项:', {
    txt: newTodo.value,
    dueDate: dueDate.value || null
  });
  
  // 重置表单
  newTodo.value = ""
  dueDate.value = ""
}
</script>

<template>
      <div class="hdContainer">
        <h1 class="hdTitle">待办事项</h1>
         <input placeholder="请输入您的待办事项，按下回车或按钮后即可添加"
          v-model.trim="newTodo"
          @keyup.enter="addNewTodo"
          class="newTodo"
          />
          <p v-show="isShowMsg">
          最多输入{{ countLimit }}个字符!!!
          </p>
          
          <!-- 日期时间选择器 -->
          <div class="datetime-container">
            <label class="datetime-label">最晚完成时间：</label>
            <input 
              type="datetime-local" 
              v-model="dueDate" 
              :min="minDate"
              class="datetime-input"
            />
            
          </div>
          <button 
          class="addBtn" 
          @click="addNewTodo"
          :disabled="!isFormValid"
        >添加待办事项</button>
      </div>

</template>

<style scoped>
.addBtn {
  margin-top: 20px;
  padding: 10px 20px;
  border: none;
  border-radius: 5px;
  background-color: #4e6ef2;
  color: #fff;
  font-size: 16px;
  cursor: pointer;
}

.addBtn:hover {
  background-color: #3a58c4;
}
.hdContainer {
  text-align: center;
  font-size: 16px;
  border: 3px solid #000000;
  border-radius: 10px;
  margin: 0 auto;
  padding: 20px 20px;
  width: 500px;
  height: 350px;
}

.hdTitle {
  color: #4e6ef2;
}

.newTodo {
  width: 100%;
  padding: 20px 20px;
  border: none;
  border-radius: 10px;
  font-size: 16px;
  box-sizing: border-box;
}

.newTodo:focus {
  outline-color: #4e6ef2;
}

.hdMsg {
  color: red;
  margin: 10px 0;
}

/* 日期时间选择器样式 */
.datetime-container {
  margin-top: 20px;
  text-align: left;
}

.datetime-label {
  display: block;
  margin-bottom: 10px;
  font-weight: bold;
}

.datetime-input {
  width: 100%;
  padding: 10px;
  border: 1px solid #ddd;
  border-radius: 5px;
  font-size: 16px;
  box-sizing: border-box;
}

.datetime-input:focus {
  outline-color: #4e6ef2;
  border-color: #4e6ef2;
}

/* 错误提示样式 */
.error-msg {
  color: #ff4757;
  margin: 5px 0 0 0;
  font-size: 14px;
}

/* 禁用按钮样式 */
.addBtn:disabled {
  background-color: #cccccc;
  color: #666666;
  cursor: not-allowed;
}

.addBtn:disabled:hover {
  background-color: #cccccc;
}
</style>
