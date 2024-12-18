<style src="./main.css"></style>
<script setup>
import { ref, onMounted, computed, watch } from 'vue'

const todos = ref([])
const name = ref('')

const input_content = ref('')
const input_category = ref(null)
const filter_category = ref(null) 
const filter_status = ref(null) 
const filter_keyword = ref('') 

const filtered_todos = computed(() => {
  return todos.value
    .filter((todo) => {
      if (filter_category.value && todo.category !== filter_category.value) {
        return false
      }
      if (filter_status.value !== null) {
        if (filter_status.value === 'completed' && !todo.done) {
          return false
        }
        if (filter_status.value === 'not_completed' && todo.done) {
          return false
        }
      }
      if (filter_keyword.value.trim() !== '' && !todo.content.toLowerCase().includes(filter_keyword.value.toLowerCase())) {
        return false
      }
      return true
    })
    .sort((a, b) => a.createdAt - b.createdAt)
})

watch(name, (newVal) => {
  localStorage.setItem('name', newVal)
})

watch(todos, (newVal) => {
  localStorage.setItem('todos', JSON.stringify(newVal))
}, {
  deep: true
})

const addTodo = () => {
  if (input_content.value.trim() === '' || input_category.value === null) {
    return
  }

  todos.value.push({
    content: input_content.value,
    category: input_category.value,
    done: false,
    editable: false,
    createdAt: new Date().getTime()
  })

  input_content.value = ''
  input_category.value = null
}

const removeTodo = (todo) => {
  todos.value = todos.value.filter((t) => t !== todo)
}

const clearFilters = () => {
  filter_category.value = null
  filter_status.value = null
  filter_keyword.value = ''
}

onMounted(() => {
  name.value = localStorage.getItem('name') || ''
  todos.value = JSON.parse(localStorage.getItem('todos')) || []
})
</script>

<template>
  <main class="app">

    <section class="create-todo">
      <h3>CREATE A TODO</h3>

      <form id="new-todo-form" @submit.prevent="addTodo">
        <h4>What's on your todo list?</h4>
        <input 
          type="text" 
          name="content" 
          id="content" 
          placeholder="Make a task"
          v-model="input_content" />
        
        <h4>Pick a category</h4>
        <div class="options">

          <label>
            <input 
              type="radio" 
              name="category" 
              id="category1" 
              value="business"
              v-model="input_category" />
            <span class="bubble business"></span>
            <div>Business</div>
          </label>

          <label>
            <input 
              type="radio" 
              name="category" 
              id="category2" 
              value="personal"
              v-model="input_category" />
            <span class="bubble personal"></span>
            <div>Personal</div>
          </label>

          <label>
            <input 
              type="radio" 
              name="category" 
              id="category3" 
              value="study"
              v-model="input_category" />
            <span class="bubble study"></span>
            <div>Study</div>
          </label>

        </div>

        <input type="submit" value="Add todo" />
      </form>
    </section>

    <section class="todo-list">
      <h3>TODO LIST</h3>

      <section class="todo-controls">
        <button @click="clearFilters">Show All</button>
        <button @click="filter_category = 'business'">Filter by Business</button>
        <button @click="filter_category = 'personal'">Filter by Personal</button>
        <button @click="filter_category = 'study'">Filter by Study</button>
        <button @click="filter_status = 'completed'">Filter by Completed</button>
        <button @click="filter_status = 'not_completed'">Filter by Not Completed</button>

        <input 
          type="text" 
          placeholder="Search todos..." 
          v-model="filter_keyword" />
      </section>

      <div class="list" id="todo-list">
        <div v-for="todo in filtered_todos" :key="todo.createdAt" :class="`todo-item ${todo.done && 'done'}`">
          <label>
            <input type="checkbox" v-model="todo.done" />
            <span :class="`bubble ${
              todo.category == 'business' 
                ? 'business'
                : todo.category === 'personal'
                ? 'personal'
                : 'study'
            }`"></span>
          </label>

          <div class="todo-content">
            <input type="text" v-model="todo.content" />
          </div>

          <div class="actions">
            <button class="delete" @click="removeTodo(todo)">Delete</button>
          </div>
        </div>
      </div>
    </section>
  </main>
</template>
