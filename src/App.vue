<script setup>
import { ref, onMounted, onUnmounted, computed, watch, nextTick } from 'vue';
import iconRevenue from './assets/total revenue.png';
import iconM365 from './assets/m365.png';
import iconAV from './assets/antivirus.png';
import iconMem from './assets/mem.png';
import iconCC from './assets/CC.png';
import iconBasket from './assets/basket.jpg';
import iconGSP from './assets/GSP.jpg';

const todos = ref([]);
const name = ref('');
const activeTab = ref('personal');
const theme = ref('light');
const editingName = ref(false);
const nameInput = ref(null);

const input_content = ref('');
const input_category = ref(null);
const activeMainTab = ref('ptracker');
const trackerEntries = ref([]);
const trackerForm = ref({
  amount: '',
  m365: 0,
  antivirus: 0,
  membership: 0,
  creditApp: 0,
  basket: 0,
  gsp: 0,
});

const showAlert = ref(false);
const alertMessage = ref('');
const now = ref(new Date());
let dateTick;

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
  
  if (!input_category.value) {
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
  input_category.value = null;
};

const removeTodo = (todo) => {
  todos.value = todos.value.filter((t) => t !== todo);
};

const autoResize = (event) => {
  const el = event?.target;
  if (!el) return;
  el.style.height = 'auto';
  el.style.height = `${el.scrollHeight}px`;
};

const resizeAllTextareas = () => {
  const nodes = document.querySelectorAll('.todo-textarea');
  nodes.forEach((el) => {
    el.style.height = 'auto';
    el.style.height = `${el.scrollHeight}px`;
  });
};

onMounted(() => {
  name.value = localStorage.getItem('name') || '';
  todos.value = JSON.parse(localStorage.getItem('todos')) || [];
  theme.value = localStorage.getItem('theme') || 'light';
  activeMainTab.value = localStorage.getItem('activeMainTab') || 'ptracker';
  document.documentElement.setAttribute('data-theme', theme.value);
  trackerEntries.value = JSON.parse(localStorage.getItem('trackerEntries')) || [];
  dateTick = setInterval(() => {
    now.value = new Date();
  }, 60 * 1000);
  nextTick(() => {
    resizeAllTextareas();
  });
});

