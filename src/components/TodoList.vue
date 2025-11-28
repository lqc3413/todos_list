<script setup>
import { ref, computed, onMounted, onUnmounted } from 'vue'

const props = defineProps({
  todos: {
    type: Array,
    default: () => []
  },
  delTodo: {
    type: Function,
  },
  topTodo: {
    type: Function,
    required: true
  }
})

// 用于存储每个todo项的倒计时信息
const countdowns = ref({})
let timer = null

// 计算剩余时间
const calculateTimeRemaining = (dueDate) => {
  if (!dueDate) return null
  
  const now = new Date()
  const due = new Date(dueDate)
  const diff = due - now
  
  // 检查是否已过期
  if (diff <= 0) {
    return { expired: true, text: '已过期', diff: diff }
  }
  
  // 计算剩余天数、小时和分钟
  const days = Math.floor(diff / (1000 * 60 * 60 * 24))
  const hours = Math.floor((diff % (1000 * 60 * 60 * 24)) / (1000 * 60 * 60))
  const minutes = Math.floor((diff % (1000 * 60 * 60)) / (1000 * 60))
  
  // 根据剩余时间长度格式化输出
  let text = ''
  if (days > 0) {
    text = `还剩${days}天${hours}小时${minutes}分钟`
  } else if (hours > 0) {
    text = `还剩${hours}小时${minutes}分钟`
  } else {
    text = `还剩${minutes}分钟`
  }
  
  const hoursRemaining = hours + (days * 24)
  let urgency = ''
  let urgencyClass = ''
  
  if (hoursRemaining < 6) {
    urgency = '紧急'
    urgencyClass = 'urgency-urgent'
  } else if (hoursRemaining < 12) {
    urgency = '急'
    urgencyClass = 'urgency-soon'
  } else {
    urgency = '不急'
    urgencyClass = 'urgency-normal'
  }
  
  return { expired: false, text, days, hours, minutes, diff: diff, urgency, urgencyClass }
}

// 排序后的待办事项列表
const sortedTodos = computed(() => {
  return [...props.todos].sort((a, b) => {
    // 首先检查置顶状态，置顶的项目总是排在前面
    if (a.pinned && !b.pinned) return -1
    if (!a.pinned && b.pinned) return 1
    
    // 如果两个项目都被置顶，按照置顶时间排序（最近置顶的排前面）
    if (a.pinned && b.pinned) {
      const aPinTime = a.pinTime || 0;
      const bPinTime = b.pinTime || 0;
      return bPinTime - aPinTime; // 时间戳大的排前面
    }
    
    // 置顶状态相同的情况下，使用原有的排序逻辑
    if (!a.dueDate && !b.dueDate) {
      return a.id - b.id // 都没有截止时间，按ID排序（相当于创建时间）
    }
    if (!a.dueDate) return 1 // a没有截止时间，排后面
    if (!b.dueDate) return -1 // b没有截止时间，排后面
    
    // 计算两个任务的剩余时间差
    const aCountdown = calculateTimeRemaining(a.dueDate)
    const bCountdown = calculateTimeRemaining(b.dueDate)
    
    // 优先按剩余时间排序（剩余时间短的排前面）
    if (aCountdown && bCountdown) {
      // 过期的任务排在前面
      if (aCountdown.expired && !bCountdown.expired) return -1
      if (!aCountdown.expired && bCountdown.expired) return 1
      // 都过期或都未过期，按剩余时间差排序
      return aCountdown.diff - bCountdown.diff
    }
    
    // 默认按ID排序
    return a.id - b.id
  })
})

// 更新所有待办项的倒计时
const updateAllCountdowns = () => {
  // 使用原始的todos数组，因为sortedTodos是计算属性
  props.todos.forEach(item => {
    if (item.dueDate) {
      countdowns.value[item.id] = calculateTimeRemaining(item.dueDate)
    }
  })
}

// 组件挂载时启动定时器
onMounted(() => {
  updateAllCountdowns() // 初始更新一次
  // 每分钟更新一次倒计时
  timer = setInterval(updateAllCountdowns, 1000)
})

