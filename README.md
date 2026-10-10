# Product Requirements Document (PRD)
## StudyLens AI: An AI-Powered Document & Study Companion

**Prepared for:** IBM Vibe Coding Masterclass Series, Final Project Submission
**Prepared by:** Gungun Verma
**Version:** 1.0
**Date:** July 2026

---

## 1. Executive Summary

StudyLens AI is a full-stack, AI-powered web application that lets a user upload or paste any document (lecture notes, research papers, articles, textbook chapters) and interact with it through three modes: **Summarize**, **Ask Questions (RAG-based Q&A)**, and **Generate Quiz**. The backend uses Retrieval-Augmented Generation (RAG) to ground the LLM's responses in the actual uploaded content, streamed to the frontend in real time. The application is containerized with Docker and deployed publicly on render and vercel.

This directly matches IBM's suggested themes of "Document Analyzer" and "Learning & Quiz Tool," combined into one cohesive product, while giving strong material for the Prompt Engineering and Technical Implementation criteria in the grading rubric.

---

## 2. Problem Statement

Students and self-learners regularly deal with dense, unstructured study material (PDFs, lecture notes, long articles) and spend significant time manually summarizing, re-reading, and creating practice questions. There is no lightweight, personal tool that lets someone drop in raw material and immediately get a summary, ask follow-up questions grounded in that specific document, and test themselves with auto-generated questions, all in one place.

---

## 3. Goals & Objectives

- Build a working RAG pipeline that grounds LLM answers in user-uploaded content (not just general model knowledge).
- Demonstrate three distinct prompting strategies (summarization, contextual Q&A, quiz generation) with documented iteration.
- Ship a fully containerized, publicly accessible application on render and vercel within free-tier limits.
- Produce a UI that is clean, responsive, and usable on both desktop and mobile.
- Document the full build process, prompts used, and architecture for the Project Report deliverable.

---

## 4. Target Users & Use Cases

| User | Use Case |
|---|---|
| College student (self) | Upload lecture PDF, get a quick summary before class |
| Exam prep | Paste textbook chapter, generate quiz questions to self-test |
| GFG Campus Mantri content | Summarize long technical articles for LinkedIn/community posts |
| Teaching assistant (4th grade CS class) | Simplify a technical topic into plain-language explanation |

---

## 5. Product Scope

### 5.1 In Scope (MVP)
- Upload a PDF or paste raw text
- Chunking and embedding of document content
- Retrieval-augmented Q&A chat interface
- One-click summarization mode
- Auto-generated quiz mode (multiple choice or short answer)
- Streamed responses (token-by-token rendering)
- Responsive frontend (desktop + mobile)
- Dockerized backend + frontend
- Deployed live on render and vercel with HTTPS

### 5.2 Out of Scope (Future Enhancements)
- Multi-document cross-referencing
- User accounts / login / saved history
- Support for scanned/handwritten PDFs (OCR)
- Multi-language document support
- Persistent vector database across sessions (MVP can use in-memory store per session)

---

## 6. Functional Requirements

| ID | Requirement | Priority |
|---|---|---|
| FR1 | User can upload a PDF file or paste plain text | Must |
| FR2 | System chunks document and generates embeddings | Must |
| FR3 | User can ask free-form questions about the document, answered via RAG | Must |
| FR4 | User can request a summary of the whole document | Must |
| FR5 | User can generate a quiz (5-10 questions) based on the document | Must |
| FR6 | Responses stream progressively in the UI, not all at once | Must |
| FR7 | Clear error handling (e.g., unsupported file, empty input) | Should |
| FR8 | Download/export quiz or summary as text | Could |
| FR9 | Toggle between "simple" and "detailed" explanation depth | Could |

---

## 7. Non-Functional Requirements

- **Security:** No API keys in frontend code or git history. All secrets loaded via environment variables on the backend/server only.
- **Performance:** Initial response token should stream within 2-3 seconds of a request.
- **Responsiveness:** UI must work cleanly on mobile viewport widths (360px+) and desktop.
- **Reliability:** Application should handle malformed or very large PDFs gracefully (size limit with clear user message).
- **Cost:** Must stay within render and vercel free-tier limits; budget alert configured.