onUnmounted(() => {
  if (dateTick) clearInterval(dateTick);
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

watch(
  trackerEntries,
  (val) => localStorage.setItem('trackerEntries', JSON.stringify(val)),
  { deep: true }
);

watch(activeMainTab, (val) => {
  localStorage.setItem('activeMainTab', val);
});

const trackerTotals = computed(() => {
  return trackerEntries.value.reduce(
    (acc, entry) => {
      acc.revenue += Number(entry.amount) || 0;
      acc.m365 += Number(entry.m365) || 0;
      acc.antivirus += Number(entry.antivirus) || 0;
      acc.membership += Number(entry.membership) || 0;
      acc.creditApp += Number(entry.creditApp) || 0;
      acc.basket += Number(entry.basket) || 0;
      acc.gsp += Number(entry.gsp) || 0;
      return acc;
    },
    { revenue: 0, m365: 0, antivirus: 0, membership: 0, creditApp: 0, basket: 0, gsp: 0 }
  );
});

const todayEntries = computed(() => {
  const today = now.value;
  return trackerEntries.value.filter((entry) =>
    isSameDay(new Date(entry.createdAt), today)
  );
});

const computeTotals = (predicate) => {
  return trackerEntries.value.reduce(
    (acc, entry) => {
      if (!predicate(entry)) return acc;
      acc.revenue += Number(entry.amount) || 0;
      acc.m365 += Number(entry.m365) || 0;
      acc.antivirus += Number(entry.antivirus) || 0;
      acc.membership += Number(entry.membership) || 0;
      acc.creditApp += Number(entry.creditApp) || 0;
      acc.basket += Number(entry.basket) || 0;
      acc.gsp += Number(entry.gsp) || 0;
      return acc;
    },
    { revenue: 0, m365: 0, antivirus: 0, membership: 0, creditApp: 0, basket: 0, gsp: 0 }
  );
};

const isSameDay = (d, ref) =>
  d.getFullYear() === ref.getFullYear() &&
  d.getMonth() === ref.getMonth() &&
  d.getDate() === ref.getDate();

const isSameMonth = (d, ref) =>
  d.getFullYear() === ref.getFullYear() &&
  d.getMonth() === ref.getMonth();

const isSameYear = (d, ref) => d.getFullYear() === ref.getFullYear();

const dailyTotals = computed(() => {
  const today = now.value;
  return computeTotals((entry) => isSameDay(new Date(entry.createdAt), today));
});

const monthlyTotals = computed(() => {
  const today = now.value;
  return computeTotals((entry) => isSameMonth(new Date(entry.createdAt), today));
});

const isSameQuarter = (d, ref) =>
  d.getFullYear() === ref.getFullYear() &&
  Math.floor(d.getMonth() / 3) === Math.floor(ref.getMonth() / 3);

const quarterlyTotals = computed(() => {
  const today = now.value;
  return computeTotals((entry) => isSameQuarter(new Date(entry.createdAt), today));
});

const yearlyTotals = computed(() => {
  const today = now.value;
  return computeTotals((entry) => isSameYear(new Date(entry.createdAt), today));
});

const allTimeTotals = computed(() => trackerTotals.value);

const addTrackerEntry = () => {
  if (!trackerForm.value.amount) {
    alertMessage.value = 'Enter receipt amount.';
    showAlert.value = true;
    return;
  }
  trackerEntries.value.push({
    ...trackerForm.value,
    createdAt: new Date().toISOString(),
  });
  trackerForm.value = { amount: '', m365: 0, antivirus: 0, membership: 0, creditApp: 0, basket: 0, gsp: 0 };
};

const removeTrackerEntry = (entry) => {
  trackerEntries.value = trackerEntries.value.filter((e) => e !== entry);
};

const bumpField = (field) => {
  trackerForm.value[field] = Number(trackerForm.value[field] || 0) + 1;
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

    <header class="top-bar">
      <div class="brand">
        <img
          class="brand-mark"
          src="./assets/icons8-walter-white-50.png"
          alt="App logo"
          v-show="!editingName && name"
        />
        <div class="brand-text">
          <p class="brand-kicker">Welcome back</p>
          <h2 class="brand-title">
            Hey,
            <span v-if="name && !editingName" class="name-display" @click="toggleEditName">{{ name }}</span>
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
          </h2>
        </div>
      </div>

      <div class="top-actions">
        <button class="theme-pill" @click="toggleTheme" :title="`Switch to ${theme === 'light' ? 'dark' : 'light'} mode`">
          <span class="theme-switch" aria-hidden="true">
            <span class="theme-icon sun">
              <svg viewBox="0 0 24 24" fill="none" aria-hidden="true">
                <circle cx="12" cy="12" r="4.5" stroke="currentColor" stroke-width="1.6" />
                <path d="M12 2.5v2.2M12 19.3v2.2M4.7 4.7l1.6 1.6M17.7 17.7l1.6 1.6M2.5 12h2.2M19.3 12h2.2M4.7 19.3l1.6-1.6M17.7 6.3l1.6-1.6" stroke="currentColor" stroke-width="1.6" stroke-linecap="round"/>
              </svg>
            </span>
            <span class="theme-icon moon">
              <svg viewBox="0 0 24 24" fill="none" aria-hidden="true">
                <path d="M14.8 3.2a7.8 7.8 0 1 0 6 13.3 8.6 8.6 0 1 1-6-13.3Z" stroke="currentColor" stroke-width="1.6" stroke-linecap="round" stroke-linejoin="round"/>
              </svg>
            </span>
            <span class="theme-thumb"></span>
          </span>
          <span class="sr-only">Switch to {{ theme === 'light' ? 'dark' : 'light' }} mode</span>
        </button>
      </div>
    </header>

    <div class="main-tabs">
      <button
        :class="['main-tab', { active: activeMainTab === 'ptracker' }]"
        @click="activeMainTab = 'ptracker'"
      >
        P-Tracker
      </button>
      <button
        :class="['main-tab', { active: activeMainTab === 'todo' }]"
        @click="activeMainTab = 'todo'"
      >
        To Do
      </button>
    </div>

    <section v-if="activeMainTab === 'todo'" class="create-todo">
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
          <label :class="['option', { selected: input_category === 'business' }]">
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

          <label :class="['option', { selected: input_category === 'personal' }]">
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

    <section v-if="activeMainTab === 'todo'" class="todo-list">
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
          :class="['todo-item', { done: todo.done }]"
        >
          <div class="todo-head">
            <div class="todo-left">
              <label class="todo-check">
                <input type="checkbox" v-model="todo.done" />
                <span
                  :class="['bubble', todo.category === 'business' ? 'business' : 'personal']"
                ></span>
              </label>
              <span
                class="todo-chip"
                :class="todo.category === 'business' ? 'business' : 'personal'"
              >
                {{ todo.category === 'business' ? 'Business' : 'Personal' }}
              </span>
            </div>
          </div>

          <div class="todo-body">
            <textarea
              class="todo-textarea"
              rows="1"
              v-model="todo.content"
              @input="autoResize($event)"
              @focus="autoResize($event)"
            ></textarea>
          </div>

          <div class="todo-foot">
            <span class="todo-meta">{{ todo.createdAt }}</span>
            <div class="actions">
              <button class="delete" @click="removeTodo(todo)">Delete</button>
            </div>
          </div>
        </div>
      </div>
    </section>

    <section v-else class="tracker main-panel">
      <div class="tracker-header">
        <div>
          <h3>Track Sales</h3>
          <p class="muted">Quick add: receipt + attaches</p>
        </div>
        <div class="tracker-pills">
          <span class="pill"><img :src="iconRevenue" alt="$" class="icon-img"> ${{ dailyTotals.revenue.toFixed(2) }}</span>
          <span class="pill"><img :src="iconM365" alt="M365" class="icon-img"> {{ dailyTotals.m365 }}</span>
          <span class="pill"><img :src="iconAV" alt="AV" class="icon-img"> {{ dailyTotals.antivirus }}</span>
          <span class="pill"><img :src="iconMem" alt="Mem" class="icon-img"> {{ dailyTotals.membership }}</span>
          <span class="pill"><img :src="iconCC" alt="App" class="icon-img"> {{ dailyTotals.creditApp }}</span>
          <span class="pill"><img :src="iconBasket" alt="Basket" class="icon-img"> {{ dailyTotals.basket }}</span>
          <span class="pill"><img :src="iconGSP" alt="GSP" class="icon-img"> {{ dailyTotals.gsp }}</span>
        </div>
      </div>

      <div class="tracker-grid">
        <div class="tracker-card">
          <form class="tracker-form" @submit.prevent="addTrackerEntry">
            <label class="amount-row">
              <img :src="iconRevenue" alt="$" class="icon-img">
              <input type="number" step="0.01" min="0" v-model="trackerForm.amount" placeholder="$0.00" />
            </label>
            <label :class="['icon-row', { 'is-selected': trackerForm.m365 > 0 }]">
              <button type="button" class="icon-btn" @click="bumpField('m365')" title="Add M365">
                <img :src="iconM365" alt="M365" class="icon-img">
              </button>
              <input type="number" min="0" v-model="trackerForm.m365" readonly />
            </label>
            <label :class="['icon-row', { 'is-selected': trackerForm.antivirus > 0 }]">
              <button type="button" class="icon-btn" @click="bumpField('antivirus')" title="Add AV">
                <img :src="iconAV" alt="AV" class="icon-img">
              </button>
              <input type="number" min="0" v-model="trackerForm.antivirus" readonly />
            </label>
            <label :class="['icon-row', { 'is-selected': trackerForm.membership > 0 }]">
              <button type="button" class="icon-btn" @click="bumpField('membership')" title="Add Mem">
                <img :src="iconMem" alt="Mem" class="icon-img">
              </button>
              <input type="number" min="0" v-model="trackerForm.membership" readonly />
            </label>
            <label :class="['icon-row', { 'is-selected': trackerForm.creditApp > 0 }]">
              <button type="button" class="icon-btn" @click="bumpField('creditApp')" title="Add App">
                <img :src="iconCC" alt="App" class="icon-img">
              </button>
              <input type="number" min="0" v-model="trackerForm.creditApp" readonly />
            </label>
            <label :class="['icon-row', { 'is-selected': trackerForm.basket > 0 }]">
              <button type="button" class="icon-btn" @click="bumpField('basket')" title="Add Basket">
                <img :src="iconBasket" alt="Basket" class="icon-img">
              </button>
              <input type="number" min="0" v-model="trackerForm.basket" readonly />
            </label>
            <label :class="['icon-row', { 'is-selected': trackerForm.gsp > 0 }]">
              <button type="button" class="icon-btn" @click="bumpField('gsp')" title="Add GSP">
                <img :src="iconGSP" alt="GSP" class="icon-img">
              </button>
              <input type="number" min="0" v-model="trackerForm.gsp" readonly />
            </label>
            <button type="submit" class="tracker-add">Add</button>
          </form>
        </div>

        <div class="tracker-card">
          <div class="tracker-card-head">
            <h4>Today</h4>
            <small class="muted">{{ todayEntries.length }} entries</small>
          </div>
          <div v-if="todayEntries.length === 0" class="empty-state">
            <p>No entries yet. Log your first sale.</p>
          </div>
          <div v-else class="tracker-table">
          <div class="tracker-row tracker-header">
            <img :src="iconRevenue" alt="$" class="icon-img">
            <img :src="iconM365" alt="M365" class="icon-img">
            <img :src="iconAV" alt="AV" class="icon-img">
            <img :src="iconMem" alt="Mem" class="icon-img">
            <img :src="iconCC" alt="App" class="icon-img">
            <img :src="iconBasket" alt="Basket" class="icon-img">
            <img :src="iconGSP" alt="GSP" class="icon-img">
            <span></span>
          </div>
            <div
              class="tracker-row"
              v-for="entry in todayEntries"
              :key="entry.createdAt"
            >
              <span>${{ Number(entry.amount).toFixed(2) }}</span>
              <span>{{ entry.m365 }}</span>
              <span>{{ entry.antivirus }}</span>
              <span>{{ entry.membership }}</span>
              <span>{{ entry.creditApp }}</span>
              <span>{{ entry.basket }}</span>
              <span>{{ entry.gsp }}</span>
              <button class="delete ghost" @click="removeTrackerEntry(entry)">Remove</button>
            </div>
          </div>
        </div>
      </div>

      <div class="tracker-summary">
        <div class="summary-card">
          <div class="summary-head">
            <span>This Month</span>
          </div>
          <div class="summary-row"><img :src="iconRevenue" class="icon-img" alt="$"> ${{ monthlyTotals.revenue.toFixed(2) }}</div>
          <div class="summary-row"><img :src="iconM365" class="icon-img" alt="M365"> {{ monthlyTotals.m365 }}</div>
          <div class="summary-row"><img :src="iconAV" class="icon-img" alt="AV"> {{ monthlyTotals.antivirus }}</div>
          <div class="summary-row"><img :src="iconMem" class="icon-img" alt="Mem"> {{ monthlyTotals.membership }}</div>
          <div class="summary-row"><img :src="iconCC" class="icon-img" alt="App"> {{ monthlyTotals.creditApp }}</div>
          <div class="summary-row"><img :src="iconBasket" class="icon-img" alt="Basket"> {{ monthlyTotals.basket }}</div>
          <div class="summary-row"><img :src="iconGSP" class="icon-img" alt="GSP"> {{ monthlyTotals.gsp }}</div>
        </div>

        <div class="summary-card">
          <div class="summary-head">
            <span>This Quarter</span>
          </div>
          <div class="summary-row"><img :src="iconRevenue" class="icon-img" alt="$"> ${{ quarterlyTotals.revenue.toFixed(2) }}</div>
          <div class="summary-row"><img :src="iconM365" class="icon-img" alt="M365"> {{ quarterlyTotals.m365 }}</div>
          <div class="summary-row"><img :src="iconAV" class="icon-img" alt="AV"> {{ quarterlyTotals.antivirus }}</div>
          <div class="summary-row"><img :src="iconMem" class="icon-img" alt="Mem"> {{ quarterlyTotals.membership }}</div>
          <div class="summary-row"><img :src="iconCC" class="icon-img" alt="App"> {{ quarterlyTotals.creditApp }}</div>
          <div class="summary-row"><img :src="iconBasket" class="icon-img" alt="Basket"> {{ quarterlyTotals.basket }}</div>
          <div class="summary-row"><img :src="iconGSP" class="icon-img" alt="GSP"> {{ quarterlyTotals.gsp }}</div>
        </div>

        <div class="summary-card">
          <div class="summary-head">
            <span>This Year</span>
          </div>
          <div class="summary-row"><img :src="iconRevenue" class="icon-img" alt="$"> ${{ yearlyTotals.revenue.toFixed(2) }}</div>
          <div class="summary-row"><img :src="iconM365" class="icon-img" alt="M365"> {{ yearlyTotals.m365 }}</div>
          <div class="summary-row"><img :src="iconAV" class="icon-img" alt="AV"> {{ yearlyTotals.antivirus }}</div>
          <div class="summary-row"><img :src="iconMem" class="icon-img" alt="Mem"> {{ yearlyTotals.membership }}</div>
          <div class="summary-row"><img :src="iconCC" class="icon-img" alt="App"> {{ yearlyTotals.creditApp }}</div>
          <div class="summary-row"><img :src="iconBasket" class="icon-img" alt="Basket"> {{ yearlyTotals.basket }}</div>
          <div class="summary-row"><img :src="iconGSP" class="icon-img" alt="GSP"> {{ yearlyTotals.gsp }}</div>
        </div>

        <div class="summary-card">
          <div class="summary-head">
            <span>All Time</span>
          </div>
          <div class="summary-row"><img :src="iconRevenue" class="icon-img" alt="$"> ${{ allTimeTotals.revenue.toFixed(2) }}</div>
          <div class="summary-row"><img :src="iconM365" class="icon-img" alt="M365"> {{ allTimeTotals.m365 }}</div>
          <div class="summary-row"><img :src="iconAV" class="icon-img" alt="AV"> {{ allTimeTotals.antivirus }}</div>
          <div class="summary-row"><img :src="iconMem" class="icon-img" alt="Mem"> {{ allTimeTotals.membership }}</div>
          <div class="summary-row"><img :src="iconCC" class="icon-img" alt="App"> {{ allTimeTotals.creditApp }}</div>
          <div class="summary-row"><img :src="iconBasket" class="icon-img" alt="Basket"> {{ allTimeTotals.basket }}</div>
          <div class="summary-row"><img :src="iconGSP" class="icon-img" alt="GSP"> {{ allTimeTotals.gsp }}</div>
        </div>
      </div>
    </section>

    <footer>
      <p>Ã‚Â© 2024 Moe Ezzeldin. All rights reserved.</p>
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


