# fin-rag-bench / medium

**A GraphRAG evaluation benchmark for financial document retrieval — 100 questions across three question sets, grounded in real SEC filings from five Fortune 500 companies.**

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
| Johnson & Johnson | JNJ | NYSE | `jnj-10k.pdf`, `jnj-10q.html`, `jnj-8k.pdf` |
| Exxon Mobil Corporation | XOM | NYSE | `xom-10k.pdf`, `xom-8k.htm`, `xom-10q.pdf` |
| Walmart Inc. | WMT | NYSE | `wmt-10k.html`, `wmt-10q.htm`, `wmt-8k.pdf` |
| JPMorgan Chase & Co. | JPM | NYSE | `jpm-10k.html`, `jpm-10q.pdf`, `jpm-8k.html` |

All filings reflect periods ending in 2025 or early 2026. Financial figures are verified against inline XBRL tags where available.

> **Note on fiscal calendars:** Walmart's fiscal year ends January 31 (FY2025 = Feb 1, 2024 – Jan 31, 2025). ExxonMobil and JPMorgan end December 31. Apple ends the last Saturday of September. Johnson & Johnson ends the last Sunday of December. Questions specify the exact period in their text to avoid ambiguity.

---

## Question Sets

Three question sets are available, designed for different evaluation purposes:

| File | IDs | Questions | Focus | Status |
|---|---|---|---|---|
| `question-set-60-html-focused.jsonl` | Q061–Q120 | 60 | HTML/HTM filings | **Ready** |
| `question-set-40-pdf-focused.jsonl` | Q121–Q160 | 40 | PDF filings | **Ready** |
| `question-set-100-all-file-types.jsonl` | Q061–Q160 | 100 | All file types (merged) | **Ready** |

- Use **`question-set-60-html-focused.jsonl`** as a standalone evaluation focused on HTML/HTM retrieval quality.
- Use **`question-set-40-pdf-focused.jsonl`** to isolate PDF parsing and retrieval performance.
- Use **`question-set-100-all-file-types.jsonl`** for a full mixed-format evaluation; it is the union of the 60 and 40 sets.

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

### 60-Question HTML-Focused Set (Q061–Q120)

#### By Company

| Company | Questions | IDs |
|---|---|---|
| Apple Inc. | 9 | Q061–Q069 |
| Johnson & Johnson | 12 | Q070–Q081 |
| Exxon Mobil Corporation | 12 | Q082–Q093 |
| Walmart Inc. | 12 | Q094–Q105 |
| JPMorgan Chase & Co. | 10 | Q106–Q115 |
| Multi-company cross-filing | 5 | Q116–Q120 |

#### By Tier

| Tier | Count |
|---|---|
| `pure_vector` | 34 |
| `hybrid` | 16 |
| `pure_graph` | 10 |

#### By Difficulty

| Difficulty | Count |
|---|---|
| easy | ~31 |
| medium | ~29 |

---

### 40-Question PDF-Focused Set (Q121–Q160)

#### By Company

| Company | Questions | IDs |
|---|---|---|
| Johnson & Johnson | 14 | Q121–Q134 |
| Exxon Mobil Corporation | 10 | Q135–Q144 |
| JPMorgan Chase & Co. | 10 | Q145–Q154 |
| Multi-company cross-filing | 6 | Q155–Q160 |

#### By Tier

| Tier | Count |
|---|---|
| `pure_vector` | 26 |
| `hybrid` | 10 |
| `pure_graph` | 4 |

#### By Difficulty

| Difficulty | Count |
|---|---|
| easy | 17 |
| medium | 23 |

---

### 100-Question Combined Set (Q061–Q160)

#### By Company

| Company | Questions |
|---|---|
| Johnson & Johnson | 27 |
| Exxon Mobil Corporation | 22 |
| JPMorgan Chase & Co. | 20 |
| Walmart Inc. | 12 |
| Multi-company cross-filing | 10 |
| Apple Inc. | 9 |

#### By Tier

| Tier | Count |
|---|---|
| `pure_vector` | 60 |
| `hybrid` | 26 |
| `pure_graph` | 14 |

#### By Difficulty

| Difficulty | Count |
|---|---|
| easy | 47 |
| medium | 53 |

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
Answer: Requires extracting total assets from five separate filings and comparing: JPM $4.42T, XOM $449B, JNJ $199B, WMT balance sheet (much smaller than revenue), AAPL $331B.

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
- Several PDF files are cover-page-only EDGAR submissions: `jnj-8k.pdf`, `wmt-8k.pdf`, and `xom-10q.pdf` contain only the SEC cover page, not the full filing content. Financial statement data cannot be retrieved from these files.
- `jnj-10k.pdf` (130 pages) and `jpm-10q.pdf` (270 pages) are complete PDF filings with full financial statement content.
- Q118 and Q119 reflect the cover-page asymmetry: three companies have machine-readable HTML 8-Ks; two (JNJ, WMT) filed PDF-only 8-Ks for the January 2026 window.
- Corpus-scoped questions (Q116–Q120) reference "all five companies" as a closed set and cannot be reused verbatim in the large test without rewriting.

---

## Curation Notes

These observations emerged during manual review of the question set and are recorded here to inform future benchmark development.

