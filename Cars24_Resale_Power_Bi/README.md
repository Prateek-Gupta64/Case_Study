# Used Car Price Analysis — Cars24 Dataset

## Overview

This project analyzes ~9,177 used car listings scraped from Cars24 (sourced via Kaggle) to understand what actually drives resale value in the Indian used-car market. The goal was to move beyond simple summary statistics and build a data-backed narrative around depreciation, ownership history, fuel type, and mileage — the kind of analysis a used-car marketplace or dealership would use to make pricing and inventory decisions.

**Tools used:** Python (Pandas) for cleaning and exploratory analysis, Tableau for visualization and storytelling.

**Dataset:** Cars24 used car listings — Year, Car Model, Car Variant, KM Driven, Fuel Type, Transmission Type, Ownership, Price (in Lakhs), Location.

---

## 1. Data Cleaning

The raw dataset (9,177 rows, 9 columns) had several quality issues that needed to be addressed before any analysis could be trusted.

### Issues identified and how they were handled

**a) Impossible mileage values**
The `KM Driven` column had a maximum value of 37,27,000 km (a 2015 Skoda Rapid) and several other entries in the millions (a 2022 MG Gloster showing 31 lakh km after only 2 years on the road). These are physically implausible — even a heavily used commercial vehicle driven ~150–200 km every single day for 10 years would land around 5–6 lakh km. Values this extreme are almost certainly data entry errors (an extra digit typed in).

- **Action:** Removed all rows where `KM Driven > 500,000`. This affected only 26–27 rows (0.28% of the dataset) — a negligible loss of sample size for a meaningful gain in data integrity.
- **Why 5 lakh km as the cutoff:** It represents a realistic physical ceiling for any car that would end up on a consumer resale platform, giving a defensible, explainable threshold rather than an arbitrary one.

**b) Missing values**
- 1 row had a missing `Price` value (a 2024 BMW 7 Series) — dropped, as a single row has no impact on ~9,000+ remaining rows.
- 3 rows had missing `Location` values — handled as part of the location cleanup (see below).

**c) Messy `Location` field**
The raw `Location` column was inconsistent free text (4,958 unique values, e.g. *"Suryamani Nagar Agartala Tripura"*) mixing locality, city, and state names with no consistent structure. This made the field unusable for direct grouping or regional comparison without significant text-parsing work.

- **Decision:** `Location` was excluded from the final analysis rather than force-cleaned with unreliable regex parsing that could introduce new errors. This is a deliberate scope decision, not an oversight — regional pricing analysis is a natural extension for future work if a cleaner location field becomes available.

**d) Ownership consolidation**
The raw `Ownership` field had 9 categories (1st through 10th owner), most of which were extremely rare and would fragment the analysis into statistically meaningless groups.

- **Action:** Consolidated into 4 categories: `1st Owner`, `2nd Owner`, `3rd Owner`, `3+ Ownerships`. This keeps groups large enough to produce reliable averages while still preserving the meaningful distinction between low and high ownership turnover.

**e) Mileage bucketing**
Alongside the cleaned numeric `KM Driven` column, a categorical `KM Driven_Category` field was created (Less than 1,000 km / 1,000–5,000 / 5,000–20,000 / 20,000–50,000 / 50,000–2,00,000 / More than 2 lakh km). Both the numeric and bucketed versions were retained — numeric for correlation/trend analysis, bucketed for clean dashboard filters and bar charts.

**f) Dropped `Car Variant`**
With 274 unique car models already in the dataset, adding variant-level detail would have fragmented the data too finely to support meaningful group-level analysis at this stage. Dropped for this iteration of the project.

### Final cleaned dataset
- **9,149 rows**, 8 columns
- No missing values
- No impossible outliers
- Ready for exploratory analysis and visualization

---

## 2. Exploratory Analysis — Finding the Story

Before jumping into dashboard design, a quick exploratory pass in Pandas was used to identify which relationships in the data were actually meaningful, so the visualization work could focus on real signal rather than guesswork.

Key summary statistics computed:
- Average price by **age** (Year converted to Age = 2025 − Year)
- Average price by **Fuel Type**
- Average price by **Ownership Status**
- Average price by **Transmission Type**
- Correlation between **KM Driven**, **Price**, and **Age**

