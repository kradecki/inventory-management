# Collapsible Sidebar Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Replace the horizontal top navigation bar with a responsive collapsible left sidebar that shows full labels (≥1024px), icons-only with tooltips (768–1023px), and a hamburger overlay (<768px).

**Architecture:** A new `Sidebar.vue` component handles all navigation, logo, and footer (profile + language). `App.vue` drops its `<header>` entirely, switches to `flex-direction: row`, and adds a `.main-column` wrapper around FilterBar + router-view. `FilterBar.vue` gets one CSS fix (sticky `top: 70px` → `top: 0`).

**Tech Stack:** Vue 3 Composition API, scoped CSS, inline SVG icons (no external library), CSS media queries, CSS custom properties.

---

## File map

| File | Action | Responsibility |
|---|---|---|
| `client/src/components/Sidebar.vue` | **Create** | All nav: logo, 7 routes, footer (LanguageSwitcher + ProfileMenu), mobile overlay + hamburger |
| `client/src/App.vue` | **Modify** | Remove `<header>`, add `<Sidebar>`, switch layout to row, add `.main-column` wrapper |
| `client/src/components/FilterBar.vue` | **Modify** | Fix `top: 70px` → `top: 0` (one line) |

No view files, composables, or backend files change.

---

## Task 1: Create `client/src/components/Sidebar.vue`

**Files:**
- Create: `client/src/components/Sidebar.vue`

- [ ] **Step 1: Create the file with complete template, script, and styles**

Write the entire file at `client/src/components/Sidebar.vue`:

```vue
<template>
  <!-- Mobile backdrop — tap to close -->
  <div
    v-if="mobileOpen"
    class="sidebar-backdrop"
    @click="mobileOpen = false"
  ></div>

  <!-- Sidebar -->
  <aside class="sidebar" :class="{ 'mobile-open': mobileOpen }">
    <!-- Logo (hidden in icons-only mode via CSS) -->
    <div class="sidebar-logo">
      <span class="logo-name">{{ t('nav.companyName') }}</span>
      <span class="logo-sub">{{ t('nav.subtitle') }}</span>
    </div>

    <!-- Nav items -->
    <nav class="sidebar-nav">
      <router-link
        v-for="item in navItems"
        :key="item.path"
        :to="item.path"
        class="sidebar-item"
        :class="{ active: $route.path === item.path }"
        :data-label="item.label"
        @click="mobileOpen = false"
      >
        <span class="sidebar-icon" v-html="item.icon"></span>
        <span class="sidebar-label">{{ item.label }}</span>
      </router-link>
    </nav>

    <!-- Footer: language + profile -->
    <div class="sidebar-footer">
      <LanguageSwitcher />
      <ProfileMenu
        @show-profile-details="$emit('show-profile-details')"
        @show-tasks="$emit('show-tasks')"
      />
    </div>
  </aside>

  <!-- Hamburger — only visible on mobile via CSS -->
  <button class="hamburger" @click="mobileOpen = true" aria-label="Open navigation">
    <svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
      <line x1="3" y1="6" x2="21" y2="6"/>
      <line x1="3" y1="12" x2="21" y2="12"/>
      <line x1="3" y1="18" x2="21" y2="18"/>
    </svg>
  </button>
</template>

<script>
import { ref } from 'vue'
import { useI18n } from '../composables/useI18n'
import LanguageSwitcher from './LanguageSwitcher.vue'
import ProfileMenu from './ProfileMenu.vue'

const ICONS = {
  overview: `<svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><rect x="3" y="3" width="7" height="7"/><rect x="14" y="3" width="7" height="7"/><rect x="3" y="14" width="7" height="7"/><rect x="14" y="14" width="7" height="7"/></svg>`,
  inventory: `<svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M21 16V8a2 2 0 0 0-1-1.73l-7-4a2 2 0 0 0-2 0l-7 4A2 2 0 0 0 3 8v8a2 2 0 0 0 1 1.73l7 4a2 2 0 0 0 2 0l7-4A2 2 0 0 0 21 16z"/><polyline points="3.27 6.96 12 12.01 20.73 6.96"/><line x1="12" y1="22.08" x2="12" y2="12"/></svg>`,
  orders: `<svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M9 5H7a2 2 0 0 0-2 2v12a2 2 0 0 0 2 2h10a2 2 0 0 0 2-2V7a2 2 0 0 0-2-2h-2"/><rect x="9" y="3" width="6" height="4" rx="1"/><line x1="9" y1="12" x2="15" y2="12"/><line x1="9" y1="16" x2="13" y2="16"/></svg>`,
  finance: `<svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><line x1="18" y1="20" x2="18" y2="10"/><line x1="12" y1="20" x2="12" y2="4"/><line x1="6" y1="20" x2="6" y2="14"/><line x1="2" y1="20" x2="22" y2="20"/></svg>`,
  demand: `<svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><polyline points="22 7 13.5 15.5 8.5 10.5 2 17"/><polyline points="16 7 22 7 22 13"/></svg>`,
  reports: `<svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M14 2H6a2 2 0 0 0-2 2v16a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V8z"/><polyline points="14 2 14 8 20 8"/><line x1="16" y1="13" x2="8" y2="13"/><line x1="16" y1="17" x2="8" y2="17"/></svg>`,
  restocking: `<svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><polyline points="23 4 23 10 17 10"/><polyline points="1 20 1 14 7 14"/><path d="M3.51 9a9 9 0 0 1 14.85-3.36L23 10M1 14l4.64 4.36A9 9 0 0 0 20.49 15"/></svg>`,
}

