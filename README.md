# AI-Assisted Covenant Monitoring for Structured Credit Facilities

**Status:** Personal technical project, complete and independently verified. Not commissioned by, or affiliated with, any company.

For a detailed, plain-language walkthrough of this project's design, methodology, and results, see [`docs/Project_Documentation.pdf`](docs/Project_Documentation.pdf).

---

## Origin

This project began as a personal exploration of a real question: can AI, applied carefully and with a human always in the loop, meaningfully help a small credit or portfolio team track the financial covenants attached to their lending facilities — even when there are only a handful of such facilities, not hundreds? The idea was prompted by studying how a real, fast-growing consumer asset-finance company structures its debt — a mix of asset-backed warehouse facilities and corporate-level borrowing, the kind of financing stack many scaling fintech and climate-tech companies build as they grow. No real company's documents, data, or confidential information were used anywhere in this project; everything from that point forward is self-authored and synthetic, described below.

---

## Confidentiality Notice

This repository is demonstrated entirely on **synthetic, self-authored loan agreements and synthetic portfolio data**. No real transaction documents belonging to any company or institution are used anywhere in this repository. In a production setting, agreement text and portfolio data would be processed entirely within a private environment — nothing shown here reflects, or is derived from, any real deal document.

---

## Premise

Companies that finance their growth through structured credit — asset-backed warehouse facilities, mezzanine debt, corporate term loans — often start small: a handful of facilities, a handful of lenders. A natural objection follows: if there are only a few facilities, why does covenant monitoring need a system at all? It's a fair question, and this project takes it seriously rather than assuming it away. The answer doesn't rest on document volume. It rests on three things that are real regardless of how many facilities exist:

1. **Recurrence.** A covenant is not read once — it is tested every reporting period, for the multi-year life of a facility. Manual tracking has to stay accurate across every cycle, not just on first read.
2. **Definitional inconsistency.** Two facilities can use the same covenant name — a delinquency ratio, a leverage ratio — with materially different governing definitions. This risk exists at three facilities as much as at three hundred.
3. **Amendment and lifecycle drift.** Facilities get amended and waived over their life. Knowing which version of a covenant is *currently governing*, as of a given date, is a real and recurring problem, not a one-time reading task.

This prototype is scoped to demonstrate exactly these three claims, using three synthetic facilities, each isolating one problem.

---

## Objective

To show, on a small and fully inspectable set of synthetic documents, that it is possible to:

- Extract covenant terms from unstructured legal text with **citation-grounded, schema-constrained** output — never a guessed field, always a source span or an explicit "not found."
- Detect **definitional inconsistency** between two facilities using the same covenant label.
- Correctly resolve **which version of a covenant is currently governing** at a given reporting date, across an original agreement and a later amendment.
- Compare live portfolio data against the correctly governing threshold, across multiple periods, producing a genuine early-warning signal — not just a pass/fail.

---

## Methodology

The pipeline runs in five stages, one notebook per stage:

**1. Extraction** (`01_extraction.ipynb`)
Each synthetic agreement is passed to the extraction model with a schema-constrained prompt: every covenant field returned must carry a citation to its exact source span in the document. Fields with no grounded citation are returned as "not found" rather than inferred.

**2. Confidence & Review** (`02_confidence_and_review.ipynb`)
Extraction is run three times per document. A covenant is marked high confidence only if every pass agrees on the same value, the citation independently verifies against the source text, and the record passes schema validation — all three, or none; there is no numeric score and no partial credit. Anything short of that is flagged for human review rather than accepted automatically. Nothing covenant-critical proceeds without sign-off.

**3. Record Store** (`03_record_store.ipynb`)
Approved extractions are written as versioned records, each carrying an effective date and, where relevant, a pointer to the record it supersedes. A resolver function answers: *what is the governing version of this covenant, as of period N?*

**4. Comparison & Alerting** (`04_comparison_and_alerting.ipynb`)
Synthetic portfolio financials, spanning several consecutive reporting periods, are compared against the correctly governing threshold for each period. Output is a period-by-period result showing headroom and trend — drift toward breach, not just a binary flag. This notebook is the full end-to-end demonstration; running it top to bottom reproduces the entire pipeline's output.

**5. Visualizations** (`05_visualizations.ipynb`)
Builds two charts directly from Notebook 4's saved output — no new computation, no API calls. Only two, deliberately: they're the only results in this project that are genuinely trends over time with a built-in comparison.

---

## Repository Structure