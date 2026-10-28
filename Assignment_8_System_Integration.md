# Assignment 8: Integrate Backend + UI + Database

**Project Title:** ReadAble – An Intelligent & Accessible Digital Reading Platform  
**Integration Scope:** React 19 Frontend + Vite Reverse Proxy + Node.js/Express API + Prisma ORM + MySQL / PostgreSQL  
**Integration Status:** Fully Integrated and Functional  

---

## 1. End-to-End System Integration Architecture

The **ReadAble** system operates as an integrated three-tier distributed architecture. The diagrams below illustrate the communication channels, proxy routing, and data flow across client, application server, and persistence tiers.

```
┌────────────────────────────────────────────────────────────────────────┐
│                        CLIENT TIER (Port 5173)                         │
│  React 19 + TypeScript + TanStack Router + Tailwind CSS + Lucide Icons │
│  Axios HTTP Client with Automatic JWT Interceptors                    │
└───────────────────────────────────┬────────────────────────────────────┘
                                    │ HTTP Request (e.g., /api/auth/login)
                                    ▼
┌────────────────────────────────────────────────────────────────────────┐
│                  VITE REVERSE PROXY GATEWAY (vite.config.ts)           │
│  Eliminates CORS preflight overhead; proxies /api/* -> localhost:3001  │
└───────────────────────────────────┬────────────────────────────────────┘
                                    │ Internal Proxy Forwarding
                                    ▼
┌────────────────────────────────────────────────────────────────────────┐
│                      APPLICATION TIER (Port 3001)                      │
│  Node.js + Express.js REST API Layer                                   │
│  - JWT Bearer Authentication & Phone Normalization Middleware          │
│  - Multer Multipart Document Ingestion Engine                          │
│  - Centralized Error & Exception Handler                               │
└───────────────────────────────────┬────────────────────────────────────┘
                                    │ Prisma Client Query Engine
                                    ▼
┌────────────────────────────────────────────────────────────────────────┐
│                       PERSISTENCE / DATABASE TIER                      │
│  Option A: Docker PostgreSQL (Port 5433 / 5432)                        │
│  Option B: WAMP / XAMPP MariaDB/MySQL (Port 3306) via readable_db     │
└────────────────────────────────────────────────────────────────────────┘
```

---

## 2. API Communication & Proxy Configurations

### 2.1 Frontend Axios Client (`frontend/src/lib/api.ts`)
The client uses an Axios instance configured with a base URL of `/api`. An automated request interceptor inspects `localStorage` and appends the cryptographic JWT Bearer token to all outgoing requests:

```typescript
import axios from "axios";

const api = axios.create({
  baseURL: "/api",
});

// Automatic JWT Token Attachment
api.interceptors.request.use((config) => {
  const token = localStorage.getItem("readable_token");
  if (token) {
    config.headers.Authorization = `Bearer ${token}`;
  }
  return config;
});

export const authApi = {
  register: (data) => api.post("/auth/register", data),
  login: (data) => api.post("/auth/login", data),
  me: async () => api.get("/auth/me"),
};

export default api;
```

---

### 2.2 Vite Reverse Proxy Configuration (`frontend/vite.config.ts`)
In development, the browser connects to Vite on port `5173`. To prevent Cross-Origin Resource Sharing (CORS) blocks, Vite proxies all `/api` traffic internally to the Express server on port `3001`:

```typescript
export default defineConfig({
  plugins: [TanStackRouterVite(), react(), tailwindcss()],
  server: {
    proxy: {
      "/api": {
        target: "http://localhost:3001",
        changeOrigin: true,
      },
    },
  },
});
```

---

### 2.3 Express CORS & Security Configuration (`backend/src/app.js`)
The backend is explicitly configured with CORS middleware to permit credentials and requests from trusted frontend origins:

```javascript
app.use(
  cors({
    origin: ["http://localhost:5173", "http://localhost:3000"],
    credentials: true,
  })
);
```

---

## 3. End-to-End Operational Workflows

### 3.1 Workflow 1: User Registration & Session Hydration
```
[User Browser]             [React Client]           [Express Backend]         [Prisma / Database]
     │                           │                          │                         │
     │── Fill Form & Submit ────►│                          │                         │
     │                           │── POST /api/auth/reg ───►│                         │
     │                           │   (Name, Phone, Pass)    │                         │
     │                           │                          │── Normalize Phone       │
     │                           │                          │── Bcrypt Hash (Salt 12) │
     │                           │                          │── INSERT INTO User ────►│
     │                           │                          │◄── Record Committed ────│
     │                           │                          │── Generate JWT Token    │
     │                           │◄── 201 Created (Token) ──│                         │
     │                           │                          │                         │
     │                           │── Save to localStorage   │                         │
     │                           │── Navigate('/dashboard') │                         │
     │◄── Render Dashboard ──────│                          │                         │
```

---

### 3.2 Workflow 2: Document Upload & Library Synchronization
1. **Selection:** User drops a PDF into the dashboard file drop zone.
2. **Payload:** The client encapsulates the file into a `FormData` object and dispatches `POST /api/documents/upload` with the authorization header.
3. **Ingestion:** Express `multer` middleware buffers the binary file, verifies that the MIME type is `application/pdf`, and saves it to local disk storage (`backend/uploads/`) or uploads it to AWS S3.
4. **Database Record:** Prisma creates a new row in the `Document` table referencing `userId: req.user.id`.
5. **State Update:** The client receives the newly minted document metadata and automatically prepends the record to the active React state library without a full page refresh.

---

### 3.3 Workflow 3: Live Accessibility Preference Synchronization
1. **User Action:** Reader adjusts the font zoom slider from $100\%$ to $115\%$.
2. **Immediate Feedback:** React invokes `document.documentElement.style.setProperty("--read-scale", "1.15")`, reflowing text instantly with zero perceptible latency.
3. **Backend Persistence:** A debounced ($300\text{ ms}$) `PATCH /api/users/preferences` request updates the `reading_preferences` table in the database, preserving the setting across devices.

---

## 4. Environment Variables Coordination

| Parameter | Frontend Expectation | Backend Production (`.env`) | WAMP / XAMPP Local Dev |
| :--- | :--- | :--- | :--- |
| **Server Port** | Proxied to `:3001` | `PORT=3001` | `PORT=3001` |
| **Database URL**| Abstracted behind API | `postgresql://readable:pwd@localhost:5433/readable_db` | `mysql://root:@localhost:3306/readable_db` |
| **JWT Secret** | Stored in `localStorage` | 256-bit secure cryptographic key | 256-bit secure cryptographic key |
| **CORS Origins**| `http://localhost:5173` | Explicit whitelist | `http://localhost:5173` |

---

## 5. Integration Verification Matrix

| Test Scenario | Input Action | Expected Behavior | Verification Status |
| :--- | :--- | :--- | :---: |
| **Health Check** | `GET /api/health` | Returns `{ status: "ok" }` with HTTP 200 | **PASSED** |
| **Dual Identifier Auth** | Login via Phone: `9876543210` | Normalizes phone, verifies hash, returns JWT | **PASSED** |
| **Route Protection Guard** | Access `/dashboard` without token | Automatically redirected to `/` | **PASSED** |
| **CORS Verification** | Frontend fetch from `localhost:5173` | Request accepted without CORS errors | **PASSED** |
| **Database Cascade Delete**| Delete User record in DB | Cascades deletion to documents and preferences | **PASSED** |
| **State Hydration** | Page refresh on `/dashboard` | Token read from `localStorage`, session active | **PASSED** |
