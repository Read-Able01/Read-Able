# Assignment 7: Develop the Backend

**Project Title:** ReadAble – An Intelligent & Accessible Digital Reading Platform  
**Backend Technology Stack:** Node.js (v18+) + Express.js + Prisma ORM / MySQL / PostgreSQL + JWT + Bcrypt  
**Design Pattern:** Modular Layered (Controller-Service-Repository) Architecture  

---

## 1. Backend Architectural Pattern

The backend of **ReadAble** is designed using a clean, layered architecture ensuring high cohesion, low coupling, and comprehensive testability.

```
                           Incoming Client HTTP Request
                                       │
                                       ▼
                       ┌───────────────────────────────┐
                       │   Express Server (app.js)     │
                       │   - CORS Middleware           │
                       │   - Express JSON Parser       │
                       │   - Health Check (/api/health)│
                       └───────────────┬───────────────┘
                                       │
                                       ▼
                       ┌───────────────────────────────┐
                       │   Routing Layer (routes/)     │
                       │   - Feature Route Mounting    │
                       └───────────────┬───────────────┘
                                       │
                        ┌──────────────┴──────────────┐
                        ▼                             ▼
       ┌────────────────────────────────┐ ┌─────────────────────────────────┐
       │ Middleware Layer               │ │ Validation Layer                │
       │ - authenticate (JWT Bearer)    │ │ - validateRegister, validateLogin│
       │ - errorMiddleware (Centralized)│ │ - File MIME & Size Validator    │
       └────────────────┬───────────────┘ └────────────────┬────────────────┘
                        │                                  │
                        └──────────────┬───────────────────┘
                                       ▼
                       ┌───────────────────────────────┐
                       │ Controller Layer              │
                       │ (auth, documents, reading)    │
                       │ - Parse HTTP Request & Params │
                       │ - Format Standard Response    │
                       └───────────────┬───────────────┘
                                       │
                                       ▼
                       ┌───────────────────────────────┐
                       │ Service Layer (Business Logic)│
                       │ - Token generation & hashing  │
                       │ - Sentence chunking & stream  │
                       │ - Storage upload & AI bridge  │
                       └───────────────┬───────────────┘
                                       │
                                       ▼
                       ┌───────────────────────────────┐
                       │ Data Access Layer (Prisma ORM)│
                       │ - Database Queries & Cascade  │
                       └───────────────────────────────┘
```

---

## 2. Core Functional Modules Implementation

### 2.1 Module 1: Authentication & Authorization (`src/features/auth/`)
- **Password Security:** Passwords are never stored in plaintext. They are hashed using `bcryptjs` with salt work factor $12$:
  ```javascript
  const passwordHash = await bcrypt.hash(password, config.bcryptSaltRounds);
  ```
- **Dual Identifier Parsing:** The authentication service accepts either Email or Phone Number. If an ambiguous identifier string is provided, it dynamically checks for an `@` symbol:
  ```javascript
  const normalizePhone = (phone) => phone.replace(/[\s\-().]/g, "").trim();

  if (identifier && !lookupEmail && !lookupPhone) {
    if (identifier.includes("@")) {
      lookupEmail = identifier;
    } else {
      lookupPhone = identifier;
    }
  }
  ```
- **Stateless Session Tokens:** Upon verification, a signed JSON Web Token (JWT) is issued containing the user ID:
  ```javascript
  const signToken = (userId) =>
    jwt.sign({ id: userId }, config.jwtSecret, { expiresIn: config.jwtExpiresIn });
  ```
- **JWT Protection Middleware (`src/shared/middleware/auth.middleware.js`):**
  Extracts the `Authorization: Bearer <token>` header, verifies the signature, and attaches `req.user = { id: decoded.id }` to subsequent handlers.

---

### 2.2 Module 2: Document Ingestion & Storage (`src/features/documents/`)
- **Multipart Upload Handling:** Uses `multer` configured with memory storage or local disk buffering.
- **Validation:** Enforces maximum file size limit of $25\text{ MB}$ and restricts MIME types to `application/pdf` and `text/plain`.
- **Database Persistence:** Associates the storage path or AWS S3 key with the authenticated user ID:
  ```javascript
  const doc = await prisma.document.create({
    data: {
      title,
      fileName,
      s3Key,
      fileSize,
      mimeType,
      userId: req.user.id
    }
  });
  ```

