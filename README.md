# EduQuest — Gamified Learning Platform

[![Microsoft Azure](https://img.shields.io/badge/Microsoft-Azure-0078D4?logo=microsoft-azure)](https://azure.microsoft.com/)
[![OpenAI](https://img.shields.io/badge/OpenAI-gpt--5--nano-412991?logo=openai)](https://openai.com/)
[![FastAPI](https://img.shields.io/badge/FastAPI-0.111-009688?logo=fastapi)](https://fastapi.tiangolo.com/)
[![React](https://img.shields.io/badge/React-18-61DAFB?logo=react)](https://react.dev/)
[![License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

> **Upload your study materials. Get AI-powered bite-sized lessons. Earn XP. Beat your classmates.**

EduQuest transforms passive study materials (PDF, PPTX, DOCX, EPUB, TXT, images) into interactive gamified lessons with an AI tutor grounded strictly in your own uploaded content.

Built for the **Microsoft Hackathon** on the Azure + Microsoft ecosystem.

---

## What It Does

| Feature | Description |
|---------|-------------|
| **File Upload** | Upload PDF, PPTX, DOCX, EPUB, TXT, or handwritten images |
| **AI Lesson Generation** | GPT auto-generates 3 lessons per document, each with MCQ, True/False, and Fill-in-blank activities |
| **Gamified Quiz** | Answer questions, earn XP, see animated feedback, track streaks |
| **RAG AI Tutor** | Ask questions answered strictly from your uploaded materials, with citations |
| **Multilingual** | Tutor responds in English, Hindi, Spanish, French, Arabic, Mandarin |
| **Resource Discovery** | AI suggests web resources; you must explicitly confirm before adding to context |
| **Leaderboard** | Weekly XP rankings with your classmates |
| **Demo Ready** | Newton's Laws pre-loaded on startup — no upload needed for the demo |

---

## Architecture

### Hackathon MVP Stack (running today)

```
┌─────────────────────────────────────┐
│   Frontend — React 18 + Vite        │  port 5173
│   Tailwind CSS · Zustand · shadcn   │
└──────────────┬──────────────────────┘
               │ /api/* proxied
┌──────────────▼──────────────────────┐
│   Backend — FastAPI + uvicorn       │  port 8000
│   Python 3.12                       │
│                                     │
│  ┌─────────────┐  ┌──────────────┐  │
│  │ OpenAI API  │  │ In-memory    │  │
│  │ gpt-5-nano  │  │ vector store │  │
│  │ embed-3-sm  │  │ (numpy)      │  │
│  └─────────────┘  └──────────────┘  │
│                                     │
│  ┌─────────────┐  ┌──────────────┐  │
│  │ Document    │  │ Local file   │  │
│  │ processor   │  │ storage      │  │
│  │ (pypdf etc) │  │ ./uploads/   │  │
│  └─────────────┘  └──────────────┘  │
└─────────────────────────────────────┘
```

### Production Azure Stack (post-hackathon migration path)

| MVP Component | Azure Equivalent |
|---------------|-----------------|
| OpenAI API | Azure OpenAI Service (GPT-4o, GPT-4o-mini) |
| In-memory vector store | Azure AI Search (HNSW vector index) |
| Local file storage | Azure Blob Storage + Event Grid |
| Python dicts (DB) | Azure Cosmos DB (serverless) |
| Local file server | Azure Functions (Python v4) |
| React + Vite (local) | Azure Static Web Apps |
| Mock resource discovery | Bing Search API + GPT evaluation |
| Console logs | Azure Monitor + Application Insights |

---

## Quick Start

### Prerequisites
- Python 3.12 (not 3.13/3.14 — binary wheels not yet available)
- Node.js 18+
- An OpenAI API key (`sk-...`)

### 1. Clone and configure

```bash
cd Azure/EduQuest

# Create .env with your API key
cp .env.example .env
# Edit .env and set:  OPENAI_API_KEY=sk-...
```

### 2. Backend

```bash
cd backend

# Create Python 3.12 virtual environment (first time only)
python3.12 -m venv .venv
.venv/bin/pip install -r requirements.txt

# Start server (auto-loads .env + pre-seeds Newton's Laws demo doc)
.venv/bin/uvicorn main:app --reload --port 8000
```

Backend is live at `http://localhost:8000`
Swagger docs at `http://localhost:8000/docs`

### 3. Frontend

```bash
cd frontend
npm install          # first time only
npm run dev
```

Frontend is live at `http://localhost:5173`

### Or use the startup script

```bash
# From the project root (Azure/EduQuest/)
./start.sh
```

---

## Environment Variables

Copy `.env.example` to `.env` and fill in:

```env
# Required
OPENAI_API_KEY=sk-...

# Optional — overrides model default (gpt-5-nano)
OPENAI_MODEL=gpt-5-nano
```

All other variables in `.env.example` are Azure service equivalents for the production migration path.

---

## Project Structure

```
Azure/EduQuest/
├── .env.example              # Environment variable template
├── .gitignore
├── README.md                 # This file
│
├── backend/
│   ├── main.py               # FastAPI app — all routes
│   ├── requirements.txt      # Python dependencies
│   ├── .venv/                # Python 3.12 virtualenv (gitignored)
│   │
│   ├── services/
│   │   ├── simple_vector_store.py   # In-memory cosine similarity search
│   │   ├── document_processor.py   # PDF/PPTX/DOCX/EPUB/TXT/image parser
│   │   ├── chunker.py              # tiktoken-based text chunker
│   │   └── lesson_generator.py     # GPT lesson + activity generation
│   │
│   ├── prompts/
│   │   └── lesson_plan.py          # LESSON_PLAN_SYSTEM + LESSON_PLAN_USER
│   │
│   ├── models/
│   │   └── __init__.py
│   │
│   ├── sample_data/
│   │   └── newtons_laws.txt        # Pre-loaded demo document
│   │
│   └── uploads/                    # Uploaded files (gitignored)
│
└── frontend/
    ├── package.json
    ├── vite.config.ts          # Vite + /api proxy → :8000
    ├── tailwind.config.js
    ├── index.html
    └── src/
        ├── App.tsx             # React Router routes
        ├── main.tsx
        ├── index.css           # Tailwind base
        ├── types/index.ts      # TypeScript interfaces
        ├── lib/
        │   ├── api.ts          # Typed API client
        │   └── utils.ts        # cn, formatXP, etc.
        ├── stores/             # Zustand state
        │   ├── authStore.ts
        │   ├── lessonStore.ts
        │   └── quizStore.ts
        ├── components/
        │   ├── layout/         # Navbar, Sidebar, Layout
        │   ├── quiz/           # QuestionCard, XPAnimation, ResultScreen
        │   ├── resources/      # ResourceCard, ResourceConfirmDialog
        │   └── ui/             # Skeleton loaders
        └── pages/
            ├── DashboardPage.tsx
            ├── UploadPage.tsx
            ├── LessonsPage.tsx
            ├── QuizPage.tsx
            ├── TutorPage.tsx
            ├── LeaderboardPage.tsx
            ├── SocialPage.tsx
            └── ProfilePage.tsx
```

---

## API Reference

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/health` | Health check |
| `GET` | `/api/demo/status` | Pre-loaded demo document info |
| `POST` | `/api/upload` | Upload and process a file |
| `GET` | `/api/documents` | List all uploaded documents |
| `GET` | `/api/documents/{id}` | Get document metadata |
| `GET` | `/api/documents/{id}/status` | Get processing status |
| `DELETE` | `/api/documents/{id}` | Delete document + chunks |
| `POST` | `/api/lessons/{docId}/generate` | Generate 3 lessons from document |
| `GET` | `/api/lessons/{docId}` | Get generated lessons |
| `GET` | `/api/lesson/{lessonId}` | Get single lesson |
| `POST` | `/api/quiz/{lessonId}/start` | Start a quiz for a lesson |
| `POST` | `/api/quiz/submit` | Submit an answer, receive XP |
| `POST` | `/api/tutor/ask` | RAG tutor query |
| `POST` | `/api/resources/discover` | Discover web resources for a topic |
| `POST` | `/api/resources/confirm` | Confirm adding a resource to context |
| `GET` | `/api/leaderboard` | Weekly XP leaderboard |
| `GET` | `/api/user/profile` | Get user profile |
| `PUT` | `/api/user/profile` | Update display name / language |

---

## Demo Script (3 Minutes)

**Pre-demo setup** (30s before presenting):
```bash
./start.sh     # both services running
# Backend auto-loads Newton's Laws — no upload needed
```

1. **(30s)** Open `localhost:5173` — show Dashboard with "Demo ready!" banner
2. **(30s)** Click "Try Demo →" — Lessons page — click "Generate Lessons"
3. **(60s)** Open Lesson 1 → Start Quiz → Answer 3 questions → show XP animation + result screen
4. **(30s)** AI Tutor → ask "Explain Newton's Second Law in simple terms" → show cited answer
5. **(15s)** Change language to Hindi → ask again → show multilingual response
6. **(15s)** "Discover related resources" → show resource cards → confirm one
7. **(15s)** Leaderboard → "Built on Azure. Works for any document, any language."

---

## Gamification Design

| Mechanic | Detail |
|----------|--------|
| XP per correct answer | Easy: 10 XP · Medium: 15 XP · Hard: 20 XP |
| Streak | Day counter shown in navbar flame pill |
| Leaderboard | Weekly XP, live-updated on quiz submit |
| Result grading | Perfect (100%) / Great (≥80%) / Good (≥60%) / Keep Going |
| XP animation | Floating gold "+XP" text on correct answer |
| Navbar pulse | XP pill bounces whenever XP increases |

---

## Microsoft Hackathon

This project showcases the Microsoft AI + Azure ecosystem:

- **OpenAI / Azure OpenAI** — `gpt-5-nano` for lesson generation, RAG tutor, resource evaluation
- **Azure Static Web Apps** — React frontend hosting (production path)
- **Azure Functions** — Python backend (production path)
- **Azure AI Search** — Vector search (production path)
- **Azure Cosmos DB** — NoSQL data store (production path)
- **Azure Blob Storage** — File storage (production path)
- **Azure AI Document Intelligence** — OCR + PDF extraction (production path)
- **Bing Search API** — Web resource discovery (production path)
- **Azure SignalR Service** — Real-time Quick Challenges (production path)
- **Microsoft Entra ID** — Authentication (production path)

The MVP uses direct OpenAI API + local in-memory stores to eliminate provisioning time during the hackathon. Every component has a documented 1:1 Azure equivalent and the swap is a config-level change.

---

## License

MIT — see [LICENSE](LICENSE)
