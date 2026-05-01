# Vitiligo Registry and Bio-resource (VOICE) - Data Cleaning Pipeline

This file details the data preparation and cleaning pipeline for the VOICE database. Data cleaning was built and run in R version 4.5.0, and drew primarily on the `tidyverse` ecosystem for data manipulation, `lubridate` for date handling and temporal calculations, and custom logic for clinical string mapping. The raw database comprised multiple overlapping snapshots, including data from separate hospital systems where data collection and warehousing infrastructure have changed since the start of data collection. A key aspect of data cleaning was to reconcile, de-duplicate and harmonise these data sources.

---

## Key Data Cleaning Steps

To maintain data fidelity across overlapping database snapshots, the pipeline applies de-confliction rules, normalizes clinical terminology, and maps raw values to standardized clinical thresholds.

### 1. Patient Demographics & Cohort Derivation
* **De-conflicting Overlapping Records:** Demographic data (DoB, Sex, Ethnicity) appearing in multiple data sources were reconciled. Priority was given to the most recent data to correct legacy dummy dates (e.g., mid-year placeholders) found in earlier database snapshots.
* **Ethnicity Standardization:** Free-text entries were mapped to standard 16-category and 5-category groupings.
* **Site of Onset Processing:** Complex, multi-site free-text strings were tokenized, cleaned, and simplified. The script derives binary flags for facial onset and identifies the most common presentation sites across the cohort.
* **Age Derivation:** Age of onset and age at diagnosis were recalculated against the corrected Dates of Birth to resolve chronological conflicts.

### 2. Comorbidities
* **Standardization:** Free-text comorbidity entries were harmonised (e.g., mapping "t1dm" to "type 1 diabetes"). 
* **Phenotypic Categorization:** Comorbidities were grouped into broad binary flags for research utility:
    * *Autoimmune:* e.g., alopecia, pernicious anaemia, thyroid disease, lupus.
    * *Atopic / Allergic:* e.g., allergic rhinitis, asthma, atopic dermatitis.
    * *Thyroid:* Differentiated into hypothyroid, hyperthyroid, and unspecified autoimmune thyroid diseases.

### 3. Treatments and Medications
* **Classification:** Diverse raw medication strings and historical prescriptions were mapped to standardized treatment classes (e.g., `jak inhibitors`, `phototherapy`, `topical/oral antioxidants`, `topical/oral immunosuppressants`, `topical/oral steroids`).
* **Key Interventions:** Dedicated flags and earliest start dates were derived for targeted therapies of interest, specifically Monobenzyl Ether of Hydroquinone (MBEH) and Phototherapy.

### 4. Laboratory & Blood Data
* **Turnaround & Duplication:** Deduplicated tests based on highest priority source, retaining the latest valid result per day per patient. 
* **Outlier & Format Handling:** Cleaned alphanumeric results (e.g., ">50", "Jan-80" for ANA titres). Extreme outliers implausible for routine vitiligo care (e.g., transient acute liver injury markers during A&E admissions) were isolated and removed to avoid skewing longitudinal baselines.
* **Reference Range Classification:** Applied clinical reference ranges based on GSTT (Guy's and St Thomas') guidelines to classify results as *Normal*, *High*, *Low*, *Deficient*, or *Borderline*. Reference ranges were dynamically applied based on the patient's age and ethnicity at the time of the test (crucial for metrics like Serum B12, Albumin, ALP, and MMA).

### 5. Clinical Assessments & Vitiligo Extent Score (VES)
* **Deduplication:** Where conflicting VES scores appeared on the same day, the highest score was retained, as lower scores were found to represent incomplete/missing sub-region assessments.
* **Date Correction:** Manually investigated and corrected clear typographical errors in assessment years by cross-referencing longitudinal follow-up intervals and PROMs data collection dates.

### 6. Patient-Reported Outcome Measures (PROMs)
* **Orphan Record Resolution:** Psychometric questionnaire records lacking a direct clinical appointment ID were mapped to the nearest clinical assessment date, using a tolerance window of ±60 days.
* **Scoring Normalization:** PHQ-8 and PHQ-9 scale inputs were merged and harmonized into a standardized 8-item score to provide continuous longitudinal tracking of PHQ-8 items.

---

## Generated Outputs

The pipeline yields a set of structured, relational data tables for downstream epidemiological and statistical modelling:
* Patient Table: Demographics and disease onset variables.
* Comorbidity Table: Comorbidities and year of diagnosis
* Treatment Table: Longitudinal treatment histories.
* Blood Table: Cleaned lab values and classifications.
* Clinical Assessments Table: Cleaned and deduplicated longitudinal clinical assessment data, including clinical markers and Vitiligo Extent Score (VES).
* Questionnaire Table: Cleaned PROMs (DLQI, VIPS, VNS, BIPQ, GAD7, PHQ8/9).
