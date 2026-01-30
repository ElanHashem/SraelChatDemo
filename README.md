# Srael - AI-Powered Research & Argumentation Chatbot

A full-stack AI chatbot application designed to assist with research and forming arguments for writing and communications work. Built with FastAPI (Python) and Next.js (TypeScript), featuring robust authentication, conversation persistence, and structured AI responses.

**Currently in beta version for the organization CAMERA.**

![Python](https://img.shields.io/badge/Python-3.10+-blue.svg)
![Next.js](https://img.shields.io/badge/Next.js-14-black.svg)
![FastAPI](https://img.shields.io/badge/FastAPI-0.115-green.svg)
![TypeScript](https://img.shields.io/badge/TypeScript-5.0-blue.svg)
![SQLite](https://img.shields.io/badge/SQLite-3-lightgrey.svg)

---

## 🎬 Demo

[![Demo Video](https://img.youtube.com/vi/foeWvQL3Xv4/maxresdefault.jpg)](https://youtu.be/foeWvQL3Xv4)

*Click the image above to watch the demo*

---

## 📋 Table of Contents

- [Demo](#demo)
- [Overview](#overview)
- [Features](#features)
- [Architecture](#architecture)
- [Tech Stack](#tech-stack)
- [Database Schema](#database-schema)
- [Authentication Flow](#authentication-flow)
- [AI Response Structure](#ai-response-structure)
- [Analytics Capabilities](#analytics-capabilities)

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

Each chat response follows a structured format with the following components:

| Field | Purpose |
|-------|---------|
| `steelman` | Presents the strongest version of the opposing argument |
| `main_points` | Core arguments supporting the position |
| `rebuttals` | Direct responses to likely counterarguments |
| `caveats` | Acknowledges nuances and limitations |
| `citations` | Verifiable sources for fact-checking |
| `_meta` | Usage tracking (model, tokens, cost) |

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

## 👤 Author

Developed by Elan Hashem

