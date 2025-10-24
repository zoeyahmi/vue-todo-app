<script setup lang="ts">
import { ref, computed, watch, onMounted } from 'vue';

interface Todo {
  id: string;
  text: string;
  completed: boolean;
}

const generateId = (): string => {
  const time = Date.now().toString(36);
  const randomPart = Math.random().toString(36).substring(2);
  return time + randomPart;
};

const LOCAL_STORAGE_KEY = 'vue-todos-list';
const DUMMY_USERNAME = 'guest@mail.com';
const DUMMY_PASSWORD = 'password123';

const isLoggedIn = ref(false);
const todos = ref<Todo[]>([]);
const newTodoText = ref('');
const loginUsername = ref('');
const loginPassword = ref('');
const loginError = ref('');

onMounted(() => {
  if (typeof window !== 'undefined') {
    const savedTodos = localStorage.getItem(LOCAL_STORAGE_KEY);
    if (savedTodos) {
      todos.value = JSON.parse(savedTodos) as Todo[];
    } else {
      todos.value = [
        { id: generateId(), text: 'Respond to urgent emails', completed: true },
        { id: generateId(), text: 'Buy groceries', completed: false },
        { id: generateId(), text: 'Go to the gym', completed: false },
      ];
    }
  }
});

watch(todos, (newTodos) => {
  if (typeof window !== 'undefined' && isLoggedIn.value) {
    localStorage.setItem(LOCAL_STORAGE_KEY, JSON.stringify(newTodos));
  }
}, { deep: true }); 

const handleLogin = (e: Event) => {
  e.preventDefault();
  loginError.value = '';

  if (loginUsername.value === DUMMY_USERNAME && loginPassword.value === DUMMY_PASSWORD) {
    isLoggedIn.value = true;
    loginUsername.value = '';
    loginPassword.value = '';
  } else {
    loginError.value = 'Invalid credentials. Hint: guest@mail.com / password123';
  }
};

const handleLogout = () => {
  isLoggedIn.value = false;
  localStorage.removeItem(LOCAL_STORAGE_KEY);
};

const addTodo = (e: Event) => {
  e.preventDefault();
  if (!isLoggedIn.value || newTodoText.value.trim() === '') return;

  const newTodoItem: Todo = {
    id: generateId(),
    text: newTodoText.value.trim(),
    completed: false,
  };

  todos.value.unshift(newTodoItem);
  newTodoText.value = '';
};

const toggleComplete = (id: string) => {
  if (!isLoggedIn.value) return;

  const index = todos.value.findIndex(t => t.id === id);
  if (index !== -1) {
    todos.value[index].completed = !todos.value[index].completed;
  }
};

const deleteTodo = (id: string) => {
  if (!isLoggedIn.value) return;

  todos.value = todos.value.filter((todo) => todo.id !== id);
};
</script>

