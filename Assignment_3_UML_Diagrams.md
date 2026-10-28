# Assignment 3 – UML Diagrams (System Modeling & Architecture)

**Project Title:** ReadAble – An Intelligent & Accessible Digital Reading Platform  
**Diagramming Standard:** Unified Modeling Language (UML 2.5)  
**Compatible Platform:** draw.io (`.drawio` format provided), Mermaid JS, and StarUML  

---

## 1. System Actors & Architecture Overview

The ReadAble system models interactions between human users and external software services:

| Actor | Type | Description & Responsibilities |
| :--- | :--- | :--- |
| **Primary Reader (Student/Researcher)** | Human Actor | Registers, logs in, uploads PDF/TXT documents, manages library, configures reading typography, triggers TTS, and requests AI explanations. |
| **Accessibility User (Dyslexic/Low Vision)** | Human Actor | Utilizes high-contrast modes, dynamic scale factors, sentence focus highlights, and audio-visual synchronization. |
| **System Administrator** | Human Actor | Monitors system health, manages database migrations, audits storage usage, and manages security policies. |
| **External AI/LLM Provider** | External System | Third-party AI service (OpenAI / Anthropic / Gemini) that processes simplification, summary, and definition requests. |
| **Cloud / Local Storage Service** | External System | AWS S3 or Local File System storing raw PDF files, thumbnails, and cache artifacts. |

---

## 2. Use Case Diagram

The Use Case Diagram captures system functionality from the perspective of primary actors, including `<<include>>` and `<<extend>>` relationships.

### 2.1 Mermaid Specification
```mermaid
graph TD
    actorUser([Student / Accessibility Reader])
    actorAdmin([System Administrator])
    actorAI([External AI Engine])
    
    subgraph ReadAble Platform Boundary
        UC1(UC-1: Register Account)
        UC2(UC-2: Authenticate / Login)
        UC3(UC-3: Upload Document)
        UC4(UC-4: Validate File & Extract Text)
        UC5(UC-5: Read Document)
        UC6(UC-6: Adjust Accessibility Settings)
        UC7(UC-7: Listen via Text-to-Speech)
        UC8(UC-8: Highlight Active Sentence)
        UC9(UC-9: Request AI Simplification)
        UC10(UC-10: Manage Bookmarks & Notes)
        UC11(UC-11: Track Reading Analytics)
        UC12(UC-12: System Health & Maintenance)
    end

    actorUser --> UC1
    actorUser --> UC2
    actorUser --> UC3
    actorUser --> UC5
    actorUser --> UC6
    actorUser --> UC10
    actorUser --> UC11

    actorAdmin --> UC12
    actorAdmin --> UC2

    UC3 -.->|<<include>>| UC4
    UC5 -.->|<<extend>>| UC7
    UC7 -.->|<<include>>| UC8
    UC5 -.->|<<extend>>| UC9
    UC9 -.-> actorAI
    UC5 -.->|<<extend>>| UC10
```

### 2.2 Detailed Use Case Descriptions
1. **UC-2 (Authenticate / Login):** Reader submits credentials (Email or Phone + Password). System normalizes phone identifier, compares bcrypt hash, and issues a 7-day JWT token.
2. **UC-3 (Upload Document) with `<<include>>` UC-4:** Ingests PDF/TXT file; automatically triggers MIME validation, file size check ($\le 25\text{MB}$), generates unique storage key, and creates database record.
3. **UC-5 (Read Document) with `<<extend>>` UC-7, UC-9, UC-10:** Reading viewport dynamically loads text. The user may optionally trigger TTS (`<<extend>>`), ask AI for simplification (`<<extend>>`), or save bookmarks and notes (`<<extend>>`).
4. **UC-7 (Listen via TTS) with `<<include>>` UC-8:** Playing synthesized audio automatically requires real-time sentence-level DOM highlighting to keep visual focus synchronized.

---

## 3. Class Diagram

