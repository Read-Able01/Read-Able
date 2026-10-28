# Assignment 2 – Requirements & Software Requirements Specification (SRS)

**Project Title:** ReadAble – An Intelligent & Accessible Digital Reading Platform  
**Standard Followed:** IEEE Std 830-1998 (Adapted SRS Format)  
**Document Constraint:** 2 Pages  

---

## 1. Introduction & Project Scope
The purpose of **ReadAble** is to deliver a web-based, accessibility-centric digital reading ecosystem that mitigates cognitive load and assists users through dense technical literature, academic papers, and digital books. The scope encompasses secure user management, document ingestion, adaptive visual rendering, synchronized text-to-speech (TTS), contextual AI assistance, and reading analytics.

---

## 2. Functional Requirements (FR)

| Requirement ID | Module / Feature | Detailed Specification | Priority |
| :--- | :--- | :--- | :--- |
| **FR-1.1** | User Registration | The system shall allow users to register an account using Full Name, unique Email or unique Phone Number, and Password. Passwords must be hashed via bcrypt (minimum 12 salt rounds). | High |
| **FR-1.2** | Dual-Identifier Authentication | The system shall enable users to log in seamlessly using either their Email or Phone number alongside their password. Normalized phone inputs shall ignore whitespace, dashes, and parentheses. | High |
| **FR-1.3** | Session Management | The system shall issue cryptographically signed JSON Web Tokens (JWT) upon successful authentication with a configurable lifespan (default: 7 days) and validate incoming bearer tokens. | High |
| **FR-2.1** | Document Ingestion | The system shall support uploading `.pdf` and `.txt` files up to 25 MB. The backend shall validate MIME types, generate unique S3/disk keys, and associate file records with the authenticated user. | High |
| **FR-2.2** | Document Library Management | The system shall allow authenticated users to browse, search, filter, download, and delete their stored documents from a personalized library dashboard. | High |
| **FR-3.1** | Dynamic Typography Engine | The system shall provide granular client-side controls to scale text font size ($100\%, 115\%, 130\%+$), alter line height ($1.6\text{--}1.9$), adjust letter tracking, and switch between standard and dyslexia-friendly typefaces. | High |
| **FR-3.2** | Theme & Contrast Toggling | The system shall provide three accessibility contrast modes: **Light Mode** (warm cream background), **Lowlight Mode** (soothing dark teal `#0E1815`), and **High Contrast Mode** without layout breaking. | High |
| **FR-4.1** | Text-to-Speech (TTS) | The system shall synthesize reading text into natural audio using the Web Speech API or backend TTS services, supporting play, pause, resume, and variable playback rates ($0.75\times\text{--}2.0\times$). | High |
| **FR-4.2** | Sentence Focus Highlighting | The system shall visually highlight the active sentence in real time as audio plays, keeping the reading focus centered and scrolling smoothly. | High |
| **FR-5.1** | Contextual AI Simplification | The system shall provide on-demand contextual simplification of selected dense paragraphs and explain technical jargon into plain language. | Medium |
| **FR-5.2** | Document Summarization | The system shall generate executive chapter-level or section-level summaries on demand to aid rapid document comprehension. | Medium |
| **FR-6.1** | Bookmarks & Annotation | The system shall allow users to highlight phrases, create sticky margin notes, and bookmark specific page numbers or percentage offsets. | Medium |
| **FR-6.2** | Reading Progress & History | The system shall track reading session duration, document completion percentage, reading streaks, and last-read timestamps. | Medium |

---

## 3. Non-Functional Requirements (NFR)

### 3.1 Performance Requirements
- **NFR-P1 (Response Time):** User authentication, preference updates, and document library queries shall respond in $< 300\text{ ms}$ under normal network conditions.
- **NFR-P2 (Document Ingestion Latency):** PDF parsing and text extraction for documents under 10 MB shall complete within $< 3.0\text{ seconds}$.
- **NFR-P3 (TTS Audio Latency):** Audio playback initialization for selected sentences shall begin within $< 250\text{ ms}$.