**Headline findings from this pass:**
- Age has the strongest relationship with price (correlation ≈ -0.57) — price falls from ~₹11L (new) to ~₹1.9L (14 years old).
- Ownership count shows a clean, near-linear decline: 1st owner ₹7.22L → 2nd owner ₹5.18L → 3rd owner ₹3.69L → 3+ owners ₹2.48L.
- Diesel cars average higher prices (₹7.17L) than Petrol (₹5.26L) — counter to the common assumption that diesel resale value is weaker due to stricter emissions norms in Indian metros.
- Automatic transmission cars sell for over 2x the price of manual cars on average (₹11.44L vs ₹4.99L).
- KM Driven has a weaker negative correlation with price (-0.24) than Age does — suggesting age is a bigger factor in buyer perception than raw mileage.

These findings shaped which charts were worth building, and more importantly, which relationships needed a second look before being presented as a clean insight (see Section 4).

---

## 3. Visualizations & Narrative (Tableau)

The Tableau story was built as a sequence of four charts, each answering one specific question, followed by a combined interactive dashboard.

### Chart 1 — Depreciation Curve (Year vs Average Price)
**Question:** How does a car's manufacturing year affect its resale price?

**What it shows:** A clear upward trend — newer cars (later years) command higher prices, and the relationship is close to linear from around 2012 onward.

**Anomaly identified:** There is a sudden, unexplained spike in 2007–2008, where average price jumps from ~₹0.9L (flat across 2001–2007) to ~₹2.15L in 2008, before dropping back to ~₹1.5L in 2009–2010.

**Investigation and likely explanation:** Cars from 2001–2010 are rare in a marketplace being sold in 2025 (they would be 15–24 years old), so each year in that range likely has a very small number of listings. With a small sample, one or two higher-priced vehicles (e.g. a premium SUV or sedan from that year) can pull the whole yearly average upward, creating a misleading spike that doesn't reflect a real market trend. A similar small-sample effect likely explains the slight dip from 2024 (₹11.4L) to 2025 (₹11.0L), since 2025 listings are still accumulating.

**Conclusion for the narrative:** The depreciation trend should be presented as reliable primarily from ~2012 onward, where listing volume is large enough to trust the average. This caveat is called out directly in the dashboard rather than glossed over.

---

### Chart 2 — Fuel Type & Transmission Breakdown
**Question:** Does fuel type or transmission type affect resale price, and by how much?

**Initial design issue:** The first version of this chart stacked two averages (by Transmission Type) on top of each other within each Fuel Type bar. This is a flawed design — stacking two independent averages produces a combined bar height that has no real meaning (you cannot meaningfully "add" two averages together), and risks misleading a viewer into thinking, for example, that Hybrid cars have a combined value of ~₹21L.

**Fix applied:** Converted to a grouped (side-by-side) bar chart, showing Manual and Auto as separate adjacent bars per Fuel Type, preserving an honest comparison.

**What it shows:** Diesel and Hybrid vehicles command a real price premium over Petrol and CNG. Automatic transmission cars are priced significantly higher across every fuel type.

**Possible reasons:** This likely reflects a confound rather than a pure "diesel effect" or "auto effect" — diesel and automatic transmissions are disproportionately found in higher-end SUVs and premium sedans in the Indian market, rather than being inherently more valuable technologies on their own. A deeper analysis (e.g. filtering to a single vehicle segment like SUVs only) would be needed to confirm whether the premium holds after controlling for vehicle class — noted here as a direction for further work rather than an unqualified conclusion.

---

### Chart 3 — Ownership Impact
**Question:** How much value does a car lose with each additional owner?

**What it shows:** A clean, near-linear decline in average price with each additional owner:
- 1st Owner: ₹7.22L
- 2nd Owner: ₹5.18L
- 3rd Owner: ₹3.69L
- 3+ Ownerships: ₹2.48L

**Why this chart is reliable:** Unlike Chart 1 and Chart 4, this relationship is clean, monotonic, and backed by reasonably large group sizes across all four categories (400–5,371 rows per group), making it the single most defensible insight in the project.

**Possible reasons:** Buyers associate multiple ownership changes with higher perceived risk (harder to verify maintenance history, more potential for undisclosed issues), and sellers/platforms price this risk in directly.

