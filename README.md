# Enpal Covenant Monitoring — Prototype

**Status:** Work in progress. Built as a personal technical exploration ahead of an interview — not an Enpal-commissioned project.

---

## Confidentiality Notice

This repository is demonstrated entirely on **synthetic, self-authored loan agreements and synthetic portfolio data**. No real Enpal transaction documents, and no real document from any other institution, are used anywhere in this repository. In a production setting, agreement text and portfolio data would be processed entirely within a private environment — nothing shown here reflects, or is derived from, any real deal document.

---

## Premise

Enpal's asset-financing model depends on a small but steadily growing set of institutional financing facilities — senior and mezzanine commitments from banks and asset managers across several ABS warehouse and public securitisation structures established since 2023. The number of facilities is deliberately small; that is not a limitation of this project, it is the point. The case for structured, AI-assisted covenant monitoring does not rest on document volume. It rests on three things that are real even at a handful of facilities:

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

The pipeline runs in four stages, one notebook per stage:

**1. Extraction** (`01_extraction.ipynb`)
Each synthetic agreement is passed to the extraction model with a schema-constrained prompt: every covenant field returned must carry a citation to its exact source span in the document. Fields with no grounded citation are returned as "not found" rather than inferred.

**2. Confidence & Review** (`02_confidence_and_review.ipynb`)
Extraction is run multiple times per document. Agreement across passes, combined with schema validation, produces a confidence score. Low-confidence or missing fields are flagged for human review rather than accepted automatically. Nothing covenant-critical proceeds without sign-off.

**3. Record Store** (`03_record_store.ipynb`)
Approved extractions are written as versioned records, each carrying an effective date and, where relevant, a pointer to the record it supersedes. A resolver function answers: *what is the governing version of this covenant, as of period N?*

**4. Comparison & Alerting** (`04_comparison_and_alerting.ipynb`)
Synthetic portfolio financials, spanning several consecutive reporting periods, are compared against the correctly governing threshold for each period. Output is a period-by-period result showing headroom and trend — drift toward breach, not just a binary flag. This notebook is the full end-to-end demonstration; running it top to bottom reproduces the entire pipeline's output.

---

## Repository Structure
enpal-covenant-monitoring-demo/
├── README.md
├── requirements.txt
├── .gitignore
├── .env # Anthropic API key — not committed
├── synthetic_agreements/
│ ├── facility_a_credit_agreement.txt # Control case — ABS-style, no complications
│ ├── facility_b_credit_agreement.txt # Definitional inconsistency, isolated
│ ├── facility_c_credit_agreement.txt # Amendment/lifecycle, isolated — original
│ └── facility_c_amendment_1.txt # Facility C — amendment, own effective date
├── synthetic_financials/
│ ├── facility_a_periods.csv # Multi-period synthetic portfolio data
│ ├── facility_b_periods.csv
│ └── facility_c_periods.csv
├── notebooks/
│ ├── 01_extraction.ipynb
│ ├── 02_confidence_and_review.ipynb
│ ├── 03_record_store.ipynb
│ └── 04_comparison_and_alerting.ipynb
└── records/
└── (structured JSON output from notebooks 01–03, committed as evidence of a real run)

---

## Intended Outputs

Running the notebooks in order produces:

- A structured, versioned record of every covenant extracted, each with a source citation and a confidence score.
- An explicit demonstration that Facility A and Facility B's shared covenant label resolves to two different numbers when the correct, facility-specific definition is applied.
- A clear cutover point in Facility C's record where the governing leverage ratio threshold changes at the amendment's effective date — with earlier periods still correctly tested against the original threshold.
- A period-by-period covenant test result for each facility, showing headroom and trend rather than a single snapshot.

---

## Limitations

- **Synthetic data only.** Every agreement and every financial figure in this repository is self-authored for demonstration purposes. Real-world drafting and data conventions were used as structural reference only (see notebook comments), never copied.
- **Starts post-OCR.** Agreements are provided as plain text. A production pipeline would need an OCR/ingestion step ahead of this one; that step is out of scope here.
- **Deliberately small facility count.** Three facilities is a design choice, not a shortcut — see Premise above.
- **Work in progress.** This is a rough, evolving prototype, not a finished product.