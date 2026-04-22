<template>
  <div class="app-layout">
    <aside class="sidebar" :class="{ 'sidebar-open': sidebarOpen }">
      <div class="sidebar-header">
        <div class="sidebar-logo">
          <div class="logo-icon">F</div>
          <div class="logo-text">
            <h1>{{ t('nav.companyName') }}</h1>
            <span class="logo-subtitle">{{ t('nav.subtitle') }}</span>
          </div>
        </div>
      </div>

      <nav class="sidebar-nav">
        <router-link to="/" class="nav-item" :class="{ active: $route.path === '/' }">
          <svg class="nav-icon" viewBox="0 0 20 20" fill="none" stroke="currentColor" stroke-width="1.5">
            <rect x="2" y="2" width="7" height="7" rx="1.5"/>
            <rect x="11" y="2" width="7" height="4" rx="1.5"/>
            <rect x="2" y="11" width="7" height="7" rx="1.5"/>
            <rect x="11" y="8" width="7" height="10" rx="1.5"/>
          </svg>
          <span class="nav-label">{{ t('nav.overview') }}</span>
        </router-link>
        <router-link to="/inventory" class="nav-item" :class="{ active: $route.path === '/inventory' }">
          <svg class="nav-icon" viewBox="0 0 20 20" fill="none" stroke="currentColor" stroke-width="1.5">
            <path d="M2 6l8-4 8 4-8 4-8-4z"/>
            <path d="M2 10l8 4 8-4"/>
            <path d="M2 14l8 4 8-4"/>
          </svg>
          <span class="nav-label">{{ t('nav.inventory') }}</span>
        </router-link>
        <router-link to="/orders" class="nav-item" :class="{ active: $route.path === '/orders' }">
          <svg class="nav-icon" viewBox="0 0 20 20" fill="none" stroke="currentColor" stroke-width="1.5">
            <rect x="4" y="2" width="12" height="16" rx="2"/>
            <path d="M7 6h6M7 10h6M7 14h3"/>
          </svg>
          <span class="nav-label">{{ t('nav.orders') }}</span>
        </router-link>
        <router-link to="/spending" class="nav-item" :class="{ active: $route.path === '/spending' }">
          <svg class="nav-icon" viewBox="0 0 20 20" fill="none" stroke="currentColor" stroke-width="1.5">
            <path d="M10 2v16M6 6c0-1.5 1.5-2.5 4-2.5s4 1 4 2.5-1.5 2.5-4 3-4 1.5-4 3 1.5 2.5 4 2.5 4-1 4-2.5"/>
          </svg>
          <span class="nav-label">{{ t('nav.finance') }}</span>
        </router-link>
        <router-link to="/demand" class="nav-item" :class="{ active: $route.path === '/demand' }">
          <svg class="nav-icon" viewBox="0 0 20 20" fill="none" stroke="currentColor" stroke-width="1.5">
            <path d="M2 16l5-5 3 3 8-10"/>
            <path d="M14 4h4v4"/>
          </svg>
          <span class="nav-label">{{ t('nav.demandForecast') }}</span>
        </router-link>
        <router-link to="/reports" class="nav-item" :class="{ active: $route.path === '/reports' }">
          <svg class="nav-icon" viewBox="0 0 20 20" fill="none" stroke="currentColor" stroke-width="1.5">
            <rect x="2" y="10" width="3" height="8" rx="1"/>
            <rect x="7" y="6" width="3" height="12" rx="1"/>
            <rect x="12" y="2" width="3" height="16" rx="1"/>
          </svg>
          <span class="nav-label">Reports</span>
        </router-link>
      </nav>

      <div class="sidebar-footer">
        <LanguageSwitcher />
        <div class="sidebar-divider"></div>
        <ProfileMenu
          @show-profile-details="showProfileDetails = true"
          @show-tasks="showTasks = true"
        />
      </div>
    </aside>

    <div v-if="sidebarOpen" class="sidebar-backdrop" @click="sidebarOpen = false"></div>

    <div class="main-area">
      <header class="top-header">
        <button class="hamburger-btn" @click="toggleSidebar">
          <svg viewBox="0 0 20 20" fill="none" stroke="currentColor" stroke-width="1.5">
            <path d="M3 5h14M3 10h14M3 15h14"/>
          </svg>
        </button>
        <h2 class="header-title">{{ pageTitle }}</h2>
      </header>
      <FilterBar />
      <main class="main-content">
        <router-view />
      </main>
    </div>

    <ProfileDetailsModal
      :is-open="showProfileDetails"
      @close="showProfileDetails = false"
    />

    <TasksModal
      :is-open="showTasks"
      :tasks="tasks"
      @close="showTasks = false"
      @add-task="addTask"
      @delete-task="deleteTask"
      @toggle-task="toggleTask"
    />
  </div>
