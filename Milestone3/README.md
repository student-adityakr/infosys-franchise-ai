# FranchiseOps AI — Infosys Springboard Internship 7.0, Milestone 3

**Batch:** 4  
**Project:** FranchiseOps AI — Agentic Franchise Operations Intelligence Platform  
**Mentor:** Mohamedsipli M  
**Evaluation:** 18–19 November 2026  
**Stack:** FastAPI · SQLite · LangGraph · scikit-learn · FAISS · BM25 · RAG · Qwen · Google Colab

## 1. Deliverables in this folder

| File | Purpose |
|---|---|
| `FranchiseOpsAI_RAG_Builder_Milestone3.ipynb` | Crawls public web sources, extracts PDFs, de-duplicates and chunks documents, builds FAISS and BM25 indexes, and writes `synthetic_kb/all_chunks.jsonl` to Drive. |
| `FranchiseOps_AI_Milestone3.ipynb` | Final FastAPI application with the original agent system, Copilot, authentication, profile, RBAC, Admin pages, simulator changes and Drive RAG import. |
| `taught_questions.csv` | 52 starter Q&A rows in the application's CSV import format (`question,answer,keywords`). |
| `candidate_gap_questions.txt` | Same 52 questions for Admin → Test them now. Confirm actual gaps before importing answers. |
| `screenshots/README.md` | Required screenshot checklist. |

## 2. Run order

### A. Run the RAG Builder first

1. Open `FranchiseOpsAI_RAG_Builder_Milestone3.ipynb` in Google Colab.
2. Choose a GPU runtime (T4 recommended) and run cells from top to bottom.
3. Allow Google Drive access.
4. The notebook writes:
   - `MyDrive/FranchiseOps_AI/rag_pdfs/` — downloaded/added PDFs.
   - `MyDrive/FranchiseOps_AI/synthetic_kb/website_pages.jsonl` — extracted public HTML pages.
   - `MyDrive/FranchiseOps_AI/synthetic_kb/pdf_extracted.jsonl` — extracted PDF page text.
   - `MyDrive/FranchiseOps_AI/synthetic_kb/all_chunks.jsonl` — final shared chunk corpus.
   - `MyDrive/FranchiseOps_AI/faiss_indexes/` — dense FAISS index and metadata.
   - `MyDrive/FranchiseOps_AI/bm25_indexes/` — lexical BM25 index.
   - `MyDrive/FranchiseOps_AI/rag_build_report.json` — unique-source and chunk counts.
5. Read `rag_build_report.json`. The target is **150+ unique websites/PDF files**, not 150 chunks. If `target_met` is false, add eligible sources / retry and rebuild.

The crawler has a cap of 180 downloaded PDFs and the website collector has a cap of 180 pages. Individual PDF downloads above 25 MiB are skipped to reduce Colab memory pressure. A source may be skipped when it is unavailable, not a PDF/HTML document, duplicate, too short, or inaccessible.

### B. Run the final app notebook

1. Open `FranchiseOps_AI_Milestone3.ipynb` in Colab.
2. Select a T4 GPU runtime.
3. Add the seven secrets below and enable notebook access.
4. Run cells top-to-bottom. Part 30 mounts Drive, reads secrets, checks the server startup and waits for `/api/health`.
5. Part 31 first tries the Colab proxy, then uses ngrok as a fallback.
6. Sign in and open **Admin → System check**. The app startup imports chunks from `MyDrive/FranchiseOps_AI/synthetic_kb/all_chunks.jsonl` into its SQLite knowledge table. System check should show the imported Drive chunk count.
7. If no Drive chunks are shown, run the RAG Builder first, confirm `all_chunks.jsonl` exists in the stated path, restart the final app, or use the Admin Knowledge Base → Import from Drive control.

## 3. Colab Secrets — exact names

Create these exact names in the Colab left-side **Secrets** panel:

