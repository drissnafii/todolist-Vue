<script setup lang="ts">
import { ref, computed, watch, onMounted } from 'vue'
import { Edit } from 'lucide-vue-next'

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
watch(
  todos,
  (newTodos) => {
    localStorage.setItem('todo_data', JSON.stringify(newTodos))
  },
  { deep: true },
)

onMounted(() => {
  const savedData = localStorage.getItem('todo_data')
  if (savedData) {
    todos.value = JSON.parse(savedData)
  }
})

const deleteTodo = (id: number) => {
  todos.value = todos.value.filter((todo) => todo.id !== id)
}

const toggleEdit = (todo: Todo) => {
  // inverse the value of editing
  todo.isEditing = !todo.isEditing
}

const activeCount = computed(() => {
  return todos.value.filter((todo) => !todo.completed).length
})

const clearCompleted = () => {
  todos.value = todos.value.filter((todo) => !todo.completed)
}
const appName = ref('My Todolist')
</script>

<template>
  <div class="todo-container">
    <header class="header">
      <h2>{{ appName }}</h2>
      <div class="input-wrapper">
        <input type="text" v-model="newTodoText" @keyup.enter="addTodo" placeholder="Title..." />
        <span @click="addTodo" class="addBtn">Add</span>
      </div>
      <div class="header-actions">
        <span class="status-info">Remaining: {{ activeCount }}</span>
        <span v-if="todos.some((t) => t.completed)" class="clear-link" @click="clearCompleted"
          >Clear Completed</span
        >
      </div>
    </header>

    <ul id="myUL">
      <li v-for="todo in todos" :key="todo.id" :class="{ checked: todo.completed }">
        <div class="todo-content" @click="todo.completed = !todo.completed">
          <!-- editing mode  -->
          <template v-if="todo.isEditing">
            <input
              type="text"
              v-model="todo.text"
              @keyup.enter="toggleEdit(todo)"
              @click.stop
              class="edit-input"
            />
          </template>
          <template v-else>
            {{ todo.text }}
          </template>
        </div>

        <div class="actions">
          <button v-if="!todo.isEditing" @click.stop="toggleEdit(todo)" class="action-btn edit">
            <Edit :size="16" />
          </button>
          <button
            v-if="!todo.isEditing"
            @click.stop="deleteTodo(todo.id)"
            class="action-btn delete"
          >
            ×
          </button>
        </div>
      </li>
    </ul>
    <p v-if="todos.length === 0" class="empty-msg">There is no task for the moment</p>
  </div>
</template>

<style scoped>
/* Brutalist Design with Soft Colors */

.todo-container {
  max-width: 600px;
  margin: 40px auto;
  font-family: 'Inter', 'Segoe UI', system-ui, sans-serif;
  border: 5px solid #2d2d2d;
  background: #fafafa;
  border-radius: 16px;
  overflow: hidden;
}

/* Style the header */
.header {
  background-color: #ffb4a2;
  padding: 30px 40px;
  color: #2d2d2d;
  text-align: center;
  border-bottom: 5px solid #2d2d2d;
}

.header h2 {
  margin: 5px 0 20px 0;
  font-size: 24px;
  font-weight: 500;
  letter-spacing: -0.5px;
}

/* Clear prefix/suffix from input */
.input-wrapper {
  display: flex;
  margin-bottom: 10px;
  border: 3px solid #2d2d2d;
  border-radius: 8px;
  overflow: hidden;
}

.header input {
  margin: 0;
  border: none;
  border-right: 3px solid #2d2d2d;
  border-radius: 0;
  width: 75%;
  padding: 14px;
  float: left;
  font-size: 16px;
  font-family: inherit;
  outline: none;
  background: #fff;
  font-weight: 500;
}

/* Style the "Add" button */
.addBtn {
  padding: 14px;
  width: 25%;
  background: #e5c6ff;
  color: #2d2d2d;
  float: left;
  text-align: center;
  font-size: 16px;
  font-weight: 500;
  cursor: pointer;
  transition: 0.2s;
  border-radius: 0;
  display: flex;
  align-items: center;
  justify-content: center;
}

.addBtn:hover {
  background-color: #d4a7ff;
}

.header-actions {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-top: 15px;
  font-size: 13px;
  font-weight: 600;
}

.clear-link {
  cursor: pointer;
  text-decoration: underline;
  text-decoration-thickness: 2px;
}

.clear-link:hover {
  opacity: 0.7;
}

/* Style the list items */
ul {
  margin: 0;
  padding: 0;
  list-style-type: none;
}

ul li {
  cursor: pointer;
  position: relative;
  padding: 16px 60px 16px 40px;
  background: #fff;
  font-size: 18px;
  font-weight: 400;
  transition: 0.2s;
  display: flex;
  align-items: center;
  justify-content: space-between;
  border-bottom: 3px solid #2d2d2d;

  /* make the list items unselectable */
  -webkit-user-select: none;
  -moz-user-select: none;
  -ms-user-select: none;
  user-select: none;
}

/* Set all odd list items to a different color (zebra-stripes) */
ul li:nth-child(odd) {
  background: #fff9e6;
}

/* Darker background-color on hover */
ul li:hover {
  background: #cfe9ff;
}

/* When clicked on, add a background color and strike out text */
ul li.checked {
  background: #b8c9d9;
  color: #2d2d2d;
  text-decoration: line-through;
  text-decoration-thickness: 3px;
}

/* Add a "checked" mark when clicked on */
ul li.checked::before {
  content: '';
  position: absolute;
  border-color: #2d2d2d;
  border-style: solid;
  border-width: 0 3px 3px 0;
  top: 18px;
  left: 16px;
  transform: rotate(45deg);
  height: 15px;
  width: 7px;
}

.todo-content {
  flex: 1;
}

.edit-input {
  width: 100%;
  padding: 8px;
  font-size: 16px;
  font-family: inherit;
  border: 3px solid #2d2d2d;
  font-weight: 500;
  border-radius: 8px;
}

/* Action buttons */
.actions {
  display: flex;
  gap: 5px;
  position: absolute;
  right: 10px;
}

.action-btn {
  border: 2px solid #2d2d2d;
  background: #ffe4b5;
  padding: 8px;
  cursor: pointer;
  border-radius: 8px;
  font-size: 20px;
  color: #2d2d2d;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: 0.2s;
}

.action-btn:hover {
  background-color: #ffd98e;
}

.action-btn.delete:hover {
  background-color: #ffb4b4;
}

.checked .action-btn {
  background: #d4dfeb;
}

.empty-msg {
  text-align: center;
  color: #666;
  padding: 40px;
  background: #f5f5f5;
  margin: 0;
  font-weight: 500;
  border-top: 3px solid #2d2d2d;
}
</style>