### 3.2 Usability & Accessibility Requirements
- **NFR-U1 (WCAG 2.1 Compliance):** The user interface shall conform strictly to **WCAG 2.1 Level AA** standards, ensuring contrast ratios of $\ge 4.5:1$ for regular text and $\ge 3:1$ for large text/interactive components.
- **NFR-U2 (Keyboard Operability):** All interactive features (reading controls, navigation, TTS triggers) must be $100\%$ operable via keyboard navigation (`Tab`, `Enter`, `Space`, and shortcut keys).
- **NFR-U3 (Responsive Presentation):** The interface shall be fully responsive across mobile ($360\text{px}$), tablet ($768\text{px}$), and desktop ($1024\text{px}+$ viewports) without text clipping or horizontal scrolling.

### 3.3 Security Requirements
- **NFR-S1 (Data Confidentiality):** All passwords shall be hashed using bcrypt with salt factor 12 before storage. Plaintext passwords shall never be logged or stored.
- **NFR-S2 (API Security):** All protected endpoints must require a valid JWT header (`Authorization: Bearer <token>`). Tokens must be signed with a minimum 256-bit secret key.
- **NFR-S3 (Injection Prevention):** Database access via Prisma ORM / parameterized queries must prevent SQL and NoSQL injection vulnerabilities. File uploads must be checked for payload safety and MIME spoofing.

### 3.4 Reliability, Availability & Maintainability
- **NFR-R1 (System Availability):** The application shall maintain $99.5\%$ operational uptime.
- **NFR-R2 (Graceful Degradation):** If AI services or remote network connectivity are unavailable, core reading, local font customization, and native browser TTS must continue operating uninterrupted.
- **NFR-M1 (Modularity):** The codebase shall enforce strict separation of concerns into Controller, Service, Route, and Middleware layers on the backend, and component-level isolation on the frontend.

---

## 4. Relevant SRS Section: System Interfaces & Operational Environment

```
+-----------------------------------------------------------------------------------+
|                            ReadAble System Architecture                           |
+-----------------------------------------------------------------------------------+
|  [Client Viewport: Browser]                                                       |
|    - React 19 + TypeScript + Vite + TanStack Router                               |
|    - Tailwind CSS + Lucide Icons + Web Speech Synthesis Audio Bridge              |
+-----------------------------------------+-----------------------------------------+
                                          | HTTP / REST (JSON + JWT via /api)
                                          v
+-----------------------------------------------------------------------------------+
|  [Application Server: Node.js & Express.js]                                       |
|    - Authentication Middleware (JWT & Bcrypt)                                     |
|    - Document Controller & Multer Ingestion Engine                                |
|    - Reading & Preferences Service                                                |
|    - Error & Validation Handler                                                   |
+-------------------+-----------------------------------+---------------------------+
                    |                                   |
                    v (Prisma ORM / MySQL / Postgres)   v (AWS SDK / File System)
+------------------------------------+   +------------------------------------------+
|  [Database Layer: SQL / MongoDB]   |   |  [Storage Layer: S3 / Local Disk]        |
|  - Users, Documents, Sessions,     |   |  - Uploaded Raw PDFs & Assets            |
|    Preferences, Bookmarks, Notes   |   +------------------------------------------+
+------------------------------------+
```

### 4.1 Target Users and Personas
1. **Academic Students / Researchers:** Need rapid comprehension of dense technical papers and ability to annotate key concepts.
2. **Readers with Neurodivergence (Dyslexia / ADHD):** Require customized font tracking, focus line rulers, high contrast, and voice-assisted reinforcement to maintain focus.
3. **Visually Impaired Users:** Rely on scalable UI typography ($130\%+$) and screen-reader/TTS narration.

### 4.2 System Constraints
- Server environment: Node.js runtime $\ge \text{v18.0}$.
- Client environment: Modern Chromium, Gecko, or WebKit browser with HTML5 and Web Speech API support.
- Database: PostgreSQL 16 / MySQL 8.0 (XAMPP/WAMP compatible) managed via Prisma ORM.
