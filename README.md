# easeui-component-library
A React + TypeScript UI component library  - buttons, cards, forms, carousels, and more, each with live demos and full API references.
# EaseUI

A React + TypeScript UI component library with a built-in documentation
site. Every component has a live interactive preview, a toggleable code
snippet, and a full props/API reference table - similar in spirit to how
libraries like shadcn/ui or Radix present their components.

---

## Table of Contents

- [Overview](#overview)
- [Tech Stack](#tech-stack)
- [Getting Started](#getting-started)
- [Project Structure](#project-structure)
- [The Component Pattern](#the-component-pattern)
- [Components](#components)
- [Dark Mode](#dark-mode)
- [Bugs Found and Fixed](#bugs-found-and-fixed)
- [Available Scripts](#available-scripts)

---

## Overview

EaseUI ships two things at once:

1. **A component library** - a set of reusable, styled, TypeScript-typed
   React components (`Button`, `Card`, `Modal`, `Input`, `Navbar`,
   `Tooltip`, `Color Picker`, `Form`, `3D Effect Card`, `Carousel`,
   `Layout`) that can be imported and used in any React project.
2. **A documentation site** - a sidebar-navigable app (built with the
   same components) where every component gets its own page showing a
   live demo, the exact code to reproduce it, and its full prop table.

The app supports light and dark mode, toggled from the navbar and
persisted across page reloads.

---

## Tech Stack

| Layer | Technology |
|---|---|
| UI library | React 19 |
| Language | TypeScript |
| Build tool | Vite (with library-mode output for `dist/`) |
| Routing | React Router v6 |
| Global state | Redux Toolkit (theme mode only) |
| Styling | Tailwind CSS v4 |
| Animation | GSAP |
| Variant styling | class-variance-authority (cva) |
| Icons | lucide-react |
| Browser APIs used | `localStorage`, `SpeechSynthesisUtterance` (Web Speech API) |

---

## Getting Started

```bash
# install dependencies
npm install

# start the local dev server (Vite, with hot module reload)
npm run dev
# -> opens at http://localhost:5173

# type-check + build for production
npm run build
# -> outputs to dist/ (both ES module and UMD bundles, since this
#    is built in library mode - see vite.config.ts)

# preview the production build locally
npm run preview
```

No environment variables or backend services are required - everything
runs entirely client-side, including the Form's "save messages" feature
(uses `localStorage`) and the text-to-speech feature (uses the browser's
built-in speech engine).

---

## Project Structure

```
src/
├── components/              # every reusable UI component
│   ├── Button/
│   ├── Card/
│   ├── Modal/
│   ├── Input/
│   ├── Navbar/
│   ├── Tooltip/
│   ├── ColorPicker/
│   ├── Form/
│   ├── Card3D/
│   ├── Carousel/
│   ├── Layout/
│   ├── Personal/            # internal helpers (CodeBlock, PropsTable)
│   └── index.ts             # re-exports everything for short imports
│
├── pages/
│   ├── components/          # one demo/docs page per component
│   └── ComponentsDemo.tsx   # shared preview-box + "Show Code" wrapper
│
├── layouts/
│   ├── HomeLayout.tsx       # navbar + outlet, used for the landing page
│   └── ComponentLayout.tsx  # sidebar + outlet, used for component pages
│
├── router/
│   └── AppRouter.tsx        # all route definitions
│
├── features/
│   └── ThemeSlice.tsx       # Redux slice for light/dark mode
│
├── store/
│   └── Store.tsx            # Redux store configuration
│
├── libs/
│   ├── utils.ts             # cn() - merges Tailwind classes safely
│   └── animations/          # reusable GSAP animation presets
│
└── index.css                 # Tailwind import + light/dark CSS variables
```

---

## The Component Pattern

Every component in this library is built from the same three pieces,
which makes the codebase predictable to extend:

**1. The component itself** - `components/<Name>/<Name>.tsx`

```tsx
// example shape, not copy-pasted from a real file
const myVariants = cva("base-tailwind-classes-here", {
  variants: {
    variant: { dark: "...", light: "..." },
    size: { sm: "...", md: "...", lg: "..." },
  },
  defaultVariants: { variant: "dark", size: "md" },
});

const MyComponent = ({ variant, size, className, ...props }: MyComponentProps) => (
  <div className={cn(myVariants({ variant, size }), className)} {...props} />
);
```

`cva()` defines style "variants" in a type-safe way - TypeScript knows
exactly which `variant`/`size` values are valid because of it.
`cn()` (from `libs/utils.ts`) merges those generated classes with
whatever custom `className` the consumer of the component passes in,
resolving any Tailwind class conflicts correctly.

**2. The demo page** - `pages/components/<Name>Page.tsx`

Wraps a live example inside `<ComponentDemo>` (gives it the preview box
+ "Show Code" toggle) and lists every prop in a `<PropsTable>` below it.

**3. Wiring** - three places every new component gets registered:
- Exported from `src/components/index.ts`
- Given a route in `src/router/AppRouter.tsx`
- Linked in the sidebar list inside `src/layouts/ComponentLayout.tsx`

---

## Components

| Component | What it does |
|---|---|
| **Button** | Styled button with multiple variants (`primary`, `secondary`, `destructive`, `ghost`, `link`, `outline`) and sizes |
| **Card** | Container with hover animations (lift, scale, rotate presets from `libs/animations`) |
| **Modal** | Overlay dialog |
| **Input** | Styled text input, supports labels, placeholder, sizes |
| **Navbar** | Top navigation bar with the light/dark theme toggle |
| **Tooltip** | Hover-triggered label with `dark`/`light` variants, `top`/`bottom` positioning, and a GSAP scale+fade entrance |
| **Color Picker** | Dropdown with preset swatches, a native color input, and a hex text field; closes on click-outside; has a playful idle nudge that wiggles, shows a "Pick me, pick me!" speech bubble, and speaks it out loud via the Web Speech API (configurable/disable-able) |
| **Form** | Config-driven form (fields described as data, not JSX) supporting `text`/`email`/`password`/`textarea`/`select`/`checkbox` with per-field validation rules |
| **3D Effect Card** | Pointer-tracked tilt with a cursor-following holographic shine, and an optional click-to-flip that reveals back-face content |
| **Carousel** | Sliding image carousel with arrows, dots, auto-play, and a visible auto-play progress bar |
| **Layout** | Content-arrangement helper with five modes: `stack`, `row`, `grid`, `auto-grid` (responsive with zero media queries), `masonry` (Pinterest-style via CSS multi-column), and `sidebar` |

### Form's message store (a small case study)

The Form demo page shows a realistic "save and read" flow without a
backend:

- On submit, the message is written to `localStorage` (no cap - stores
  everything ever submitted)
- On page load, `useState`'s lazy initializer reads straight from
  `localStorage`, so saved messages appear immediately, automatically -
  no manual "load" step needed
- Clicking any saved message uses the browser's built-in
  `SpeechSynthesisUtterance` API to read it out loud (click again to
  stop; starting a new one cancels whatever was playing)

---

## Dark Mode

Theme state lives in Redux (`features/ThemeSlice.tsx`). Toggling it:

1. Flips `mode` between `"light"` / `"dark"` in the Redux store
2. Sets `data-theme="dark"` (or removes it) on the `<html>` element
3. Persists the choice to `localStorage` so it survives a refresh

For this to actually affect styling, `index.css` registers a custom
Tailwind v4 variant:

```css
@custom-variant dark (&:where([data-theme="dark"], [data-theme="dark"] *));
```

Without this line, Tailwind's `dark:` utility classes fall back to
following the operating system's `prefers-color-scheme` setting instead
of the app's own toggle - which was actually a real bug found and fixed
in this project (see below).

---

## Bugs Found and Fixed

This codebase was extended from an existing, partially-built starting
point. Along the way, several real bugs were found and fixed - not
just cosmetic issues, but ones that silently broke functionality
without throwing any errors.

### 1. Sidebar linked to pages that didn't exist - the main bug
**Cause:** `ComponentLayout.tsx`'s sidebar already listed "Carousel",
"Tooltip", and "Layout" as nav links, but `AppRouter.tsx` had no
matching routes registered for any of them. Clicking them changed the
URL correctly but then hit React Router's default error boundary:
`Unexpected Application Error! 404 Not Found`.
**Fix:** Built all three components and their demo pages from scratch,
following the existing pattern, and registered their routes.

### 2. Dark mode text stayed unreadable, but only on component pages
**Important nuance:** the main light/dark switching (page background
and text color) already worked correctly across the app, driven by
plain CSS custom properties in `index.css` (`--bg-color`, `--text-color`)
that respond to the `data-theme` attribute Redux sets. This part was
never broken.
**Cause:** `ComponentLayout.tsx` (used only on `/components/*` pages)
had `text-gray-900` hardcoded, which overrode that CSS-variable-driven
color specifically in the sidebar and content area of the Components
section - nowhere else.
**Fix:** Switched to `text-[var(--text-color)]` and added `dark:`
classes to the sidebar background/border. Also added a Tailwind v4
`@custom-variant dark (...)` rule so any `dark:`-prefixed utility class
elsewhere in the app responds to the in-app toggle instead of only the
OS-level `prefers-color-scheme` setting.

### 3. Carousel went blank after sliding (caught while building it)
**Note:** the Carousel didn't exist in the original codebase at all -
this wasn't a bug found in old code, it's a mistake made while writing
its animation logic and caught by testing it before calling it done.
**Cause:** A math bug, not a styling issue. GSAP's `xPercent` moves an
element relative to its **own** width - and the carousel's track is
`slides.length * 100%` wide since every slide sits side by side. The
first version of the code moved the track by `-100 * index` percent,
wildly overshooting past the real slides into empty space for any
carousel with more than one slide.
**Fix:**
```js
gsap.to(track, { xPercent: (-100 * index) / slides.length })
```

### 4. Duplicate "Email" label on the Input demo page
**Cause:** A copy-paste leftover - two input fields were both labeled
"Email" instead of the second one showing a different field to
demonstrate size variation.
**Fix:** Changed the second field to "Phone Number".

---

## Available Scripts

| Command | What it does |
|---|---|
| `npm run dev` | Starts the Vite dev server with hot reload |
| `npm run build` | Type-checks and builds the production library bundle |
| `npm run preview` | Serves the production build locally |
| `npm run lint` | Runs ESLint across the project |

---

## Code Quality

`npm run lint` passes clean with **0 errors, 0 warnings**. This wasn't
always the case - the ESLint config file existed but its dependencies
were never installed, so linting silently never ran. Once wired up, it
surfaced 25 real issues: unsafe `any` types, a React hook being called
conditionally (which breaks React's rules of hooks), a component prop
that was declared but never actually used, and a handful of
Fast-Refresh/empty-type warnings. All fixed - see `CHANGES.md` for the
full list.

`tsconfig.app.json` also had an invalid `ignoreDeprecations` value that
was silently preventing `tsc` from ever type-checking the actual source
files - fixed, and `npx tsc --noEmit -p tsconfig.app.json` now passes
clean too.





