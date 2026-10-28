# Assignment 6: Design the UI/Frontend

**Project Title:** ReadAble – An Intelligent & Accessible Digital Reading Platform  
**Frontend Framework:** React 19 + TypeScript + Vite + TanStack Router  
**Styling & Design System:** Tailwind CSS v4 + Dynamic CSS Custom Properties (`oklch` Color Palette) + Lucide Icons  

---

## 1. UI/UX Design Philosophy & Cognitive Accessibility Principles

The user interface of **ReadAble** is designed from the ground up on the principles of **Universal Design** and **Cognitive Accessibility**. Traditional reading tools assume standard neurological and visual processing; ReadAble adapts directly to the reader's unique cognitive profile.

### 1.1 Cognitive Load Reduction Strategies
1. **Minimizing Visual Crowding:** Standard dense layouts trigger visual crowding for readers with dyslexia. ReadAble introduces adjustable letter tracking (`--read-tracking: 0.02em - 0.06em`) and line heights (`--read-leading: 1.6 - 2.2`).
2. **Focus Centering & Visual Anchors:** When audio playback is active, the active sentence receives an ambient soft highlight while surrounding sentences gently fade to 60% opacity. This prevents involuntary saccades (wandering gaze).
3. **Progressive Disclosure:** Advanced settings (AI model selection, prompt customization, export options) are tucked into collapsible drawers, keeping the primary document reading canvas uncluttered and peaceful.

### 1.2 WCAG 2.1 Compliance Matrix

| WCAG Guideline | ReadAble Implementation | Standard Met |
| :--- | :--- | :--- |
| **1.4.3 Contrast (Minimum)** | Regular text contrast exceeds $7.2:1$ in Light Cream mode and $11.5:1$ in Lowlight Dark mode. | **WCAG AAA** |
| **1.4.4 Resize Text** | Dynamic font scaling up to $200\%$ using `calc(16px * var(--read-scale))` without text clipping or horizontal scrolling. | **WCAG AA** |
| **1.4.12 Text Spacing** | Users can independently adjust line height, letter spacing, and word spacing via the `ReadingPreferences` control panel. | **WCAG AAA** |
| **2.1.1 Keyboard Navigation** | Every interactive element, toolbar button, and slider is fully navigable with `Tab`, `Arrow keys`, and `Spacebar`. | **WCAG AA** |
| **2.4.7 Focus Visible** | High-visibility $2\text{px}$ ring with contrasting brand outline (`--ring: #3DD9AE`) on all focused elements. | **WCAG AA** |

---

## 2. Design System Tokens & Style Guide

The styling is defined declaratively in `frontend/src/index.css` using modern CSS variables and color models.

### 2.1 Curated Color Palettes
- **Light Theme (Soothing Cream):**
  - Background: `oklch(0.976 0.012 85)` (warm parchment that eliminates glare)
  - Foreground: `oklch(0.22 0.02 160)` (deep charcoal-forest green)
  - Primary Brand: `oklch(0.42 0.078 178)` (calming dark teal)
  - Accent / Highlighting: `oklch(0.9 0.055 172)` (soft mint wash)
- **Lowlight Dark Theme (`.lowlight`):**
  - Background: `#0E1815` (deep ambient slate)
  - Foreground: `#EAF7F1` (crisp mint-tinted off-white)
  - Primary Brand: `#3DD9AE` (neon emerald focus ring)
  - Surface Card: `#152420` (elevated dark container)

### 2.2 Dynamic CSS Typography Engine
```css
:root {
  --read-scale: 1.0;
  --read-leading: 1.6;
  --read-tracking: 0em;
  --read-word-spacing: normal;
}

html {
  font-size: calc(16px * var(--read-scale, 1));
  scroll-behavior: smooth;
}

body {
  line-height: var(--read-leading, 1.6);
  letter-spacing: var(--read-tracking, 0em);
  word-spacing: var(--read-word-spacing, normal);
}
```

---

## 3. Screen Layouts & Detailed ASCII Wireframes

### 3.1 Homepage / Landing View (`/`)
```
+------------------------------------------------------------------------------------+
|  [Logo] ReadAble           [Features]  [Accessibility]  [Help]     [Sign In / Demo] |
+------------------------------------------------------------------------------------+
|                                                                                    |
|                  Read. Understand. Listen. Adapt.                                  |
|         Transform dense academic PDFs into effortless knowledge.                    |
|                                                                                    |
|       [ Upload Document Free -> ]          [ Explore Live Interactive Demo ]       |
|                                                                                    |
|  +------------------------------------------------------------------------------+  |
|  | Interactive Accessibility Demo Widget:                                       |  |
|  | Text Scale: [  -  ] [ 115% ] [  +  ]   | Theme: (x) Cream  ( ) Dark  ( ) HC  |  |
|  | "Doing well with money has a little to do with how smart you are and a lot   |  |
|  | to do with how you behave. ReadAble simplifies complex academic reading."    |  |
|  +------------------------------------------------------------------------------+  |
|                                                                                    |
|  [Feature Cards: 1. Smart Typography | 2. Synchronized Audio | 3. In-Situ AI]      |
+------------------------------------------------------------------------------------+
|  (C) 2026 ReadAble Platform | WCAG 2.1 AAA Compliant | Open Source Software         |
+------------------------------------------------------------------------------------+
```

