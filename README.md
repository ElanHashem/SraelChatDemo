# Srael - AI-Powered Research & Argumentation Chatbot

A full-stack AI chatbot application designed to assist with research and forming arguments for writing and communications work. Built with FastAPI (Python) and Next.js (TypeScript), featuring robust authentication, conversation persistence, and structured AI responses.

**Currently in beta version for the organization CAMERA.**

![Python](https://img.shields.io/badge/Python-3.10+-blue.svg)
![Next.js](https://img.shields.io/badge/Next.js-14-black.svg)
![FastAPI](https://img.shields.io/badge/FastAPI-0.115-green.svg)
![TypeScript](https://img.shields.io/badge/TypeScript-5.0-blue.svg)
![SQLite](https://img.shields.io/badge/SQLite-3-lightgrey.svg)

---

## 📋 Table of Contents

- [Overview](#overview)
- [Features](#features)
- [Architecture](#architecture)
- [Tech Stack](#tech-stack)
- [Database Schema](#database-schema)
- [API Endpoints](#api-endpoints)
- [Authentication Flow](#authentication-flow)
- [AI Response Structure](#ai-response-structure)
- [Setup & Installation](#setup--installation)
- [Deployment](#deployment)
- [Analytics Capabilities](#analytics-capabilities)
- [Future Improvements](#future-improvements)

---

## 🎯 Overview

Srael is a specialized chatbot that generates structured argumentative responses for research and communications professionals. Unlike generic chatbots, it produces comprehensive responses that include:

- **Steelman arguments** - Charitable representations of opposing viewpoints
- **Main points** - Key arguments supporting a position
- **Rebuttals** - Direct responses to counterarguments
- **Caveats** - Important nuances and considerations
- **Citations** - Sources and references for fact-checking

This structure helps users understand multiple perspectives and craft well-informed, balanced communications.

---

## ✨ Features

### Core Functionality
- 🤖 **Structured AI Responses** - Generates organized arguments with opposing views, rebuttals, and citations
- 💬 **Conversation Persistence** - Full chat history saved and retrievable across sessions
- 🔍 **Smart Web Search** - Automatically searches for current events when queries mention recent topics
- 📊 **Usage Tracking** - Monitors API costs and token usage per request

### Authentication & Security
- 🔐 **JWT Authentication** - Secure token-based authentication with 30-day expiry
- 📧 **Email Verification** - 6-digit code verification via Resend API
- 📋 **Email Whitelist** - Admin-controlled access list for beta testing
- 🔒 **Password Requirements** - Enforces strong passwords (8+ chars, numbers, special characters)
- 🛡️ **CORS Protection** - Configurable origin restrictions

### User Experience
- 📱 **Responsive Design** - Works on desktop and mobile devices
- 📂 **Conversation Sidebar** - Easy navigation between chat threads
- ✏️ **Rename & Delete** - Manage conversation history
- 🎨 **Auto-Generated Titles** - AI creates descriptive titles for each conversation

---

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                           CLIENT (Vercel)                           │
│  ┌───────────────────────────────────────────────────────────────┐  │
│  │                    Next.js 14 Frontend                        │  │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐   │  │
│  │  │   Auth      │  │   Chat      │  │   Sidebar           │   │  │
│  │  │   Context   │  │   Component │  │   (Conversations)   │   │  │
│  │  └─────────────┘  └─────────────┘  └─────────────────────┘   │  │
│  └───────────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────────┘
                                   │
                                   │ HTTPS (JWT Auth)
                                   ▼
┌─────────────────────────────────────────────────────────────────────┐
│                          SERVER (Railway)                           │
│  ┌───────────────────────────────────────────────────────────────┐  │
│  │                    FastAPI Backend                            │  │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐   │  │
│  │  │   Auth      │  │   Chat      │  │   Admin             │   │  │
│  │  │   Routes    │  │   Routes    │  │   Routes            │   │  │
│  │  └─────────────┘  └─────────────┘  └─────────────────────┘   │  │
│  │         │                │                    │               │  │
│  │         ▼                ▼                    ▼               │  │
│  │  ┌─────────────────────────────────────────────────────────┐ │  │
│  │  │                   SQLite Database                       │ │  │
│  │  │   users | conversations | chat_history | whitelist      │ │  │
│  │  └─────────────────────────────────────────────────────────┘ │  │
│  └───────────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────────┘
                                   │
                    ┌──────────────┼──────────────┐
                    ▼              ▼              ▼
              ┌──────────┐  ┌──────────┐  ┌──────────┐
              │ OpenAI   │  │  Brave   │  │  Resend  │
              │ API      │  │  Search  │  │  Email   │
              └──────────┘  └──────────┘  └──────────┘
```

---

## 🛠️ Tech Stack

### Backend
| Technology | Purpose |
|------------|---------|
| **FastAPI** | High-performance Python web framework |
| **SQLite** | Lightweight database with WAL mode for concurrency |
| **JWT (python-jose)** | Secure token-based authentication |
| **bcrypt** | Password hashing |
| **OpenAI API** | GPT models for response generation |
| **Brave Search API** | Real-time web search for current events |
| **Resend** | Transactional email for verification |

### Frontend
| Technology | Purpose |
|------------|---------|
| **Next.js 14** | React framework with App Router |
| **TypeScript** | Type-safe JavaScript |
| **Tailwind CSS** | Utility-first styling |
| **React Context** | Global state management for auth |

### Infrastructure
| Service | Purpose |
|---------|---------|
| **Railway** | Backend hosting with persistent storage |
| **Vercel** | Frontend hosting with edge network |
| **Custom Domain** | Professional branding |

---

## 🗄️ Database Schema

```sql
┌─────────────────────┐       ┌─────────────────────┐
│       users         │       │    conversations    │
├─────────────────────┤       ├─────────────────────┤
│ id (PK)             │◄──┐   │ id (PK)             │
│ email (UNIQUE)      │   │   │ user_id (FK)────────┤
│ password_hash       │   │   │ title               │
│ email_verified      │   │   │ created_at          │
│ created_at          │   │   │ updated_at          │
└─────────────────────┘   │   └─────────────────────┘
                          │              │
┌─────────────────────┐   │              │
│   email_whitelist   │   │              │
├─────────────────────┤   │              ▼
│ id (PK)             │   │   ┌─────────────────────┐
│ email (UNIQUE)      │   │   │    chat_history     │
│ added_by (FK)───────┤   │   ├─────────────────────┤
│ added_at            │   │   │ id (PK)             │
│ notes               │   │   │ conversation_id (FK)│
└─────────────────────┘   │   │ user_id (FK)────────┘
                          │   │ message (TEXT)      │
                          │   │ response (JSON)     │
                          │   │ created_at          │
                          │   └─────────────────────┘
```

### Key Design Decisions

- **JSON Response Storage** - AI responses stored as JSON for flexibility and rich metadata
- **WAL Mode** - Write-Ahead Logging for better concurrent read/write performance
- **Cascade Deletes** - Deleting a conversation removes all associated messages
- **Indexed Queries** - Optimized lookups on `user_id` and `conversation_id`

---

## 🔌 API Endpoints

### Authentication
| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/auth/signup` | Create new account (requires whitelisted email) |
| `POST` | `/auth/login` | Authenticate and receive JWT |
| `GET` | `/auth/me` | Get current user info |
| `POST` | `/auth/verify-email` | Verify email with 6-digit code |
| `POST` | `/auth/resend-verification` | Resend verification code |

### Conversations
| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/conversations` | Create new conversation |
| `GET` | `/conversations` | List all user conversations |
| `GET` | `/conversations/{id}` | Get conversation with messages |
| `PUT` | `/conversations/{id}` | Rename conversation |
| `DELETE` | `/conversations/{id}` | Delete conversation |

### Chat
| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/chat` | Send message and receive AI response |

### Admin (Protected)
| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/admin/whitelist/add` | Add email to whitelist |
| `POST` | `/admin/whitelist/remove` | Remove email from whitelist |
| `GET` | `/admin/whitelist` | List all whitelisted emails |

---

## 🔐 Authentication Flow

```
┌──────────┐     ┌──────────┐     ┌──────────┐     ┌──────────┐
│  User    │     │ Frontend │     │ Backend  │     │  Resend  │
└────┬─────┘     └────┬─────┘     └────┬─────┘     └────┬─────┘
     │                │                │                │
     │  1. Sign Up    │                │                │
     │───────────────>│                │                │
     │                │  2. POST /auth/signup           │
     │                │───────────────>│                │
     │                │                │  3. Check      │
     │                │                │     Whitelist  │
     │                │                │                │
     │                │                │  4. Send Code  │
     │                │                │───────────────>│
     │                │                │                │
     │                │  5. Return JWT │                │
     │                │<───────────────│                │
     │                │                │                │
     │  6. Show Verification Modal     │                │
     │<───────────────│                │                │
     │                │                │                │
     │  7. Enter Code │                │                │
     │───────────────>│                │                │
     │                │  8. POST /auth/verify-email     │
     │                │───────────────>│                │
     │                │                │                │
     │                │  9. Verified!  │                │
     │                │<───────────────│                │
     │                │                │                │
     │  10. Access    │                │                │
     │      Granted   │                │                │
     │<───────────────│                │                │
     │                │                │                │
```

---

## 🤖 AI Response Structure

Each chat response follows a structured JSON format:

```json
{
  "stance": "pro_position",
  "steelman": "The opposing view argues that...",
  "main_points": [
    "First key argument supporting the position",
    "Second key argument with evidence",
    "Third argument addressing common concerns"
  ],
  "rebuttals": [
    "Response to counterargument #1",
    "Response to counterargument #2"
  ],
  "caveats": "Important nuances to consider...",
  "citations": [
    {
      "title": "Source Title",
      "url": "https://example.com/source",
      "publisher": "Publisher Name"
    }
  ],
  "tone_notes": "Suggested tone for communication",
  "word_count": 245,
  "_meta": {
    "model": "gpt-4o-mini",
    "input_tokens": 150,
    "output_tokens": 320,
    "estimated_cost_usd": 0.000215,
    "used_web_search": false,
    "conversation_id": 42
  }
}
```

### Response Components Explained

| Field | Purpose |
|-------|---------|
| `steelman` | Presents the strongest version of the opposing argument |
| `main_points` | Core arguments supporting the position |
| `rebuttals` | Direct responses to likely counterarguments |
| `caveats` | Acknowledges nuances and limitations |
| `citations` | Verifiable sources for fact-checking |
| `_meta` | Usage tracking (model, tokens, cost) |

---

## 🚀 Setup & Installation

### Prerequisites
- Python 3.10+
- Node.js 18+
- OpenAI API key
- Brave Search API key (optional)
- Resend API key

### Backend Setup

```bash
# Clone and navigate to backend
cd backend

# Create virtual environment
python -m venv venv
source venv/bin/activate  # Windows: venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt

# Configure environment
cp .env.example .env
# Edit .env with your API keys

# Run development server
uvicorn app:app --reload --port 8000
```

### Frontend Setup

```bash
# Navigate to frontend
cd frontend

# Install dependencies
npm install

# Configure environment
cp .env.local.example .env.local
# Edit with your backend URL

# Run development server
npm run dev
```

### Environment Variables

**Backend (`.env`)**
```env
OPENAI_API_KEY=sk-your-openai-key
JWT_SECRET_KEY=your-long-random-secret-key
RESEND_API_KEY=re_your-resend-key
FROM_EMAIL=noreply@yourdomain.com
BRAVE_API_KEY=your-brave-key
SOFT_LIMIT_USD=10.0
HARD_LIMIT_USD=20.0
```

**Frontend (`.env.local`)**
```env
NEXT_PUBLIC_API_URL=http://localhost:8000
```

---

## ☁️ Deployment

### Railway (Backend)

1. Connect your GitHub repository
2. Set environment variables in Railway dashboard
3. Configure persistent volume for SQLite at `/app/data`
4. Deploy automatically on push

### Vercel (Frontend)

1. Import project from GitHub
2. Set `NEXT_PUBLIC_API_URL` to your Railway URL
3. Deploy automatically on push

### Custom Domain Setup

- **API**: `api.yourdomain.com` → Railway
- **Frontend**: `yourdomain.com` → Vercel

---

## 📊 Analytics Capabilities

The database structure enables rich analytics:

### User Engagement
```sql
-- Messages per user
SELECT user_id, COUNT(*) as message_count
FROM chat_history
GROUP BY user_id
ORDER BY message_count DESC;

-- Daily active users
SELECT DATE(created_at) as day, COUNT(DISTINCT user_id) as dau
FROM chat_history
GROUP BY DATE(created_at);
```

### Cost Analysis
```sql
-- Total spend by model
SELECT 
    JSON_EXTRACT(response, '$._meta.model') as model,
    SUM(JSON_EXTRACT(response, '$._meta.estimated_cost_usd')) as total_cost
FROM chat_history
GROUP BY model;
```

### Content Analysis
- Topic modeling on user queries
- Sentiment analysis on conversations
- Peak usage time identification
- Most common question patterns

---

## 🔮 Future Improvements

- [ ] **Rate Limiting** - Prevent brute-force attacks on login
- [ ] **PostgreSQL Migration** - Better scalability for production
- [ ] **Admin Dashboard** - Web UI for whitelist management
- [ ] **Export Conversations** - PDF/Markdown export for users
- [ ] **Feedback System** - Thumbs up/down on responses
- [ ] **Analytics Dashboard** - Visualize usage patterns
- [ ] **Multi-language Support** - Internationalization
- [ ] **Response Caching** - Reduce API costs for similar queries

---

## 📄 License

This project is proprietary software developed for CAMERA.

---

## 👤 Author

Developed by [Your Name]

- GitHub: [@yourusername](https://github.com/yourusername)
- LinkedIn: [Your LinkedIn](https://linkedin.com/in/yourprofile)

---

## 🙏 Acknowledgments

- [FastAPI](https://fastapi.tiangolo.com/) - Modern Python web framework
- [Next.js](https://nextjs.org/) - React framework
- [OpenAI](https://openai.com/) - GPT models
- [Railway](https://railway.app/) - Backend hosting
- [Vercel](https://vercel.com/) - Frontend hosting
