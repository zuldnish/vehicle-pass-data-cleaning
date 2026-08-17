# Monthly Vehicle Pass Records — Data Entry, Cleaning & Analysis

Cleaning and analysing 1,025 monthly vehicle pass sales from a multi-tenant trading complex, from handwritten receipts through to a dashboard.

I was hired for data entry with some light analysis attached. I keyed the records in by hand, and when the job was nearly over I went back through what I'd typed and found it wasn't ready to be analysed at all. This repository is the before and the after.

**Headline:** RM199,680 across 1,025 passes over six trading days. Along the way I found every date in the file was stored in the wrong month, 20 passes charged at the wrong rate, and roughly 140 repeat purchases that the original anonymisation had made invisible.

---

## Contents

| File | What it is |
|---|---|
| `01_raw_daily_records.xlsx` | The data as originally kept — one sheet per trading day, 1,021 records, every defect intact |
| `02_cleaned_analysis_workbook.xlsx` | Dashboard, analysis, cleaned master and unit occupancy |

Both files are anonymised. See [Privacy](#privacy).

---

## How it actually happened

**1. Manual entry.** Thousands of fields typed from physical receipts into one sheet per trading day, in Malay — `TARIKH`, `NO RESIT`, `NAMA`, `NO KAD PENGENALAN`, and so on. Six days: 1, 2, 3, 6, 7 and 8 July 2026.

**2. Combining.** On the last day I stacked the six daily sheets into a single master using `VSTACK` rather than copy-pasting by hand — repeatable, and it doesn't silently drop rows the way manual selection does.

**3. Cleaning.** This is where the work actually was. Details below.

**4. Analysis.** Category and daily revenue breakdowns, payment mix, document compliance, and unit-level occupancy, all built on live formulas so the numbers follow any edit to the source.

---

## What was wrong with the raw data

I keyed most of this in myself, so a fair share of these are mine. Finding them is the point.

**Every date was stored in the wrong month.** Dates were entered day-first but stored month-first, so `1/7/2026` was sitting in the file as 7 January 2026. All 1,025 records were affected, spread across seven different months.

I caught it because the pattern didn't survive a sanity check. Read month-first, the data claimed sales on the 7th of January, February, March, June, July and August — with April and May missing for no reason. Swap day and month and you get Wednesday 1, Thursday 2, Friday 3, Monday 6, Tuesday 7, Wednesday 8 July: a clean six-day run skipping Saturday and Sunday. A site that doesn't trade weekends explains the gap; six random months don't explain anything. The daily sheet tabs were named `172026`, `272026` and so on — day-first — which confirmed it independently.

**20 passes were charged at the wrong rate.** Rates are fixed per vehicle category, but 20 records disagreed with their own category — trader cars billed at RM450, lorries at RM150. Raw revenue as actually priced was RM198,770 against RM199,680 once reconciled: a **RM910 gap**.

I've flagged rather than silently fixed these, because the correction runs both ways. If the price is right and the category was mistyped, revenue was correct all along. Deciding that needs the physical receipts, not a spreadsheet.

**`"E-Wallet "` with a trailing space.** 87 entries had it; 15 didn't. Excel treated them as two separate payment methods, so every pivot table split e-wallet revenue across two rows and understated it. A second variant had three *leading* spaces. Invisible on screen, and the sort of thing that quietly corrupts a report.

**Receipt numbers aren't unique.** 117 duplicates, 67 of them within a single day, ranging from 1 to 16,775 with large gaps — consistent with several receipt books running at different counters simultaneously. Receipt number alone can't be a primary key here; it needs book plus number. 48 records had no receipt number at all, and 131 were stored as text with leading zeros alongside numeric ones.

**Blank cells meaning different things.** Unit number was blank on 42 records, `-` on 33, `N/A` on 3. These aren't missing data — the intake form for those vehicle types never asked for a unit number. Treating them as gaps would have inflated the missing-data rate by 7 percentage points and pointed at a problem that doesn't exist.

**Record count mismatch.** 1,021 records across the daily sheets against 1,025 in the master. Four records in the master have no counterpart in the source, and four rows carry no date at all. Unresolved, and documented rather than quietly reconciled.

---

## Cleaning decisions

Every rule below was confirmed with the people who collect the data rather than assumed from the values.

| Issue | Decision |
|---|---|
| Transposed dates | Swapped day and month across all records; verified against the weekday pattern and tab names |
| Two records dated 1/8 | Corrected to 1 July — they sat inside the 1 July block, and 1 August is a Saturday |
| Duplicate `BIL` | One hardcoded value broke the sequence; restored the formula so it runs 1–1025 |
| Whitespace | Trimmed across all text fields |
| Prices as text | Converted to numbers driven by a lookup against a documented rate table |
| Blank / `-` / `N/A` unit numbers | Normalised to a single "not provided" state, not counted as missing |
| Two units in one cell | Split into separate columns, one row per pass — splitting into rows would double-count revenue |
| Missing receipts | Flagged separately; percentages reported against recorded payments only, but the revenue still counts |

**Document compliance is category-dependent**, which took a few passes to get right. Trader cars must supply a full document set. Every other category needs only an IC — or any other document that traces back to the holder, such as a lorry grant. The same status can therefore be compliant on one row and non-compliant on another, so a single global rule would have been wrong in both directions.

---

## Findings

**Sales collapse after opening day.** 502 of 1,025 passes sold on 1 July, falling to 13 by 8 July. Day one took 49% of passes but only 44% of revenue, because it skewed toward RM150 trader cars while the higher-value lorries came in later.

**Lorries are 15% of passes but 30% of revenue.** Trader vehicles dominate volume at 74.5% yet contribute 57% of revenue. Volume and value point in different directions here.

**Compliance is a single-category problem.** 92.8% of records are complete, and 68 of the 74 shortfalls are trader cars — the only category required to produce a full document set.

**Nearly nine in ten passes are paid in cash**, which is an operational and reconciliation risk as much as a statistic.

**Around 140 passes are repeat purchases.** 874 distinct IC numbers across 1,014 records with an IC; 103 holders bought more than once and one bought seven times. This one is worth dwelling on: the original anonymisation assigned placeholders row by row, so every record looked like a different person and the repeat structure vanished. The dataset isn't "1,025 passes" — it's roughly 880 customers, some buying for several vehicles. That's a different business question.

---

## What I'd do differently

**Validate at entry, not after.** Dropdowns for category, payment and document status, and a date column formatted and checked on the way in. Most of what I spent days fixing could not have been typed in the first place. The file *had* dropdowns on some columns — padded values were bypassing them, which taught me that validation you don't test isn't validation.

**Never anonymise by row position.** Placeholder codes should map consistently per person, or you destroy exactly the relationships that make the data worth analysing. I only noticed because I went back to the raw file.

**Keep the raw file.** Every conclusion here depends on being able to compare against what was actually collected. The cleaned version alone would have hidden the rate mismatches entirely — it looks perfectly consistent, because someone had already smoothed them over.

**Reconcile counts before analysing.** The four-record discrepancy should have been caught at the combining step, not after the analysis was built.

---

## Privacy

The raw records contained real names, Malaysian NRIC numbers, phone numbers, company names and vehicle plates for over a thousand people. **None of that is in this repository.**

Identities were replaced with placeholders, mapped consistently so the same person keeps the same code across every sheet — repeat purchases survive, re-identification doesn't. Blanks stay blank, and NRIC and passport holders remain distinguishable. Document metadata was scrubbed, and the file was scanned at the XML level to confirm nothing survived in cached form.

Every data-quality defect described above is preserved deliberately in the raw sample. None of it is an artefact of anonymisation.

---

## Tools

Excel — `VSTACK`, `INDEX`/`MATCH`, `SUMIFS`/`COUNTIFS`, dynamic arrays, data validation, pivot tables, charts.

Summary tables are formula-driven rather than static pivots, so the workbook stays correct when the underlying data changes.