<template>
  <div class="min-h-screen bg-gray-100 flex items-start justify-center p-4 sm:p-8 font-grotesk">
    <div class="w-full max-w-lg bg-white shadow-xl rounded-xl overflow-hidden border border-gray-200">

      <div class="p-6 bg-black text-white flex justify-between items-center rounded-t-lg">
        <h1 class="text-3xl font-extrabold uppercase tracking-wide">Todo List</h1>
        <div v-if="isLoggedIn" class="flex items-center gap-2 text-sm">
          <span class="hidden sm:inline">Logged in as guest@mail.com</span>
          <button @click="handleLogout"
            class="p-2 bg-white text-black font-semibold rounded-lg hover:bg-gray-300 transition-colors flex items-center gap-1">
            <svg class="w-4 h-4" fill="currentColor" viewBox="0 0 20 20" xmlns="http://www.w3.org/2000/svg">
              <path fill-rule="evenodd"
                d="M3 3a1 1 0 011-1h6a1 1 0 110 2H4v14h14v-6a1 1 0 112 0v6a2 2 0 01-2 2H4a2 2 0 01-2-2V4a2 2 0 012-2z"
                clip-rule="evenodd" />
              <path fill-rule="evenodd"
                d="M17 9V3h-6a1 1 0 010-2h7a1 1 0 011 1v7a1 1 0 11-2 0z" clip-rule="evenodd" />
              <path fill-rule="evenodd"
                d="M16 3a1 1 0 011-1h-6a1 1 0 110-2h7a1 1 0 011 1v7a1 1 0 11-2 0z" clip-rule="evenodd" />
            </svg>
            Logout
          </button>
        </div>
      </div>

      <div v-if="!isLoggedIn" class="p-8">
        <h2 class="text-2xl font-bold mb-6 text-center text-gray-900 flex items-center justify-center gap-2">
          <svg class="w-5 h-5" fill="currentColor" viewBox="0 0 20 20" xmlns="http://www.w3.org/2000/svg">
            <path fill-rule="evenodd"
              d="M10 2a8 8 0 100 16A8 8 0 0010 2zm0 14a6 6 0 110-12 6 6 0 010 12zm0-10a1 1 0 00-1 1v4a1 1 0 102 0V7a1 1 0 00-1-1zM9 13a1 1 0 102 0 1 1 0 00-2 0z"
              clip-rule="evenodd" />
          </svg>
          Login Required
        </h2>
        <form @submit="handleLogin" class="space-y-4">
          <div>
            <input type="email" placeholder="Username: guest@mail.com" v-model="loginUsername"
              class="w-full p-3 border border-gray-400 rounded-lg focus:outline-none focus:ring-2 focus:ring-gray-900"
              aria-label="Username input" />
          </div>
          <div>
            <input type="password" placeholder="Password: password123" v-model="loginPassword"
              class="w-full p-3 border border-gray-400 rounded-lg focus:outline-none focus:ring-2 focus:ring-gray-900"
              aria-label="Password input" />
          </div>
          <p v-if="loginError" class="text-red-500 text-sm">{{ loginError }}</p>
          <button type="submit"
            class="w-full p-3 bg-black text-white font-bold rounded-lg hover:bg-gray-700 transition-colors shadow-md flex items-center justify-center gap-2"
            aria-label="Login button">
            <svg class="w-4 h-4" fill="currentColor" viewBox="0 0 20 20" xmlns="http://www.w3.org/2000/svg">
              <path fill-rule="evenodd"
                d="M5 9V7a5 5 0 0110 0v2a2 2 0 012 2v5a2 2 0 01-2 2H5a2 2 0 01-2-2v-5a2 2 0 012-2zm8-2V7a3 3 0 10-6 0v2h6z"
                clip-rule="evenodd" />
            </svg>
            Sign In
          </button>
        </form>
      </div>

      <template v-else>
        <form @submit="addTodo" class="p-6 border-b border-gray-400">
          <div class="flex space-x-3">
            <input type="text"
              class="flex-1 p-3 text-lg text-gray-900 border border-gray-400 rounded-lg focus:outline-none focus:border-gray-900 focus:ring-2 focus:ring-gray-900 transition-colors placeholder:text-gray-500"
              placeholder="What needs to be done?" v-model="newTodoText" aria-label="New todo text" />
            <button type="submit"
              class="p-3 bg-black text-white font-bold rounded-lg hover:bg-gray-700 transition-colors shadow-md flex items-center justify-center focus:outline-none focus:ring-2 focus:ring-gray-900"
              aria-label="Add todo">
              Add to list
            </button>
          </div>
        </form>

        <div class="divide-y divide-gray-100">
          <div v-if="todos.length === 0" class="p-8 text-center text-gray-500 italic">
            Add a new task to begin!
          </div>

          <div v-for="todo in todos" :key="todo.id"
            class="flex items-center justify-between p-4 border-b border-gray-200 transition duration-150 ease-in-out hover:bg-gray-50">

            <div class="flex items-center flex-1 min-w-0" @click="toggleComplete(todo.id)">
              <button
                class="w-6 h-6 mr-4 focus:outline-none transition-transform duration-150"
                :aria-label="todo.completed ? 'Mark incomplete' : 'Mark complete'"
                :disabled="!isLoggedIn">
                <svg v-if="todo.completed" class="w-6 h-6 text-gray-900 fill-gray-900 stroke-white" viewBox="0 0 20 20"
                  fill="currentColor" xmlns="http://www.w3.org/2000/svg">
                  <path fill-rule="evenodd" d="M10 18a8 8 0 100-16 8 8 0 000 16zm3.707-9.293a1 1 0 00-1.414-1.414L9 10.586 7.707 9.293a1 1 0 00-1.414 1.414l2 2a1 1 0 001.414 0l4-4z"
                    clip-rule="evenodd" />
                </svg>
                <svg v-else class="w-6 h-6 text-gray-400" fill="none" stroke="currentColor" viewBox="0 0 24 24"
                  xmlns="http://www.w3.org/2000/svg">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                    d="M10 14l2-2m0 0l2-2m-2 2l-2-2m2 2l2 2m7-2a9 9 0 11-18 0 9 9 0 0118 0z" />
                </svg>
              </button>
              <span
                :class="['text-lg font-inter truncate', todo.completed ? 'line-through text-gray-500' : 'text-gray-900']">
                {{ todo.text }}
              </span>
            </div>

            <button @click="deleteTodo(todo.id)"
              :class="['p-2 rounded-full transition-colors duration-150 focus:outline-none ml-4', isLoggedIn ? 'text-gray-400 hover:text-red-600 focus:ring-2 focus:ring-red-600 focus:ring-opacity-50' : 'text-gray-200 cursor-not-allowed']"
              aria-label="Delete todo" :disabled="!isLoggedIn">
              <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                  d="M19 7l-.867 12.142A2 2 0 0116.138 21H7.862a2 2 0 01-1.995-1.858L5 7m5 4v6m4-6v6m1-10V4a1 1 0 00-1-1h-4a1 1 0 00-1 1v3M4 7h16" />
              </svg>
            </button>
          </div>
        </div>
      </template>

    </div>
  </div>
</template>
