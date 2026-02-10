<script setup>
import { ref, onMounted, computed, watch, nextTick } from 'vue';

const todos = ref([]);
const name = ref('');
const activeTab = ref('personal');
const theme = ref('light');
const editingName = ref(false);
const nameInput = ref(null);

const input_content = ref('');
const input_category = ref(null);

const showAlert = ref(false);
const alertMessage = ref('');

const todos_asc = computed(() => {
  const sorted = todos.value.sort((a, b) => {
    return a.createdAt - b.createdAt;
  });
  return sorted.filter(todo => todo.category === activeTab.value);
});

watch(name, (newVal) => {
  localStorage.setItem('name', newVal);
});

watch(theme, (newVal) => {
  localStorage.setItem('theme', newVal);
  document.documentElement.setAttribute('data-theme', newVal);
});

watch(
  todos,
  (newVal) => {
    localStorage.setItem('todos', JSON.stringify(newVal));
  },
  {
    deep: true,
  }
);

const addTodo = () => {
  if (input_content.value.trim() === '') {
    alertMessage.value = 'Please enter a task!';
    showAlert.value = true;
    return;
  }
  
  if (input_category.value === null) {
    alertMessage.value = 'Please select a category (Business or Personal)!';
    showAlert.value = true;
    return;
  }

  function getFormattedDateTime() {
    const date = new Date();

    const daysOfWeek = [
      'Sunday',
      'Monday',
      'Tuesday',
      'Wednesday',
      'Thursday',
      'Friday',
      'Saturday',
    ];
    const day = daysOfWeek[date.getDay()];

    const month = String(date.getMonth() + 1).padStart(2, '0'); // Months are zero-indexed
    const dayOfMonth = String(date.getDate()).padStart(2, '0');
    const year = date.getFullYear();

    let hours = date.getHours();
    const minutes = String(date.getMinutes()).padStart(2, '0');
    const seconds = String(date.getSeconds()).padStart(2, '0');

    const ampm = hours >= 12 ? 'PM' : 'AM';
    hours = hours % 12;
    hours = hours ? hours : 12; // The hour '0' should be '12'
    const formattedHours = String(hours).padStart(2, '0');

    return `${day}, ${month}/${dayOfMonth}/${year}, ${formattedHours}:${minutes}:${seconds} ${ampm}`;
  }

  todos.value.push({
    content: input_content.value,
    category: input_category.value,
    done: false,
    editable: false,
    createdAt: getFormattedDateTime(),
  });
  input_content.value = '';
  input_category.value = '';
};

const removeTodo = (todo) => {
  todos.value = todos.value.filter((t) => t !== todo);
};

onMounted(() => {
  name.value = localStorage.getItem('name') || '';
  todos.value = JSON.parse(localStorage.getItem('todos')) || [];
  theme.value = localStorage.getItem('theme') || 'light';
  document.documentElement.setAttribute('data-theme', theme.value);
});

const toggleTheme = () => {
  theme.value = theme.value === 'light' ? 'dark' : 'light';
};

const toggleEditName = async () => {
  editingName.value = true;
  await nextTick();
  nameInput.value?.focus();
};

const finishEditingName = () => {
  editingName.value = false;
};
</script>

<template>
  <main class="app">
    <!-- Custom Alert Modal -->
    <div v-if="showAlert" class="alert-modal">
      <div class="alert-content">
        <p>{{ alertMessage }}</p>
        <button @click="showAlert = false" class="alert-btn">Okay</button>
      </div>
    </div>

    <div class="theme-toggle">
      <button class="toggle-btn" @click="toggleTheme" :title="`Switch to ${theme === 'light' ? 'dark' : 'light'} mode`">
        {{ theme === 'light' ? '🌙' : '☀️' }}
      </button>
    </div>
    <section class="greeting" >
      <div class="greeting-content" >
        <h2 class="title">
          What's up,<span v-if="name && !editingName" class="name-display" @click="toggleEditName">{{ name }}</span>
          <input 
            v-if="!name || editingName" 
            ref="nameInput"
            type="text" 
            id="name" 
            placeholder="Name" 
            v-model="name" 
            class="name-input"
            @blur="finishEditingName"
            @keydown.enter="finishEditingName"
          />
          <img
            v-show="!!name"
            src="./assets/icons8-walter-white-50.png"
            alt="user pic"
            class="greeting-img"
          />
        </h2>
      </div>
    </section>

    <section class="create-todo">
      <h3>CREATE A TASK</h3>

      <form id="new-todo-form" @submit.prevent="addTodo">
        <h4>What's on your to do list?</h4>
        <input
          type="text"
          name="content"
          id="content"
          placeholder="e.g. make a video"
          v-model="input_content"
        />

        <h4>Pick a category</h4>
        <div class="options">
          <label>
            <input
              type="radio"
              name="category"
              id="category1"
              value="business"
              v-model="input_category"
            />
            <span class="bubble business"></span>
            <div>Business</div>
          </label>

          <label>
            <input
              type="radio"
              name="category"
              id="category2"
              value="personal"
              v-model="input_category"
            />
            <span class="bubble personal"></span>
            <div>Personal</div>
          </label>
        </div>

        <input type="submit" value="Add Task!" />
      </form>
    </section>

    <section class="todo-list">
      <div class="tabs">
        <button 
          :class="['tab', { active: activeTab === 'personal' }]" 
          @click="activeTab = 'personal'"
        >
          Personal
        </button>
        <button 
          :class="['tab', { active: activeTab === 'business' }]" 
          @click="activeTab = 'business'"
        >
          Business
        </button>
      </div>
      <h3>TO DO LIST</h3>
      <div class="list" id="todo-list">
        <div
          v-for="todo in todos_asc"
          :class="`todo-item ${todo.done && 'done'}`"
        >
          <label>
            <input type="checkbox" v-model="todo.done" />
            <span
              :class="`bubble ${
                todo.category == 'business' ? 'business' : 'personal'
              }`"
            ></span>
          </label>

          <div class="todo-content">
            <textarea type="text" v-model="todo.content" /> <br>
            <span>{{ todo.createdAt }}</span>
          </div>

          
          <div class="actions">
            <button class="delete" @click="removeTodo(todo)">Delete</button>
          </div>
        </div>
      </div>
    </section>
    <footer>
      <p>© 2024 Moe Ezzeldin. All rights reserved.</p>
      <p>
        This is a personal project created for learning purposes. Use at your
        own risk.
      </p>
      <p>
        Todo App: A simple application to manage your tasks locally. Feedback
        and suggestions are welcome!
      </p>
      <p>
        Contact:
        <a href="mailto:ezzeldin.mo3@gmail.com:">Ezzeldin.mo3@gmail.com</a>
      </p>
    </footer>
  </main>
</template>
