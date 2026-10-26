# ReadAble: An Intelligent & Accessible Digital Reading Platform
## Master Software Engineering Project Report (Assignments 1 – 9 Compiled)

**Course:** Software Engineering & Systems Development  
**Academic Year:** 2024–2026  
**Repository:** [https://github.com/RohitRabade2006/Read-Able.git](https://github.com/RohitRabade2006/Read-Able.git)  
**Document Compilation:** Assignment 10 – Final Master Synthesis  

---

## Executive Summary & Abstract
The proliferation of digital documentation in academia and enterprise has created an unintentional barrier for millions of students and professionals. Conventional digital document viewers (e.g., standard PDF readers) function strictly as passive visual renderers. They are indifferent to the reader's cognitive capacity, sensory limitations, and neurodivergent processing needs.

**ReadAble** is an open-source, full-stack, accessibility-centric digital reading ecosystem designed to transform passive reading into an active, multi-modal comprehension experience. Built on the core paradigm:
$$\mathbf{Read \longrightarrow Understand \longrightarrow Listen \longrightarrow Adapt}$$

ReadAble integrates dynamic typography scaling, custom letter tracking and line spacing, WCAG 2.1 AAA high-contrast themes, synchronized sentence-level text-to-speech (TTS) audio narration, and contextual in-situ generative AI assistance for vocabulary lookup and paragraph simplification.

This master document compiles and integrates all nine technical milestones of the ReadAble system—encompassing problem feasibility, formal requirements (SRS), UML system modeling, data flow and entity-relationship diagrams, normalized database design implemented on WAMP/XAMPP and MongoDB, responsive frontend engineering, modular layered backend API development, three-tier end-to-end integration, iterative waterfall SDLC execution, comprehensive testing matrices, real-world incident post-mortems, automated GitHub Actions CI/CD workflows, and long-term maintenance protocols.

---

## Table of Contents
1. [Chapter 1: Problem Statement & Feasibility Analysis](#chapter-1-problem-statement--feasibility-analysis)
2. [Chapter 2: Software Requirements Specification (SRS)](#chapter-2-software-requirements-specification-srs)
3. [Chapter 3: System Modeling & UML Diagrams](#chapter-3-system-modeling--uml-diagrams)
4. [Chapter 4: Data Modeling, DFDs, State Machines & ER Diagram](#chapter-4-data-modeling-dfds-state-machines--er-diagram)
5. [Chapter 5: Database Design, Normalization & WAMP/XAMPP Implementation](#chapter-5-database-design-normalization--wampxampp-implementation)
6. [Chapter 6: UI/Frontend Architecture & Accessibility Design](#chapter-6-uifrontend-architecture--accessibility-design)
7. [Chapter 7: Backend Architecture & RESTful API Catalog](#chapter-7-backend-architecture--restful-api-catalog)
8. [Chapter 8: Three-Tier System Integration & Network Flow](#chapter-8-three-tier-system-integration--network-flow)
9. [Chapter 9: SDLC Methodology, Testing, Debugging, CI/CD & Maintenance](#chapter-9-sdlc-methodology-testing-debugging-cicd--maintenance)
10. [Chapter 10: Conclusion, Future Roadmap & References](#chapter-10-conclusion-future-roadmap--references)

---

## Chapter 1: Problem Statement & Feasibility Analysis

### 1.1 Problem Statement
Readers of academic, scientific, and technical PDF documents encounter substantial cognitive fatigue, visual crowding, and comprehension bottlenecks. Standard PDF viewers fail to provide:
- Adaptive typography (line height, letter tracking, dyslexia-friendly fonts).
- Synchronized multi-modal reinforcement (speech audio matching visual sentence focus).
- In-situ technical jargon clarification and paragraph simplification.
- Persistent cross-device personalization of accessibility preferences.

### 1.2 Feasibility Study Summary
- **Technical Feasibility:** Highly feasible using React 19, TypeScript, Vite, Node.js/Express, Prisma ORM, and MySQL/PostgreSQL.
- **Operational Feasibility:** Zero-install web application operating across desktop, tablet, and mobile browsers.
- **Economic Feasibility:** Built entirely on open-source frameworks; operable locally on WAMP/XAMPP and Docker with minimal infrastructure overhead.
- **Schedule Feasibility:** Completed across 10 disciplined sprint milestones within an academic semester.
- **Legal/Ethical Compliance:** Conforms with WCAG 2.1 Level AA/AAA and GDPR tokenized privacy standards.

---

## Chapter 2: Software Requirements Specification (SRS)

### 2.1 Functional Requirements Summary
- **User Management (FR-1):** Registration, dual-identifier login (Email or Phone), bcrypt password hashing (12 salt rounds), stateless 7-day JWT session issuance.
- **Document Ingestion (FR-2):** Multipart upload of `.pdf` and `.txt` files up to $25\text{MB}$, MIME validation, unique S3/disk key indexing, library management.
- **Adaptive UI Engine (FR-3):** Client-side typography scaling ($100\%\text{--}130\%+$), letter spacing, line height ($1.6\text{--}1.9$), and themes (Light Cream, Lowlight Dark `#0E1815`, High Contrast).
- **Assisted Audio & Focus (FR-4):** Web Speech API synthesis, adjustable speed ($0.75\times\text{--}2.0\times$), real-time sentence-level DOM highlighting.
- **Comprehension AI (FR-5):** Contextual simplification of dense passages, executive document summarization, vocabulary definitions.
- **Telemetry & Notes (FR-6):** Bookmarks, margin annotations, reading duration telemetry, reading streak tracking.

### 2.2 Non-Functional Requirements Summary
- **Performance:** UI response $< 300\text{ ms}$, document ingestion $< 3.0\text{ s}$, audio start $< 250\text{ ms}$.
- **Accessibility:** 100% keyboard operability, contrast ratio $\ge 7:1$, dynamic font scaling without 2D scrolling.
- **Security:** Bcrypt salted password storage, JWT bearer header validation, parameterized database queries.
- **Reliability:** 99.5% uptime target, graceful degradation to offline reading.

---

## Chapter 3: System Modeling & UML Diagrams

### 3.1 Actors & Relationships
- **Primary Reader:** Interacts with reading viewport, adjusts accessibility sliders, invokes TTS, creates notes.
- **Accessibility User:** Relies on high-contrast palettes, font tracking, and synchronized voice-text focus.
- **System Administrator:** Monitors database health, executes schema migrations, manages cloud storage.
- **External AI Provider:** Generates simplifications and definitions via REST API.

### 3.2 Diagram Highlights
- **Use Case Diagram:** Defines 12 use cases including `<<include>>` relationships (e.g., Upload includes Validate File; Listen TTS includes Highlight Sentence) and `<<extend>>` features (Read Document extended by AI Assistance and Bookmarks).
- **Class Diagram:** Full object-oriented design featuring `User`, `ReadingPreference`, `Document`, `DocumentChunk`, `Bookmark`, `Annotation`, `ReadingSession`, `AudioPlayerService`, and `AIService`.
- **Sequence Diagram:** Detailed lifecycle trace from file upload, MIME inspection, database commit, to viewport text rendering and TTS audio trigger.
- **Activity Diagram:** Decision tree tracking reader interactions, preference reflow, and telemetry logging.
- *Vector interchange models provided in `Assignments/Assignment_3_UML_Diagrams.drawio`.*

---

## Chapter 4: Data Modeling, DFDs, State Machines & ER Diagram

### 4.1 Data Flow Diagrams
- **Level 0 (Context):** Maps information flow between Reader, External Storage, AI Engine, and the ReadAble System.
- **Level 1 (Subsystems):** Decomposes system into Processes 1.0 through 6.0 and Data Stores D1 through D5.
- **Level 2 (Deep Dive):** Details the active reading loop, sentence boundary parser, and DOM focus highlighter.

### 4.2 State Machine Models
- **Document Lifecycle:** `Draft -> Validating -> Processing -> Chunked -> Ready -> Active Reading -> Archived -> Deleted`.
- **Session State Machine:** `Initializing -> Visual Reading <-> Playing TTS <-> Inspecting AI -> Terminated`.

### 4.3 Entity-Relationship Architecture
- 8 fully normalized tables with primary keys, foreign keys, cascading deletion rules, and candidate unique indexes (`email`, `phone`, `s3Key`).
- *Vector interchange models provided in `Assignments/Assignment_4_Data_Modelling.drawio`.*

---

## Chapter 5: Database Design, Normalization & WAMP/XAMPP Implementation

### 5.1 Relational Schema & Normalization
- **1NF:** Atomic attributes; repeating chunks and bookmarks factored into distinct tables.
- **2NF:** No partial dependencies; all non-key attributes depend on atomic surrogate primary keys (`id`).
- **3NF & BCNF:** Transitive preferences factored into `reading_preferences`; candidate keys (`email`, `phone`) act as deterministic determinants.

### 5.2 WAMP / XAMPP Implementation Deliverable
- Executable script `Assignments/readable_db_xampp.sql` contains complete DDL statements, constraints, sample seed data, and analytical validation queries.
- Verified in phpMyAdmin under `http://localhost/phpmyadmin` on MariaDB/MySQL port 3306.
- Comprehensive Mongoose NoSQL schemas provided in `Assignments/readable_mongodb_schemas.js`.

---

## Chapter 6: UI/Frontend Architecture & Accessibility Design

### 6.1 Design Tokens & WCAG 2.1 Conformance
- Dynamic CSS Custom Properties: `--read-scale`, `--read-leading`, `--read-tracking`, `--read-word-spacing`.
- Theme Color Schemes: Soothing Cream (`oklch(0.976 0.012 85)`), Lowlight Dark (`#0E1815`), and High Contrast.
- Typography: Plus Jakarta Sans with fallbacks to OpenDyslexic.

### 6.2 Screen Architecture & Wireframes
- **Landing Page (`/`):** Hero showcase with live interactive accessibility demonstration slider.
- **Auth Page (`/login`):** Split card with dual-identifier (Email/Phone) toggle and real-time typography preview.
- **Dashboard (`/dashboard`):** 3-column layout featuring collapsible navigation drawer, active reading viewport with sentence focus overlay, and collapsible AI assistance drawer.
- **User Manual (`/help`):** Comprehensive guide and keyboard shortcut catalog.

---

## Chapter 7: Backend Architecture & RESTful API Catalog

### 7.1 Layered Architecture Pattern
- Presentation Layer: Express router, CORS, JSON parsers, authentication guard middleware.
- Business Logic Layer: Isolated service modules (`auth.service.js`, `document.service.js`, `reading.service.js`).
- Data Access Layer: Prisma ORM abstraction mapping to relational databases.
- Centralized Error Handling: Custom operational errors (`ValidationError`, `UnauthorizedError`, `ConflictError`) handled through unified Express error middleware.

### 7.2 REST API Catalog
Ten core production endpoints documented with HTTP verbs, request payloads, response schemas, and status codes.

---

## Chapter 8: Three-Tier System Integration & Network Flow

### 8.1 Integration Architecture
- Frontend: React 19 on `http://localhost:5173`.
- Gateway: Vite dev server reverse proxy mapping `/api/*` to backend port 3001.
- Backend: Express.js server on `http://localhost:3001`.
- Database: PostgreSQL on port 5433 / WAMP/XAMPP MySQL on port 3306.

### 8.2 Interceptors & Workflows
- Automated Axios request interceptor injects JWT tokens from `localStorage`.
- End-to-end trace of User Registration, Document Upload, and Live Accessibility Synchronization.
- Verification matrix confirms all integration test scenarios pass with zero CORS violations.

---

## Chapter 9: SDLC Methodology, Testing, Debugging, CI/CD & Maintenance

### 9.1 SDLC Model Justification
Selected the **Iterative Waterfall (Hybrid)** model to balance strict upfront accessibility and database design standards with agile iterative tuning of UI and AI components.

### 9.2 Comprehensive Testing Matrix
Twelve formal test cases (TC-01 through TC-12) executed across authentication, phone normalization, route protection, file upload validation, accessibility scaling, and database cascading.

### 9.3 Debugging Case Study: Prisma Error P2022
Detailed analysis of real-world database schema drift between `schema.prisma` and persistent Docker PostgreSQL volume. Documented root cause analysis, diagnostic commands, manual repair SQL, and long-term migration policy.

### 9.4 CI/CD Pipeline
Fully automated GitHub Actions workflow (`.github/workflows/ci-cd.yml`) executing linting (`oxlint`), TypeScript type checking (`tsc --noEmit`), backend environment tests, and Vite production bundle compilation.

### 9.5 Maintenance Protocol
Structured maintenance plan covering Corrective (bug patching), Adaptive (browser engine updates), Perfective (offline PWA reading), and Preventive (database indexing and security audits) maintenance.

---

## Chapter 10: Conclusion, Future Roadmap & References

### 10.1 Project Conclusion
The ReadAble project successfully proves that academic reading software can transcend passive document rendering. By uniting adaptive typography, synchronized audio-visual focus, and in-situ AI comprehension, ReadAble empowers readers of all neurodivergent and visual profiles to study complex literature with confidence and comfort.

### 10.2 Future Roadmap
1. **Offline PWA Support:** Service worker caching and IndexedDB storage for offline reading in transit.
2. **Multilingual Speech Synthesis:** Multi-language audio narration and real-time translation for international scholars.
3. **Advanced Math & Diagram OCR:** Mathematical formula transcription into accessible Speech Synthesis Markup Language (SSML).

### 10.3 References
1. IEEE Std 830-1998: *IEEE Recommended Practice for Software Requirements Specifications*.
2. W3C Web Accessibility Initiative (WAI): *Web Content Accessibility Guidelines (WCAG) 2.1*.
3. Gamma, E., Helm, R., Johnson, R., & Vlissides, J.: *Design Patterns: Elements of Reusable Object-Oriented Software*. Addison-Wesley.
4. Prisma Documentation: *Prisma Client & Migration Workflows* (https://www.prisma.io/docs).
5. React 19 & TanStack Router Specifications: (https://tanstack.com/router).