---

## 8. System Architecture

```
[React Frontend]
      |
      | HTTPS (fetch / SSE for streaming)
      v
[FastAPI Backend on render and vercel  ]
      |
      |-- Document Processing: PDF text extraction, chunking
      |-- Embedding Layer: sentence-transformers (local) or Claude embeddings
      |-- In-memory Vector Store: cosine similarity search over chunks
      |-- Prompt Orchestration Layer: builds mode-specific prompts
      |
      v
[Anthropic Gemini API] --> streamed response --> [Backend] --> [Frontend]
```

### Component Breakdown
- **Frontend (React + Tailwind):** Upload widget, mode selector (Summarize / Ask / Quiz), chat-style response panel with streaming text.
- **Backend (FastAPI):** Handles file parsing, chunking, retrieval, and prompt construction; proxies streamed responses from Gemini API to frontend via Server-Sent Events (SSE).
- **Retrieval Layer:** Chunks document into ~300-500 token segments, embeds them, and retrieves top-k relevant chunks per query.
- **LLM Layer:** Gemini API call with retrieved context injected into the system/user prompt, response streamed back token-by-token.

---

## 9. Tech Stack

| Layer | Technology |
|---|---|
| Frontend | React, Tailwind CSS |
| Backend | Python, FastAPI |
| LLM | Anthropic Gemini API |
| Embeddings | sentence-transformers (all-MiniLM-L6-v2) or equivalent lightweight model |
| Vector Search | In-memory cosine similarity (numpy) or Chroma (lightweight, optional) |
| PDF Parsing | PyPDF2 / pdfplumber |
| Containerization | Docker (single container serving both API and built frontend) |
| Cloud | render and vercel   |
| Secrets Management | render and vercel environment variables |

---

## 10. LLM & Prompting Strategy

Three distinct prompt templates, one per mode. All use a shared "grounding" instruction to reduce hallucination.

**Summarize Mode (system prompt intent):** Instructs the model to produce a structured summary (key points, main argument, notable details) strictly from the provided document text, explicitly told not to add outside information.

**Q&A Mode (RAG, system prompt intent):** Provides the top-k retrieved chunks as context, instructs the model to answer only using that context, and to state clearly if the answer isn't present in the document rather than guessing.

**Quiz Mode (system prompt intent):** Instructs the model to generate a fixed number of questions (mix of multiple-choice and short-answer) strictly testing concepts found in the document, with answer keys included separately from the question text.

For the Project Report, document 2-3 iterations per mode: the first naive prompt attempt, what went wrong (e.g., model added outside knowledge, or quiz questions were too vague), and the refined version that fixed it. This iteration story is what the rubric rewards under "Prompt Engineering Quality & Documentation."

---

## 11. API Design (Backend Endpoints)

| Endpoint | Method | Description |
|---|---|---|
| `/upload` | POST | Accepts PDF or text, returns document ID after chunking + embedding |
| `/summarize/{doc_id}` | POST | Streams a summary of the document |
| `/ask/{doc_id}` | POST | Accepts a question, streams a RAG-grounded answer |
| `/quiz/{doc_id}` | POST | Streams generated quiz questions + answer key |
| `/health` | GET | Health check for render and vercel deployment monitoring |

---

## 12. Development Roadmap

| Phase | Deliverable | Maps to IBM Requirement |
|---|---|---|
| Phase 1: Concept & Design | Finalize concept note, sketch architecture, draft prompts | Concept Note (PDF) |
| Phase 2: Backend Core | PDF parsing, chunking, embeddings, in-memory retrieval | Technical Implementation |
| Phase 3: LLM Integration | Gemini API calls for all 3 modes, streaming via SSE | LLM Integration, Streaming requirement |
| Phase 4: Frontend | React UI, upload flow, mode selector, streamed chat display, responsive design | Application Design & UX |
| Phase 5: Containerization | Dockerfile, local docker run test | Containerization requirement |
| Phase 6: render and vercel Deployment | Push to render and vercel  , configure env vars, HTTPS, budget alert | Cloud Deployment & render and vercel Architecture |
| Phase 7: Documentation | Write Project Report: architecture, prompts, challenges, learnings | Report Quality & Reflection |