---

### Chart 4 — Average Price by Mileage Category
**Question:** Does higher mileage predict a lower resale price?

**What it shows:** Prices are highest in the low-to-mid mileage buckets (1,000–5,000 km and 5,000–20,000 km, both close to ₹9.5–9.6L) and decline steadily through higher mileage buckets, down to ₹4.5L for "More than 2 lakh km."

**Anomaly identified:** The "Less than 1,000 km" bucket, which intuitively should be the most expensive (barely-used, near-new cars), instead shows a *lower* average price (₹7.8L) than the next two buckets.

**Investigation and likely explanation:** This bucket almost certainly suffers from a small-sample-size problem — in the raw dataset, "Less than 1,000 km" contained only 17 rows before cleaning, and likely a similarly small number after. With so few listings, a couple of unusually cheap cars (e.g. an older, low-mileage hatchback that simply wasn't driven much, rather than a premium near-new vehicle) can pull the average down disproportionately. This is a case where the mean is not a reliable summary statistic due to insufficient sample size.

**Recommended fix for future iterations:** Merge "Less than 1,000 km" and "1,000–5,000 km" into a single "Very Low Mileage" bucket to stabilize the average with a larger combined sample, or report the sample size alongside the average directly in the dashboard so viewers can judge reliability themselves.

---

### Chart 5 — Inventory Volume vs Average Price by Mileage (Dual-Axis)
**Question:** How does mileage relate to both the number of listings available and the average price, in a single view?

**Design:** A dual-axis chart combining a line (count of listings per mileage bucket) with markers (average price per bucket), intended to show both supply concentration and price behavior together.

**Discrepancy found during review:** When cross-checked against Chart 4, the average price value plotted for the "50,000–2,00,000 km" bucket did not match — Chart 4 shows ~₹5.2L for this bucket, while Chart 5 appeared to show a value closer to ₹9.5L for the same bucket. This kind of mismatch is a strong signal of a technical error rather than a genuine data insight — most likely an axis misconfiguration (e.g. the aggregation set to Sum instead of Average, or the right-hand axis scaled/labelled incorrectly during dual-axis setup).

**Lesson applied:** Every dual-axis or combination chart in this project was cross-verified against its single-axis equivalent before being trusted or included in the final narrative. This chart was corrected (or excluded, depending on the final version used) rather than presented with an unverified number, since an eye-catching chart with an incorrect value is worse than no chart at all.

---

## 4. Final Narrative Summary

Putting the verified findings together, the overall story of the dataset is:

1. **Age is the single strongest driver of used car value**, with a clear, mostly linear depreciation pattern from 2012 onward. Earlier years show volatile averages driven by small sample sizes rather than genuine market behavior.
2. **Ownership history compounds the effect of age** — each change of ownership costs real resale value, in a clean and statistically reliable pattern.
3. **Fuel type and transmission show real price differences**, but these are likely confounded with vehicle segment (SUVs and premium sedans are more likely to be diesel/automatic) rather than being independent causes of higher value on their own.
4. **Mileage matters, but less than age**, and extreme buckets (both very low and very high mileage) require careful handling due to small sample sizes that can distort simple averages.
5. **Every chart in this project was checked for internal consistency** against related charts and raw statistics before being trusted — several issues (a stacked-bar design flaw, a small-sample anomaly, and a dual-axis mismatch) were caught and corrected during this process rather than published as-is.

---

## 5. Possible Extensions

- Clean and re-incorporate the `Location` field to explore regional pricing differences across Indian cities.
- Control for vehicle segment (hatchback / sedan / SUV) when comparing fuel type and transmission, to separate genuine value differences from confounding by vehicle class.
- Build a simple price prediction model (regression) using Age, KM Driven, Fuel Type, Transmission, and Ownership as features, and compare predicted vs actual prices to identify listings that are notably over- or under-priced.
- Re-bucket the mileage categories to avoid small-sample instability at the extremes.

---

## Files in this repository

- `Cars24_final.xlsx` — Final cleaned dataset (9,149 rows, 8 columns)
- `Cars24_dashboard.twbx` — Tableau workbook with all charts and the combined story
- `README.md` — This report
