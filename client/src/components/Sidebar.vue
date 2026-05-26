<template>
  <div class="sidebar-backdrop" v-if="mobileOpen" @click="mobileOpen = false"></div>

  <aside class="sidebar" :class="{ 'mobile-open': mobileOpen }">
    <div class="sidebar-logo">
      <span class="logo-name">{{ t('nav.companyName') }}</span>
      <span class="logo-sub">{{ t('nav.subtitle') }}</span>
    </div>

    <nav class="sidebar-nav">
      <router-link
        v-for="item in navItems"
        :key="item.path"
        class="sidebar-item"
        :class="{ active: $route.path === item.path }"
        :to="item.path"
        :data-label="item.label"
        @click="mobileOpen = false"
      >
        <span class="sidebar-icon" v-html="item.icon"></span>
        <span class="sidebar-label">{{ item.label }}</span>
      </router-link>
    </nav>

    <div class="sidebar-footer">
      <LanguageSwitcher />
      <ProfileMenu
        @show-profile-details="$emit('show-profile-details')"
        @show-tasks="$emit('show-tasks')"
      />
    </div>
  </aside>

  <button class="hamburger" @click="mobileOpen = true">
    <svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
      <line x1="3" y1="6" x2="21" y2="6"/>
      <line x1="3" y1="12" x2="21" y2="12"/>
      <line x1="3" y1="18" x2="21" y2="18"/>
    </svg>
  </button>
</template>

<script>
import { ref, computed } from 'vue'
import { useI18n } from '../composables/useI18n'
import LanguageSwitcher from './LanguageSwitcher.vue'
import ProfileMenu from './ProfileMenu.vue'

const ICONS = {
  overview:    `<svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><rect x="3" y="3" width="7" height="7"/><rect x="14" y="3" width="7" height="7"/><rect x="3" y="14" width="7" height="7"/><rect x="14" y="14" width="7" height="7"/></svg>`,
  inventory:   `<svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M21 16V8a2 2 0 0 0-1-1.73l-7-4a2 2 0 0 0-2 0l-7 4A2 2 0 0 0 3 8v8a2 2 0 0 0 1 1.73l7 4a2 2 0 0 0 2 0l7-4A2 2 0 0 0 21 16z"/><polyline points="3.27 6.96 12 12.01 20.73 6.96"/><line x1="12" y1="22.08" x2="12" y2="12"/></svg>`,
  orders:      `<svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M9 5H7a2 2 0 0 0-2 2v12a2 2 0 0 0 2 2h10a2 2 0 0 0 2-2V7a2 2 0 0 0-2-2h-2"/><rect x="9" y="3" width="6" height="4" rx="1"/><line x1="9" y1="12" x2="15" y2="12"/><line x1="9" y1="16" x2="13" y2="16"/></svg>`,
  finance:     `<svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><line x1="18" y1="20" x2="18" y2="10"/><line x1="12" y1="20" x2="12" y2="4"/><line x1="6" y1="20" x2="6" y2="14"/><line x1="2" y1="20" x2="22" y2="20"/></svg>`,
  demand:      `<svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><polyline points="22 7 13.5 15.5 8.5 10.5 2 17"/><polyline points="16 7 22 7 22 13"/></svg>`,
  reports:     `<svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M14 2H6a2 2 0 0 0-2 2v16a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V8z"/><polyline points="14 2 14 8 20 8"/><line x1="16" y1="13" x2="8" y2="13"/><line x1="16" y1="17" x2="8" y2="17"/></svg>`,
  restocking:  `<svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><polyline points="23 4 23 10 17 10"/><polyline points="1 20 1 14 7 14"/><path d="M3.51 9a9 9 0 0 1 14.85-3.36L23 10M1 14l4.64 4.36A9 9 0 0 0 20.49 15"/></svg>`,
}

export default {
  name: 'Sidebar',
  components: { LanguageSwitcher, ProfileMenu },
  emits: ['show-profile-details', 'show-tasks'],
  setup() {
    const { t } = useI18n()
    const mobileOpen = ref(false)

    const navItems = computed(() => [
      { path: '/',           label: t('nav.overview'),       icon: ICONS.overview },
      { path: '/inventory',  label: t('nav.inventory'),      icon: ICONS.inventory },
      { path: '/orders',     label: t('nav.orders'),         icon: ICONS.orders },
      { path: '/spending',   label: t('nav.finance'),        icon: ICONS.finance },
      { path: '/demand',     label: t('nav.demandForecast'), icon: ICONS.demand },
      { path: '/reports',    label: t('nav.reports'),        icon: ICONS.reports },
      { path: '/restocking', label: t('nav.restocking'),     icon: ICONS.restocking },
    ])

    return { t, mobileOpen, navItems }
  }
}
</script>

<style scoped>
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
  /* overflow: hidden removed — would clip dropdown menus and tooltips */
  transition: width 0.2s ease, transform 0.2s ease;
}

.sidebar-backdrop {
  position: fixed;
  inset: 0;
  background: rgba(0, 0, 0, 0.45);
  z-index: 99;
}

.sidebar-logo {
  padding: 1.25rem 1rem 1rem;
  border-bottom: 1px solid #334155;
  flex-shrink: 0;
  white-space: nowrap;
  overflow: hidden; /* hide logo text during width collapse */
}

.logo-name {
  display: block;
  font-size: 0.9rem;
  font-weight: 700;
  color: #e2e8f0;
  margin-bottom: 0.15rem;
}

.logo-sub {
  display: block;
  font-size: 0.7rem;
  color: #64748b;
}

.sidebar-nav {
  flex: 1;
  padding: 0.75rem 0.5rem;
  display: flex;
  flex-direction: column;
  gap: 0.125rem;
  overflow-y: auto;
  /* overflow-x intentionally not set — allows tooltips to extend beyond sidebar */
}

.sidebar-item {
  display: flex;
  align-items: center;
  gap: 0.75rem;
  padding: 0.625rem;
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

.sidebar-footer {
  padding: 0.75rem 0.5rem;
  border-top: 1px solid #334155;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 0.5rem;
  flex-shrink: 0;
}

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

/* 768–1023px: icons only + tooltip */
@media (max-width: 1023px) and (min-width: 768px) {
  .sidebar {
    width: var(--sidebar-icon, 64px);
  }

  .sidebar-logo {
    padding: 1rem 0;
    display: flex;
    align-items: center;
    justify-content: center;
  }

  .logo-name,
  .logo-sub {
    display: none;
  }

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
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.3);
  }

  .sidebar-item:hover::after {
    opacity: 1;
  }
}

/* <768px: hidden sidebar + hamburger */
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
