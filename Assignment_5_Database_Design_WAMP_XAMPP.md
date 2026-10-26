# Assignment 5: Design the Database using SQL/MongoDB and Implement it using WAMP/XAMPP

**Project Title:** ReadAble – An Intelligent & Accessible Digital Reading Platform  
**Target Environments:** WAMP (Windows, Apache, MySQL, PHP) / XAMPP (Cross-Platform, Apache, MariaDB/MySQL, PHP, Perl), and MongoDB NoSQL  
**Deliverable Files:**  
- `Assignments/readable_db_xampp.sql` (Production-ready MySQL/MariaDB DDL, seed data & queries)  
- `Assignments/readable_mongodb_schemas.js` (Mongoose document schema definitions)  

---

## 1. Relational Database Design (SQL)

The relational database for **ReadAble** is engineered to maintain strong ACID consistency, referential integrity with cascading constraints, and optimized indexes for fast retrieval of documents, preferences, and reading telemetry.

### 1.1 Relational Schema Architecture (8 Core Tables)
1. **`users`**: Manages identities with dual-identifier support (unique `email` or unique `phone`), encrypted passwords (`passwordHash`), and audit timestamps.
2. **`reading_preferences`**: A $1\text{--}1$ extension of `users` storing personalized accessibility configurations (font scale, line height, letter tracking, OpenDyslexic font family, and contrast theme).
3. **`documents`**: Ingested files linked to the uploading user via `userId` foreign key. Includes S3 storage keys, MIME types, and file sizes.
4. **`document_chunks`**: Decomposed text segments for sentence-level DOM highlighting and streaming text-to-speech (TTS).
5. **`bookmarks`**: Persistent scroll offsets and page markers per user-document combination.
6. **`annotations`**: Reader highlights and margin notes tied to specific text selections.
7. **`reading_sessions`**: Quantitative telemetry recording active reading duration (minutes) and progress percentages for reading streak calculations.
8. **`ai_interactions`**: Contextual cache storing generative AI simplifications and vocabulary definitions to prevent redundant LLM API calls.

---

## 2. Database Normalization Analysis

To ensure data integrity, eradicate update/delete anomalies, and minimize storage redundancy, the relational schema was normalized up to Boyce-Codd Normal Form (BCNF).

### 2.1 First Normal Form (1NF)
- **Requirement:** Every column must hold atomic values; repeating groups or arrays are prohibited.
- **Implementation:** Rather than storing bookmarks, text chunks, or annotations as delimited strings or JSON arrays inside a single `documents` record, each item is extracted into distinct first-class tables (`document_chunks`, `bookmarks`, `annotations`) with discrete primary keys and individual rows.

### 2.2 Second Normal Form (2NF)
- **Requirement:** Must be in 1NF and all non-key attributes must be fully functionally dependent on the entire primary key (no partial dependencies on composite keys).
- **Implementation:** Every table in the schema utilizes an atomic surrogate primary key (`id INT AUTO_INCREMENT`). Consequently, no partial dependency exists because no primary key consists of composite attributes. Functional dependencies such as:
  $$\text{documentId, chunkIndex} \longrightarrow \text{textContent}$$
  are represented cleanly with `documentId` acting as a foreign key and `id` as the primary key.

### 2.3 Third Normal Form (3NF)
- **Requirement:** Must be in 2NF and no non-key attribute may be transitively dependent on the primary key ($X \to Y$ where neither $X$ nor $Y$ is a superkey).
- **Implementation:** User typography preferences (`fontScale`, `themeMode`, `letterSpacing`) are factored out into a dedicated `reading_preferences` table rather than residing redundantly across `reading_sessions` or `documents`. If a user updates their font scale, it modifies exactly one row without cascading updates across historical sessions.

### 2.4 Boyce-Codd Normal Form (BCNF)
- **Requirement:** For every functional dependency $X \to Y$, $X$ must be a superkey.
- **Verification:** In `users`, dependencies $\text{email} \to \text{id, name}$ and $\text{phone} \to \text{id, name}$ both hold true because `email` and `phone` are declared with unique constraints, rendering them candidate keys. The schema adheres to BCNF.

---

## 3. NoSQL / MongoDB Alternative Design

For high-throughput, unstructured reading telemetry and nested document notes, MongoDB offers horizontal scalability and schema flexibility.

### 3.1 Design Tradeoff Comparison