</template>

<script>
import { ref, onMounted, computed } from 'vue'
import { useRoute } from 'vue-router'
import { api } from './api'
import { useAuth } from './composables/useAuth'
import { useI18n } from './composables/useI18n'
import FilterBar from './components/FilterBar.vue'
import ProfileMenu from './components/ProfileMenu.vue'
import ProfileDetailsModal from './components/ProfileDetailsModal.vue'
import TasksModal from './components/TasksModal.vue'
import LanguageSwitcher from './components/LanguageSwitcher.vue'

export default {
  name: 'App',
  components: {
    FilterBar,
    ProfileMenu,
    ProfileDetailsModal,
    TasksModal,
    LanguageSwitcher
  },
  setup() {
    const { currentUser } = useAuth()
    const { t } = useI18n()
    const showProfileDetails = ref(false)
    const showTasks = ref(false)
    const apiTasks = ref([])

    const route = useRoute()
    const pageTitle = computed(() => {
      const titles = {
        '/': t('nav.overview'),
        '/inventory': t('nav.inventory'),
        '/orders': t('nav.orders'),
        '/spending': t('nav.finance'),
        '/demand': t('nav.demandForecast'),
        '/reports': 'Reports'
      }
      return titles[route.path] || ''
    })

    const sidebarOpen = ref(false)
    const toggleSidebar = () => {
      sidebarOpen.value = !sidebarOpen.value
    }

    // Merge mock tasks from currentUser with API tasks
    const tasks = computed(() => {
      return [...currentUser.value.tasks, ...apiTasks.value]
    })

    const loadTasks = async () => {
      try {
        apiTasks.value = await api.getTasks()
      } catch (err) {
        console.error('Failed to load tasks:', err)
      }
    }

    const addTask = async (taskData) => {
      try {
        const newTask = await api.createTask(taskData)
        // Add new task to the beginning of the array
        apiTasks.value.unshift(newTask)
      } catch (err) {
        console.error('Failed to add task:', err)
      }
    }

    const deleteTask = async (taskId) => {
      try {
        // Check if it's a mock task (from currentUser)
        const isMockTask = currentUser.value.tasks.some(t => t.id === taskId)

        if (isMockTask) {
          // Remove from mock tasks
          const index = currentUser.value.tasks.findIndex(t => t.id === taskId)
          if (index !== -1) {
            currentUser.value.tasks.splice(index, 1)
          }
        } else {
          // Remove from API tasks
          await api.deleteTask(taskId)
          apiTasks.value = apiTasks.value.filter(t => t.id !== taskId)
        }
      } catch (err) {
        console.error('Failed to delete task:', err)
      }
    }

    const toggleTask = async (taskId) => {
      try {
        // Check if it's a mock task (from currentUser)
        const mockTask = currentUser.value.tasks.find(t => t.id === taskId)

        if (mockTask) {
          // Toggle mock task status
          mockTask.status = mockTask.status === 'pending' ? 'completed' : 'pending'
        } else {
          // Toggle API task
          const updatedTask = await api.toggleTask(taskId)
          const index = apiTasks.value.findIndex(t => t.id === taskId)
          if (index !== -1) {
            apiTasks.value[index] = updatedTask
          }
        }
      } catch (err) {
        console.error('Failed to toggle task:', err)
      }
    }

    onMounted(loadTasks)

    return {
      t,
      showProfileDetails,
      showTasks,
      tasks,
      addTask,
      deleteTask,
      toggleTask,
      pageTitle,
      sidebarOpen,
      toggleSidebar
    }
  }
}
</script>

