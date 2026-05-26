# Collapsible Sidebar Design

**Date:** 2026-05-26
**Status:** Approved

## Context

The app currently uses a horizontal top navigation bar. As the number of nav items grew to 7, the bar became cramped and offered no responsive behaviour. This spec replaces it with a collapsible left sidebar that adapts automatically across screen sizes.

## Decisions

- **Collapse behaviour:** Responsive auto-collapse (not user-toggled). No state to persist.
- **FilterBar:** Stays as a sticky horizontal bar at the top of the content area (unchanged position, just sticky `top: 0` instead of `top: 70px`).
- **Profile menu + language switcher:** Move to the sidebar footer (bottom of sidebar).

## Breakpoints

| Range | Sidebar state | Width |
|---|---|---|
| ≥ 1024px | Full — icon + label | 240px |
| 768–1023px | Icons only — icon + tooltip | 64px |
| < 768px | Hidden — hamburger opens overlay | 0px |

## Sidebar Anatomy (top → bottom)

1. **Logo / company name** — "Catalyst Components" + subtitle. Hidden when icons-only.
2. **Nav items** (7) — icon + label. Label hidden when icons-only. Active item: blue background + bright icon.
3. **Footer** — `LanguageSwitcher` and `ProfileMenu` avatar stacked vertically. Labels hidden when icons-only.

### Nav items and icons (inline SVG, no external library)

| Route | Label | Icon |
|---|---|---|
| `/` | Overview | Grid / dashboard squares |
| `/inventory` | Inventory | Box / cube |
| `/orders` | Orders | List / clipboard |
| `/spending` | Finance | Bar chart |
| `/demand` | Demand Forecast | Trend arrow |
| `/reports` | Reports | Document |
| `/restocking` | Restocking | Refresh / cycle arrows |

### Icons-only tooltips

In the 768–1023px range, hovering an icon shows a CSS tooltip with the nav item label. Implemented with a CSS `::after` pseudo-element tooltip on the nav item — no JS required. The tooltip appears to the right of the icon.

## Mobile overlay (< 768px)

- A hamburger button (`☰`) appears in the top-left of the content area.
- Tapping it slides the full 240px sidebar in from the left.
- A semi-transparent backdrop covers the content; tapping it closes the sidebar.
- No persistent sidebar rendered at this size.

## Layout structure

```
.app (flex-direction: row)
├── Sidebar.vue (fixed, full height, left: 0)
│   ├── .sidebar-logo
│   ├── .sidebar-nav (7 router-links)
│   └── .sidebar-footer (LanguageSwitcher + ProfileMenu)
└── .main-column (flex: 1, margin-left: var(--sidebar-width))
    ├── FilterBar.vue (sticky, top: 0)
    └── .main-content (router-view)
```

CSS custom property `--sidebar-width` drives all margins:
- `240px` at ≥ 1024px
- `64px` at 768–1023px
- `0` at < 768px (sidebar becomes overlay)

Smooth width transition: `transition: width 0.2s ease, margin-left 0.2s ease`.

## Files changed

| File | Change |
|---|---|
| `client/src/components/Sidebar.vue` | **New** — full sidebar component |
| `client/src/App.vue` | Remove `.top-nav` header; add `<Sidebar>`; change `.app` to `flex-direction: row`; update main layout |
| `client/src/components/FilterBar.vue` | Change `top: 70px` → `top: 0` in sticky positioning |

## What does NOT change

- `FilterBar.vue` internals — filters, dropdowns, reset button unchanged.
- All view components (`Dashboard.vue`, `Orders.vue`, etc.) — zero changes needed.
- `useFilters.js`, `useI18n.js`, `useAuth.js` composables — unchanged.
- Route structure — unchanged.

## Active state

- Background: `#1d4ed8` (blue-700)
- Icon color: `#93c5fd` (blue-300)
- Text color: `#e0f2fe` (blue-50)
- Non-active hover: subtle `#334155` background

## Design tokens

```css
--sidebar-full-width: 240px;
--sidebar-icon-width: 64px;
--sidebar-bg: #1e293b;
--sidebar-border: #334155;
--sidebar-active-bg: #1d4ed8;
--sidebar-active-icon: #93c5fd;
--sidebar-text: #94a3b8;
--sidebar-text-active: #e0f2fe;
```
