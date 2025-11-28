<script setup>
import TodoHeader from './components/TodoHeader.vue'
import TodoList from './components/TodoList.vue'
import TodoFooter from './components/TodoFooter.vue'



import { ref,computed,watch } from 'vue'
const todos = ref([])
const tabOrder = ref(0)



const getAddTodo = (todoData) => {
  const todo = typeof todoData === 'string' ? { txt: todoData } : todoData;
  todos.value.push({
    id: todos.value.length + 1,
    txt: todo.txt,
    completed: false,
    dueDate: todo.dueDate 
  })
}

const delTodo = (item) =>{
  todos.value = todos.value.filter(todo => todo.id !==item)
}

const topTodo = (id) => {
  // 找到要置顶的待办事项
  const todoIndex = todos.value.findIndex(todo => todo.id === id)
  if (todoIndex !== -1) {
    // 设置置顶标记和置顶时间戳
    todos.value[todoIndex].pinned = true;
    todos.value[todoIndex].pinTime = new Date().getTime();
  }
}

const changeTabType = (tabType) => {
  tabOrder.value = tabType
  }
const todoList = computed(() => {
  if(tabOrder.value === 0) {
    return todos.value
  } else if(tabOrder.value === 1) {
    return todos.value.filter(todo => todo.completed === false)
  } else if(tabOrder.value === 2) {
    return todos.value.filter(todo => todo.completed === true)
  }
})

todos.value = JSON.parse(localStorage.getItem('vue-todos') || '[]')

watch(todos, (newTodos) => {
  localStorage.setItem('vue-todos', JSON.stringify(newTodos))
}, { deep: true })
</script>

<template>
  <TodoHeader @addTodo="getAddTodo" />
  <TodoList :todos="todoList" :delTodo="delTodo" :topTodo="topTodo" />
  <TodoFooter :count="todoList.length" :tabType="tabOrder" @changeTabType="changeTabType" />
</template>

<style >
html,
body {
  background-color: #f5f5f5;
}

#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}

.container {
  max-width: 980px;

  min-height: 100%;
  margin: 0 auto;
}
</style>
