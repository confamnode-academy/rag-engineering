# Module 03 — Document Ingestion

**Score entering:** hit@1 = 0.444 (16/36), ceiling 0.667
**Score leaving:** hit@1 = **0.750** (27/36), ceiling 1.0

Module 02 read seven PDFs. This module reads all fifteen documents in eight
formats, repairs what arrives broken, keeps structure intact, and reads a
register recording which documents are still in force.

| # | Notebook | What it does |
| --- | --- | --- |
| 1 | Parsing by format | A handler per format, each showing its failure first |
| 2 | General parsers | Unstructured, Docling, MarkItDown — scored against known traps |
| 3 | Cleaning | Scan detection, OCR, encoding damage, boilerplate |
| 4 | Structure preservation | Heading paths, speaker attribution, table headers |
| 5 | Metadata and provenance | What to capture, and why it comes from the questions |
| 6 | The pipeline | Assemble, make it restartable, measure |

## What actually happened

Three predictions were written down before running anything.

**1. Twelve questions become reachable.** Seven of the twelve now hit. The other
five are readable but don't retrieve — a spreadsheet, a CSV and an email thread
losing to longer PDF documents. That's a retrieval problem, and it waits for
module 07.

**2. The staleness failures convert.** Yes. Four of the five converted and the
class went from 2/6 to 5/6. Dropping the superseded handbook worked exactly as
intended.

**3. Q12 and Q13 must not break.** They didn't. Keeping the amended circular in
the index preserved both.

## But three questions regressed

| | before | after | why |
| --- | --- | --- | --- |
| Q05 | pass | fail | The HR memo is now indexed and also says "25 working days" |
| Q09 | pass | fail | The amended 2024 circular still competes with its amendment |
| Q10 | pass | fail | Retrieved the **HTML copy** of the right circular, not the PDF |

Q05 is the price of reading more documents: a new file genuinely relevant to the
question now outranks the one the golden set expects.

Q10 is arguably not a failure at all. `nfsc-circular-2025-02.html` is the same
circular as the PDF, in another format. The system found the right content in the
wrong file, and document-level scoring can't tell the difference. Cross-format
deduplication is module 04.

Q09 is the real one.

## The claim that failed

The supersession register fixed staleness and **did nothing for amendments.**

| class | before | after |
| --- | --- | --- |
| staleness | 2/6 | **5/6** |
| amendment | 1/3 | **0/3** |

All three amendment questions now fail. The reason is visible in the data:
marking the 2024 circular `amended` keeps it retrievable, which is correct — two
questions still need it — and means it goes on competing with the document that
amended it.

That is a straight trade-off, not a bug:

- keep it → Q12 and Q13 pass, Q08 and Q09 fail
- drop it → Q08 and Q09 pass, Q12 and Q13 fail

The register moves which questions fail. It does not reduce how many.

**Document-level status cannot express "this document is right for some questions
and wrong for others."** The 2025 circular amended two provisions and left the
rest standing, so correctness is a property of *sections*, not of the file.

Two ways forward, neither of them ingestion:

- section-level supersession — which provisions were replaced (harder than it
  sounds, and it needs the amendment to say so precisely)
- retrieve both and let generation resolve the conflict, with the relationship
  in the prompt (module 11)

Worth stating plainly: this module set out to fix seven failures and fixed four,
broke three, and revealed that one of its two mechanisms was the wrong shape for
the problem.

## The idea worth keeping

Every failure in this module was silent.

The parser returning an empty string for the scan. `python-docx` dropping both
sides of a tracked change. An OCR setting that recovered 60% more text and lost
the one phrase a question needed. A boilerplate rule that would have deleted a
policy sentence. A supersession rule that fixed one class and broke another.

None raised an error. All were found by printing the output or measuring against
known answers — and the amendment result was only visible because the evaluation
set scores per failure class rather than in aggregate.

The headline went from 0.444 to 0.750. Inside that gain, a whole category went to
zero.
