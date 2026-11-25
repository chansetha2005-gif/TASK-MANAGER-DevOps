
<script setup lang="ts">
import { ref, onMounted } from 'vue'

interface Task {
  _id: string
  title: string
  description: string
  completed: boolean
}

const API_URL = '/api/tasks'

const tasks = ref<Task[]>([])
const title = ref('')
const description = ref('')
const loading = ref(false)
const error = ref('')

async function fetchTasks() {
  loading.value = true
  error.value = ''

  try {
    const response = await fetch(API_URL)

    if (!response.ok) {
      throw new Error('Failed to load tasks')
    }

    tasks.value = await response.json()
  } catch (err) {
    error.value = 'Cannot connect to the backend. Please check the server.'
    console.error(err)
  } finally {
    loading.value = false
  }
}

async function addTask() {
  if (!title.value.trim()) {
    error.value = 'Please enter a task title.'
    return
  }

  try {
    const response = await fetch(API_URL, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({
        title: title.value,
        description: description.value,
      }),
    })

    if (!response.ok) {
      throw new Error('Failed to create task')
    }

    title.value = ''
    description.value = ''
    error.value = ''

    await fetchTasks()
  } catch (err) {
    error.value = 'Failed to add task.'
    console.error(err)
  }
}

async function updateTask(task: Task) {
  try {
    const response = await fetch(`${API_URL}/${task._id}`, {
      method: 'PUT',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({
        title: task.title,
        description: task.description,
        completed: !task.completed,
      }),
    })

    if (!response.ok) {
      throw new Error('Failed to update task')
    }

    await fetchTasks()
  } catch (err) {
    error.value = 'Failed to update task.'
    console.error(err)
  }
}

async function deleteTask(id: string) {
  if (!confirm('Are you sure you want to delete this task?')) {
    return
  }

  try {
    const response = await fetch(`${API_URL}/${id}`, {
      method: 'DELETE',
    })

    if (!response.ok) {
      throw new Error('Failed to delete task')
    }

    await fetchTasks()
  } catch (err) {
    error.value = 'Failed to delete task.'
    console.error(err)
  }
}

onMounted(fetchTasks)
</script>

<template>
  <main class="container">
    <header class="header">
      <h1>Task Manager</h1>
      <p>Manage your daily tasks in one place.</p>
    </header>

    <section class="form-card">
      <h2>Add New Task</h2>

      <form @submit.prevent="addTask">
        <input
          v-model="title"
          type="text"
          placeholder="Enter task title"
        />

        <textarea
          v-model="description"
          placeholder="Enter task description"
        ></textarea>

        <button type="submit">Add Task</button>
      </form>
    </section>

    <section class="task-section">
      <h2>My Tasks ({{ tasks.length }})</h2>

      <p v-if="loading">Loading tasks...</p>
      <p v-if="error" class="error">{{ error }}</p>

      <p v-if="!loading && tasks.length === 0">
        No tasks found. Add your first task!
      </p>

      <div
        v-for="task in tasks"
        :key="task._id"
        class="task-card"
      >
        <div>
          <h3 :class="{ completed: task.completed }">
            {{ task.title }}
          </h3>

          <p>{{ task.description }}</p>

          <span>
            {{ task.completed ? 'Completed' : 'Pending' }}
          </span>
        </div>

        <div class="actions">
          <button @click="updateTask(task)">
            {{ task.completed ? 'Undo' : 'Complete' }}
          </button>

          <button class="delete-btn" @click="deleteTask(task._id)">
            Delete
          </button>
        </div>
      </div>
    </section>
  </main>
</template>

<style scoped>
.container {
  max-width: 850px;
  margin: 40px auto;
  padding: 24px;
  font-family: Arial, sans-serif;
  color: #222;
}

.header {
  margin-bottom: 30px;
}

.header h1 {
  font-size: 32px;
  margin-bottom: 8px;
}

.header p {
  color: #666;
}

.form-card,
.task-card {
  background: white;
  padding: 22px;
  border: 1px solid #ddd;
  border-radius: 10px;
  margin-bottom: 20px;
}

h2 {
  margin-bottom: 16px;
}

form {
  display: flex;
  flex-direction: column;
  gap: 12px;
}

input,
textarea {
  width: 100%;
  padding: 12px;
  border: 1px solid #ccc;
  border-radius: 6px;
  font-size: 15px;
}

textarea {
  min-height: 90px;
  resize: vertical;
}

button {
  padding: 10px 16px;
  border: none;
  border-radius: 6px;
  background: #2563eb;
  color: white;
  cursor: pointer;
}

.task-card {
  display: flex;
  justify-content: space-between;
  align-items: center;
  gap: 15px;
}

.task-card h3 {
  margin-bottom: 6px;
}

.task-card p {
  color: #666;
  margin-bottom: 8px;
}

.completed {
  text-decoration: line-through;
  color: #888;
}

.actions {
  display: flex;
  gap: 8px;
}

.delete-btn {
  background: #dc2626;
}

.error {
  color: #dc2626;
  margin-bottom: 15px;
}

@media (max-width: 600px) {
  .task-card {
    flex-direction: column;
    align-items: flex-start;
  }
}
</style>
