# Srael - AI Research Assistant

A full-stack AI research assistant that answers questions with structured, well-sourced responses. Every citation points to a real article from a curated library, so you get answers you can actually check. Built with FastAPI (Python) and Next.js (TypeScript), with retrieval-augmented generation (RAG), semantic search, authentication, and saved conversations.

![Python](https://img.shields.io/badge/Python-3.10+-blue.svg)
![Next.js](https://img.shields.io/badge/Next.js-14-black.svg)
![FastAPI](https://img.shields.io/badge/FastAPI-0.115-green.svg)
![TypeScript](https://img.shields.io/badge/TypeScript-5.0-blue.svg)
![SQLite](https://img.shields.io/badge/SQLite-FTS5-lightgrey.svg)

**Currently in beta version for the organization CAMERA.**

> This repository is a public showcase of the Srael project. The production source code is private.

---

## 🎬 Demo

[![Demo Video](https://img.youtube.com/vi/foeWvQL3Xv4/maxresdefault.jpg)](https://youtu.be/foeWvQL3Xv4)

*Click the image above to watch the demo*

---

## 📋 Table of Contents

- [Demo](#-demo)
- [Overview](#-overview)
- [Features](#-features)
- [How It Works: The RAG Pipeline](#-how-it-works-the-rag-pipeline)
- [Architecture](#️-architecture)
- [Tech Stack](#️-tech-stack)
- [Database Schema](#️-database-schema)
- [Authentication Flow](#-authentication-flow)
- [AI Response Structure](#-ai-response-structure)
- [Analytics Capabilities](#-analytics-capabilities)

---

## 🎯 Overview

Srael helps researchers, writers, and students dig into a topic fast without having to second-guess the sources. Ask a question and you get back a structured answer:

- **Steelman**: the strongest, fairest version of the opposing view
- **Main points**: the key arguments and evidence
- **Rebuttals**: responses to likely counterarguments
- **Caveats**: nuances and limitations worth knowing
- **Citations**: numbered, clickable links to the real articles the answer drew on

### The problem it solves

General-purpose chatbots often **hallucinate citations**. They invent URLs that look plausible but return a 404, or cite articles that were never written. For a tool meant to produce citable research, that's a dealbreaker.

Srael fixes this by grounding every answer in a local library of real, pre-fetched articles. The model only sees, and can only cite, sources that actually exist.

| | Generic chatbot | Srael |
|---|---|---|
| **Source of facts** | Training data only | Curated article library |
| **URLs** | Often fabricated | Real, verified, clickable |
| **Retrieval** | None | Semantic search + BM25 fallback |
| **Citation accuracy** | Unreliable | Validated against retrieved sources |
| **Consistency** | Varies per query | Grounded in a fixed corpus |

---

## ✨ Features

### Research & Retrieval
- 🧠 **Semantic Search**: articles are split into chunks and embedded, so retrieval matches on meaning, not just keywords
- 🔎 **BM25 Full-Text Fallback**: SQLite FTS5 ranking takes over when embeddings aren't available
- ✍️ **Query Rewriting**: an LLM step fixes typos and turns casual questions into search-optimized queries, using conversation context for follow-ups
- 📚 **Verified Citations**: every citation the model returns is checked against the retrieved sources; anything unverified gets dropped
- 🔢 **Clean Citation Numbering**: citations are renumbered in order of first use and deduplicated, so the list never has gaps or repeats
- 📌 **Curated Knowledge Base**: hand-picked articles can be pinned to specific topics

### Conversations
- 💬 **Persistent History**: every conversation is saved and can be reopened later
- 🔁 **Context-Aware Follow-ups**: recent messages are included so follow-up questions just work
- 🎨 **Auto-Generated Titles**: the AI names each conversation
- ✏️ **Rename & Delete**: manage threads from the sidebar
- 📏 **Adjustable Length**: choose how long you want responses to be

### Authentication & Security
- 🔐 **JWT Authentication**: token-based sessions
- 📧 **Email Verification**: 6-digit codes sent through Resend
- 📋 **Invite Whitelist**: admin-controlled access during beta
- 🔒 **Password Requirements**: 8+ characters with numbers and special characters
- 🛡️ **CORS Protection**: restricted allowed origins
- 💰 **Budget Guardrails**: soft and hard spending caps on API usage

---

## 🔬 How It Works: The RAG Pipeline

```
 User question
      │
      ▼
┌───────────────────────┐
│ 1. Query Rewriting    │  LLM fixes spelling, adds conversation context,
│                       │  produces a search-optimized query
└──────────┬────────────┘
           ▼
┌───────────────────────┐
│ 2. Curated KB Lookup  │  Topic-pinned, hand-picked articles
└──────────┬────────────┘
           ▼
┌───────────────────────┐
│ 3. Semantic Chunk     │  Query embedded → cosine similarity against
│    Search             │  ~3,000-char article chunks → top 12 chunks,
│                       │  grouped by source article
└──────────┬────────────┘
           │  (no results?)
           ▼
┌───────────────────────┐
│ 4. FTS5 / BM25        │  Keyword relevance ranking over full articles
│    Fallback           │
└──────────┬────────────┘
           ▼
┌───────────────────────┐
│ 5. Context Assembly   │  Sources numbered [1]..[N], trimmed to fit
│                       │  the token budget, passed to the LLM
└──────────┬────────────┘
           ▼
┌───────────────────────┐
│ 6. Generation         │  Structured JSON answer citing only
│                       │  the provided sources
└──────────┬────────────┘
           ▼
┌───────────────────────┐
│ 7. Citation           │  Unverified URLs dropped, numbers renumbered
│    Validation         │  sequentially, duplicates merged
└──────────┬────────────┘
           ▼
     Response to user
```

### Building the article library

Articles are collected ahead of time from a list of trusted publishers, then cleaned (ads, navigation, and scripts stripped) and stored in SQLite. An offline chunking script then:

1. Splits each article into overlapping chunks (3,000 characters with 400 characters of overlap, breaking at sentence boundaries)
2. Generates an embedding for each chunk with `text-embedding-3-small`
3. Stores the chunks and embeddings alongside the articles

Since all content is fetched in advance, nothing is scraped while a user waits. That keeps responses fast and consistent, and sidesteps sites that block automated requests.

### Why chunking?

Whole-article retrieval tends to fill the context window with irrelevant paragraphs. Chunk-level retrieval pulls only the passages that answer the question, which means:
- Better answers from more focused context
- More distinct sources fit in a single prompt
- Lower token costs

---

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                           CLIENT (Vercel)                           │
│  ┌───────────────────────────────────────────────────────────────┐  │
│  │                    Next.js 14 Frontend                        │  │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐    │  │
│  │  │   Auth      │  │   Chat      │  │   Sidebar           │    │  │
│  │  │   Context   │  │   Component │  │   (Conversations)   │    │  │
│  │  └─────────────┘  └─────────────┘  └─────────────────────┘    │  │
│  └───────────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────────┘
                                   │
                                   │ HTTPS (JWT Auth)
                                   ▼
┌─────────────────────────────────────────────────────────────────────┐
│                          SERVER (Railway)                           │
│  ┌───────────────────────────────────────────────────────────────┐  │
│  │                    FastAPI Backend                            │  │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────────┐   │  │
│  │  │  Auth    │  │  Chat    │  │  Admin   │  │  RAG Engine  │   │  │
│  │  │  Routes  │  │  Routes  │  │  Routes  │  │  (search.py) │   │  │
│  │  └──────────┘  └──────────┘  └──────────┘  └──────────────┘   │  │
│  │                         │                                     │  │
│  │                         ▼                                     │  │
│  │  ┌─────────────────────────────────────────────────────────┐  │  │
│  │  │                  SQLite (WAL mode)                      │  │  │
│  │  │  users │ conversations │ chat_history │ email_whitelist │  │  │
│  │  │  cached_articles │ cached_articles_fts │ article_chunks │  │  │
│  │  └─────────────────────────────────────────────────────────┘  │  │
│  └───────────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────────┘
                                   │
                    ┌──────────────┴──────────────┐
                    ▼                             ▼
          ┌──────────────────┐          ┌──────────────────┐
          │   OpenAI API     │          │   Resend Email   │
          │ (chat+embeddings)│          │                  │
          └──────────────────┘          └──────────────────┘
```

---

## 🛠️ Tech Stack

### Backend
| Technology | Purpose |
|------------|---------|
| **FastAPI** | Async Python web framework |
| **SQLite + FTS5** | App data, article library, and BM25 full-text index |
| **OpenAI Chat API** | Query rewriting, titles, and structured answer generation |
| **OpenAI Embeddings** | `text-embedding-3-small` for semantic chunk search |
| **JWT (python-jose)** | Token-based authentication |
| **bcrypt (passlib)** | Password hashing |
| **BeautifulSoup** | HTML cleaning when building the article library |
| **httpx** | Async HTTP client |
| **Resend** | Transactional email for verification |

### Frontend
| Technology | Purpose |
|------------|---------|
| **Next.js 14** | React framework with App Router |
| **TypeScript** | Type-safe JavaScript |
| **Tailwind CSS** | Utility-first styling |
| **React Context** | Global auth state |

### Infrastructure
| Service | Purpose |
|---------|---------|
| **Railway** | Backend hosting with a persistent volume for SQLite |
| **Vercel** | Frontend hosting |

---

## 🗄️ Database Schema

### Application data

```
┌─────────────────────┐       ┌─────────────────────┐
│       users         │       │    conversations    │
├─────────────────────┤       ├─────────────────────┤
│ id (PK)             │◄──┐   │ id (PK)             │
│ email (UNIQUE)      │   ├───┤ user_id (FK)        │
│ password_hash       │   │   │ title               │
│ email_verified      │   │   │ created_at          │
│ created_at          │   │   │ updated_at          │
└─────────────────────┘   │   └─────────┬───────────┘
                          │             │
┌─────────────────────┐   │             ▼
│   email_whitelist   │   │   ┌─────────────────────┐
├─────────────────────┤   │   │    chat_history     │
│ id (PK)             │   │   ├─────────────────────┤
│ email (UNIQUE)      │   │   │ id (PK)             │
│ added_by (FK)       │   │   │ conversation_id (FK)│
│ added_at            │   └───┤ user_id (FK)        │
│ notes               │       │ message (TEXT)      │
└─────────────────────┘       │ response (JSON)     │
                              │ created_at          │
                              └─────────────────────┘
```

### Article library

```
┌──────────────────────┐      ┌──────────────────────┐
│   cached_articles    │      │ cached_articles_fts  │
├──────────────────────┤      │   (FTS5 virtual)     │
│ id (PK)              │─────►├──────────────────────┤
│ url (UNIQUE)         │      │ url (unindexed)      │
│ url_hash             │      │ domain (unindexed)   │
│ domain               │      │ title                │
│ title                │      │ content              │
│ content              │      └──────────────────────┘
│ content_length       │
│ fetched_at           │      ┌──────────────────────┐
│ last_used_at         │      │    article_chunks    │
│ times_used           │      ├──────────────────────┤
└──────────────────────┘      │ id (PK)              │
                              │ article_url          │
                              │ article_title        │
                              │ domain               │
                              │ chunk_index          │
                              │ chunk_text           │
                              │ embedding (BLOB)     │
                              │ char_start, char_end │
                              │ created_at           │
                              └──────────────────────┘
```

### Key design decisions

- **One database file**: app data, articles, the full-text index, and embeddings all live in one SQLite file on a persistent volume. That keeps deployment simple.
- **Embeddings as BLOBs**: vectors are stored as packed floats and compared with cosine similarity in Python. At this corpus size there's no need for a separate vector database.
- **FTS5 with BM25**: SQLite's built-in full-text engine gives relevance-ranked keyword search with no extra infrastructure.
- **JSON response storage**: AI responses are stored whole, metadata included, which makes them easy to analyze later.
- **WAL mode**: better concurrent read/write performance.
- **Cascade deletes**: deleting a conversation also removes its messages.

---

## 🔐 Authentication Flow

```
┌──────────┐     ┌──────────┐     ┌──────────┐     ┌──────────┐
│  User    │     │ Frontend │     │ Backend  │     │  Resend  │
└────┬─────┘     └────┬─────┘     └────┬─────┘     └────┬─────┘
     │  1. Sign Up    │                │                │
     │───────────────>│                │                │
     │                │ 2. POST /auth/signup            │
     │                │───────────────>│                │
     │                │                │ 3. Check       │
     │                │                │    whitelist   │
     │                │                │ 4. Send code   │
     │                │                │───────────────>│
     │                │ 5. Return JWT  │                │
     │                │<───────────────│                │
     │ 6. Verification modal           │                │
     │<───────────────│                │                │
     │ 7. Enter code  │                │                │
     │───────────────>│                │                │
     │                │ 8. POST /auth/verify-email      │
     │                │───────────────>│                │
     │                │ 9. Verified    │                │
     │                │<───────────────│                │
     │ 10. Access     │                │                │
     │     granted    │                │                │
     │<───────────────│                │                │
```

---

## 🤖 AI Response Structure

Each chat response comes back as structured JSON. Citation markers like `[1]` in the text line up with entries in the `citations` array:

```json
{
  "steelman": "The strongest opposing view holds that... [2]",
  "main_points": [
    "First key point, supported by evidence [1]",
    "Second point drawing on another source [2]",
    "Third point addressing a common concern [3]"
  ],
  "rebuttals": [
    "Response to counterargument #1 [1]",
    "Response to counterargument #2 [3]"
  ],
  "caveats": "Important nuances to keep in mind...",
  "citations": [
    { "title": "Source Article One",   "url": "https://example.com/a", "publisher": "example.com" },
    { "title": "Source Article Two",   "url": "https://example.org/b", "publisher": "example.org" },
    { "title": "Source Article Three", "url": "https://example.net/c", "publisher": "example.net" }
  ],
  "tone_notes": "Suggested tone for communication",
  "word_count": 245,
  "_meta": {
    "model": "gpt-5.1",
    "input_tokens": 18450,
    "output_tokens": 620,
    "estimated_cost_usd": 0.029263,
    "sources_searched": 7,
    "citations_verified": true,
    "conversation_id": 42,
    "is_first_message": false
  }
}
```

| Field | Purpose |
|-------|---------|
| `steelman` | The strongest version of the opposing argument |
| `main_points` | Core arguments and evidence |
| `rebuttals` | Responses to likely counterarguments |
| `caveats` | Nuances and limitations |
| `citations` | Verified sources, numbered in order of first use |
| `_meta` | Model, token usage, cost, and retrieval stats |

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
