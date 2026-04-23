# FinRAGBench

Here's the SEC EDGAR files I hand-picked for finance domain RAG system test, will update with full QA set. The QA set currently focuse on information retrieval quality evluation and graph building quality for GraphRAG system in `.html`, `.htm` and `.pdf` formats. All files are from the official site `https://www.sec.gov/edgar/search/`

*The current question sets targeting the hybrid search (graph + vector) pipeline evaluation*

**Folder List**
- `small/` - 2 sample files for smoke test
- `medium/` - 15 files for a regular test
- `large/` - 60 files for big end-to-end system test

**File List**
- `sample-question-set.jsonl` - the sample questions made from the SEC filings inside `medium/`, the answers are yet to varify, please don't use for now : )

**Question Types**
1. Exact answer
2. Short phrase
3. Boolean / Yes-No
4. Ordered List
5. Unordered List
6. Numerical range
