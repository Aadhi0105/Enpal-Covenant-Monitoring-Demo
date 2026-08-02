# Synthetic Financials

Three CSVs, one per facility, each structured deliberately differently — the schema itself is part of the design, not an afterthought. This file explains every column, the actual numbers, and why each file is shaped the way it is.

All figures are self-authored for this project. None are derived from, or represent, any real Enpal or third-party financial data.

---

## `facility_a_periods.csv` — the control case

**Columns:** `facility, period, determination_date, original_pool_balance, outstanding_pool_balance_eligible, advance_balance, bucket_31_60, bucket_61_90, bucket_91_120, bucket_120_plus`

**Coverage:** 8 monthly reporting periods, March 2024 through October 2024.

**The numbers:** the current Outstanding Pool Balance grows gently across the year — €150.0M rising to €172.5M — reflecting a revolving warehouse facility where new receivables continuously replace amortizing ones, keeping the pool roughly stable-to-growing rather than steadily shrinking. The Advance Balance is held at a flat 88% of the pool balance every period, which keeps Overcollateralization steady at exactly 12.0% throughout — comfortably, unchangingly above the 8.00% target. The four aging buckets (31–60, 61–90, 91–120, 120+ days) rise slowly and proportionally, producing a Delinquency Ratio that climbs from 0.800% to 1.536% — nowhere near the 5.00% trigger.

**Why aging-bucket granularity, not a single pre-summed figure:** even though Facility A's own covenant only needs the 61–90/91–120/120+ buckets summed together, the raw data is still captured at full bucket-level detail, matching the real structure of a servicing report. This also means the *31–60* bucket is present but genuinely unused by any computation in this facility — included for realism and schema consistency with Facility B, not because Facility A needs it.

**Why `original_pool_balance` is present, even though Facility A's own covenant never uses it:** it's set flat, equal to the very first period's outstanding balance, specifically so Facility B's calculation convention could, if wanted, be tested against Facility A's data too — a fully symmetric cross-facility comparison, not just B tested against A's convention.

**Why every number here was chosen to stay healthy:** this file exists to prove the pipeline handles the easy case cleanly before anything is stress-tested. Nothing here is meant to be dramatic.

---

## `facility_b_periods.csv` — the definitional-inconsistency proof

**Columns:** `facility, period, determination_date, original_pool_balance, outstanding_pool_balance_eligible, bucket_31_60, bucket_61_90, bucket_91_120, bucket_120_plus, reserve_account_balance`

**Coverage:** 6 monthly reporting periods, November 2024 through April 2025.

**The numbers:** `original_pool_balance` is fixed at €220.0M in every single row — deliberately never changing, in direct contrast to Facility A's growing balance, because Facility B's own real covenant is defined against this fixed figure. `outstanding_pool_balance_eligible`, by contrast, amortizes steadily downward with no replenishment — €218.0M falling to €183.0M — because this facility isn't shown revolving, unlike A. The four aging buckets are deliberately back-loaded: they start small and accelerate sharply in the final two periods, specifically so that a *wrong* calculation applied to this data produces a rising false signal, while the *correct* calculation stays flat and low throughout. `reserve_account_balance` is held flat at exactly €4.4M every period — precisely 2.00% of the fixed original pool balance, satisfying the Reserve Account covenant with no drama, since that covenant's role in this project is to test the review pipeline, not to carry its own dramatic arc.

**Why this file must be bucket-level, not optional:** this is the single most important design decision behind the entire Facility B narrative. Facility B's real covenant needs the 91–120 and 120+ buckets (90-or-more-days delinquency) divided by the fixed original balance. A wrongly-applied calculation instead needs the 61–90, 91–120, and 120+ buckets (60-or-more-days delinquency) divided by the current, shrinking balance. Both calculations draw from the *same* underlying rows — the only reason it's possible to show "same data, two answers" at all is that the raw data was captured granular enough to support both computations from a single source. A pre-summed "delinquent balance" column would have made this comparison structurally impossible.

**Why the trajectory was tuned the way it was:** the back-loaded delinquency growth was deliberately shaped so the wrongly-calculated ratio crosses Facility B's own real 4.00% trigger by the second-to-last period — a genuine, computed false breach, not a hypothetical one.

---

## `facility_c_periods.csv` — the amendment-resolution proof

**Columns:** `facility, test_period_end, consolidated_net_debt, consolidated_ebitda_ltm, cure_contribution`

**Coverage:** 8 quarterly Test Periods, September 2023 through June 2025.

**The numbers:** Consolidated Net Debt climbs steadily from €178.0M to a peak of €215.0M, while Consolidated EBITDA (LTM) declines steadily from €64.5M down to €57.0M before a modest recovery — a combination deliberately chosen to push the raw leverage ratio (Net Debt ÷ EBITDA) from a comfortable 2.760x up toward, and briefly past, the original 3.50x limit. At the Test Period ended December 31, 2024, the raw ratio is 3.667x — a genuine breach of the original threshold — and a `cure_contribution` of exactly €3.0M is applied that period only, bringing the adjusted ratio to 3.483x: compliant, but by a razor-thin +0.017x, matching the original agreement's cure mechanic exactly as drafted. The two final periods, March and June 2025, sit at 3.720x and 3.610x respectively — both of which would breach the *original* 3.50x limit if tested against it, and both of which are genuinely compliant under the *amended* 4.00x limit that actually governs those dates.

**Why this schema is simpler than Facilities A and B's:** deliberately so. A Consolidated Net Leverage Ratio needs exactly three inputs — net debt, EBITDA, and any cure contribution — and nothing else feeds its calculation. Where Facilities A and B needed bucket-level granularity to make their specific proof possible, Facility C's proof (correct amendment resolution) depends on the *threshold* being correctly resolved per period, not on the underlying financial data being especially granular. Adding unused columns here would only obscure that the real complexity in this facility lives in the record store's resolver, not in the raw numbers.

**Why the December 2024 and March/June 2025 periods are the ones that matter most:** they're the three data points that make every claim in this project's results concrete rather than asserted — the cure mechanic working exactly as drafted, and the amendment's tiered threshold being the only thing standing between "correctly compliant" and "wrongly flagged as breach" for the two most recent quarters.