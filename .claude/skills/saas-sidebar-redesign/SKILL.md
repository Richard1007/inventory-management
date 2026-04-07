---
name: saas-sidebar-redesign
description: Transform a Vue 3 application from horizontal top-nav to a modern SaaS-style vertical sidebar layout with consistent spacing and polished design. Use when redesigning navigation to a sidebar pattern.
---

# SaaS Sidebar Redesign

Transform a Vue 3 application's horizontal top-navigation into a modern SaaS-style vertical sidebar layout.

## Target Layout

```
+------------------+--------------------------------------------------+
|                  |  Top Header (60px)                               |
|   Sidebar        |  [Page Title]                    [Search/Actions]|
|   (260px)        +--------------------------------------------------+
|                  |  Filter Bar (sticky)                             |
|   [Logo]         |  [Period] [Location] [Category] [Status] [Reset]|
|                  +--------------------------------------------------+
|   [Dashboard  ]  |                                                  |
|   [Inventory  ]  |  Main Content Area                               |
|   [Orders     ]  |  (scrollable, full width)                        |
|   [Finance    ]  |                                                  |
|   [Demand     ]  |                                                  |
|   [Reports    ]  |                                                  |
|                  |                                                  |
|   -----------    |                                                  |
|   [Language]     |                                                  |
|   [Profile ]     |                                                  |
+------------------+--------------------------------------------------+
```

## Design Tokens

Inject these CSS custom properties into the `:root` block in App.vue's global (unscoped) `<style>` section. These are the single source of truth for the redesign.

```css
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

  /* Colors (preserve existing palette as variables) */
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
```

## Target DOM Hierarchy

The new App.vue template should produce this structure:

```html
<div class="app-layout">
  <!-- SIDEBAR: fixed left column -->
  <aside class="sidebar">
    <div class="sidebar-header">
      <div class="sidebar-logo">
        <div class="logo-icon"><!-- Single letter or SVG mark --></div>
        <div class="logo-text">
          <h1>Company Name</h1>
          <span class="logo-subtitle">Subtitle</span>
        </div>
      </div>
    </div>

    <nav class="sidebar-nav">
      <!-- One .nav-item per route -->
      <router-link to="/" class="nav-item">
        <svg class="nav-icon"><!-- icon --></svg>
        <span class="nav-label">Overview</span>
      </router-link>
      <!-- ... repeat for each route ... -->
    </nav>

    <div class="sidebar-footer">
      <LanguageSwitcher />
      <div class="sidebar-divider"></div>
      <ProfileMenu />
    </div>
  </aside>

  <!-- MAIN AREA: right of sidebar -->
  <div class="main-area">
    <header class="top-header">
      <h2 class="header-title"><!-- Dynamic page title from route --></h2>
    </header>

    <FilterBar />

    <main class="main-content">
      <router-view />
    </main>
  </div>

  <!-- Modals (Teleport to body) stay unchanged -->
</div>
```

## SVG Nav Icons

Use these inline SVG icons for common page types. Each icon should be 20x20, stroke-based, currentColor.

```html
<!-- Dashboard / Overview -->
<svg class="nav-icon" viewBox="0 0 20 20" fill="none" stroke="currentColor" stroke-width="1.5">
  <rect x="2" y="2" width="7" height="7" rx="1.5"/>
  <rect x="11" y="2" width="7" height="4" rx="1.5"/>
  <rect x="2" y="11" width="7" height="7" rx="1.5"/>
  <rect x="11" y="8" width="7" height="10" rx="1.5"/>
</svg>

<!-- Inventory / Stock -->
<svg class="nav-icon" viewBox="0 0 20 20" fill="none" stroke="currentColor" stroke-width="1.5">
  <path d="M2 6l8-4 8 4-8 4-8-4z"/>
  <path d="M2 10l8 4 8-4"/>
  <path d="M2 14l8 4 8-4"/>
</svg>

<!-- Orders / Clipboard -->
<svg class="nav-icon" viewBox="0 0 20 20" fill="none" stroke="currentColor" stroke-width="1.5">
  <rect x="4" y="2" width="12" height="16" rx="2"/>
  <path d="M7 6h6M7 10h6M7 14h3"/>
</svg>

<!-- Finance / Spending -->
<svg class="nav-icon" viewBox="0 0 20 20" fill="none" stroke="currentColor" stroke-width="1.5">
  <path d="M10 2v16M6 6c0-1.5 1.5-2.5 4-2.5s4 1 4 2.5-1.5 2.5-4 3-4 1.5-4 3 1.5 2.5 4 2.5 4-1 4-2.5"/>
</svg>

<!-- Demand / Trending Up -->
<svg class="nav-icon" viewBox="0 0 20 20" fill="none" stroke="currentColor" stroke-width="1.5">
  <path d="M2 16l5-5 3 3 8-10"/>
  <path d="M14 4h4v4"/>
</svg>

<!-- Reports / Chart -->
<svg class="nav-icon" viewBox="0 0 20 20" fill="none" stroke="currentColor" stroke-width="1.5">
  <rect x="2" y="10" width="3" height="8" rx="1"/>
  <rect x="7" y="6" width="3" height="12" rx="1"/>
  <rect x="12" y="2" width="3" height="16" rx="1"/>
</svg>
```

Adapt icon selection to match the actual pages in the target application.

## CSS Patterns

### Layout Shell

```css
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
  transition: width var(--transition-slow);
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
```

### Sidebar Header (Logo)

```css
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
```

### Sidebar Navigation

