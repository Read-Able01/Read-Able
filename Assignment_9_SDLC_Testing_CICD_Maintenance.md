# Assignment 9: Implement Waterfall Model or Others, Debugging & Testing, CI/CD Pipeline, and Maintenance

**Project Title:** ReadAble – An Intelligent & Accessible Digital Reading Platform  
**SDLC Methodology:** Iterative Waterfall (Hybrid Agile-Waterfall) Framework  
**Testing Scope:** Unit, Integration, Accessibility (a11y), and End-to-End Testing  
**Automation:** GitHub Actions Continuous Integration & Continuous Deployment (CI/CD) Pipeline  

---

## 1. Software Development Life Cycle (SDLC) Methodology

### 1.1 Comparison of Candidate SDLC Models

| SDLC Model | Core Philosophy | Advantages | Disadvantages | Suitability for ReadAble |
| :--- | :--- | :--- | :--- | :---: |
| **Classical Waterfall** | Strict sequential progression through distinct non-overlapping phases. | Clear milestone deliverables, comprehensive documentation. | Inflexible to changing accessibility requirements, late testing. | Moderate |
| **Pure Agile (Scrum)** | Rapid sprint cycles with minimal formal initial documentation. | Fast adaptation, continuous user feedback. | High risk of architectural drift, weak formal UML baselining. | Moderate |
| **V-Model** | Rigorous verification and validation at every development stage. | High quality, robust testing matrices. | High initial overhead, rigid change management. | Good |
| **Iterative Waterfall (Hybrid)** | Structured sequential phases (Requirements $\to$ Design $\to$ Build) executed in iterative feedback loops. | Combines rigorous architectural documentation with iterative frontend/AI refinement. | Requires disciplined milestone tracking. | **Best Fit (Selected)** |

---

### 1.2 Justification for the Iterative Waterfall Model
ReadAble selected the **Iterative Waterfall (Hybrid Agile-Waterfall) Model** for the following technical reasons:
1. **Accessibility Standards Require Upfront Discipline:** Conformance with WCAG 2.1 AA/AAA, color contrast algorithms, and typography ratios demands clear, documented specifications before writing code.
2. **Database Integrity & Schema Constraints:** Relational schemas with cascading rules (Users $\to$ Documents $\to$ Chunks) must be architected upfront to prevent costly database refactoring.
3. **Iterative Refinement of AI and Audio Features:** While the architecture is established sequentially, speech synthesis parameters, sentence highlighting algorithms, and prompt templates benefit from iterative prototyping cycles.

### 1.3 Project Phase Mapping Across Milestones
```
   [Phase 1: Requirements Engineering] ──► Assignments 1 & 2 (Problem Statement, Feasibility, SRS)
                   │
                   ▼
   [Phase 2: System & Data Architecture] ──► Assignments 3 & 4 (UML Diagrams, DFDs, ER Modeling)
                   │
                   ▼
   [Phase 3: Database & UI Prototyping] ──► Assignments 5 & 6 (WAMP/XAMPP SQL, UI/UX Design System)
                   │
                   ▼
   [Phase 4: Backend & System Integration] ──► Assignments 7 & 8 (Express APIs, End-to-End Integration)
                   │
                   ▼
   [Phase 5: Verification, CI/CD & Maintenance] ──► Assignments 9 & 10 (Testing, GitHub Actions, Final Report)
```

---

## 2. Comprehensive Testing Strategy & Test Case Matrix

### 2.1 Testing Pyramid Implementation
- **Unit Testing:** Testing isolated utility functions (`normalizePhone`, `splitIntoSentences`, password hashing) using Vitest/Jest.
- **Integration Testing:** Testing Express HTTP routes, authentication guards, and Prisma database queries using Supertest.
- **Accessibility (a11y) Testing:** Automated scanning using `@axe-core/react` and Chrome Lighthouse to guarantee zero contrast violations.
- **Manual End-to-End Testing:** Human testing of keyboard navigation (`Tab`/`Enter`), audio playback rate synchronization, and responsive layouts across viewports.

### 2.2 Test Case Specification Matrix