<style>
:root {
  /* Sidebar */
  --sidebar-width: 260px;
  --sidebar-collapsed-width: 68px;
  --sidebar-bg: #0f172a;
  --sidebar-text: #94a3b8;
  --sidebar-text-hover: #f1f5f9;
  --sidebar-text-active: #ffffff;
  --sidebar-active-bg: rgba(59, 130, 246, 0.15);
  --sidebar-active-border: #3b82f6;
  --sidebar-divider: rgba(148, 163, 184, 0.12);
  --sidebar-logo-text: #ffffff;
  --sidebar-z: 200;

  /* Top header bar */
  --header-height: 60px;
  --header-bg: #ffffff;
  --header-border: #e2e8f0;
  --header-z: 100;

  /* Content */
  --content-bg: #f8fafc;
  --content-padding: 1.5rem 2rem;

  /* Spacing scale */
  --space-1: 0.25rem;
  --space-2: 0.5rem;
  --space-3: 0.75rem;
  --space-4: 1rem;
  --space-5: 1.25rem;
  --space-6: 1.5rem;
  --space-8: 2rem;
  --space-10: 2.5rem;
  --space-12: 3rem;

  /* Transitions */
  --transition-fast: 0.15s ease;
  --transition-base: 0.2s ease;
  --transition-slow: 0.3s cubic-bezier(0.4, 0, 0.2, 1);

  /* Typography */
  --font-family: 'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
  --nav-font-size: 0.875rem;
  --nav-font-weight: 500;
  --nav-icon-size: 20px;

  /* Border radius */
  --radius-sm: 6px;
  --radius-md: 8px;
  --radius-lg: 10px;

  /* Colors */
  --color-primary: #2563eb;
  --color-primary-light: #eff6ff;
  --color-text: #1e293b;
  --color-text-dark: #0f172a;
  --color-text-muted: #64748b;
  --color-border: #e2e8f0;
  --color-border-hover: #cbd5e1;
  --color-surface: #ffffff;
  --color-bg: #f8fafc;
}

* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: var(--font-family);
  background: var(--color-bg);
  color: var(--color-text);
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

/* ===== Layout Shell ===== */
.app-layout {
  display: flex;
  min-height: 100vh;
}

.sidebar {
  position: fixed;
  top: 0;
  left: 0;
  bottom: 0;
  width: var(--sidebar-width);
  background: var(--sidebar-bg);
  display: flex;
  flex-direction: column;
  z-index: var(--sidebar-z);
  overflow-y: auto;
  overflow-x: hidden;
  transition: width var(--transition-slow), transform var(--transition-slow);
}

.main-area {
  margin-left: var(--sidebar-width);
  flex: 1;
  display: flex;
  flex-direction: column;
  min-height: 100vh;
  background: var(--content-bg);
  transition: margin-left var(--transition-slow);
}

/* ===== Sidebar Header (Logo) ===== */
.sidebar-header {
  padding: var(--space-6) var(--space-5);
  border-bottom: 1px solid var(--sidebar-divider);
}

.sidebar-logo {
  display: flex;
  align-items: center;
  gap: var(--space-3);
}

.logo-icon {
  width: 36px;
  height: 36px;
  background: var(--color-primary);
  border-radius: var(--radius-md);
  display: flex;
  align-items: center;
  justify-content: center;
  color: #fff;
  font-weight: 700;
  font-size: 1rem;
  flex-shrink: 0;
}

.sidebar-logo h1 {
  font-size: 1rem;
  font-weight: 700;
  color: var(--sidebar-logo-text);
  line-height: 1.2;
}

.logo-subtitle {
  font-size: 0.7rem;
  color: var(--sidebar-text);
  letter-spacing: 0.02em;
}

.logo-text {
  overflow: hidden;
}

/* ===== Sidebar Navigation ===== */
.sidebar-nav {
  flex: 1;
  padding: var(--space-4) 0;
  display: flex;
  flex-direction: column;
  gap: var(--space-1);
}