export default {
  name: 'Sidebar',
  components: { LanguageSwitcher, ProfileMenu },
  emits: ['show-profile-details', 'show-tasks'],
  setup() {
    const { t } = useI18n()
    const mobileOpen = ref(false)

    const navItems = [
      { path: '/',            label: t('nav.overview'),       icon: ICONS.overview },
      { path: '/inventory',   label: t('nav.inventory'),      icon: ICONS.inventory },
      { path: '/orders',      label: t('nav.orders'),         icon: ICONS.orders },
      { path: '/spending',    label: t('nav.finance'),        icon: ICONS.finance },
      { path: '/demand',      label: t('nav.demandForecast'), icon: ICONS.demand },
      { path: '/reports',     label: 'Reports',               icon: ICONS.reports },
      { path: '/restocking',  label: t('nav.restocking'),     icon: ICONS.restocking },
    ]

    return { t, mobileOpen, navItems }
  }
}
</script>

<style scoped>
/* ── Variables ─────────────────────────────────────── */
:root {
  --sidebar-full: 240px;
  --sidebar-icon: 64px;
}

/* ── Backdrop (mobile only) ────────────────────────── */
.sidebar-backdrop {
  position: fixed;
  inset: 0;
  background: rgba(0, 0, 0, 0.45);
  z-index: 99;
}

/* ── Sidebar shell ─────────────────────────────────── */
.sidebar {
  position: fixed;
  top: 0;
  left: 0;
  height: 100vh;
  width: var(--sidebar-full, 240px);
  background: #1e293b;
  border-right: 1px solid #334155;
  display: flex;
  flex-direction: column;
  z-index: 100;
  overflow: hidden;
  transition: width 0.2s ease, transform 0.2s ease;
}

/* ── Logo ──────────────────────────────────────────── */
.sidebar-logo {
  padding: 1.25rem 1rem 1rem;
  border-bottom: 1px solid #334155;
  flex-shrink: 0;
  overflow: hidden;
  white-space: nowrap;
}

.logo-name {
  display: block;
  font-size: 0.9rem;
  font-weight: 700;
  color: #e2e8f0;
  letter-spacing: -0.01em;
  margin-bottom: 0.15rem;
}

.logo-sub {
  display: block;
  font-size: 0.7rem;
  color: #64748b;
}

/* ── Nav ───────────────────────────────────────────── */
.sidebar-nav {
  flex: 1;
  padding: 0.75rem 0.5rem;
  display: flex;
  flex-direction: column;
  gap: 0.125rem;
  overflow-y: auto;
  overflow-x: hidden;
}

.sidebar-item {
  display: flex;
  align-items: center;
  gap: 0.75rem;
  padding: 0.625rem 0.625rem;
  border-radius: 6px;
  color: #94a3b8;
  text-decoration: none;
  font-size: 0.875rem;
  font-weight: 500;
  white-space: nowrap;
  transition: background 0.15s, color 0.15s;
  position: relative;
}

.sidebar-item:hover {
  background: #334155;
  color: #e2e8f0;
}

.sidebar-item.active {
  background: #1d4ed8;
  color: #e0f2fe;
}

.sidebar-item.active .sidebar-icon {
  color: #93c5fd;
}

.sidebar-icon {
  flex-shrink: 0;
  display: flex;
  align-items: center;
  justify-content: center;
  width: 20px;
  height: 20px;
}

.sidebar-label {
  overflow: hidden;
  text-overflow: ellipsis;
}

/* ── Footer ────────────────────────────────────────── */
.sidebar-footer {
  padding: 0.75rem 0.5rem;
  border-top: 1px solid #334155;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 0.5rem;
  flex-shrink: 0;
}

