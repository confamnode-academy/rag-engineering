# RAG Engineering — Course Corpus (synthetic)

Fifteen documents in eight formats from three fictional organisations, built to fail in
specific ways.

**Every document in this corpus is synthetic.** The organisations do not exist. The figures are
invented. Each PDF carries a footer saying so. Nothing here should be cited as fact about Nigerian
regulation, taxation or any real company — and that footer doubles as the boilerplate-removal
exercise in Module 03.

Fictional entities were used deliberately. Realistic-looking fabricated filings attributed to real
companies or real regulators would be a liability the moment this repo goes public.

---

## The organisations

| Entity | Type | Documents |
| --- | --- | --- |
| Sahel Microfinance Bank Plc | 22-branch MFB, Lagos HQ | Employee handbook (2023, 2025), procurement policy |
| Kaduna Agro Processing Limited | Agro-processor, RC 1284477 | Annual report 2024, board minutes |
| National Financial Services Commission | Fictional regulator | Circular 2024/07, Circular 2025/02 |
| Kaduna State Internal Revenue Service | Fictional state revenue body | Guidance Note 4 of 2024 (scanned) |

---

## What is planted where

### 1. The staleness trap — the most important thing in this corpus

`sahel-employee-handbook-2023.pdf` and `sahel-employee-handbook-2025.pdf` are structurally identical
documents with changed values:

| Provision | 2023 edition | 2025 edition |
| --- | --- | --- |
| Annual leave, confirmed staff | 21 working days | 25 working days |
| Leave carryover cap | 5 days | 10 days |
| Senior grade additional leave | 3 days | 5 days |
| Probation period | 6 months | 4 months |
| Paternity leave | 5 days | 10 days |
| Domestic per diem | NGN 35,000/night | NGN 60,000/night |
| Expense claim window | 30 days | 60 days |
| Notice period, confirmed staff | 1 month | 2 months |
| Remote work | 1 day/week | 2 days/week + hybrid agreement |

A naive pipeline retrieves both and either contradicts itself or picks one at random. When it picks
2023, it produces an answer that is **perfectly faithful to the retrieved chunk and wrong** — the
failure mode the course calls *faithful incorrectness*, and the one most real systems have.

*Used in:* Module 01 (failure taxonomy), Module 03 (versioning and deduplication), Module 06
(evaluation catches it), Module 09 (grounding), Module 15 (tombstoning superseded documents).

### 2. The amendment trap

`nfsc-circular-2024-07-cybersecurity.pdf` sets the material incident notification window at **72
hours** and the loss threshold at **NGN 10 million**. `nfsc-circular-2025-02-amendment.pdf` reduces
these to **24 hours** and **NGN 5 million**, and brings one compliance date forward.

Harder than the handbook pair, because the amendment does not restate the original — it says "the
window is reduced from seventy-two hours to twenty-four hours." Answering correctly requires both
documents *and* the right one to win. Retrieving only the 2024 circular gives a confident wrong
answer with a real citation.

*Used in:* Module 07 (retrieval), Module 08 (context assembly and ordering), Module 11 (multi-hop).

### 3. The page-spanning table

`kaduna-agro-annual-report-2024.pdf` contains a 46-row distribution table (state, LGA, volume,
revenue, utilisation) that breaks across pages **with the header row repeated**. That is the exact
case the stitching exercise handles: match consecutive pages by column count, detect and strip the
duplicate header, concatenate.

Fixed-size chunking on this table produces headless chunks — bare rows with no idea what the columns
mean.

*Used in:* Module 04 (headless chunks), Module 13 (table extraction and stitching).

### 4. Chart-only data

Figure 1 of the annual report plots revenue and gross margin for 2019–2024. **The values appear
nowhere in the prose.** The text says margin "recovered in the second half" and revenue "rose
substantially" and gives no numbers.

Ask for 2020 revenue and a text-only pipeline cannot answer. This is the ablation that motivates the
whole multimodal module — text only, then image summary, then raw image to a VLM, with the answer
getting measurably better each time.