.nav-item {
  display: flex;
  align-items: center;
  gap: var(--space-3);
  padding: var(--space-3) var(--space-5);
  margin: 0 var(--space-3);
  border-radius: var(--radius-md);
  color: var(--sidebar-text);
  text-decoration: none;
  font-size: var(--nav-font-size);
  font-weight: var(--nav-font-weight);
  transition: all var(--transition-fast);
  position: relative;
}

.nav-icon {
  width: var(--nav-icon-size);
  height: var(--nav-icon-size);
  flex-shrink: 0;
}

.nav-item:hover {
  color: var(--sidebar-text-hover);
  background: rgba(255, 255, 255, 0.06);
}

.nav-item.router-link-exact-active,
.nav-item.active {
  color: var(--sidebar-text-active);
  background: var(--sidebar-active-bg);
}

.nav-item.router-link-exact-active::before,
.nav-item.active::before {
  content: '';
  position: absolute;
  left: 0;
  top: var(--space-2);
  bottom: var(--space-2);
  width: 3px;
  border-radius: 0 3px 3px 0;
  background: var(--sidebar-active-border);
}

.nav-label {
  overflow: hidden;
  white-space: nowrap;
}

/* ===== Sidebar Footer ===== */
.sidebar-footer {
  padding: var(--space-4) var(--space-5);
  border-top: 1px solid var(--sidebar-divider);
  display: flex;
  flex-direction: column;
  gap: var(--space-3);
}

.sidebar-divider {
  height: 1px;
  background: var(--sidebar-divider);
}

/* ===== Top Header ===== */
.top-header {
  position: sticky;
  top: 0;
  height: var(--header-height);
  background: var(--header-bg);
  border-bottom: 1px solid var(--header-border);
  z-index: var(--header-z);
  display: flex;
  align-items: center;
  padding: 0 var(--space-8);
  flex-shrink: 0;
}

.header-title {
  font-size: 1.25rem;
  font-weight: 700;
  color: var(--color-text-dark);
}

.hamburger-btn {
  display: none;
  align-items: center;
  justify-content: center;
  width: 36px;
  height: 36px;
  padding: 0;
  margin-right: var(--space-3);
  background: none;
  border: 1px solid var(--color-border);
  border-radius: var(--radius-sm);
  cursor: pointer;
  color: var(--color-text-muted);
  transition: all var(--transition-fast);
}

.hamburger-btn:hover {
  background: var(--color-bg);
  color: var(--color-text);
}

.hamburger-btn svg {
  width: 20px;
  height: 20px;
}

/* ===== Main Content ===== */
.main-content {
  flex: 1;
  padding: var(--content-padding);
}

/* ===== Sidebar Backdrop (mobile) ===== */
.sidebar-backdrop {
  display: none;
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.5);
  z-index: 199;
}

/* ===== Responsive: Tablet (collapsed sidebar) ===== */
@media (max-width: 1024px) {
  .sidebar {
    width: var(--sidebar-collapsed-width);
  }

  .main-area {
    margin-left: var(--sidebar-collapsed-width);
  }

  .nav-label,
  .logo-text,
  .logo-subtitle {
    display: none;
  }

  .nav-item {
    justify-content: center;
    padding: var(--space-3);
    margin: 0 var(--space-2);
  }

  .sidebar-header {
    padding: var(--space-4) var(--space-3);
    display: flex;
    justify-content: center;
  }

  .sidebar-logo {
    justify-content: center;
  }

  .sidebar-footer {
    padding: var(--space-4) var(--space-3);
    align-items: center;
  }
}

/* ===== Responsive: Mobile (off-screen sidebar) ===== */
@media (max-width: 768px) {
  .sidebar {
    width: var(--sidebar-width);
    transform: translateX(-100%);
  }

  .sidebar.sidebar-open {
    transform: translateX(0);
  }

  .main-area {
    margin-left: 0;
  }

  .hamburger-btn {
    display: flex;
  }

  .sidebar-backdrop {
    display: block;
  }

  .nav-label,
  .logo-text,
  .logo-subtitle {
    display: block;
  }

  .nav-item {
    justify-content: flex-start;
    padding: var(--space-3) var(--space-5);
    margin: 0 var(--space-3);
  }

  .sidebar-header {
    padding: var(--space-6) var(--space-5);
    display: block;
    justify-content: initial;
  }

  .sidebar-logo {
    justify-content: flex-start;
  }

  .sidebar-footer {
    padding: var(--space-4) var(--space-5);
    align-items: stretch;
  }
}

