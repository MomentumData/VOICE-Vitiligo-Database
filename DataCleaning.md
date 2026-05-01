# Vitiligo Database (VDB) - Data Cleaning Pipeline

This repository contains the data preparation and cleaning pipeline for the Vitiligo Database (VDB). The primary script (`VDB_cleaning_v0.1.Rmd`) processes raw cuts from the clinic database software and hospital records, transforming them into a standardized, research-ready registry. 

This document serves as a methodological reference for research manuscripts utilizing this dataset, ensuring transparency in how patient data, clinical assessments, and laboratory values were curated.

---

## 📂 Data Sources

The database aggregates information from three sequential data cuts to capture primary clinical data, updated demographics, and auxiliary hospital records:
* **October 2024 ("Original")**: Primary clinic database cut containing enrolled patient demographics, comorbidities, clinical assessments, audiology/ophthalmology, bloods, and treatments.
* **November 2024**: Refresh containing psychological data (PROMs), updated demographics, and hospital inpatient/outpatient data (BP, BMI, Labs, Medications) for a subset of patients.
* **December 2024**: Final refresh containing further updated demographics and a larger pool of longitudinal hospital data.

---

## 🧹 Key Data Cleaning Steps

To maintain data integrity across multiple overlapping snapshots, the pipeline applies strict de-confliction rules, normalizes clinical terminology, and maps raw values to standardized clinical thresholds.

### 1. Patient Demographics & Cohort Derivation
* **De-conflicting Overlapping Records:** Demographic data (DOB, Sex, Ethnicity) appearing in multiple data cuts were reconciled. Priority was given to the most recent data (December > November > October) to correct legacy dummy dates (e.g., mid-year placeholders) found in earlier database cuts.
* **Ethnicity Standardization:** Free-text and varied ethnicity codes were mapped to standard 18-category and 5-category groupings, cross-referenced with CPRD Aurum standards. Where missing, skin phototype (Fitzpatrick scale, Tanning/Burning responses) was used as a secondary validation tool.
* **Site of Onset Processing:** Complex, multi-site free-text strings were tokenized, cleaned, and simplified. The script derives binary flags for facial onset and identifies the most common presentation sites across the cohort.
* **Age Derivation:** Age of onset and age at diagnosis were strictly recalculated against the corrected Dates of Birth to resolve chronological conflicts.

### 2. Comorbidities
* **Standardization:** Free-text comorbidity entries were lowercased and harmonized (e.g., mapping "t1dm" to "type 1 diabetes"). 
* **Phenotypic Categorization:** Comorbidities were grouped into broad binary flags for research utility:
    * *Autoimmune:* e.g., alopecia, pernicious anaemia, thyroid disease, lupus.
    * *Atopic / Allergic:* e.g., allergic rhinitis, asthma, atopic dermatitis.
    * *Thyroid:* Differentiated into hypothyroid, hyperthyroid, and unspecified autoimmune thyroid diseases.
    * *VKH:* Vogt-Koyanagi-Harada syndrome and Alezzandrini syndrome.

### 3. Treatments and Medications
* **Classification:** Diverse raw medication strings and historical prescriptions were mapped to standardized treatment classes (`cosmetic camouflage`, `depigmenting treatment`, `jak inhibitors`, `phototherapy`, `topical/oral antioxidants`, `topical/oral immunosuppressants`, `topical/oral steroids`, and `surgical approaches`).
* **Key Interventions:** Dedicated flags and earliest start dates were derived for targeted therapies of interest, specifically Monobenzyl Ether of Hydroquinone (MBEH) and Phototherapy.

### 4. Laboratory & Blood Data
* **Turnaround & Duplication:** Deduplicated tests based on highest priority source, retaining the latest valid result per day per patient. 
* **Outlier & Format Handling:** Cleaned alphanumeric results (e.g., ">50", "Jan-80" for ANA titres). Extreme outliers implausible for routine vitiligo care (e.g., transient acute liver injury markers during A&E admissions) were isolated and removed to avoid skewing longitudinal baselines.
* **Reference Range Classification:** Applied clinical reference ranges based on GSTT (Guy's and St Thomas') guidelines to classify results as *Normal*, *High*, *Low*, *Deficient*, or *Borderline*. Reference ranges were dynamically applied based on the patient's age and ethnicity at the time of the test (crucial for metrics like Serum B12, Albumin, ALP, and MMA).

### 5. Clinical Assessments & Vitiligo Extent Score (VES)
* **Deduplication:** Where conflicting VES scores appeared on the same day, the highest score was retained (assuming lower scores represented incomplete/missing sub-region assessments).
* **Date Correction:** Manually investigated and corrected clear typographical errors in assessment years (e.g., 2034 corrected to 2024) by cross-referencing longitudinal follow-up intervals and psych data collection dates.
* **Missing Data Imputation (Universalis):** Investigated instances of zero or `NA` VES scores. Where clinical subtype was persistently recorded as "Vitiligo Universalis" or historically stable at ~100% involvement, missing scores were imputed by carrying forward the previous maximum extent, counteracting recording bias for severe, stable disease.

### 6. Patient-Reported Outcome Measures (PROMs)
* **Orphan Record Resolution:** Psychometric questionnaire records lacking a direct clinical appointment ID were mapped to the nearest clinical assessment date. A strict tolerance window of ±60 days was applied.
* **Scoring Normalization:** PHQ-8 and PHQ-9 scale inputs were merged and harmonized into a standardized 8-item score to ensure continuous longitudinal tracking. Validated clinical cut-points (e.g., ≥ 10 for probable major depression) were automatically applied to generate categorical variables.

---

## 📦 Generated Outputs

The pipeline yields a set of structured, relational `.RData` files ready for downstream epidemiological and statistical modelling:
* `patient_table.RData`: Master demographics and disease onset variables.
* `comorbidities_table.RData` & `comorbidity_profile.RData`: Harmonized diagnoses and phenotype flags.
* `treatment_table.RData`: Longitudinal treatment histories and classifications.
* `blood_table.RData`: Cleaned lab values with dynamic reference classifications.
* `clinical_assessments_table.RData`: Deduplicated progression timeline (VES scores).
* `questionnaire_table.RData`: Cleaned PROMs (DLQI, VIPS, VNS, BIPQ, GAD7, PHQ8/9).

## 🛠️ Dependencies
The cleaning pipeline is built in R and heavily relies on the `tidyverse` ecosystem for data manipulation, `lubridate` for temporal calculations, and custom logic for clinical string mapping.