```css
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

/* Active state — match with router-link-active or a manual active class */
.nav-item.router-link-exact-active,
.nav-item.active {
  color: var(--sidebar-text-active);
  background: var(--sidebar-active-bg);
}

/* Blue accent bar on left edge of active item */
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
```

### Sidebar Footer

```css
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
```

### Top Header

```css
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
```

### Main Content

```css
.main-content {
  flex: 1;
  padding: var(--content-padding);
  /* No max-width — content fills available space right of sidebar */
}
```

## Step-by-Step Transformation

Execute these steps in order. **Delegate all .vue file modifications to the `vue-expert` subagent.**

### Step 1: Inject Design Tokens

Add the CSS custom properties from the Design Tokens section into the `:root` block in App.vue's global (unscoped) `<style>` section. If no `:root` block exists, create one at the top of the style block.

### Step 2: Restructure App.vue Template

Replace the current vertical-flow layout (`<header class="top-nav">` + `<FilterBar>` + `<main class="main-content">`) with the two-column sidebar layout from the Target DOM Hierarchy section above.

Key changes:
- Wrap everything in `<div class="app-layout">`
- Move logo and nav links into `<aside class="sidebar">`
- Move LanguageSwitcher and ProfileMenu into `.sidebar-footer`
- Create `<div class="main-area">` containing the header, FilterBar, and main content
- Add a computed `pageTitle` based on the current route for the header
- Keep modal Teleports outside the layout wrapper (unchanged)

### Step 3: Add SVG Nav Icons

Add an inline SVG icon before each nav label in the sidebar. Use the icons from the SVG Nav Icons section, choosing the icon that best matches each page's purpose. Each icon should use `class="nav-icon"`.

### Step 4: Rewrite App.vue Layout CSS

Replace old layout classes (`.app`, `.top-nav`, `.nav-container`, `.nav-tabs`, `.logo`, `.subtitle`, `.main-content`) with the new CSS patterns from the CSS Patterns section.

**Preserve unchanged:** All shared utility styles (`.card`, `.badge`, `.stat-card`, `.page-header`, table styles, modal styles, status colors, etc.) must remain untouched.

### Step 5: Update FilterBar.vue

- Change sticky `top` from the old header height (e.g., `70px`) to `var(--header-height)` (60px)
- Remove any `max-width` constraint on the filter container — it should fill the content area
- Keep z-index at 90

### Step 6: Adapt ProfileMenu for Sidebar

The ProfileMenu dropdown currently opens downward-right. In the sidebar footer it should:
- Open **upward** (change `top: calc(100% + ...)` to `bottom: calc(100% + 0.5rem)`) or **rightward** (`left: calc(100% + 0.5rem)`)
- Be wider to fill sidebar width
- Keep z-index at 1000 (above sidebar's 200)
- Style the trigger button to match the dark sidebar theme (light text on transparent background)

### Step 7: Adapt LanguageSwitcher for Sidebar

Same adjustments as ProfileMenu:
- Dropdown opens upward or rightward
- Button styled for dark sidebar background
- z-index stays at 1000

### Step 8: Update Z-Index Strategy

Ensure this z-index hierarchy:

| Layer | z-index |
|-------|---------|
| Modals (Teleport to body) | 2000+ |
| Sidebar dropdown menus | 1000 |
| Sidebar | 200 |
| Top header | 100 |
| FilterBar | 90 |

### Step 9: Add Responsive Breakpoints

Add `@media` queries in App.vue's global styles:

**At 1024px and below:**
- Sidebar collapses to `var(--sidebar-collapsed-width)` (68px)
- Only icons visible, nav labels hidden
- Logo text hidden, only logo icon shown
- `.main-area` margin adjusts to collapsed width

**At 768px and below:**
- Sidebar hidden off-screen (`transform: translateX(-100%)`)
- Add a hamburger toggle button in the top header
- Sidebar overlays content when opened (with semi-transparent backdrop)
- `.main-area` margin-left becomes 0

### Step 10: Clean Up

- Remove old `.top-nav`, `.nav-tabs`, `.nav-container` styles entirely
- Remove the old `max-width: 1600px` from `.main-content` (or keep a sensible max if the views rely on it)
- Verify all old header-related classes are gone
- Ensure page-level `.page-header` styles in views still work (leave them as-is)

## Execution

**IMPORTANT:** Delegate all .vue file modifications to the `vue-expert` subagent. Provide the subagent with:

1. The complete target template for App.vue
2. The CSS design tokens to inject
3. The specific style changes for FilterBar.vue, ProfileMenu.vue, and LanguageSwitcher.vue
4. The z-index strategy
5. The responsive breakpoint rules

**Modification order:**
1. `App.vue` (template + global styles — this is the primary change)
2. `FilterBar.vue` (sticky position + width adjustments)
3. `ProfileMenu.vue` (dropdown direction + dark theme styling)
4. `LanguageSwitcher.vue` (dropdown direction + dark theme styling)

## Verification Checklist

After implementation, verify using Playwright MCP tools against `http://localhost:3000`:

- [ ] Sidebar renders on the left with dark background
- [ ] All 6 navigation links are visible with icons
- [ ] Active route is highlighted with blue accent bar
- [ ] Logo displays in sidebar header
- [ ] LanguageSwitcher and ProfileMenu work in sidebar footer
- [ ] Filter bar spans the content area (not the sidebar)
- [ ] Filter dropdowns still function correctly
- [ ] All page routes render content in the main area
- [ ] Modals appear above the sidebar (z-index correct)
- [ ] Profile/Language dropdowns open correctly in sidebar context
- [ ] At 1024px viewport: sidebar collapses to icons only
- [ ] At 768px viewport: sidebar hidden with hamburger toggle
