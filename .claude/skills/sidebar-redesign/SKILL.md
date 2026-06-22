---
name: sidebar-redesign
description: Transforms a Vue 3 app from a horizontal top nav bar to a fixed vertical sidebar SaaS layout. Invoke when asked to modernize the navigation, redesign the UI shell, or convert to a sidebar layout.
---

# Sidebar Redesign Skill

This skill transforms a Vue 3 app shell from a horizontal top navigation bar into a modern SaaS-style fixed vertical sidebar. The changes are scoped to **4 files only**: `App.vue`, `FilterBar.vue`, `ProfileMenu.vue`, and `LanguageSwitcher.vue`. No view files in `client/src/views/` are touched. No new dependencies are added.

---

## Step 1: Read These Files First

Read all 5 files before making any edits:

1. `client/src/App.vue` — understand the full template structure, all component imports, the `<script setup>` block (refs, event handlers), and the entire global `<style>` block
2. `client/src/components/FilterBar.vue` — find the `position: sticky; top: 70px` rule that must be updated
3. `client/src/components/ProfileMenu.vue` — note the dropdown anchoring (`right: 0`) that needs direction correction
4. `client/src/components/LanguageSwitcher.vue` — same dropdown direction issue
5. `client/src/main.js` — confirm the exact route paths for all 6 nav links

---

## Step 2: Redesign App.vue

### Template Changes

Remove the entire `<header class="top-nav">` block and the standalone `<FilterBar />` that follows it.

Replace the root `<div class="app">` contents with this layout:

```vue
<div class="app">
  <!-- Fixed vertical sidebar -->
  <aside class="sidebar">
    <div class="sidebar-logo">
      <h1>FactoryOS</h1>
      <span class="sidebar-subtitle">Inventory Platform</span>
    </div>

    <nav class="sidebar-nav">
      <router-link to="/" class="sidebar-link" :class="{ active: $route.path === '/' }">
        <svg viewBox="0 0 18 18" fill="none" xmlns="http://www.w3.org/2000/svg">
          <path d="M2.25 12L9 4.5L15.75 12V16.5H11.25V12.75H6.75V16.5H2.25V12Z" stroke="currentColor" stroke-width="1.5" stroke-linejoin="round"/>
        </svg>
        Dashboard
      </router-link>

      <router-link to="/inventory" class="sidebar-link" :class="{ active: $route.path === '/inventory' }">
        <svg viewBox="0 0 18 18" fill="none" xmlns="http://www.w3.org/2000/svg">
          <path d="M9 1.5L16.5 5.25V12.75L9 16.5L1.5 12.75V5.25L9 1.5Z" stroke="currentColor" stroke-width="1.5" stroke-linejoin="round"/>
          <path d="M9 9L16.5 5.25M9 9L1.5 5.25M9 9V16.5" stroke="currentColor" stroke-width="1.5" stroke-linecap="round"/>
        </svg>
        Inventory
      </router-link>

      <router-link to="/orders" class="sidebar-link" :class="{ active: $route.path === '/orders' }">
        <svg viewBox="0 0 18 18" fill="none" xmlns="http://www.w3.org/2000/svg">
          <path d="M1.5 1.5H3.75L5.25 10.5H13.5L15 4.5H4.5" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/>
          <circle cx="6.75" cy="13.5" r="1.125" stroke="currentColor" stroke-width="1.5"/>
          <circle cx="12" cy="13.5" r="1.125" stroke="currentColor" stroke-width="1.5"/>
        </svg>
        Orders
      </router-link>

      <router-link to="/spending" class="sidebar-link" :class="{ active: $route.path === '/spending' }">
        <svg viewBox="0 0 18 18" fill="none" xmlns="http://www.w3.org/2000/svg">
          <rect x="1.5" y="7.5" width="3" height="9" rx="0.75" stroke="currentColor" stroke-width="1.5"/>
          <rect x="7.5" y="4.5" width="3" height="12" rx="0.75" stroke="currentColor" stroke-width="1.5"/>
          <rect x="13.5" y="1.5" width="3" height="15" rx="0.75" stroke="currentColor" stroke-width="1.5"/>
        </svg>
        Finance
      </router-link>

      <router-link to="/demand" class="sidebar-link" :class="{ active: $route.path === '/demand' }">
        <svg viewBox="0 0 18 18" fill="none" xmlns="http://www.w3.org/2000/svg">
          <path d="M1.5 13.5L6.75 8.25L10.5 11.25L16.5 4.5" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/>
          <path d="M13.5 4.5H16.5V7.5" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/>
        </svg>
        Demand
      </router-link>

      <router-link to="/reports" class="sidebar-link" :class="{ active: $route.path === '/reports' }">
        <svg viewBox="0 0 18 18" fill="none" xmlns="http://www.w3.org/2000/svg">
          <path d="M10.5 1.5H4.5C3.672 1.5 3 2.172 3 3V15C3 15.828 3.672 16.5 4.5 16.5H13.5C14.328 16.5 15 15.828 15 15V6L10.5 1.5Z" stroke="currentColor" stroke-width="1.5" stroke-linejoin="round"/>
          <path d="M10.5 1.5V6H15" stroke="currentColor" stroke-width="1.5" stroke-linejoin="round"/>
          <path d="M6 10.5H12M6 13.5H10.5" stroke="currentColor" stroke-width="1.5" stroke-linecap="round"/>
        </svg>
        Reports
      </router-link>
    </nav>

    <div class="sidebar-footer">
      <LanguageSwitcher />
      <ProfileMenu
        @show-profile-details="showProfileDetails = true"
        @show-tasks="showTasks = true"
      />
    </div>
  </aside>

  <!-- Scrollable main area -->
  <div class="main-wrapper">
    <FilterBar />
    <main class="main-content">
      <router-view />
    </main>
  </div>

  <!-- Modals (unchanged) -->
  <ProfileDetailsModal
    :is-open="showProfileDetails"
    @close="showProfileDetails = false"
  />
  <TasksModal
    :is-open="showTasks"
    @close="showTasks = false"
  />
</div>
```