/* ===== Shared Utility Styles (preserved) ===== */
.page-header {
  margin-bottom: 1.5rem;
}

.page-header h2 {
  font-size: 1.875rem;
  font-weight: 700;
  color: #0f172a;
  margin-bottom: 0.375rem;
  letter-spacing: -0.025em;
}

.page-header p {
  color: #64748b;
  font-size: 0.938rem;
}

.stats-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
  gap: 1.25rem;
  margin-bottom: 1.5rem;
}

.stat-card {
  background: white;
  padding: 1.25rem;
  border-radius: 10px;
  border: 1px solid #e2e8f0;
  transition: all 0.2s ease;
}

.stat-card:hover {
  border-color: #cbd5e1;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.06);
}

.stat-label {
  color: #64748b;
  font-size: 0.875rem;
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.5px;
  margin-bottom: 0.625rem;
}

.stat-value {
  font-size: 2.25rem;
  font-weight: 700;
  color: #0f172a;
  letter-spacing: -0.025em;
}

.stat-card.warning .stat-value {
  color: #ea580c;
}

.stat-card.success .stat-value {
  color: #059669;
}

.stat-card.danger .stat-value {
  color: #dc2626;
}

.stat-card.info .stat-value {
  color: #2563eb;
}

.card {
  background: white;
  border-radius: 10px;
  padding: 1.25rem;
  border: 1px solid #e2e8f0;
  margin-bottom: 1.25rem;
}

.card-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 1rem;
  padding-bottom: 0.875rem;
  border-bottom: 1px solid #e2e8f0;
}

.card-title {
  font-size: 1.125rem;
  font-weight: 700;
  color: #0f172a;
  letter-spacing: -0.025em;
}

.table-container {
  overflow-x: auto;
}

table {
  width: 100%;
  border-collapse: collapse;
}

thead {
  background: #f8fafc;
  border-top: 1px solid #e2e8f0;
  border-bottom: 1px solid #e2e8f0;
}

th {
  text-align: left;
  padding: 0.5rem 0.75rem;
  font-weight: 600;
  color: #475569;
  font-size: 0.75rem;
  text-transform: uppercase;
  letter-spacing: 0.05em;
}

td {
  padding: 0.5rem 0.75rem;
  border-top: 1px solid #f1f5f9;
  color: #334155;
  font-size: 0.875rem;
}

tbody tr {
  transition: background-color 0.15s ease;
}

tbody tr:hover {
  background: #f8fafc;
}

.badge {
  display: inline-block;
  padding: 0.313rem 0.75rem;
  border-radius: 6px;
  font-size: 0.75rem;
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.025em;
}

.badge.success {
  background: #d1fae5;
  color: #065f46;
}

.badge.warning {
  background: #fed7aa;
  color: #92400e;
}

.badge.danger {
  background: #fecaca;
  color: #991b1b;
}

.badge.info {
  background: #dbeafe;
  color: #1e40af;
}

.badge.increasing {
  background: #d1fae5;
  color: #065f46;
}

.badge.decreasing {
  background: #fecaca;
  color: #991b1b;
}

.badge.stable {
  background: #e0e7ff;
  color: #3730a3;
}

.badge.high {
  background: #fecaca;
  color: #991b1b;
}

.badge.medium {
  background: #fed7aa;
  color: #92400e;
}

.badge.low {
  background: #dbeafe;
  color: #1e40af;
}

.loading {
  text-align: center;
  padding: 3rem;
  color: #64748b;
  font-size: 0.938rem;
}

.error {
  background: #fef2f2;
  border: 1px solid #fecaca;
  color: #991b1b;
  padding: 1rem;
  border-radius: 8px;
  margin: 1rem 0;
  font-size: 0.938rem;
}
</style>