**Fiscal calendar pitfall (Q094 — Walmart revenue).**
Walmart's fiscal year ends January 31, meaning FY2025 spans February 2024 through January 2025. This caused the initial answer to pull $642,637M from the FY2024 comparative column instead of $674,538M from the FY2025 primary column — a plausible but wrong answer. Questions covering non-standard fiscal calendars should include the exact date range in the question text, as Q094 now does, to eliminate column ambiguity.

**Cross-period comparative questions (Q099).**
Q099 asks about Walmart's FY2024 net income as reported *inside* the FY2025 10-K comparative column. While correct and intentional, this framing can confuse both human reviewers and LLM evaluators who expect the filing year and the data year to match. Questions of this type should be labeled clearly in `answer_notes` with the XBRL context reference.

**Terminology mismatch (Q107 — JPMorgan investment banking).**
"Investment banking revenue" in Q107 corresponds to the line item labeled **"Investment banking fees"** in the JPMorgan 10-K. The XBRL tag (`InvestmentBankingRevenue`) uses the canonical term, but a system matching against display labels may miss this. This type of question is a useful stress test for terminology normalization.

**JNJ revenue label (Q121 — "Sales to customers").**
Johnson & Johnson labels its top-line revenue "Sales to customers" rather than "Revenue" or "Net revenue." This label is used consistently in both the HTML 10-Q and PDF 10-K. A system that searches for "revenue" may miss the JNJ income statement entirely. Both segment revenues sum to the same consolidated total — JNJ has no revenue outside its two reporting segments (Innovative Medicine and MedTech).

**Intentional cover-page-only PDF files (Q118, Q160).**
`jnj-8k.pdf`, `wmt-8k.pdf`, and `xom-10q.pdf` were included deliberately to test hallucination resilience. All three are real EDGAR filings but contain only the SEC cover page. Notably, `xom-10q.pdf` appears as a single-page inline EDGAR viewer screenshot rather than a full quarterly report — a system that attempts to extract XOM quarterly financials from it is failing. By contrast, `jpm-10q.pdf` is a complete 270-page 10-Q with full financial statements, making the JPM/XOM 10-Q pair a useful asymmetry test (Q160).

**Shared auditor across three companies (Q156).**
PricewaterhouseCoopers LLP (PCAOB ID 238) audits Johnson & Johnson, Exxon Mobil, and JPMorgan Chase — confirmed from their respective 10-K filings. Apple and Walmart both use Ernst & Young LLP. The initial Q156 draft incorrectly stated JPMorgan's auditor was unverified; this was caught during manual review of the JPM 10-K Item 8 and corrected.

**Corpus-scoped questions are not portable to larger test sets (Q116–Q120).**
Several questions reference "all five companies in the corpus" as a closed set. These questions are specific to the medium test (5 companies) and cannot be reused verbatim in the large test (20 companies) without rewriting. When scaling questions to larger corpora, replace closed-set phrasing with explicit company lists or add a corpus-scope field to the schema.

**System correctly prioritized logical correctness over recency (Q116).**
Q116 asks which company had the *largest* total assets at its most recent fiscal year-end. JPMorgan's year-end 2024 balance ($4,002,814M) is lower than year-end 2025 ($4,424,900M), so the system correctly used the 2025 figure (the most recent) rather than the larger historical value. This is the expected behavior — a signal that the system understood "most recent" as a temporal constraint, not a maximization objective.

---

## Project Context

This benchmark was built as part of a GraphRAG pipeline evaluation project targeting financial document understanding. The goal is to produce a question set where retrieval tier is the primary differentiator of system performance — not question difficulty or domain knowledge — so that architectural comparisons between pure vector search, hybrid graph+vector, and structured graph query are meaningful and reproducible.

The three question sets here form a layered evaluation suite. The 60-question HTML-focused set provides a clean baseline for dense retrieval. The 40-question PDF-focused set isolates format-specific parsing challenges. The 100-question combined set enables full mixed-format evaluation and held-out test/validation splits.

---

## Quick Start

```bash
# Run against the HTML-focused 60-question set (Q061–Q120)
python evaluate.py \
  --questions question-set-60-html-focused.jsonl \
  --corpus files/ \
  --output results-60.jsonl

# Run against the PDF-focused 40-question set (Q121–Q160)
python evaluate.py \
  --questions question-set-40-pdf-focused.jsonl \
  --corpus files/ \
  --output results-40.jsonl

# Run against the full mixed-format 100-question set (Q061–Q160)
python evaluate.py \
  --questions question-set-100-all-file-types.jsonl \
  --corpus files/ \
  --output results-100.jsonl

# Filter to a specific tier
jq 'select(.tier == "hybrid")' question-set-100-all-file-types.jsonl

# Count questions by company
jq -r '.company' question-set-100-all-file-types.jsonl | sort | uniq -c | sort -rn

# Count questions by tier across all three sets
for f in question-set-60-html-focused.jsonl question-set-40-pdf-focused.jsonl question-set-100-all-file-types.jsonl; do
  echo "=== $f ==="; jq -r '.tier' "$f" | sort | uniq -c
done
```

---

*Data sourced from SEC EDGAR public filings. All financial figures are as reported; no adjustments have been made.*