> **Note:** Use the exact route paths confirmed from `main.js` in Step 1. Keep all existing `<script>` refs and event handlers — only the `<template>` changes.

---

### CSS Changes for App.vue

In the global `<style>` block, **replace** the following rule sets entirely:
- `.app`
- `.top-nav` and all `.top-nav *` descendants
- `.nav-container`
- `.logo`, `.logo h1`, `.subtitle`
- `.nav-tabs`, `.nav-tabs a`, `.nav-tabs a:hover`, `.nav-tabs a.active`, `.nav-tabs a.active::after`
- `.main-content`

**Replace them with:**

```css
.app {
  display: flex;
  flex-direction: row;
  min-height: 100vh;
}

/* ── Sidebar ── */
.sidebar {
  width: 240px;
  min-width: 240px;
  height: 100vh;
  position: fixed;
  top: 0;
  left: 0;
  background: #0f172a;
  border-right: 1px solid rgba(255, 255, 255, 0.08);
  display: flex;
  flex-direction: column;
  z-index: 100;
  overflow: hidden;
}

.sidebar-logo {
  padding: 1.5rem 1.25rem 1.25rem;
  border-bottom: 1px solid rgba(255, 255, 255, 0.08);
  flex-shrink: 0;
}

.sidebar-logo h1 {
  font-size: 1rem;
  font-weight: 700;
  color: #ffffff;
  letter-spacing: -0.025em;
  line-height: 1.3;
  margin: 0;
}

.sidebar-subtitle {
  font-size: 0.688rem;
  color: #64748b;
  font-weight: 400;
  display: block;
  margin-top: 0.25rem;
  text-transform: uppercase;
  letter-spacing: 0.05em;
}

.sidebar-nav {
  flex: 1;
  padding: 0.75rem;
  display: flex;
  flex-direction: column;
  gap: 0.125rem;
  overflow-y: auto;
}

.sidebar-link {
  display: flex;
  align-items: center;
  gap: 0.75rem;
  padding: 0.625rem 0.875rem;
  color: #94a3b8;
  text-decoration: none;
  font-weight: 500;
  font-size: 0.875rem;
  border-radius: 6px;
  border-left: 3px solid transparent;
  transition: all 0.15s ease;
}

.sidebar-link:hover {
  color: #f1f5f9;
  background: rgba(255, 255, 255, 0.06);
}

.sidebar-link.active {
  color: #ffffff;
  background: rgba(37, 99, 235, 0.25);
  border-left-color: #2563eb;
}

.sidebar-link svg {
  width: 18px;
  height: 18px;
  flex-shrink: 0;
}

.sidebar-footer {
  padding: 0.75rem;
  border-top: 1px solid rgba(255, 255, 255, 0.08);
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
  flex-shrink: 0;
}

/* ── Main area ── */
.main-wrapper {
  margin-left: 240px;
  flex: 1;
  min-height: 100vh;
  display: flex;
  flex-direction: column;
  min-width: 0;
  background: #f8fafc;
}

.main-content {
  flex: 1;
  padding: 1.5rem 2rem;
  max-width: 1360px;
  width: 100%;
}
```

