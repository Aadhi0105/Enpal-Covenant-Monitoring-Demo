# Synthetic Agreements

Four documents, three facilities. Each facility is a deliberate, self-authored design choice — none of the covenant registers, definitions, or amendment structure were picked at random. This file explains what each document is, what it's built to prove, and why it was drafted the way it was.

None of these are real agreements. No real Enpal document, and no real document belonging to any other institution, was used or copied anywhere in this folder. Several real, publicly filed agreements — asset-backed servicing agreements and credit agreement amendments filed with the U.S. Securities and Exchange Commission — were read purely as structural and stylistic reference, to make the drafting conventions here realistic. Nothing from those real filings was copied into these documents.

---

## Facility A — `facility_a_credit_agreement.txt`

**The control case.**

**Structure:** A Warehouse Receivables Facility Agreement dated March 14, 2024, between Heliora Solar Funding I GmbH (the special-purpose vehicle borrower), Heliora Energy GmbH (originator and servicer), Meridian Capital Bank AG (facility agent and senior lender), and Nordbrücke Trust Services GmbH (collateral agent). Facility amount: EUR 180,000,000.

**Covenants:**
- **Delinquency Ratio / Delinquency Trigger** — the ratio of Delinquent Receivables (60 or more days past due, measured against the current Outstanding Pool Balance) to the total Outstanding Pool Balance. A Delinquency Trigger occurs if this ratio exceeds **5.00%**.
- **Overcollateralization Test** — satisfied if the Overcollateralization Amount (Outstanding Pool Balance of Eligible Receivables minus the Advance Balance) is at least **8.00%** of that same pool balance.

**Why this register:** an asset-backed warehouse facility, because that mirrors Enpal's actual, primary financing model — pools of customer receivables refinanced through a special-purpose vehicle.

**Why these two covenants specifically:** between them, they cover the two fundamental covenant families present in almost any securitization — a pool-performance trigger (delinquency) and a collateral-sufficiency test (overcollateralization). Both were drafted with real, publicly filed asset-backed servicing agreements and investor reports as structural reference, so the drafting conventions — how a delinquency trigger is defined, how an overcollateralization target is calculated — reflect genuine market practice.

**Why it's deliberately uneventful:** every period in the accompanying financial data stays comfortably compliant. This is intentional. If the extraction and monitoring pipeline can't handle the clean, healthy case correctly, nothing built on top of it is worth trusting. Facility A is also the reference point Facility B needs to exist — see below.

---

## Facility B — `facility_b_credit_agreement.txt`

**Isolates definitional inconsistency.**

**Structure:** A Warehouse Receivables Facility Agreement dated November 4, 2024, between Heliora Solar Funding II GmbH (borrower), Heliora Energy GmbH (originator and servicer), Solventa Bank SE (facility agent and senior lender), and Kastellan Trust & Agency GmbH (collateral agent). Facility amount: EUR 250,000,000.

**Covenants:**
- **Delinquency Ratio / Delinquency Trigger** — carries the **identical name** as Facility A's covenant, on purpose. Its actual definition is materially different: Delinquent Receivables here means 90 or more days past due, and the denominator is the **fixed Original Pool Balance** at closing, not the current, amortizing balance. A Delinquency Trigger occurs if this ratio exceeds **4.00%**.
- **Reserve Account Covenant** — a Required Reserve Amount equal to 2.00% of the Original Pool Balance must be maintained in a segregated Reserve Account.

**Why the same register as Facility A:** deliberately kept constant, so the *only* variable between the two facilities is the covenant definition itself, not the surrounding facility type.

**Why this specific definitional difference:** the combination — a different delinquency threshold (60 vs. 90 days) *and* a different denominator (current vs. original pool balance) — wasn't invented for effect. It mirrors a real, observed variation across actual filed asset-backed servicing agreements, where both conventions genuinely appear. The point of Facility B is to test whether reusing a familiar calculation convention across facilities, without re-checking the specific contract in front of you, produces a materially wrong answer. It does — see `synthetic_financials/README.md` for the numbers.

**Why the Reserve Account covenant exists:** to give Facility B its own independent identity — a real, distinct facility, not merely "Facility A with different numbers" — and to give the confidence-review pipeline (Notebook 2) a second, unrelated covenant to test. It's this covenant, not the Delinquency Ratio, that ended up genuinely flagged for human review during testing (a phrasing inconsistency across extraction passes, not a value disagreement).

---

## Facility C — `facility_c_credit_agreement.txt` (original) and `facility_c_amendment_1.txt` (amendment)

**Isolates amendment and lifecycle drift.**

### Original agreement

**Structure:** A Term Loan Credit Agreement dated June 3, 2023, between Heliora Group Holdco GmbH (borrower), Heliora Energy GmbH (guarantor), Ardenne Capital Bank AG (administrative agent), and its lenders. Facility amount: EUR 60,000,000.

**Covenant:** Consolidated Net Leverage Ratio must not exceed **3.50:1.00**, tested quarterly, for Test Periods ending on or after September 30, 2023 — no stated end date within this document alone.

**Cure right:** Section 6.02 permits the borrower to cure a breach, up to twice during the facility's term, by having the guarantor or its equityholders make a cash contribution, added to Consolidated EBITDA solely for the purpose of calculating the ratio.

**Why the register changes here:** a straightforward corporate term loan, leveraged-finance style, deliberately different from Facilities A and B. Two reasons: it more closely mirrors how Enpal's own real financing stack actually mixes asset-backed warehouse facilities with corporate-level borrowing; and it keeps the covenant itself instantly recognizable — a leverage ratio — so that any complexity in what follows comes from the amendment mechanic, not from an unfamiliar covenant type.

**Why the cure right is there:** genuinely common market practice in leveraged facilities, and it gives the amendment (below) a believable backstory — a cure already used once, making a formal amendment the natural next step rather than the first resort.

### Amendment

**Structure:** A First Amendment to Term Loan Credit Agreement and Waiver, dated April 11, 2025.

**What it does:**
- **Recitals** confirm the cure right was already used once, for the Test Period ended December 31, 2024, and that the borrower anticipates a further breach — in excess of 3.50:1.00 — for the Test Period ending March 31, 2025 (the "Specified Default"), and does not intend to use a second cure.
- **Waiver** of that one specific anticipated default.
- **Restated covenant**, Section 6.01, now tiered: **3.50:1.00** for Test Periods ending September 30, 2023 through December 31, 2024; **4.00:1.00** for Test Periods ending on or after March 31, 2025.

**Why this exact structure:** it follows the real pattern observed in actual filed credit agreement amendments — a recital acknowledging an anticipated breach, a waiver of that specific default, and a covenant restated with a new, tiered threshold carrying its own effective date. The tiering is deliberately set so the cutover falls exactly on a fiscal quarter boundary, and so that correctly resolving it requires keying off the **Test Period end date**, not the amendment's own signing date (April 2025) — the harder, more realistic version of the resolution problem, and the one this project's resolver (Notebook 3) is built to solve.

---

## A note on what these documents leave out

Every document here contains exactly the terms needed to demonstrate its intended point, and nothing extraneous. Facility A and B's agreements do not define a leverage ratio; Facility C's does not define a delinquency ratio. This isn't an oversight — a synthetic document scoped to prove one thing cleanly shouldn't carry unrelated complexity that no downstream notebook ever tests against.