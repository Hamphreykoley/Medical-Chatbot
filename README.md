# Doctor_LLM (MedBot)

A Retrieval-Augmented Generation (RAG) system that answers medical-information questions using approved, ingested medical documents. Built with Flask, Pinecone, OpenAI, and Supabase — with JWT authentication, PDF ingestion, and source-cited, disclaimer-backed answers.

> ⚠️ **Status: In Progress** — core architecture is defined (see [SRS](#documentation)); backend implementation is under active development. Not yet deployed.

---

## ⚕️ Safety Notice

Doctor_LLM provides **educational information only**. It is not a doctor, does not diagnose conditions, does not prescribe treatment, and is not a substitute for urgent or professional medical care. Every response includes a disclaimer and directs users to seek professional or emergency help where relevant.

---

## Overview

Doctor_LLM retrieves relevant passages from a vector database and uses an LLM to generate grounded, context-aware answers to medical-information questions — rather than relying on the model's raw parametric knowledge alone. The current knowledge base is built from the **Encyclopedia of Medicine, Vol. 3**, chunked and embedded for semantic retrieval.

## Features

- 🔐 **JWT-based authentication** — registration, login, and protected endpoints with bcrypt-hashed passwords
- 📄 **PDF ingestion pipeline** — extraction (PyPDF), configurable chunking, and embedding generation
- 🔎 **Retrieval-Augmented Generation** — top-K semantic retrieval from Pinecone, grounded LLM responses via LangChain + OpenAI
- 📚 **Source-aware answers** — responses cite the source document and page when available
- 🩺 **Built-in safety disclaimers** — every answer flags itself as general information, not medical advice, with emergency guidance where relevant
- 🗂️ **Conversation history** — authenticated users can retrieve their own chat history (Supabase-backed)
- 📝 **Structured logging** — console and file logging with configurable log levels, no duplicate handlers
- 🛡️ **Security-conscious defaults** — secrets isolated via `.env`, no stack traces or internal errors exposed to clients

## Tech Stack

| Area | Technology |
|---|---|
| Backend API | Flask, Flask-CORS |
| LLM & Embeddings | OpenAI / LangChain OpenAI |
| RAG Orchestration | LangChain |
| Vector Database | Pinecone |
| PDF Parsing | PyPDF |
| Database / User Platform | Supabase |
| Authentication | PyJWT, bcrypt |
| Configuration | python-dotenv |
| Logging | Python `logging`, colorlog |

## Architecture

```
PDF Document (Encyclopedia of Medicine, Vol. 3)
        │
        ▼
  Text Extraction (PyPDF)
        │
        ▼
  Chunking (configurable size/overlap)
        │
        ▼
  Embedding (OpenAI text-embedding-3-small)
        │
        ▼
  Pinecone Vector Index
        │
        ▼
User Question ──▶ Embed Query ──▶ Top-K Retrieval ──▶ LLM (OpenAI) ──▶ Grounded Answer + Sources + Disclaimer
```

## API Reference

| Method | Endpoint | Auth | Purpose |
|---|---|---|---|
| `POST` | `/api/auth/register` | No | Create user account |
| `POST` | `/api/auth/login` | No | Authenticate and return JWT |
| `POST` | `/api/documents/upload` | Admin | Upload and ingest a PDF |
| `POST` | `/api/chat` | User | Ask a question, receive RAG response |
| `GET` | `/api/chat/history` | User | Retrieve current user's chat history |
| `GET` | `/api/health` | No | Service health check |

**Example — `POST /api/chat`**

Request:
```json
{ "question": "What are common symptoms of diabetes?" }
```

Response:
```json
{
  "answer": "...",
  "sources": [{ "document": "encyclopedia-of-medicine-vol3.pdf", "page": 4 }],
  "disclaimer": "This is general medical information, not medical advice."
}
```

## Getting Started

### Prerequisites
- Python 3.10+
- OpenAI API key
- Pinecone account and API key
- Supabase project (URL + service key)

### Setup

```bash
git clone https://github.com/Hamphreykoley/doctor-llm.git
cd doctor-llm
python -m venv venv
source venv/bin/activate   # or venv\Scripts\activate on Windows
pip install -r requirements.txt
```

Create a `backend/.env` file with the required configuration:

```env
OPENAI_API_KEY=your_openai_key
PINECONE_API_KEY=your_pinecone_key
SUPABASE_URL=your_supabase_url
SUPABASE_SERVICE_KEY=your_supabase_service_key
FLASK_SECRET_KEY=your_flask_secret
JWT_SECRET=your_jwt_secret
JWT_EXPIRY_HOURS=24
CHUNK_SIZE=1000
CHUNK_OVERLAP=200
RETRIEVER_TOP_K=4
LOG_LEVEL=INFO
```

Run the backend:

```bash
python app.py
```

> The backend will fail to start with a clear error if any required configuration value is missing — this is intentional (see SRS §4, FR-1).

## Data Source

Current knowledge base: **Encyclopedia of Medicine, Vol. 3**, ingested via the document upload pipeline and indexed in Pinecone. Additional approved medical documents can be added by an administrator through `/api/documents/upload`.

## Roadmap

- [ ] Complete backend implementation (auth, ingestion, chat endpoints)
- [ ] Unit tests for config, auth, ingestion validation, and API behavior
- [ ] Rate limiting on auth and chat endpoints
- [ ] Deployment (backend + hosted vector index)
- [ ] Admin document approval workflow
- [ ] Retention/deletion policy for chat history and documents

## Documentation

Full requirements, acceptance criteria, and design decisions are documented in the [Software Requirements Specification](./docs/Doctor_LLM_SRS.pdf).

## License

All rights reserved. This project is not currently licensed for reuse or distribution.

## Author

**Hamphrey Koley**
B.Tech Information Technology, Netaji Subhash Engineering College
