# Blitz Protocol – Architecture Overview

This doc explains how the app works so you can run it and present it clearly.

---

## What the app does

**Blitz Protocol** is an AI customer-service platform for e‑commerce:

1. **Chat** – Users ask questions in natural language.
2. **Intent detection** – A GenAI node (Perplexity or Gemini) classifies the intent (e.g. order tracking, refund, FAQ).
3. **RAG (optional)** – For knowledge-base answers, text is chunked, embedded (local Xenova model), stored in Pinecone, and retrieved for context.
4. **Module execution** – Dedicated executors handle intents (tracking, cancellation, refund, FAQ, etc.) and call Groq for final answers when needed.
5. **Workflow builder** – You design the flow visually (nodes + edges) and configure GenAI + RAG per workflow.

So: **User message → GenAI intent → RAG (if needed) → Module executor → Response.**

---

## High-level architecture

```
User message (e.g. /chat)
        ↓
   Chat API (/api/chat)
        ↓
   WorkflowOrchestrator (loads workflow from DB, finds GenAI + RAG nodes)
        ↓
   GenAI Intent Node (Perplexity/Gemini) → intent label
        ↓
   RAG (if configured) → chunk + embed (Xenova) → Pinecone search → context
        ↓
   Module executors (Tracking, Refund, FAQ, etc.) + Groq for generation
        ↓
   Response back to user
```

- **Auth:** Clerk (sign-in/sign-up). Most routes are protected; only sign-in, sign-up, and webhooks are public.
- **Data:** Supabase (PostgreSQL). Workflows, node configs (including encrypted API keys), businesses, chat history.
- **AI:** Groq (RAG answers + some modules), Perplexity or Gemini (intent), Xenova (embeddings, no API key).

---

## Where things live in the repo

| Area | Path | Purpose |
|------|------|--------|
| **API routes** | `app/api/` | Chat, workflows, RAG (ingest/search/answer/delete), node config, businesses |
| **Workflow UI** | `app/workflows/`, `app/components/workflow/` | Canvas, palette, node config panel, GenAI/RAG config |
| **Orchestration** | `app/lib/nodes/executors/WorkflowOrchestrator.ts` | Runs workflow: load → intent → RAG → module |
| **Executors** | `app/lib/nodes/executors/*.ts` | GenAINodeExecutor, RAGNodeExecutor, TrackingModuleExecutor, etc. |
| **RAG** | `app/lib/rag/` | Chunking, embeddings (Xenova), Pinecone service, config |
| **DB helpers** | `app/lib/db/` | Workflows, node configs (with encryption), businesses, chat |
| **Auth** | `middleware.ts`, Clerk in `layout.tsx` | Protects routes; sign-in/sign-up under `(auth)/` |

---

## Credentials and env vars

Credentials are **not** in the repo. You configure them via environment variables (and one key is set in the UI).

1. **Create env file**  
   In the `blitz` folder:
   ```bash
   cp .env.example .env.local
   ```
2. **Fill in `.env.local`** (see list below).  
   If your friend already gave you a `.env` or `.env.local`, you can use that; just ensure the same variable names are set.

**Required:**

| Variable | Used for |
|----------|----------|
| `SUPABASE_URL` | Supabase project URL |
| `SUPABASE_SERVICE_ROLE_KEY` | Supabase server access (workflows, configs, DB) |
| `API_ENCRYPTION_KEY` | Encrypting/decrypting API keys stored in DB (use a long random string) |
| `NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY` | Clerk auth (frontend) |
| `CLERK_SECRET_KEY` | Clerk auth (backend) |
| `GROQ_API_KEY` | RAG answer + some module responses |
| `PINECONE_API_KEY` | Vector store for RAG |

**Optional:**  
`PINECONE_INDEX_NAME`, `PINECONE_INDEX_HOST`, and RAG tuning vars (see `.env.example`).

**Per-workflow API keys (GenAI node):**  
Perplexity or Gemini API keys are set in the **workflow builder** (GenAI node config). They are encrypted with `API_ENCRYPTION_KEY` and stored in Supabase; no env vars for those.

---

## How to run it

1. **Install and env**
   ```bash
   cd blitz
   npm install
   cp .env.example .env.local
   # Edit .env.local with your (or your friend’s) credentials
   ```
2. **Start dev server**
   ```bash
   npm run dev
   ```
3. **Open**  
   [http://localhost:3000](http://localhost:3000)  
   Sign in with Clerk, then go to **Workflows** to configure a workflow (GenAI node + optional RAG) and test chat.

**Note:** The README mentions `npm run db:migrate`; there is no such script in `package.json`. If the Supabase schema was already applied (e.g. by your friend), you don’t need it. If you get DB errors, you’ll need to run Supabase migrations separately (e.g. from Supabase dashboard or CLI).

---

## Making it “yours” for a presentation

- **Replace branding** – App title is in `app/layout.tsx` (e.g. “Blitz Protocol Console”). Update it and any other “Blitz” strings if you want your own product name.
- **README / docs** – Swap in your name/repo in `README.md` and in this file if you keep it; remove or rewrite “Made with ❤️ by the Blitz Commerce team” and support links if they’re not yours.
- **Env** – Use your own Supabase project, Clerk app, Pinecone index, and API keys so the demo runs under your accounts.
- **One flow you know cold** – Pick one path (e.g. “user asks about order → intent → RAG → response”) and walk through it using this doc and the code paths above so you can explain it confidently.

If you tell me your target product name and how much you want to change (e.g. “only README and title” vs “full rebrand”), I can suggest exact edits.