// 组件卸载时清除定时器
onUnmounted(() => {
  if (timer) {
    clearInterval(timer)
  }
})
</script>




<template>
    <div class="tdContainer">
      <ul class="tdList">
          <li class="tdItem" v-for="item in sortedTodos" :key="item.id">
            <div class="tdItem-main">
                <input type="checkbox" v-model="item.completed" 
                class="toToggle"/>
                <span class="tdTxt" :class="{completed:item.completed}">
                {{ item.txt }}

                </span>
                <!-- 显示倒计时 -->
                <span 
                  v-if="item.dueDate && countdowns[item.id]" 
                  class="countdown"
                  :class="{ 'expired': countdowns[item.id].expired, 'completed-item': item.completed }"
                >
                  {{ countdowns[item.id].text }}
                </span>
                <!-- 显示紧急程度 -->
                <span 
                  v-if="item.dueDate && countdowns[item.id] && !countdowns[item.id].expired" 
                  class="urgency-badge"
                  :class="[countdowns[item.id].urgencyClass, { 'completed-item': item.completed }]"
                >
                  {{ countdowns[item.id].urgency }}
                </span>
              </div>
              <button class="topBtn" @click="props.topTodo(item.id)">置顶</button>
          <div class="tdItem-acts">
            <a @click="props.delTodo(item.id)">删除</a>
          </div>
        </li>
      </ul>
    </div>
</template>

<style scoped>
.topBtn {
    margin-left: 10px;
    padding: 4px 8px;
    border: none;
    border-radius: 4px;
    background-color: #5567af;
    color: #fff;
    font-size: 12px;
    cursor: pointer;
    margin-right: 10px;
}

.tdList {
    list-style: none;
    padding: 0;
    text-align: left;
    background-color: #fff;
    border-radius: 10px;
}

.tdItem {
    padding: 10px 20px 10px 10px;
    border-bottom: 1px solid #ddd;
    cursor: pointer;
    display: flex;
    justify-content: space-between;
}

.tdItem:last-child {
    border-bottom: 0;
}

.tdToggle {
    cursor: pointer;
}

.tdItem-main {
    display: flex;
    align-items: center;
    flex: 1;
}

.tdTxt {
    padding-left: 10px;
    flex: 1;
}

.completed {
    text-decoration: line-through;
    color: #999;
}

/* 已完成项的所有文本提示内容添加删除线 */
.completed-item {
    text-decoration: line-through;
    color: #999;
}

/* 确保删除按钮不受影响 */
.tdItem-acts {
    color: red;
    text-decoration: none !important;
}

/* 倒计时样式 */
.countdown {
    font-size: 12px;
    padding: 4px 8px;
    border-radius: 12px;
    background-color: #f0f0f0;
    color: #666;
    margin-left: 10px;
}

/* 过期状态样式 */
.countdown.expired {
    background-color: #ff4757;
    color: white;
    font-weight: bold;
    animation: pulse 1s infinite alternate; /* 添加脉冲动画强调 */
}

/* 紧急程度样式 */
.urgency-badge {
    padding: 3px 8px;
    border-radius: 12px;
    font-size: 12px;
    font-weight: 500;
    margin-left: 8px;
    border: 1px solid transparent;
}

/* 紧急状态 - 6小时以内 */
.urgency-urgent {
    background-color: #ffebee;
    color: #c62828;
    border-color: #ffcdd2;
}

/* 急状态 - 6-12小时 */
.urgency-soon {
    background-color: #fff3e0;
    color: #ef6c00;
    border-color: #ffcc80;
}

/* 不急状态 - 12小时以上 */
.urgency-normal {
    background-color: #e8f5e9;
    color: #2e7d32;
    border-color: #c8e6c9;
}

@keyframes pulse {
    from { opacity: 0.8; }
    to { opacity: 1; }
}

.tdItem-acts {
    /* display: none; */
    color: red;
}

.tdItem:hover .tdItem-acts {
    display: block;
}
</style>