/* ── Hamburger (hidden on desktop) ────────────────── */
.hamburger {
  display: none;
  position: fixed;
  top: 0.75rem;
  left: 0.75rem;
  z-index: 98;
  background: #1e293b;
  border: 1px solid #334155;
  border-radius: 6px;
  padding: 0.375rem;
  color: #e2e8f0;
  cursor: pointer;
  align-items: center;
  justify-content: center;
}

/* ── 768–1023px: icons-only ────────────────────────── */
@media (max-width: 1023px) and (min-width: 768px) {
  .sidebar {
    width: var(--sidebar-icon, 64px);
  }

  .sidebar-logo {
    padding: 1rem 0;
    display: flex;
    align-items: center;
    justify-content: center;
    border-bottom: 1px solid #334155;
  }

  .logo-name,
  .logo-sub {
    display: none;
  }

  /* Show a small brand mark in place of the logo text */
  .sidebar-logo::after {
    content: '';
    display: block;
    width: 28px;
    height: 28px;
    background: #2563eb;
    border-radius: 6px;
  }

  .sidebar-nav {
    padding: 0.75rem 0.375rem;
    align-items: center;
  }

  .sidebar-item {
    justify-content: center;
    padding: 0.625rem;
    width: 100%;
  }

  .sidebar-label {
    display: none;
  }

  /* CSS tooltip: appears to the right of the icon on hover */
  .sidebar-item::after {
    content: attr(data-label);
    position: absolute;
    left: calc(100% + 10px);
    top: 50%;
    transform: translateY(-50%);
    background: #0f172a;
    color: #e2e8f0;
    padding: 0.3rem 0.6rem;
    border-radius: 5px;
    font-size: 0.75rem;
    font-weight: 500;
    white-space: nowrap;
    pointer-events: none;
    opacity: 0;
    transition: opacity 0.15s;
    z-index: 200;
    border: 1px solid #334155;
    box-shadow: 0 2px 8px rgba(0,0,0,0.3);
  }

  .sidebar-item:hover::after {
    opacity: 1;
  }
}

/* ── <768px: hidden sidebar + hamburger ─────────────── */
@media (max-width: 767px) {
  .sidebar {
    width: var(--sidebar-full, 240px);
    transform: translateX(-100%);
  }

  .sidebar.mobile-open {
    transform: translateX(0);
  }

  .hamburger {
    display: flex;
  }
}
</style>
```

- [ ] **Step 2: Verify the file was created correctly**

```bash
ls client/src/components/Sidebar.vue
```
Expected: file exists, no error.

- [ ] **Step 3: Commit**

```bash
git add client/src/components/Sidebar.vue
git commit -m "feat: add Sidebar component with responsive icon/label/overlay modes"
```

---

## Task 2: Update `App.vue` — replace top nav with sidebar

**Files:**
- Modify: `client/src/App.vue`

- [ ] **Step 1: Replace the entire template**

Replace the `<template>` block (lines 1–58) with:

```vue
<template>
  <div class="app">
    <Sidebar
      @show-profile-details="showProfileDetails = true"
      @show-tasks="showTasks = true"
    />

    <div class="main-column">
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
```

- [ ] **Step 2: Update the script — swap imports**

Replace the `<script>` block with:

```vue
<script>
import { ref, onMounted, computed } from 'vue'
import { api } from './api'
import { useAuth } from './composables/useAuth'
import FilterBar from './components/FilterBar.vue'
import Sidebar from './components/Sidebar.vue'
import ProfileDetailsModal from './components/ProfileDetailsModal.vue'
import TasksModal from './components/TasksModal.vue'

export default {
  name: 'App',
  components: {
    Sidebar,
    FilterBar,
    ProfileDetailsModal,
    TasksModal
  },
  setup() {
    const { currentUser } = useAuth()
    const showProfileDetails = ref(false)
    const showTasks = ref(false)
    const apiTasks = ref([])

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
        apiTasks.value.unshift(newTask)
      } catch (err) {
        console.error('Failed to add task:', err)
      }
    }

    const deleteTask = async (taskId) => {
      try {
        const isMockTask = currentUser.value.tasks.some(t => t.id === taskId)
        if (isMockTask) {
          const index = currentUser.value.tasks.findIndex(t => t.id === taskId)
          if (index !== -1) currentUser.value.tasks.splice(index, 1)
        } else {
          await api.deleteTask(taskId)
          apiTasks.value = apiTasks.value.filter(t => t.id !== taskId)
        }
      } catch (err) {
        console.error('Failed to delete task:', err)
      }
    }

    const toggleTask = async (taskId) => {
      try {
        const mockTask = currentUser.value.tasks.find(t => t.id === taskId)
        if (mockTask) {
          mockTask.status = mockTask.status === 'pending' ? 'completed' : 'pending'
        } else {
          const updatedTask = await api.toggleTask(taskId)
          const index = apiTasks.value.findIndex(t => t.id === taskId)
          if (index !== -1) apiTasks.value[index] = updatedTask
        }
      } catch (err) {
        console.error('Failed to toggle task:', err)
      }
    }

    onMounted(loadTasks)

    return {
      showProfileDetails,
      showTasks,
      tasks,
      addTask,
      deleteTask,
      toggleTask
    }
  }
}
</script>
```

- [ ] **Step 3: Update the global CSS in `<style>` — replace layout-related rules**

In the `<style>` block (unscoped, global), delete these rules entirely: `.top-nav`, `.nav-container`, `.nav-container > .nav-tabs`, `.nav-container > .language-switcher`, `.logo`, `.logo h1`, `.nav-tabs`, `.nav-tabs a`, `.nav-tabs a:hover`, `.nav-tabs a.active`, `.nav-tabs a.active::after`. Keep `.subtitle` (it's used by page headers in views). Replace the `.app` rule and add `.main-column`. Replace those rules with:

```css
.app {
  display: flex;
  flex-direction: row;
  min-height: 100vh;
}