| Secret | Required? | Used for |
|---|---|---|
| `ADMIN_USERNAME` | Yes | Initial Admin username. |
| `ADMIN_PASSWORD` | Yes | Initial Admin password. |
| `EMAIL_ADDRESS` | No, unless email features are used | Gmail sender address. |
| `EMAIL_PASSWORD` | No, unless email features are used | Gmail App Password, not the normal Google password. |
| `HF_TOKEN` | Only for gated Hugging Face model access | Hugging Face model download/access. |
| `JWT_SECRET` | Yes | Signs session tokens; use at least 32 random characters. |
| `NGROK_AUTHTOKEN` | Fallback | Public ngrok tunnel if the Colab proxy is unavailable. |

Do not print secret values, paste them into code, commit them, or leave them in notebook outputs. `EMAIL_ADDRESS` and `EMAIL_PASSWORD` are optional for the application to start if email/OTP notifications are not being used.

## 4. RAG and Copilot flow

The RAG Builder produces one chunk corpus. The final app imports that corpus into SQLite and combines it with the application's retrieval system:

1. **BM25** finds lexical matches.
2. **Dense embeddings** find semantic matches.
3. **Reciprocal Rank Fusion (RRF)** combines the two rankings.
4. A **cross-encoder reranker** reorders top passages.
5. The language model writes a short explanation and includes source references.

The Copilot has several answer layers:
- taught answers from Admin → Teach Copilot;
- exact, read-only SQL intents for numeric/data questions;
- agent trace/results for operational questions;
- hybrid RAG for policies and how-to questions.

Numbers must be sourced from SQL or retrieved evidence. If the evidence is not adequate, the Copilot should say **“I do not know”** rather than invent a number.

## 5. Teach Copilot: 52 starter questions

`candidate_gap_questions.txt` is a test list. Use Admin → Test them now before importing. Mark a question as a genuine gap only when the running Copilot fails, returns an irrelevant/weak match, or cannot find a useful source.

`taught_questions.csv` has 52 sample Q&A rows with the exact columns expected by the application's import endpoint:
- `question`
- `answer`
- `keywords`

To import: Admin → Teach Copilot → import CSV → choose `taught_questions.csv`. Then retest each question and check the taught count and Copilot insights. The CSV contains **illustrative internal demo policy answers** based on the sample FranchiseOps policy pack and project explanation. Verify each answer against your own approved SOPs before teaching it as an operational rule. These sample policies are not statutory legal guidance.

The milestone counter reaches 50/50 only after the application successfully imports 50 distinct questions; the file does not itself prove that the live Copilot originally failed those questions.

## 6. Copilot SQL intent improvements

`copilot_pro.py` adds 16 franchise-specific patterns for questions including:
- lowest / highest CSAT outlets;
- staff count for a named outlet;
- inventory below reorder level;
- stock-out risk;
- highest campaign ROI;
- campaign performance by channel;
- negative reviews in the last seven days;
- lowest audit scores;
- attrition by tenure band;
- overtime by outlet;
- operating-cost ratios;
- wait-time complaints;
- days of stock cover;
- top revenue outlets;
- open audit/compliance findings.

The intent executor uses read-only SQL. For scoped users, queries with `outlet_id` are wrapped so that filtering is applied before aggregation. The Copilot remains a SQL data assistant; it does not use an LLM to invent numeric answers.

## 7. Agent simulator changes (`sim_fr.py`)

Each simulator reads current database rows and runs a seeded what-if model. These simulations do not write scenario outputs back to live business tables.

| Simulator | Milestone 3 changes |
|---|---|
| Inventory | Demand seasonality, random supplier delay, stock-out probability, three reorder-rule comparisons and P10–P90 lost-sales ranges. |
| Workforce | Attrition adjustment by tenure band, hiring lead-time assumption, overtime vs temp/hiring cost comparison and P10–P90 total-cost range. |
| P&L | Price elasticity and wage/rent inflation uncertainty with P10–P90 profit range. |
| Marketing | Diminishing returns, campaign fatigue and P10–P90 conversion range. |
| Sentiment | Keyword-based top complaint and assumption-based CSAT / repeat-visit impact range. |
| Audit | Open audit/compliance findings and remediation delay affect failure-risk estimate, with P10–P90 range. |