| Evaluation Metric | Relational SQL (MySQL / MariaDB in XAMPP) | Document Store (MongoDB) | ReadAble Decision |
| :--- | :--- | :--- | :--- |
| **Referential Integrity** | Strictly enforced via engine foreign keys (`ON DELETE CASCADE`). | Manual application-level validation required. | **SQL Preferred for core auth & documents.** |
| **Document Chunks & Notes** | Requires multi-table `JOIN` across 3+ tables. | Stored natively as embedded sub-document arrays. | **MongoDB optimal for nested chunk streams.** |
| **Audit & ACID Transactions**| Native multi-table transactional guarantees. | Single-document ACID native; multi-document transactions require replica sets. | **SQL chosen for financial/academic auditability.** |
| **Deployment Complexity** | Pre-bundled with WAMP / XAMPP; zero external cloud dependency. | Requires separate MongoDB Community Server daemon or Atlas cluster. | **XAMPP MySQL ideal for local development.** |

---

## 4. Step-by-Step WAMP / XAMPP Implementation Guide

This section provides actionable steps to initialize the database in WAMP or XAMPP and wire it directly into the ReadAble backend.

### Step 1: Start Services in XAMPP / WAMP Control Panel
1. Open **XAMPP Control Panel** (or WAMP Server).
2. Click **Start** beside the **Apache** module (Port 80/443).
3. Click **Start** beside the **MySQL** module (Port 3306).
4. Verify that the status indicators turn **Green** with PID and port assignments.

```
+-------------------------------------------------------------------+
|                     XAMPP Control Panel v3.3.0                    |
+-------------------------------------------------------------------+
|  Module      PID(s)      Port(s)      Actions                     |
|  Apache      7412        80, 443      [Stop]  [Admin]  [Config]   |
|  MySQL       5820        3306         [Stop]  [Admin]  [Config]   |
+-------------------------------------------------------------------+
```

---

### Step 2: Access phpMyAdmin and Create Database
1. Launch any web browser and navigate to:  
   `http://localhost/phpmyadmin`
2. In the top navigation bar, click on **Databases**.
3. Under **Create database**, enter `readable_db`, select Collation **`utf8mb4_unicode_ci`**, and click **Create**.

---

### Step 3: Import the SQL Schema Script
1. Select `readable_db` from the left-hand database list.
2. Click on the **Import** tab on the top menu.
3. Click **Choose File** and select:  
   `c:\Users\hp\OneDrive\Desktop\Read_Able\Read-Able\Assignments\readable_db_xampp.sql`
4. Leave format set to **SQL** and click **Import** at the bottom.
5. phpMyAdmin will execute all statements, displaying:  
   *`Import has been successfully finished, 8 queries executed.`*

---

### Step 4: Configure Node.js Backend for WAMP / XAMPP MySQL

#### Option A: Prisma ORM with MySQL Provider
Update `backend/prisma/schema.prisma` to use the MySQL datasource:
```prisma
datasource db {
  provider = "mysql"
  url      = env("DATABASE_URL")
}
```

Update `backend/.env` with your XAMPP credentials (default XAMPP MySQL has username `root` with no password):
```env
DATABASE_URL="mysql://root:@localhost:3306/readable_db"
JWT_SECRET="your_actual_random_secret_at_least_32_characters"
JWT_EXPIRES_IN="7d"
BCRYPT_SALT_ROUNDS=12
PORT=3001
```

Generate the Prisma Client:
```powershell
# From backend/ directory
npx prisma generate
```

#### Option B: Direct Native MySQL2 Pool
If connecting using the native `mysql2` driver:
```javascript
// backend/src/config/database.js
import mysql from "mysql2/promise";

const pool = mysql.createPool({
  host: "localhost",
  port: 3306,
  user: "root",
  password: "", // Default XAMPP password is empty
  database: "readable_db",
  waitForConnections: true,
  connectionLimit: 10,
  queueLimit: 0,
});

export default pool;
```

---

### Step 5: Verification & Execution of Analytical Queries

Run this validation query in phpMyAdmin's **SQL** tab to confirm proper data integration:

```sql
SELECT 
  u.name AS Reader_Name,
  u.email,
  p.themeMode AS Active_Theme,
  COUNT(d.id) AS Documents_Uploaded,
  COALESCE(SUM(s.durationMinutes), 0) AS Total_Reading_Mins
FROM users u
LEFT JOIN reading_preferences p ON u.id = p.userId
LEFT JOIN documents d ON u.id = d.userId
LEFT JOIN reading_sessions s ON u.id = s.userId
WHERE u.id = 1
GROUP BY u.id, u.name, u.email, p.themeMode;
```

#### Expected Output Table
| Reader_Name | email | Active_Theme | Documents_Uploaded | Total_Reading_Mins |
| :--- | :--- | :--- | :--- | :--- |
| **Rohit Rabade** | `rohit@readable.org` | `lowlight` | 2 | 75 |

The database is successfully designed, normalized, implemented, and operational in WAMP/XAMPP.