Ground truth: revenue (NGN bn) 8.4, 7.1, 11.9, 16.3, 22.8, 31.6 · margin (%) 11.2, 6.8, 13.4, 15.1,
14.2, 17.9 for 2019–2024.

*Used in:* Module 01 (when text-only RAG fails), Module 13 (the three-way ablation).

### 5. The scan

`kdirs-guidance-note-4-2024-SCANNED.pdf` is image-only. No text layer, zero embedded fonts —
`pdftotext` returns nothing and `pdffonts` shows an empty table. It has been degraded to third-generation
photocopy quality: 1.5° skew, uneven platen illumination, a dark left edge where the lid didn't close,
a horizontal fold shadow, toner speckle and heavy JPEG artefacts.

Tesseract currently returns "Guidanice" for "Guidance," "Girectorate" for "Directorate," and drops
line-ends where the illumination gradient darkens the page. That is the 60–80% accuracy regime made
concrete rather than asserted.

*Used in:* Module 03 (triage and routing — detect before you parse), advanced ingestion (OCR),
Module 13 (VLM parsing as fallback).

### 6. Threshold tables

`sahel-procurement-policy-v3.pdf` has a five-tier approval threshold table, and the NFSC circular has
an eight-row implementation schedule where the compliance date depends on the institution's asset
size.

Both punish flattened tables. "What approval is needed for NGN 3 million?" requires reading a row and
a range together, and the 2.5m–15m band is easy to get wrong if the table was chunked mid-row.

*Used in:* Module 04, Module 06 (these make good golden-set questions), Module 13.

### 7. Speaker attribution

`kaduna-agro-board-minutes-2024-10-17.pdf` attributes statements to named individuals, with one
director withdrawing for a conflicted item and rejoining afterwards.

"Who questioned the payback sensitivity?" needs attribution preserved through chunking. "Who was
present for the haulage decision?" needs the withdrawal noticed. This stands in for meeting
transcripts before the audio module introduces real diarization.

*Used in:* Module 03 (structure preservation), Module 12 (agentic multi-hop).

### 8. Exact identifiers

Scattered throughout: RC 1284477, SMB/HR/HANDBOOK/2025, NFSC/CIR/2024/07, SMB/OPS/PROC/2024.

Dense embeddings handle these badly — this is what motivates BM25 and hybrid retrieval. Ask "what
does circular NFSC/CIR/2025/02 change?" and watch semantic search underperform keyword search.

*Used in:* Module 05 (where embeddings leak), Module 07 (hybrid retrieval).

### 9. Boilerplate

Every PDF carries the same synthetic-document footer on every page. Left in deliberately: repeated
per-page boilerplate polluting chunks is a real ingestion problem and this is the exercise for it.

*Used in:* Module 03 (cleaning).

---

## Scale

This corpus is small — roughly a dozen pages. That is correct for teaching: everything runs on a
laptop and every failure is traceable to a specific document.

It is **not** large enough to demonstrate retrieval degradation at scale, which is a real phenomenon
the course should cover. Before Module 07, pad it to a few hundred pages with genuine public Nigerian
documents — CBN circulars, NGX-listed annual reports, university handbooks, state government policy
PDFs. Keep the eight synthetic documents as the scored set; the padding exists to make retrieval hard,
not to be evaluated against.

Keep the two sets in separate folders so the golden questions stay valid.

---

---

## Format-specific failures

PDF is not the hard part of enterprise ingestion; it is only the loudest. These documents carry
failures that belong to their format.

### 10. DOCX tracked changes — two tools, two different wrong answers

`sahel-hr-memo-2024-41-TRACKED.docx` is the memorandum that produced the 2025 handbook changes. It
carries unaccepted tracked changes: `NGN 35,000` deleted, `NGN 60,000` inserted, plus the same
pattern for leave days and the claim window. It also has a header, a footer, and a reviewer note.