Every simulator uses a seed (default `7`) so the same settings reproduce the same scenario. Assumptions are displayed beside each simulator. These are educational scenario assumptions, not guaranteed forecasts or legal/financial advice.

## 8. Role access matrix

| Role | Copilot | Workforce | Outlets | Inventory | Marketing | Sentiment | Audit | Run agents | Users |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| Admin | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| Franchise Owner | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✗ |
| Store Manager | ✓ | ✓ | ✓ | ✓ | ✗ | ✓ | ✓ | ✗ | ✗ |
| Staff | ✓ | ✗ | ✗ | ✗ | ✗ | ✗ | ✗ | ✗ | ✗ |
| Auditor | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✗ | ✗ |

The Store Manager and Staff role must be restricted to their own outlet. During the live demo, sign in as two different roles and call a route/action the lower-privilege role is not allowed to use; capture the HTTP 403 response. Verify both menu visibility and server-side denial.

## 9. Six-slide presentation and 10-minute demo

Prepare a concise six-slide presentation for the 5-minute evaluation slot. The outline and timing below match the mentor's email:

**Presentation (5 minutes)**
1. Project title and team / mentor.
2. Problem statement and demo scale.
3. Solution architecture: agents, planner, approvals and Copilot.
4. Technology stack and why each layer is used.
5. Three challenges and fixes.
6. Evaluation targets and evidence to show.

**Live demo (10 minutes)**
- 2:00 — Sign in as two roles; demonstrate a refused (403) action.
- 1:30 — Admin dashboard and System check (RAG source/chunk count).
- 1:30 — Run an agent cycle and one simulator.
- 2:00 — SQL Copilot: typo, follow-up, and chart.
- 2:00 — RAG Copilot: cited sources, a taught question, and an “I do not know” case.
- 1:00 — Evaluator questions and code Q&A.

Start the Colab runtime early, pre-warm the models, keep the ngrok URL ready, and record a two-minute backup demo video.

## 10. Before GitHub submission

- [ ] Clear all output from both notebooks.
- [ ] Remove any credentials, API keys, JWT values, ngrok tokens and personal data from notebook source and output.
- [ ] Do not upload `.db`, logs, cache folders, model weights or secrets.
- [ ] Verify the RAG report says `target_met: true` and record the actual source/chunk count.
- [ ] Import and retest at least 50 distinct taught questions; capture the Admin counter.
- [ ] Capture a Copilot answer with numbered sources `[1]`, `[2]`.
- [ ] Capture each simulator before and after.
- [ ] Capture Admin → Copilot insights.
- [ ] Capture two-role sign-in and a restricted 403 response.
- [ ] Add actual screenshots under `screenshots/`.
- [ ] Push this folder to `infosys-franchise-ai/Milestone3/`.

## 11. Official source starting points

The RAG Builder seeds public resources for FSSAI food-safety/licensing, labour and workplace rules, GST/tax, supply chain and market information. These are source starting points; inspect the crawled documents and dates before answering any regulatory question.

- FSSAI — Hygiene Requirements: https://fssai.gov.in/inspection/hygiene-requirements
- FSSAI — Licensing: https://www.fssai.gov.in/business/licensing
- FSSAI — Regulations: https://fssai.gov.in/food-law/regulations
- Ministry of Labour & Employment — Labour Codes (including Central Rules and FAQs): https://www.labour.gov.in/offerings/schemes-and-services/details/labour-codes-gzNzQzMtQWa
- CBIC — Tax Information Portal: https://taxinformation.cbic.gov.in/
- GST Council FAQs: https://gstcouncil.gov.in/faq
- APEDA: https://apeda.gov.in/
- eNAM: https://enam.gov.in/web/
- AGMARKNET: https://agmarknet.gov.in/
- FAO/WHO Codex — Codes of Practice: https://www.fao.org/fao-who-codexalimentarius/codex-texts/codes-of-practice/en/

Regulations change. Use the current original notification / statute and check amendments. FSSAI itself notes that its compendia are reference documents and the original notifications/amendments prevail where there is any conflict.
