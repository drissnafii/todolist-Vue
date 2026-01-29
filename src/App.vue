<script setup lang="ts">
import { ref } from 'vue'
import { Edit, PlusIcon, Save, Trash2Icon } from 'lucide-vue-next'

interface Todo {
  id: number
  text: string
  completed: boolean
  isEditing: boolean
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
    isEditing: false,
  }

  // push task into the ref using .value
  todos.value.push(newTodo)
  newTodoText.value = ''
}

const deleteTodo = (id: number) => {
  todos.value = todos.value.filter((todo) => todo.id !== id)
}

const toggleEdit = (todo: Todo) => {
  // inverse the value of editing
  todo.isEditing = !todo.isEditing
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
      <button @click="addTodo">
        <PlusIcon :size="16" />
      </button>
    </div>
    <ul>
      <li v-for="todo in todos" :key="todo.id">
        <!-- editing mode  -->
        <template v-if="todo.isEditing">
          <input type="text" v-model="todo.text" @keyup.enter="toggleEdit(todo)" />
          <button @click="toggleEdit(todo)">
            <Save :size="16" />
          </button>
        </template>

        <template v-else>
          <input type="checkbox" v-model="todo.completed" />
          <span :class="{ completed: todo.completed }">
            {{ todo.text }}
          </span>
          <button @click="toggleEdit(todo)" :class="'edit-btn'">
            <Edit :size="16" />
          </button>
          <button @click="deleteTodo(todo.id)" :class="'delete-btn'">
            <Trash2Icon :size="16" />
          </button>
        </template>
      </li>
    </ul>
    <p v-if="todos.length === 0">There is no task for the moment</p>
  </main>
</template>

<style scoped>
.delete-btn {
  color: #ef4444;
  background: none;
  border: none;
  cursor: pointer;
  padding: 4px;
  margin-left: 8px;
}

.delete-btn:hover {
  color: #f11010;
}

.edit-btn {
  color: #1f2a1c9f;
  background: none;
  border: none;
  cursor: pointer;
  padding: 4px;
  margin-left: 8px;
}

.completed {
  text-decoration: line-through;
  opacity: 0.6;
}
</style>