Two approaches fail differently on the same file:

- **python-docx** reads `paragraph.text` and sees *neither* value. It skips `w:del` and `w:ins` runs
  entirely, so the sentence arrives as "The domestic per diem ... is revised to ." The number is
  gone and nothing warns you.
- **Naive XML stripping** yields `"revised to NGN 35,000 per nightNGN 60,000 per night"` — both
  values concatenated without a space. The chunk now asserts two contradictory figures.

Neither approach errors. This is the best single demonstration in the corpus of why "we use a
supported library" is not the same as "we parse correctly."

*Used in:* Module 03 (parsing, and parser selection), Module 06 (the golden set catches it).

### 11. XLSX — formulas, hidden sheets, merged headers

`kaduna-agro-distribution-2024.xlsx` has three sheets:

- **Distribution 2024** — a merged header cell (`C1:D1`), dates formatted for display but stored as
  serials, and a derived column holding `=D3*1000/C3` with **no cached value**. Reading with
  `data_only=True` returns `None` for every cell in that column. Reading without it returns the
  formula string, which is worse — you embed `=D3*1000/C3` as if it were content.
- **Notes** — basis of preparation, needed to interpret the Zaria figures correctly.
- **Draft - DO NOT CIRCULATE** — *hidden*, holding a superseded pre-audit total of 264,100 tonnes
  against the audited 271,400. A pipeline that iterates all sheets ingests the wrong number, and it
  is marked as not for circulation.

*Used in:* Module 03 (parsing beyond text), Module 06, Module 14 (should a hidden sheet be indexed
at all?).

### 12. PPTX — the answer is in the speaker notes

`kaduna-agro-board-deck-2025-01.pptx` puts material facts only in the notes. Slide 3 says three
closures were approved; the notes name them. Slide 2 says nothing about capital requirement; the
notes give NGN 2.4 billion with NGN 1.6 billion uncommitted, and state it has not been disclosed
externally.

Extractors that walk slide shapes and stop there lose all of it. Extractors that include notes
indiscriminately ingest material explicitly marked as internal — which is a governance question, not
a parsing one.

*Used in:* Module 03, Module 14 (classification-aware ingestion).

### 13. EML — quoted replies and signature blocks

`sahel-per-diem-thread.eml` is a three-deep reply chain. The original announcement is quoted twice,
so naive chunking indexes near-identical content three times and retrieval returns three copies of
one fact. Signature blocks add addresses and phone numbers that dilute every chunk they land in.

The answers to both thread questions are in the top message; everything below it is noise that looks
like signal.

*Used in:* Module 03 (cleaning), Module 04 (deduplication), Module 07 (near-duplicate results).

### 14. HTML — the same circular, wrapped in furniture

`nfsc-circular-2025-02.html` is the web version of the 2025 circular, surrounded by a skip link,
site header, seven-item nav, search form, breadcrumb, a maintenance banner, a related-circulars box,
a sidebar and a footer.

The body is identical to the PDF, so this doubles as a near-duplicate across formats: does your
pipeline notice it already has this content?

*Used in:* Module 03 (boilerplate stripping), Module 04 (cross-format deduplication).

### 15. CSV — line count is not record count

`sahel-approved-vendors.csv` has addresses containing embedded newlines inside quoted fields.
`wc -l` reports 14; there are 5 vendor records. It also ends with a `#` comment row that naive
readers parse as data.

*Used in:* Module 03 (structured data ingestion).

### 16. Markdown — the easy case

`sahel-branch-runbook.md` has clean headings and a clean table. Included deliberately as the
control: structure is free, and comparing it against the same question asked of a PDF table shows
how much of the difficulty is inherent to the format rather than to RAG.

*Used in:* Module 03 (contrast), Module 04.

---

## Files