| Test ID | Module | Scenario & Pre-conditions | Test Steps | Expected Result | Status |
| :---: | :--- | :--- | :--- | :--- | :---: |
| **TC-01** | Auth | User registration with valid email | Submit `{ name: "Alice", email: "alice@test.org", password: "PassWord!23" }` | HTTP 201; User record created; JWT returned | **PASS** |
| **TC-02** | Auth | User registration with duplicate email | Submit identical email as TC-01 | HTTP 409 Conflict; "Account already exists" | **PASS** |
| **TC-03** | Auth | Phone normalization | Submit phone `"(987) 654-3210"` | Saved as `"9876543210"`; whitespaces stripped | **PASS** |
| **TC-04** | Auth | Dual identifier login via phone | Submit `{ identifier: "9876543210", password: "PassWord!23" }` | HTTP 200; Valid JWT token returned | **PASS** |
| **TC-05** | Security | Protected endpoint access without token | Dispatches `GET /api/documents` with no header | HTTP 401 Unauthorized; Request blocked | **PASS** |
| **TC-06** | Ingestion | PDF upload within size limit ($<25\text{MB}$) | Upload valid $1.4\text{MB}$ PDF via dashboard | HTTP 201; Stored with unique S3 key | **PASS** |
| **TC-07** | Ingestion | Upload non-permitted file type (`.exe`) | Upload malicious binary | HTTP 400 Bad Request; File rejected | **PASS** |
| **TC-08** | Ingestion | Upload file exceeding size limit ($>25\text{MB}$) | Upload $30\text{MB}$ archive | HTTP 413 Payload Too Large; Upload stopped | **PASS** |
| **TC-09** | UI / a11y | Font scale slider adjustment | Move scale slider from $100\%$ to $130\%$ | `--read-scale` set to $1.3$; text reflows cleanly | **PASS** |
| **TC-10** | UI / a11y | Theme toggle to Lowlight Mode | Click Lowlight Mode button | `.lowlight` applied to `<body>`; contrast $\ge 7:1$ | **PASS** |
| **TC-11** | Audio | Text-to-Speech Playback | Click Play on sentence 1 | `speechSynthesis` triggers; sentence highlighted | **PASS** |
| **TC-12** | Database | Cascade deletion of user | Delete User ID in database | Associated documents and preferences purged | **PASS** |

---

## 3. Real-World Debugging Case Study & Incident Post-Mortem

### 3.1 Incident Overview: Prisma Error P2022
During integration testing between the backend and database, the application experienced a critical runtime failure:
```
PrismaClientKnownRequestError: 
Invalid `prisma.user.findUnique()` invocation:
The column `User.phone` does not exist in the current database.
Error Code: P2022
```

### 3.2 Root Cause Analysis (RCA)
1. **Schema Drift:** The developer added `phone String? @unique` to `backend/prisma/schema.prisma`.
2. **Volume Persistence vs. Declarative Migrations:** The PostgreSQL database running inside Docker used a persistent named volume (`readable_postgres_data`). While `schema.prisma` was modified, no formal migration was generated and applied to the database container.
3. **Misleading Status:** `prisma migrate status` reported "up to date" because no unapplied migration files were tracked in the Git repository, despite the live database schema being out of sync with Prisma's JavaScript client.

### 3.3 Diagnostic Procedure & Resolution
1. **Inspection via Container Shell:**
   ```powershell
   docker exec -it readable-postgres psql -U readable -d readable_db
   SELECT column_name, data_type FROM information_schema.columns WHERE table_name = 'User';
   ```
   *Result confirmed that the `phone` column was completely missing.*
2. **Database Repair:**
   ```sql
   ALTER TABLE public."User" ADD COLUMN "phone" TEXT;
   CREATE UNIQUE INDEX "User_phone_key" ON public."User"("phone");
   ```
3. **Regenerate Client:**
   ```powershell
   npx prisma generate
   ```

### 3.4 Long-Term Architectural Prevention Policy
- **Strict Rule:** Never edit `schema.prisma` without immediately running `npx prisma migrate dev --name <description>`.
- Both the schema and the migration SQL script must be committed together to version control.
- In production CI/CD pipelines, execute `npx prisma migrate deploy` automatically before launching server processes.

---

## 4. Continuous Integration & Continuous Deployment (CI/CD) Pipeline

An automated CI/CD pipeline is implemented using **GitHub Actions** (`.github/workflows/ci-cd.yml`).

### 4.1 Pipeline Architecture & Workflow Stages
```
[Git Push / Pull Request to main]
             │
             ▼
   [Stage 1: Code Quality & Linting] ──► Runs oxlint & ESLint across frontend and backend
             │
             ▼
   [Stage 2: Type Checking] ──────────► Executes tsc --noEmit (TypeScript compiler)
             │
             ▼
   [Stage 3: Automated Unit Tests] ───► Executes backend & frontend test suites
             │
             ▼
   [Stage 4: Production Build] ───────► Bundles Vite frontend assets & verifies bundle size
             │
             ▼
   [Stage 5: Container Build & Scan] ─► Builds Docker images & verifies Docker Compose health
```

---

## 5. Software Maintenance & System Evolution Plan

To ensure long-term stability, ReadAble adopts four structured maintenance protocols:

1. **Corrective Maintenance (Reactive):**
   - Active error logging with Winston/Morgan.
   - Centralized health monitoring route (`GET /api/health`) returning container status.
   - Automated error reporting alerting administrators to 500-level HTTP responses.
2. **Adaptive Maintenance (Environmental):**
   - Routine dependency updates via `npm update` and `dependabot`.
   - Adapting to Web Speech API changes across evolving browser engine versions (Chromium, WebKit, Gecko).
3. **Perfective Maintenance (Enhancements):**
   - Roadmap for offline Progressive Web App (PWA) reading with IndexedDB local caching.
   - Incorporation of optical character recognition (OCR) for scanned historical manuscripts.
4. **Preventive Maintenance (Proactive):**
   - Weekly automated database vacuuming and B-Tree index optimization on `readable_db`.
   - Security auditing using `npm audit` to identify vulnerable transitive dependencies.