**Do not modify** any other rules in the global `<style>` block (`.card`, `.stat-card`, `.badge`, `.table-container`, `.page-header`, table rules, etc.).

---

## Step 3: Update FilterBar.vue

Make exactly **2 CSS changes** in the `<style scoped>` block. Do not touch the `<template>` or `<script>`.

**Change 1** — `.filters-bar` positioning:
```css
/* Before */
position: sticky;
top: 70px;
z-index: 90;

/* After */
position: sticky;
top: 0;
z-index: 10;
```

**Change 2** — `.filters-container` width constraint:
Remove `max-width: 1600px` and `margin: 0 auto` from `.filters-container`. The filter bar now lives inside `.main-wrapper` which constrains width; centering is no longer needed. Keep `padding: 0 2rem` if present.

---

## Step 4: Fix Dropdown Direction in ProfileMenu.vue and LanguageSwitcher.vue

Both components open their `.dropdown-menu` with `right: 0`, which anchors the dropdown to the right edge of its parent. This works fine in a top-right nav but clips content when the components are placed in a left sidebar.

In both files, find the `.dropdown-menu` rule in `<style scoped>` and change:
```css
/* Before */
right: 0;

/* After */
left: 0;
right: auto;
```

This makes the dropdown open rightward from the left edge of the button, staying within the viewport.

---

## Step 5: View Files — No Changes

Do **not** modify any file in `client/src/views/`. All global utility classes (`.card`, `.stat-card`, `.badge`, `.page-header`, `.table-container`, etc.) remain in `App.vue`'s global `<style>` block and continue to work for all views without change.

---

## Step 6: Verification Checklist

Run through all 9 checks before declaring the redesign complete:

1. **Dev server starts** — `npm run dev` in `client/` completes without errors
2. **Sidebar renders** — left sidebar visible at 240px wide with dark (`#0f172a`) background
3. **No content overlap** — main content area starts at 240px from the left; no overlap with sidebar
4. **All 6 routes work** — navigate to each route; the correct sidebar link highlights as `.active`; page content renders correctly
5. **FilterBar works** — appears sticky at the top of the main content area; all 4 filters change data correctly
6. **ProfileMenu dropdown** — opens without overflowing the viewport left edge
7. **LanguageSwitcher dropdown** — same check
8. **1024px viewport** — resize browser to 1024px wide; no horizontal scroll; sidebar remains visible; content scrolls normally
9. **Modals still work** — open ProfileDetailsModal via ProfileMenu and TasksModal via tasks button; both render correctly over the layout
