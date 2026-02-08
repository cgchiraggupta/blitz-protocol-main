# Making the code “yours” – presentation checklist

Use this so the project clearly looks like your work when you run and present it.

---

## 1. Credentials (to run it)

- [ ] Copy `blitz/.env.example` to `blitz/.env.local`.
- [ ] Fill in **required** env vars (see `.env.example` or ARCHITECTURE.md):
  - `SUPABASE_URL`, `SUPABASE_SERVICE_ROLE_KEY`
  - `API_ENCRYPTION_KEY` (long random string, e.g. `openssl rand -hex 32`)
  - `NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY`, `CLERK_SECRET_KEY`
  - `GROQ_API_KEY`, `PINECONE_API_KEY`
- [ ] If your friend already gave you a `.env` or `.env.local`, put it in `blitz/` as `.env.local` and confirm the same variable names are used.
- [ ] In the app: sign in (Clerk), go to **Workflows**, add a workflow, and in the **GenAI node** set a Perplexity or Gemini API key (so intent detection works).

---

## 2. Run and verify

- [ ] From repo root: `cd blitz && npm install && npm run dev`
- [ ] Open http://localhost:3000, sign in, open Workflows.
- [ ] Create or open a workflow, configure the GenAI node (model + API key), save.
- [ ] Use the chat (or store/demo page) and send a message that hits that workflow so you can show it end-to-end.

---

## 3. Make it look like your project (optional but recommended)

- [ ] **App title** – In `blitz/app/layout.tsx`, change the `title` and `description` in `metadata` to your project name and description.
- [ ] **README** – In `blitz/README.md` (and root `README.md` if you use it): replace “Blitz Commerce” / “Blitz Protocol” with your name, repo URL, and remove or change “Made with ❤️ by the Blitz Commerce team” and any support links that aren’t yours.
- [ ] **Your name in the repo** – If you’re pushing to your GitHub: your profile and commit history will show you as the owner; consider one or two small commits (e.g. “Update README and app title”) so the history reflects your involvement.

---

## 4. What to say in the presentation

- **What it is:** “An AI customer-service platform: user asks a question → we detect intent with an LLM → we optionally pull in knowledge-base context (RAG) → we run the right module (tracking, refund, FAQ, etc.) and return an answer.”
- **How it works:** “We use a visual workflow (nodes and edges). The GenAI node does intent detection; the RAG node handles our docs; then module executors plus Groq generate the reply. Auth and data are Clerk and Supabase.”
- **If asked about the code:** Point to `ARCHITECTURE.md` and the main flow: Chat API → WorkflowOrchestrator → GenAI node → RAG (if used) → module executors.

---

## 5. Before you push

- [ ] **Never commit real secrets.** Ensure `.env` and `.env.local` are in `.gitignore` (they are via `.env*`). Only `.env.example` (with placeholders) should be committed.
- [ ] If you added your own `.env.example`, keep it to placeholder values only.

You’re set to run the app and present it as your own.