---

### 3.2 Authentication & Onboarding View (`/login`)
```
+------------------------------------------------------------------------------------+
|  [Logo] ReadAble                                                [Back to Home]     |
+------------------------------------------------------------------------------------+
|                                                                                    |
|                      +--------------------------------------+                      |
|                      |  [ Sign In ]      |    [ Register ]  |                      |
|                      +--------------------------------------+                      |
|                      | Welcome back to ReadAble             |                      |
|                      |                                      |                      |
|                      | Identifier (Email or Phone Number)   |                      |
|                      | [ rohit@readable.org or 9876543210 ] |                      |
|                      |                                      |                      |
|                      | Password                             |                      |
|                      | [ ****************           (eye) ] |                      |
|                      |                                      |                      |
|                      | [X] Remember my reading settings     |                      |
|                      |                                      |                      |
|                      | [      Sign In to Dashboard ->     ] |                      |
|                      |                                      |                      |
|                      | Accessibility Preview:               |                      |
|                      | Scale: [ 100% =====O======= 130% ]   |                      |
|                      | [X] Comfortable Line Height (1.9x)   |                      |
|                      +--------------------------------------+                      |
|                                                                                    |
+------------------------------------------------------------------------------------+
```

---

### 3.3 The Core Reading Dashboard (`/dashboard`)
```
+--------------------------------------------------------------------------------------------------------+
| [=] ReadAble   | The Psychology of Money.pdf [75% Read]   | Scale [115%] Theme [Lowlight] | [User Menu] |
+----------------+--------------------------------------------------------------+-------------------------+
| NAVIGATION     | DOCUMENT READING VIEWPORT                                    | AI & ASSISTANCE DRAWER  |
|                |                                                              |                         |
| [*] Dashboard  | Chapter 2: Confounding Compounding                           | [Sparkles] AI Explainer |
| [ ] My Library |                                                              |                         |
| [ ] Upload Doc | "Doing well with money has a little to do with how smart    | Selected Sentence:      |
| [ ] Read Aloud | you are and a lot to do with how you behave."                 | "Confounding            |
| [ ] Bookmarks  |                                                              | compounding generates   |
| [ ] History    | >>> [Active Voice Playback Highlighted in Soft Mint Box] <<< | nonlinear returns..."   |
| [ ] Notes      | "And behavior is hard to teach, even to really smart         |                         |
| [ ] Settings   | people. Ordinary folks with no financial education can be    | Plain Language:         |
|                | wealthy if they have behavioral skills."                     | "Compounding means you  |
|                |                                                              | earn interest on top of |
|                |                                                              | previous interest. It   |
| AUDIO TOOLBAR  |                                                              | grows faster than our   |
| [|<] [Play] [>|]|                                                              | brain expects."         |
| Rate: [ 1.0x ] |                                                              |                         |
| Progress: 75%  |                                                              | [Copy] [Save to Notes]  |
+----------------+--------------------------------------------------------------+-------------------------+
```

---

## 4. Frontend Component Hierarchy & State Architecture

### 4.1 Component Tree Diagram
```
App Root (__root.tsx)
 ├── Navbar.tsx (Global accessible header, theme toggler, route links)
 ├── Route Component Router (TanStack Router)
 │    ├── / (index.tsx: Homepage, Hero, Demo, Feature Grid)
 │    ├── /login (login.tsx: Auth forms, validation, scale preview)
 │    ├── /dashboard (dashboard.tsx: Auth guard redirect)
 │    │    └── ReadAbleDashboard.tsx (Main reader state manager)
 │    │         ├── SidebarNav (Nav items, collapse state)
 │    │         ├── ReadingViewport (Document text stream, sentence focus)
 │    │         ├── ReadingPreferences.tsx (Typography sliders, themes)
 │    │         ├── AudioController (TTS synthesis, playback speed)
 │    │         └── AIAssistantDrawer (Contextual simplification)
 │    └── /help (help.tsx: User manual, keyboard shortcuts)
 └── Footer.tsx (Site credits, accessibility badges)
```

### 4.2 State Hydration & API Integration (`src/lib/api.ts`)
- The authentication state initializes by inspecting `localStorage.getItem("readable_token")`.
- When an authenticated user logs in, `authApi.me()` fetches their latest profile and persisted `reading_preferences`.
- Sliders on the UI bind two-way with local state and apply direct DOM modifications through `document.documentElement.style.setProperty("--read-scale", scale)`.
- Updates to preferences are debounced ($300\text{ ms}$) and synchronized via `PATCH /api/users/preferences`.
