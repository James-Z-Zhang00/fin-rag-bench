# fin-rag-bench / small

**A minimal 2-file smoke test corpus for fast pipeline sanity checks — 6 questions, 2 companies, 2 file formats.**

---

## Purpose

The small corpus is not an evaluation set. It is a **smoke test**: a minimal, fast check that a pipeline can ingest filings, parse both HTML and PDF formats, and return correct answers before running a full evaluation. A passing run on these 6 questions confirms the pipeline is wired up correctly; a failing run surfaces format or parsing issues early.

---

## Document Corpus

| Company | File | Format | Content |
|---|---|---|---|
| Apple Inc. | `aapl-10k.htm` | HTML (inline XBRL) | Full FY2025 10-K annual report |
| Walmart Inc. | `wmt-8k.pdf` | PDF | 8-K cover page only (filed January 16, 2026) |

The two files are intentionally different in format and content depth. `aapl-10k.htm` is a complete financial filing with dense XBRL-tagged data. `wmt-8k.pdf` is a single-page SEC cover sheet with no financial statement content — only company metadata (EIN, ticker, exchange, state of incorporation).

---

## Question Set

`question-set-6.jsonl` — 6 questions (IDs S001–S006), one per line.

| ID | Company | File | Tier | What it tests |
|---|---|---|---|---|
| S001 | Apple | `aapl-10k.htm` | `pure_vector` | Revenue lookup from HTML 10-K |
| S002 | Apple | `aapl-10k.htm` | `pure_vector` | Net income lookup from HTML 10-K |
| S003 | Apple | `aapl-10k.htm` | `pure_vector` | Non-financial entity (auditor name) from HTML |
| S004 | Apple | `aapl-10k.htm` | `hybrid` | Gross margin — requires dividing two retrieved values |
| S005 | Walmart | `wmt-8k.pdf` | `pure_vector` | EIN from PDF cover — tests PDF metadata without hallucination |
| S006 | Both | both | `pure_graph` | Exchange listing — cross-format entity property comparison |

**Tier distribution:** 4 `pure_vector`, 1 `hybrid`, 1 `pure_graph`

---

## Design Notes

**S005 is a hallucination trap.** `wmt-8k.pdf` contains only the SEC cover page — no earnings data, no financial statements. A system that hallucinates Walmart's revenue or income from this file is failing. The correct answer is limited to what appears on the cover: EIN, ticker, exchange, and incorporation state.

**S006 tests mixed-format reasoning.** The exchange listing for Apple appears in the HTML filing; Walmart's appears in the PDF cover. A system that only indexes one format will give an incomplete or wrong answer.

---

*For full evaluation methodology and scoring rubrics, see [`medium/README.md`](../medium/README.md).*
