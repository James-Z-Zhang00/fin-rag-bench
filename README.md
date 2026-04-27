# FinRAGBench

A curated SEC EDGAR benchmark for evaluating financial RAG systems — covering information retrieval quality, cross-document reasoning, and graph structure understanding. All source filings are hand-picked from the official [SEC EDGAR](https://www.sec.gov/edgar/search/) public database in `.html`, `.htm`, and `.pdf` formats.

The question sets are designed to evaluate **hybrid search pipelines** (graph + vector), with questions stratified by retrieval tier so that architectural differences between pure vector, graph-guided, and entity-topology approaches are measurable and reproducible.

---

## Folder Structure

| Folder | Files | Purpose |
|---|---|---|
| `small/` | 2 | Smoke test — 1 HTML 10-K + 1 PDF cover-page 8-K |
| `medium/` | 15 | Standard evaluation — 5 Fortune 500 companies, 3 filing types, mixed HTML and PDF |
| `large/` | 60 | End-to-end system test (in progress) |

---

## Question Sets

| File | IDs | Questions | Status | Description |
|---|---|---|---|---|
| `small/question-set-6.jsonl` | S001–S006 | 6 | **Ready** | Smoke-test questions; see `small/README.md` |
| `medium/question-set-60-html-focused.jsonl` | Q061–Q120 | 60 | **Ready** | HTML/HTM-focused evaluation set |
| `medium/question-set-40-pdf-focused.jsonl` | Q121–Q160 | 40 | **Ready** | PDF-focused evaluation set |
| `medium/question-set-100-all-file-types.jsonl` | Q061–Q160 | 100 | **Ready** | Full mixed-format set (union of the 60 and 40) |
| `sample-question-set.jsonl` | — | — | Draft | Early sample questions, answers unverified — do not use for evaluation |

---

## Question Types

| Type | Description |
|---|---|
| `exact_number` | A specific financial figure (dollar amount, share count) |
| `exact_name` | A proper name, ticker symbol, or identifier |
| `short_phrase` | A descriptive or multi-part answer |
| `boolean` | Yes / No with supporting evidence |
| `numerical_range` | A computed ratio or percentage with an acceptable tolerance |

---

## Retrieval Tiers

- **`pure_vector`** — Single-document lookup; baseline for dense retrieval quality
- **`hybrid`** — Cross-document or cross-period comparison requiring graph navigation + vector extraction
- **`pure_graph`** — Entity and relationship queries (incorporation state, ticker, filing structure)

For full methodology, scoring rubrics, corpus details, and curation notes, see [`medium/README.md`](medium/README.md).