The Class Diagram illustrates the object-oriented structure, class attributes, method signatures, visibility constraints, and entity relationships within ReadAble.

### 3.1 Mermaid Specification
```mermaid
classDiagram
    class User {
        -int id
        -string name
        -string email
        -string phone
        -string passwordHash
        -DateTime createdAt
        -DateTime updatedAt
        +register(name, email, phone, password) User
        +login(identifier, password) string
        +getProfile() User
        +updatePreferences(prefs) void
    }

    class ReadingPreference {
        -int id
        -int userId
        -float fontScale
        -float lineHeight
        -float letterSpacing
        -string fontFamily
        -string themeMode
        -boolean focusRulerEnabled
        +applyPreferences() void
        +resetDefaults() void
    }

    class Document {
        -int id
        -string title
        -string fileName
        -string s3Key
        -int fileSize
        -string mimeType
        -int userId
        -DateTime createdAt
        -DateTime updatedAt
        +upload() boolean
        +delete() boolean
        +extractContent() DocumentChunk[]
        +getSummary() string
    }

    class DocumentChunk {
        -int id
        -int documentId
        -int chunkIndex
        -string textContent
        -int pageNumber
        +getSentences() string[]
    }

    class Bookmark {
        -int id
        -int userId
        -int documentId
        -int pageNumber
        -float scrollOffset
        -string label
        -DateTime createdAt
        +createBookmark() void
        +removeBookmark() void
    }

    class Annotation {
        -int id
        -int userId
        -int documentId
        -string selectedText
        -string noteContent
        -string highlightColor
        -DateTime createdAt
        +saveAnnotation() void
        +deleteAnnotation() void
    }

    class ReadingSession {
        -int id
        -int userId
        -int documentId
        -DateTime startTime
        -DateTime endTime
        -int durationMinutes
        -float progressPercentage
        +startSession() void
        +endSession() void
        +recordStreak() void
    }

    class AudioPlayerService {
        -SpeechSynthesis utterance
        -float playbackRate
        -string voiceUri
        -boolean isPlaying
        +play(text) void
        +pause() void
        +resume() void
        +setRate(rate) void
        +onBoundary(callback) void
    }

    class AIService {
        -string apiKey
        -string modelName
        +simplify(text) string
        +summarize(text) string
        +defineTerm(word, context) string
        +explainDiagram(imageKey) string
    }

    User "1" *-- "1" ReadingPreference : owns
    User "1" o-- "0..*" Document : manages
    User "1" o-- "0..*" Bookmark : creates
    User "1" o-- "0..*" Annotation : writes
    User "1" o-- "0..*" ReadingSession : logs
    Document "1" *-- "1..*" DocumentChunk : contains
    Document "1" o-- "0..*" Bookmark : referenced_by
    Document "1" o-- "0..*" Annotation : annotated_in
    Document "1" o-- "0..*" ReadingSession : tracked_in
    ReadingSession ..> AudioPlayerService : uses
    ReadingSession ..> AIService : consults
```

### 3.2 Design Rationale
- **Composition (`*--`):** A `User` strictly owns their `ReadingPreference`; a `Document` is composed of `DocumentChunk`s. If the parent document is deleted, the chunks are cascaded.
- **Aggregation (`o--`):** Documents, Bookmarks, and Sessions are associated with Users without strict lifecycle encapsulation (supports archiving).
- **Dependency (`..>`):** The `ReadingSession` client relies on stateless services `AudioPlayerService` and `AIService` to fulfill auxiliary multi-modal features.

---

## 4. Sequence Diagram: Document Ingestion and Reading Session

This sequence diagram depicts the chronological message exchange for document upload, text parsing, and starting an assisted reading session.