---

## 13. render and vercel Deployment Plan

- Build Docker image locally, test with `docker run` and confirm `/health` endpoint responds.
- Push image to Amazon ECR (Elastic Container Registry).
- Deploy via **render and vercel  **, pointing to the ECR image.
- Set Gemini API key as an environment variable in   configuration, never hardcoded.
- Confirm public HTTPS URL is issued automatically by  .
- Set up an **render and vercel Budget Alert** (e.g., threshold at $1-5) to catch unexpected usage early.
- Test the live URL on both desktop and mobile browsers before submission.

---

## 14. Security Checklist

- [ ] API keys stored only as environment variables, never committed to git
- [ ] `.env` file added to `.gitignore`
- [ ] No secrets present in frontend bundle (verify via browser dev tools/network tab)
- [ ] CORS configured to only allow expected origins
- [ ] File upload size limits enforced to prevent abuse
- [ ] render and vercel budget alert configured

---

## 15. Testing Plan

- Unit test PDF chunking function with sample documents of varying length.
- Manually test all 3 modes with at least 3 different document types (lecture notes, article, textbook excerpt).
- Test streaming behavior on slow network (throttled) to confirm progressive rendering.
- Cross-browser check: Chrome, Safari (mobile), and one more.
- Load test with a large PDF (10+ pages) to confirm graceful handling or clear size-limit messaging.

---

## 16. Evaluation Criteria Mapping

| IBM Rubric Criteria | Weight | How This Project Addresses It |
|---|---|---|
| Technical Implementation & Vibe Coding Methodology | 25% | RAG pipeline (chunking, embeddings, retrieval) is more substantial than a simple chatbot wrapper |
| Prompt Engineering Quality & Documentation | 20% | Three distinct prompt modes, documented iteration and refinement per mode |
| Cloud Deployment & render and vercel Architecture | 20% | Dockerized single-container deploy via render and vercel   with proper env var security |
| Application Design & User Experience | 20% | Clean responsive React UI, streamed chat-style interaction, mode switching |
| Report Quality, Reflection & Clarity | 15% | Structured report covering architecture, prompts, phases, challenges, learnings |

---

## 17. Deliverables Checklist

**1. Project Concept Note (PDF)**
- [ ] Project title and application name
- [ ] Problem statement / objective
- [ ] Target user and use case
- [ ] LLM model and API used (Gemini API)
- [ ] Key features
- [ ] Expected user experience and outcomes

**2. Live Application**
- [ ] Deployed and publicly accessible via render and vercel URL
- [ ] Fully functional at submission time
- [ ] URL pasted in both Concept Note and Report

**3. Project Report**
- [ ] Application overview and tech stack
- [ ] Prompting strategy with sample prompts (before/after iterations)
- [ ] Phase-by-phase development summary
- [ ] Architecture diagram and explanation
- [ ] Challenges encountered and resolutions
- [ ] Key learnings and reflection

---

## 18. Risks & Mitigations

| Risk | Mitigation |
|---|---|
| render and vercel costs exceed free tier | Configure budget alert early; use  's smallest instance size |
| Large PDFs slow down embedding step | Enforce a page/size limit with a clear UI message |
| Streaming doesn't work smoothly on render and vercel | Test SSE behavior locally in Docker before deploying, confirm   supports long-lived connections |
| Retrieval returns irrelevant chunks | Tune chunk size and top-k value; test with varied documents before finalizing |
| Running out of time before deadline | Treat Phases 1-4 (core RAG + UI) as non-negotiable MVP; containerization and deployment must start early, not last-minute |

---

## 19. Appendix

**Tools used during development (to document in report):**
- Claude (Anthropic) for code generation, debugging, and architecture guidance
- Gemini API for the live LLM integration in the deployed app
- render and vercel   for deployment
- Docker for containerization

**Suggested sample document types to test with:**
- A GFG technical article
- A lecture note PDF from your CSE coursework
- A page from Ed Donner's LLM engineering course material (for variety in content type)

---

*End of PRD*
