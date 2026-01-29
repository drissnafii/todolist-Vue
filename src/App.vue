<script setup lang="ts">
import { ref } from 'vue'

interface Todo {
  id: number
  text: string
  completed: boolean
}

const newTodoText = ref('') // for the input
const todos = ref<Todo[]>([]) // for the list of tasks

// function to add a todo
const addTodo = () => {
  if (newTodoText.value.trim() === '') return // Validation simple

  const newTodo: Todo = {
    id: Date.now(),
    text: newTodoText.value,
    completed: false,
  }

  // push task into the ref using .value
  todos.value.push(newTodo)
  newTodoText.value = ''
}
</script>

<template>
  <main>
    <h1>My todolist</h1>
    <div>
      <input
        type="text"
        v-model="newTodoText"
        @keyup.enter="addTodo"
        placeholder="add new task..."
      />
      <button @click="addTodo">Add</button>
    </div>
    <ul>
      <li v-for="todo in todos" :key="todo.id">
        <input type="checkbox" v-model="todo.completed">
        {{ todo.text }}
      </li>
    </ul>
    <p v-if="todos.length === 0">There is no task for the moment</p>
  </main>
</template>

<style scoped></style>