---

### 2.3 Module 3: Reading Engine & Text Chunking (`src/features/reading/`)
- **Sentence Splitting:** Ingested documents are divided into indexable sentence chunks for real-time highlighting:
  ```javascript
  export const splitIntoSentences = (text) => {
    return text.match(/[^.!?]+[.!?]+(\s|$)/g) || [text];
  };
  ```
- **Reading Progress Tracking:** Records last read sentence index, page number, and scroll percentage, allowing the user to resume reading seamlessly on any device.

---

### 2.4 Module 4: Assistive AI Integration
- Connects to generative AI endpoints (e.g., Google Gemini or OpenAI) using structured prompt templates:
  ```javascript
  export const simplifyText = async (complexText) => {
    const prompt = `Rewrite the following academic text for an 8th-grade reading level.
Retain core factual accuracy while simplifying dense vocabulary and long clauses:
"${complexText}"`;
    return await callAIService(prompt);
  };
  ```

---

## 3. RESTful API Specification Catalog

| Method | Endpoint Route | Auth Required | Request Body / Params | Success Status | Response Payload | Error Codes |
| :--- | :--- | :---: | :--- | :---: | :--- | :---: |
| `POST` | `/api/auth/register` | No | `{ name, email?, phone?, password }` | `201 Created` | `{ token, user: { id, name, email } }` | `400, 409` |
| `POST` | `/api/auth/login` | No | `{ identifier?, password }` | `200 OK` | `{ token, user: { id, name, email } }` | `400, 401` |
| `GET` | `/api/auth/me` | **Yes** | None (Header Token) | `200 OK` | `{ id, name, email, phone, createdAt }`| `401` |
| `POST` | `/api/documents/upload` | **Yes** | `multipart/form-data` (file) | `201 Created` | `{ id, title, s3Key, fileSize }` | `400, 413` |
| `GET` | `/api/documents` | **Yes** | Query: `?page=1&limit=10` | `200 OK` | `[ { id, title, fileName, fileSize } ]` | `401` |
| `GET` | `/api/documents/:id` | **Yes** | Param: `id` | `200 OK` | `{ id, title, chunks: [...] }` | `401, 404` |
| `DELETE`| `/api/documents/:id` | **Yes** | Param: `id` | `200 OK` | `{ message: "Document deleted" }` | `401, 404` |
| `PATCH`| `/api/users/preferences`| **Yes** | `{ fontScale, themeMode, lineHeight }`| `200 OK` | `{ success: true, preferences }` | `400, 401` |
| `POST` | `/api/ai/simplify` | **Yes** | `{ text: string }` | `200 OK` | `{ simplifiedText: string }` | `400, 502` |
| `GET` | `/api/health` | No | None | `200 OK` | `{ status: "ok" }` | None |

---

## 4. Centralized Error Handling & Security Architecture

### 4.1 Custom Error Hierarchy (`src/shared/utils/errors.js`)
ReadAble implements domain-specific operational errors extending standard `Error`:
- `ValidationError` (HTTP 400)
- `UnauthorizedError` (HTTP 401)
- `ForbiddenError` (HTTP 403)
- `NotFoundError` (HTTP 404)
- `ConflictError` (HTTP 409)

### 4.2 Centralized Error Middleware (`src/shared/middleware/error.middleware.js`)
```javascript
export const errorMiddleware = (err, req, res, next) => {
  const statusCode = err.statusCode || 500;
  const message = err.isOperational ? err.message : "Internal server error";

  if (process.env.NODE_ENV !== "production") {
    console.error("Backend Error Stack:", err);
  }

  res.status(statusCode).json({
    success: false,
    error: {
      message,
      statusCode,
    },
  });
};
```

### 4.3 Startup Environment Integrity Check (`src/config/env.js`)
The server validates critical environment parameters at boot. If `JWT_SECRET` is omitted or contains fewer than 32 characters, the application terminates immediately with an explicit fatal error message, preventing insecure deployment.