.main-column {
  flex: 1;
  display: flex;
  flex-direction: column;
  min-width: 0;
  margin-left: 240px;
  transition: margin-left 0.2s ease;
}

@media (max-width: 1023px) and (min-width: 768px) {
  .main-column {
    margin-left: 64px;
  }
}

@media (max-width: 767px) {
  .main-column {
    margin-left: 0;
  }
}
```

Keep all other global styles (`.main-content`, `.page-header`, `.stats-grid`, `.stat-card`, `.card`, `.badge`, `.loading`, `.error`, etc.) exactly as-is.

- [ ] **Step 4: Verify the dev server has no console errors**

The Vite dev server is running at http://localhost:3000. Open the browser console and confirm no errors. Navigate to the app and confirm:
- Sidebar appears on the left
- Top nav is gone
- Clicking nav items navigates correctly
- FilterBar is still visible at the top of the content area

- [ ] **Step 5: Commit**

```bash
git add client/src/App.vue
git commit -m "feat: replace top nav with collapsible sidebar in App.vue"
```

---

## Task 3: Fix `FilterBar.vue` sticky positioning

**Files:**
- Modify: `client/src/components/FilterBar.vue:109`

- [ ] **Step 1: Change `top: 70px` to `top: 0`**

In `client/src/components/FilterBar.vue`, find the `.filters-bar` CSS rule. It currently has `top: 70px` (the old nav height). Change it to `top: 0`:

```css
/* Before */
.filters-bar {
  position: sticky;
  top: 70px;   /* ← old nav height */
  z-index: 90;
  ...
}

/* After */
.filters-bar {
  position: sticky;
  top: 0;      /* ← sidebar is on the left, no top nav */
  z-index: 90;
  ...
}
```

- [ ] **Step 2: Verify FilterBar sticks correctly**

Scroll down on any page (e.g., http://localhost:3000/inventory with many rows). The filter bar should stick to the top of the viewport, not leave a 70px gap.

- [ ] **Step 3: Commit**

```bash
git add client/src/components/FilterBar.vue
git commit -m "fix: FilterBar sticky top 70px → 0 after removing top nav"
```

---

## Task 4: End-to-end verification

No code changes. Verify the three breakpoint states manually.

- [ ] **Step 1: Verify full sidebar (≥1024px)**

In the browser at http://localhost:3000, resize to at least 1024px wide. Confirm:
- Sidebar is 240px wide, showing icon + label for all 7 nav items
- Logo and subtitle are visible
- Profile menu and language switcher are in the sidebar footer
- Clicking each nav item navigates to the correct route
- Active item has blue background
- FilterBar sticks to top when scrolling

- [ ] **Step 2: Verify icons-only mode (768–1023px)**

Resize to ~900px. Confirm:
- Sidebar shrinks to ~64px, showing icons only
- Labels are hidden, logo text is hidden
- Hovering an icon shows a tooltip with the label
- Active icon has blue background
- Content area fills the remaining width

- [ ] **Step 3: Verify mobile overlay (<768px)**

Resize to ~375px (mobile). Confirm:
- Sidebar is not visible
- Hamburger button (☰) appears top-left
- Tapping hamburger slides sidebar in from left
- Semi-transparent backdrop covers content
- Tapping backdrop closes sidebar
- Tapping a nav item closes sidebar and navigates

- [ ] **Step 4: Run backend tests to confirm nothing broke**

```bash
uv run --directory server pytest ../tests/backend/ -v
```
Expected: 40 passed.

- [ ] **Step 5: Commit**

```bash
git commit --allow-empty -m "chore: verify sidebar end-to-end across all breakpoints"
```