```mermaid
sequenceDiagram
    autonumber
    actor Reader as Reader / Student
    participant UI as React Frontend
    participant Auth as Auth Middleware
    participant DocCtrl as Document Controller
    participant Storage as File Storage (S3 / Disk)
    participant DB as Prisma / Database
    participant Audio as Web Speech Engine
    participant AI as AI Assistance Service

    Reader->>UI: Selects PDF & clicks "Upload"
    UI->>Auth: POST /api/documents/upload (Bearer JWT + FormData)
    Auth->>Auth: Validate JWT signature & claims
    Auth->>DocCtrl: Forward verified request (req.user)
    DocCtrl->>Storage: Store binary file & generate s3Key
    Storage-->>DocCtrl: Return file path & confirmation
    DocCtrl->>DB: INSERT INTO Document (title, s3Key, size, userId)
    DB-->>DocCtrl: Document record created (id: 42)
    DocCtrl-->>UI: 201 Created { docId: 42, title: "Deep Work.pdf" }
    UI-->>Reader: Display upload success in Library

    Reader->>UI: Clicks "Open & Read"
    UI->>DocCtrl: GET /api/documents/42
    DocCtrl->>DB: Query Document + Chunks
    DB-->>DocCtrl: Return document data
    DocCtrl-->>UI: 200 OK (Document text & metadata)
    UI-->>Reader: Render accessible viewport with custom typography

    Reader->>UI: Clicks "Play Audio (Read Aloud)"
    UI->>Audio: Initialize SpeechSynthesisUtterance(sentence)
    Audio-->>UI: onboundary(charIndex, sentenceIndex)
    UI->>UI: Highlight active sentence in DOM
    Audio-->>Reader: Audio speech output

    Reader->>UI: Highlights complex paragraph & clicks "Explain with AI"
    UI->>AI: POST /api/ai/simplify { text: selectedText }
    AI-->>UI: Return simplified explanation
    UI-->>Reader: Display explanation tooltip beside paragraph
```

---

## 5. Activity Diagram: Reading Session with Adaptive Accessibility

The Activity Diagram models the workflow and decision branches executed during a user's reading session.

```mermaid
graph TD
    Start([Start Reading Session]) --> LoadDoc[Load Document into Viewport]
    LoadDoc --> CheckPrefs{Stored User Preferences?}
    
    CheckPrefs -->|Yes| ApplyPrefs[Apply Font Scale, Theme, Line Height]
    CheckPrefs -->|No| ApplyDefault[Apply Default System Theme: Cream]
    
    ApplyPrefs --> ReadView[Display Text Viewport]
    ApplyDefault --> ReadView

    ReadView --> ActionChoice{User Action?}
    
    ActionChoice -->|Change Typography| AdjustUI[Modify Font Scale / Contrast Theme]
    AdjustUI --> UpdateCSS[Update Root CSS Variables --read-scale]
    UpdateCSS --> ReadView

    ActionChoice -->|Activate Read-Aloud| InitAudio[Fetch Next Sentence]
    InitAudio --> PlayTTS[Synthesize Speech via Audio Engine]
    PlayTTS --> HighLight[Highlight Matching Sentence in DOM]
    HighLight --> AudioCheck{Audio Finished?}
    AudioCheck -->|Next Sentence| InitAudio
    AudioCheck -->|Pause/Stop| ReadView

    ActionChoice -->|Encounter Jargon| SelectText[Select Word or Paragraph]
    SelectText --> QueryAI[Send Prompt to AI Service]
    QueryAI --> ShowTooltip[Render Contextual Simplification]
    ShowTooltip --> ReadView

    ActionChoice -->|Add Note / Bookmark| SaveAnnot[Save Bookmark & Scroll Offset to DB]
    SaveAnnot --> ReadView

    ActionChoice -->|Exit| EndSession[Calculate Time Spent & Update Streak]
    EndSession --> Finish([End Reading Session])
```

---

## 6. Draw.io Interchange File
The raw vector XML structure for this architecture is compiled in `Assignments/Assignment_3_UML_Diagrams.drawio`. It can be directly loaded into [app.diagrams.net](https://app.diagrams.net) or the VS Code Draw.io Integration extension for editing and visual presentation.
