# Assignment 1 – Problem Statement & Feasibility Analysis

**Project Title:** ReadAble – An Intelligent & Accessible Digital Reading Platform  
**Course:** Software Engineering / Software Development Project  
**Repository:** [https://github.com/RohitRabade2006/Read-Able.git](https://github.com/RohitRabade2006/Read-Able.git)  
**Academic Year:** 2024–2026  
**Document Constraint:** Maximum 2 Pages  

---

## 1. Problem Statement

### 1.1 Context and Background
In today's academic and professional environments, digital documents—predominantly Portable Document Format (PDF) and structured text files—serve as the primary medium for disseminating research publications, technical documentation, course textbooks, and white papers. However, existing PDF viewing tools (such as Adobe Acrobat Reader, native browser viewers, and generic e-readers) are fundamentally designed as **passive document renderers**. They display visual layouts accurately but remain entirely indifferent to the reader's cognitive capacity, sensory limitations, and comprehension needs.

### 1.2 Core Problem Definition
Readers encounter significant friction when engaging with complex, technical, or lengthy documents due to several critical barriers:
1. **Cognitive Overload & Dense Typography:** Long academic papers with narrow margins, small standard serif/sans-serif typefaces, and rigid line spacing induce visual fatigue, eye strain, and loss of concentration, especially for neurodivergent individuals (e.g., those diagnosed with Dyslexia, ADHD, or visual processing disorders).
2. **Technical Jargon & Complex Syntactical Constructs:** Domain-specific vocabulary, dense mathematical formulations, and convoluted paragraphs force readers to break their reading flow to manually look up external dictionaries, search engines, or AI chat interfaces.
3. **Inaccessible Visual Figures:** Diagrams, flowcharts, and technical figures often lack semantic textual descriptions or contextual breakdowns, posing high cognitive barriers for both visually impaired users and novices.
4. **Lack of Synchronized Multi-Modal Assistance:** Traditional screen readers often read raw text linearly without highlighting the active sentence, losing reading position and preventing simultaneous visual-auditory reinforcement.
5. **Absence of Adaptive Personalization:** Readers lack an intelligent environment that remembers their preferred typography, reading speeds, and comprehension struggles across reading sessions.

### 1.3 The Proposed Solution: ReadAble
**ReadAble** is an accessibility-first, intelligent digital reading platform engineered to transform passive reading into an active, personalized, and multi-modal comprehension experience. Built on the core paradigm:
$$\mathbf{Read \longrightarrow Understand \longrightarrow Listen \longrightarrow Adapt}$$

ReadAble delivers:
- **Dynamic Accessibility Controls:** Real-time font scaling ($100\%\text{--}130\%+$), custom leading (line height), tracking (letter spacing), word spacing, and curated high-contrast reading themes (e.g., Cream & Teal, Lowlight Dark, and High Contrast).
- **Synchronized Text-to-Speech (TTS) & Focus Mode:** Sentence-by-sentence visual highlighting synchronized with audio playback, mitigating attention drift and supporting auditory learners.
- **Contextual AI Comprehension Engine:** In-situ paragraph simplification, technical term explanation, executive summarization, and visual figure analysis without leaving the active document viewport.
- **Persistent Personalization:** Secure user session management tracking reading progress, bookmarks, annotations, and reading streaks.

---

## 2. Feasibility Analysis

A multidimensional feasibility study was conducted to evaluate the viability, operational sustainability, and economic justification of the ReadAble platform.

| Feasibility Dimension | Assessment & Architecture Considerations | Verdict |
| :--- | :--- | :--- |
| **Technical Feasibility** | The system leverages proven modern web standards: **React 19 + TypeScript + Vite** for a lightweight, accessible UI; **TanStack Router** for client-side routing; **Tailwind CSS** with CSS variables for dynamic theme rendering; **Node.js & Express.js** for RESTful services; **Prisma ORM** interfacing with relational databases (**PostgreSQL / MySQL / WAMP / XAMPP**); and standard **Web Speech API / Cloud TTS** alongside LLM APIs for AI assistance. The modular layered architecture guarantees modularity and testability. | **Highly Feasible** |
| **Operational Feasibility** | The user interface follows a progressive disclosure pattern. Readers can seamlessly upload documents (`.pdf`, `.txt`) or paste text, toggle accessibility bars with a single click, and activate read-aloud features. The platform requires zero installation for end users, operating across modern web browsers on desktop, tablet, and mobile devices. | **Highly Feasible** |
| **Economic Feasibility** | Development is accomplished entirely using open-source, zero-licensing frameworks (React, Node.js, Express, Prisma, Docker, MySQL/PostgreSQL). Initial development and university hosting operate locally (via WAMP/XAMPP or Docker Compose) with near-zero initial infrastructure cost. Scaled cloud deployments (AWS S3, Supabase/RDS, serverless LLM APIs) can be funded on a pay-per-use tier. | **Feasible & Cost-Effective** |
| **Schedule / Temporal Feasibility** | The project is structured into manageable, iterative development milestones (10 sprint assignments) spanning requirements, UML modeling, database schema implementation, frontend/backend engineering, integration, and CI/CD automated deployment within an academic semester schedule. | **Feasible within Timeline** |
| **Legal & Accessibility Compliance** | The platform is designed strictly aligned with **WCAG 2.1 Level AA & AAA** guidelines (contrast ratio $\ge 4.5:1$ and $\ge 7:1$, keyboard operability, text scalability without clipping). User documents and personal study notes are protected under GDPR-compliant tokenized authentication; uploaded texts are processed strictly under educational fair use. | **Fully Compliant** |

### 2.1 Risk Assessment and Mitigation Strategy
1. **Schema Drift between ORM and Database Containers:** Mitigated by enforcing strict migration versioning (`npx prisma migrate dev`), declarative schema sync, and verified connection scripts.
2. **Third-Party AI Latency & Rate Limits:** Mitigated by caching generated summaries and definitions locally in PostgreSQL/MySQL and providing fallbacks to native browser dictionary lookup.
3. **Heavy PDF Parsing Load on Client/Server:** Mitigated through asynchronous chunked document processing and streaming text rendering.

---

## 3. Conclusion
The problem identified is acute, widespread, and underserved by conventional document viewers. The feasibility evaluation confirms that **ReadAble** is technically viable, economically efficient, operationally intuitive, and ethically compliant. The project is strongly recommended for full development and deployment.
