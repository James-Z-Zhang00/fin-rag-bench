# fin-rag-bench / medium

**A GraphRAG evaluation benchmark for financial document retrieval — 60 questions grounded in real SEC filings from five Fortune 500 companies.**

---

## Overview

This benchmark tests retrieval-augmented generation (RAG) systems that operate over SEC financial filings. It is specifically designed to distinguish between three retrieval strategies:

| Tier | What it tests |
|---|---|
| `pure_vector` | Dense embedding search against a single filing |
| `hybrid` | Graph-guided narrowing followed by vector extraction across multiple filings or periods |
| `pure_graph` | Entity and relationship topology queries (ticker symbols, incorporation states, filing structure) |

A system that scores well on `pure_vector` but poorly on `hybrid` has strong retrieval but weak cross-document reasoning. A system that scores well on `pure_graph` demonstrates structured entity understanding beyond passage-level similarity.

---

## Document Corpus

15 SEC filings covering five large-cap US companies, sourced directly from [SEC EDGAR](https://www.sec.gov/cgi-bin/browse-edgar):

| Company | Ticker | Exchange | Files |
|---|---|---|---|
| Apple Inc. | AAPL | Nasdaq | `aapl-10k.htm`, `aapl-10q.html`, `aapl-8k.html` |
| Johnson & Johnson | JNJ | NYSE | `jnj-10q.html`, `jnj-8k.pdf` |
| Exxon Mobil Corporation | XOM | NYSE | `xom-10k.pdf`, `xom-8k.htm` |
| Walmart Inc. | WMT | NYSE | `wmt-10k.html`, `wmt-10q.htm`, `wmt-8k.pdf` |
| JPMorgan Chase & Co. | JPM | NYSE | `jpm-10k.html`, `jpm-8k.html` |

All filings reflect periods ending in 2025 or early 2026. Financial figures are verified against inline XBRL tags where available.

> **Note on fiscal calendars:** Walmart's fiscal year ends January 31 (FY2025 = Feb 1, 2024 – Jan 31, 2025). ExxonMobil and JPMorgan end December 31. Apple ends the last Saturday of September. Johnson & Johnson ends December 31. Questions specify the exact period in their text to avoid ambiguity.

---

## Question Set

`question-set-60.jsonl` contains 60 questions (IDs Q061–Q120), one JSON object per line.

### Schema

```jsonc
{
  "id":            "Q061",                    // Unique question ID
  "company":       "Apple Inc.",              // Primary company, or "Multiple"
  "filing_source": "aapl-10k.htm",           // Source file(s) in /files/
  "filing_type":   "10-K",                   // SEC form type
  "fiscal_period": "FY2025",                 // Period the question covers
  "tier":          "pure_vector",            // Retrieval tier (see below)
  "difficulty":    "easy",                   // easy | medium
  "answer_type":   "exact_number",           // See answer types below
  "question":      "...",                    // Question text
  "answer":        "$416,161 million",       // Ground-truth answer
  "answer_notes":  "XBRL: Revenue..., c-1"  // XBRL source tag and context
}
```

### Answer Types

| Type | Description | Example |
|---|---|---|
| `exact_number` | A specific dollar amount or share count | `$674,538 million` |
| `exact_name` | A proper name, ticker, or identifier | `PricewaterhouseCoopers LLP` |
| `short_phrase` | A multi-part or descriptive answer | `Innovative Medicine and MedTech` |
| `boolean` | Yes / No with supporting evidence | `Yes (+91.2% YoY)` |
| `numerical_range` | A computed ratio or percentage | `~46.9%` (±0.3% acceptable) |

---

## Coverage

### By Company

| Company | Questions | IDs |
|---|---|---|
| Apple Inc. | 9 | Q061–Q069 |
| Johnson & Johnson | 12 | Q070–Q081 |
| Exxon Mobil Corporation | 12 | Q082–Q093 |
| Walmart Inc. | 12 | Q094–Q105 |
| JPMorgan Chase & Co. | 10 | Q106–Q115 |
| Multi-company cross-filing | 5 | Q116–Q120 |

### By Tier

| Tier | Count | Notes |
|---|---|---|
| `pure_vector` | 34 | Single-filing lookups; tests dense retrieval precision |
| `hybrid` | 16 | Cross-period or cross-company comparisons requiring both graph and vector |
| `pure_graph` | 10 | Entity properties (state of incorporation, ticker symbol, filing structure) |

### By Difficulty

| Difficulty | Count |
|---|---|
| easy | ~31 |
| medium | ~29 |

---

## Tier Definitions

### `pure_vector`
The answer exists verbatim (or near-verbatim) within a single document. A well-tuned dense retriever should surface the correct passage. These questions establish a baseline.

**Example (Q094):** *"What were Walmart's total net revenues for fiscal year 2025 (fiscal year ended January 31, 2025)?"*
Answer: `$674,538 million` — a single XBRL-tagged value in `wmt-10k.html`.

### `hybrid`
The answer requires navigating a graph structure to identify which documents are relevant, then extracting and comparing values across those documents. Systems that rely on pure embedding similarity frequently fail here because the relevant passages are spread across filing periods or companies.

**Example (Q111):** *"Did JPMorgan Chase's net income increase or decrease from fiscal year 2024 to fiscal year 2025, and by approximately how much?"*
Answer: Requires reading both the FY2024 and FY2025 columns from the comparative income statement — same file, different temporal contexts.

**Example (Q116):** *"Among all five companies in the corpus, which company had the largest total assets as of its most recent fiscal year-end?"*
Answer: Requires extracting total assets from five separate filings and comparing: JPM $4.42T, XOM $449B, WMT balance sheet << revenue figure, AAPL $331B, JNJ $192.8B.

### `pure_graph`
The answer is a graph-level property of an entity node (company metadata, filing relationships, listed securities) rather than financial statement data. These questions evaluate whether the system models entities and their relationships, not just passages.

**Example (Q093):** *"How many total securities does Exxon Mobil Corporation have listed on the NYSE in its January 2026 8-K?"*
Answer: Four — common stock plus three note series (XOM, XOM28, XOM32, XOM39A).

---

## Evaluation Notes

### Scoring Recommendations
- `exact_number`: Accept ±0 unless the question specifies a range; unit mismatch (millions vs billions) counts as wrong.
- `numerical_range`: Accept ±0.3 percentage points from the stated value.
- `boolean`: Full credit requires the correct Yes/No; partial credit can be given if supporting figures are accurate but the conclusion is inverted.
- `exact_name` / `short_phrase`: Case-insensitive exact match or unambiguous semantic equivalence.

### XBRL Grounding
`answer_notes` fields cite the specific XBRL element name and context reference (e.g., `c-1: Feb 1, 2024 – Jan 31, 2025`) that produces the ground-truth value. This enables deterministic verification without re-reading prose.

### Known Corpus Limitations
- `jnj-8k.pdf`, `jpm-10q.pdf`, `wmt-8k.pdf`, `xom-10q.pdf`, and `jnj-10k.pdf` are PDF-format filings. Cover-page metadata (entity name, EIN, address) is accessible, but dense financial statement content requires PDF parsing.
- Q118 and Q119 reflect this asymmetry: three companies have machine-readable HTML 8-Ks; two (JNJ, WMT) filed PDF-only 8-Ks for the January 2026 window.
- This batch (Q061–Q120) skews toward HTML/HTM sources. Only the final two questions involve PDF-format content. Future batches should distribute questions more evenly across file formats to avoid retrieval format bias.

---

## Curation Notes

These observations emerged during manual review of the question set and are recorded here to inform future benchmark development.

**Fiscal calendar pitfall (Q094 — Walmart revenue).**
Walmart's fiscal year ends January 31, meaning FY2025 spans February 2024 through January 2025. This caused the initial answer to pull $642,637M from the FY2024 comparative column instead of $674,538M from the FY2025 primary column — a plausible but wrong answer. Questions covering non-standard fiscal calendars should include the exact date range in the question text, as Q094 now does, to eliminate column ambiguity.

**Cross-period comparative questions (Q099).**
Q099 asks about Walmart's FY2024 net income as reported *inside* the FY2025 10-K comparative column. While correct and intentional, this framing can confuse both human reviewers and LLM evaluators who expect the filing year and the data year to match. Questions of this type should be labeled clearly in `answer_notes` with the XBRL context reference.

**Terminology mismatch (Q107 — JPMorgan investment banking).**
"Investment banking revenue" in Q107 corresponds to the line item labeled **"Investment banking fees"** in the JPMorgan 10-K. The XBRL tag (`InvestmentBankingRevenue`) uses the canonical term, but a system matching against display labels may miss this. This type of question is a useful stress test for terminology normalization.

**Intentional cover-page-only PDF files (Q118).**
`jnj-8k.pdf` and `wmt-8k.pdf` were added deliberately to test hallucination resilience: both are real 8-K filings (JNJ: January 21, 2026; WMT: January 15, 2026) but contain only the SEC cover page, not the earnings press release. A well-behaved system should acknowledge these filings exist while correctly reporting that their content cannot be verified from the corpus. The initial `answer_notes` draft incorrectly stated the two companies had no 8-K filings at all — this was caught and corrected.

**Corpus-scoped questions are not portable to larger test sets (Q116–Q120).**
Several questions reference "all five companies in the corpus" as a closed set. These questions are specific to the medium test (5 companies) and cannot be reused verbatim in the large test (20 companies) without rewriting. When scaling questions to larger corpora, replace closed-set phrasing with explicit company lists or add a corpus-scope field to the schema.

**System correctly prioritized logical correctness over recency (Q116).**
Q116 asks which company had the *largest* total assets at its most recent fiscal year-end. JPMorgan's year-end 2024 balance ($4,002,814M) is lower than year-end 2025 ($4,424,900M), so the system correctly used the 2025 figure (the most recent) rather than the larger historical value. This is the expected behavior — and a signal that the system understood "most recent" as a temporal constraint, not a maximization objective.

---

## Project Context

This benchmark was built as part of a GraphRAG pipeline evaluation project targeting financial document understanding. The goal is to produce a question set where retrieval tier is the primary differentiator of system performance — not question difficulty or domain knowledge — so that architectural comparisons between pure vector search, hybrid graph+vector, and structured graph query are meaningful and reproducible.

The 60 questions here (Q061–Q120) form a self-contained evaluation set. A complementary first batch (Q001–Q060) covers the same corpus with overlapping but distinct coverage, enabling held-out test / validation splits.

---

## Quick Start

```bash
# Run your RAG system against the question set
python evaluate.py \
  --questions question-set-60.jsonl \
  --corpus files/ \
  --output results.jsonl

# Filter to a specific tier
jq 'select(.tier == "hybrid")' question-set-60.jsonl

# Count questions by company
jq -r '.company' question-set-60.jsonl | sort | uniq -c | sort -rn
```

---

*Data sourced from SEC EDGAR public filings. All financial figures are as reported; no adjustments have been made.*
