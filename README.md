# Enpal Covenant Monitoring — Prototype

**Status:** Work in progress. Built as a personal technical exploration ahead of an interview — not an Enpal-commissioned project.

For a detailed, plain-language walkthrough of this project's design, methodology, and results, see [`docs/Enpal_Covenant_Monitoring_Project_Documentation.pdf`](docs/Enpal_Covenant_Monitoring_Project_Documentation.pdf).

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

```
enpal-covenant-monitoring-demo/
├── README.md
├── requirements.txt
├── .gitignore
├── .env                                     # Anthropic API key — not committed
├── synthetic_agreements/
│   ├── facility_a_credit_agreement.txt      # Control case — ABS-style, no complications
│   ├── facility_b_credit_agreement.txt      # Definitional inconsistency, isolated
│   ├── facility_c_credit_agreement.txt      # Amendment/lifecycle, isolated — original
│   └── facility_c_amendment_1.txt           # Facility C — amendment, own effective date
├── synthetic_financials/
│   ├── facility_a_periods.csv               # Multi-period synthetic portfolio data
│   ├── facility_b_periods.csv
│   └── facility_c_periods.csv
├── notebooks/
│   ├── 01_extraction.ipynb
│   ├── 02_confidence_and_review.ipynb
│   ├── 03_record_store.ipynb
│   └── 04_comparison_and_alerting.ipynb
└── records/
    └── (structured JSON output from every notebook, committed as evidence of a real run)
```

---

## Results

All four notebooks have been run end to end against the synthetic agreements and financial data. What follows are the actual, independently verified results — not a description of what the pipeline is intended to do.

**Extraction and confidence.** All three facilities' covenants were extracted with citation-grounded values, each independently checked against the source text. Across three independent extraction passes per document, six of seven distinct covenant terms reached full agreement and were approved. One — Facility B's Reserve Account covenant — was correctly withheld from monitoring and flagged for human review: the three passes agreed on the underlying citation, but expressed the threshold value in two different, equally valid phrasings ("2.00%" vs. "2.00% of the Original Pool Balance"). It remains excluded pending review, exactly as intended — nothing covenant-critical proceeds without sign-off.

**Facility A (control).** Delinquency Ratio rose gently from 0.80% to 1.54% over eight periods, well under the 5.00% trigger. Overcollateralization held steady at 12.0%, comfortably above the 8.00% target, throughout. No drama — the clean case, as designed.

**Facility B (definitional inconsistency).** Using its own contractual definition (90+ days delinquent ÷ fixed original pool balance), Facility B's Delinquency Ratio rose from 0.273% to 2.795% across six periods — never approaching its 4.00% trigger. Applying Facility A's definition (60+ days ÷ current, shrinking balance) to the *identical* underlying data instead produces 0.688% rising to 7.623%, crossing 5% by the final period. Same facts, two definitions, two entirely different risk conclusions — the core claim this facility exists to prove, demonstrated on real computed output.

**Facility C (amendment resolution).** The Consolidated Net Leverage Ratio was correctly tested against 3.50:1.00 through the Test Period ended December 31, 2024 — headroom narrowing steadily to a thin +0.017x that period, cured via a permitted equity contribution — and against the amended 4.00:1.00 threshold from March 31, 2025 onward, with headroom recovering to +0.280x and +0.390x. A resolver using only the original agreement, never learning of the amendment, would have wrongly flagged **both** post-amendment periods as covenant breaches. The correct resolver, built in this repository, did not.

---

## Limitations

- **Synthetic data only.** Every agreement and every financial figure in this repository is self-authored for demonstration purposes. Real-world drafting and data conventions were used as structural reference only (see notebook comments), never copied.
- **Starts post-OCR.** Agreements are provided as plain text. A production pipeline would need an OCR/ingestion step ahead of this one; that step is out of scope here.
- **Deliberately small facility count.** Three facilities is a design choice, not a shortcut — see Premise above.
- **Work in progress.** This is a rough, evolving prototype, not a finished product.