```
corpus/
  README.md                                    this file
  golden_questions.csv                         38-question starter evaluation set
  corpus_src/                                  build scripts, fixed seeds
  docs/
    sahel-employee-handbook-2023.pdf           staleness trap (superseded)
    sahel-employee-handbook-2025.pdf           staleness trap (current)
    sahel-procurement-policy-v3.pdf            threshold table
    nfsc-circular-2024-07-cybersecurity.pdf    amendment trap (superseded in part)
    nfsc-circular-2025-02-amendment.pdf        amendment trap (amending)
    nfsc-circular-2025-02.html                 same content, wrapped in web furniture
    kaduna-agro-annual-report-2024.pdf         page-spanning table + chart-only data
    kaduna-agro-board-minutes-2024-10-17.pdf   speaker attribution
    kaduna-agro-board-deck-2025-01.pptx        facts only in speaker notes
    kaduna-agro-distribution-2024.xlsx         formulas, hidden sheet, merged headers
    kdirs-guidance-note-4-2024-SCANNED.pdf     image-only, no text layer
    sahel-hr-memo-2024-41-TRACKED.docx         unaccepted tracked changes
    sahel-per-diem-thread.eml                  quoted replies, signatures
    sahel-approved-vendors.csv                 embedded newlines, comment row
    sahel-branch-runbook.md                    clean control case
```

---

## A note on PyMuPDF4LLM

Run `pymupdf4llm.to_markdown()` over `kaduna-agro-annual-report-2024.pdf` before teaching Module 03.
It produces clean markdown with a proper header row for the first part of the distribution table —
and then emits the page-two fragment as a **second** markdown table whose header row is
`|Ogun|Ijebu-Ode|43,379|18,564.4|66%|`. A data row has been promoted to a header. Forty rows lose
their real column names, and the output looks entirely well-formed.

It also silently invokes OCR on a page of this natively-generated PDF. The library's own
documentation notes that the markdown makes no distinction between natively extracted and
OCR-recovered pages, and that OCR is roughly 1,000× slower. So a pipeline can be running OCR without
knowing it, on content that never needed it, with no marker in the output saying which text is which.

Both behaviours are silent. That is the point.

---

## Regenerating

Build scripts are in `corpus_src/`. Random seeds are fixed, so the distribution table and the scan
degradation reproduce exactly. Change the seed in `annual_report.py` for a different table; change
the parameters in `degrade.py` to make the scan easier or harder.

Worth doing once: rebuild the scan at a harsher setting and watch OCR accuracy collapse. It makes the
argument for late-interaction visual retrieval better than any benchmark number.

---

## Parser behaviour on this corpus

Measured, not asserted. Re-run these before teaching Module 03 — the tools move.

| Planted failure | python-docx / naive XML | MarkItDown | PyMuPDF4LLM |
| --- | --- | --- | --- |
| DOCX tracked changes | python-docx sees **neither** value; naive XML strip yields both concatenated | **Correct** — resolves to NGN 60,000 | Office formats need PyMuPDF Pro (commercial) |
| XLSX hidden sheet (DO NOT CIRCULATE) | openpyxl exposes it unless you filter `sheet_state` | **Leaks it** — the superseded 264,100 figure is ingested | n/a |
| PPTX speaker notes marked internal | python-pptx exposes notes only if you ask | **Captures them** — including the NGN 2.4bn not for external disclosure | n/a |
| HTML nav and boilerplate | — | **Not stripped** — "Returns Portal" banner survives into chunks | n/a |
| Page-spanning PDF table | raw PyMuPDF blocks mix table and prose | — | **Emits two tables; second header is a data row** |
| Native-text PDF | — | — | **Silently invokes OCR**, with no marker distinguishing OCR'd text from native |

The headline: **the tool that handles the DOCX best is the one that leaks the most.** MarkItDown
resolves tracked changes correctly and then ingests a hidden sheet marked DO NOT CIRCULATE and
speaker notes marked as not for external disclosure.

That reframes parser selection. It is not "which is most accurate." It is "which failure can I live
with, and what control do I put in front of it" — and the answer for a bank or a law firm is
different from the answer for an internal wiki.
