<h1 align="center">Diabetes Care & Patient Follow-Up Analysis</h1>

<p align="center">
  <strong>Turning patient journeys into evidence for more focused outreach.</strong><br>
  2026 ASA DataFest · Stormont Vail Health, Kansas · Team Mangaussian
</p>

<p align="center">
  <a href="#overview">Overview</a> ·
  <a href="#key-findings">Findings</a> ·
  <a href="#predictive-modelling">Modelling</a> ·
  <a href="Report_Team_Mangaussian.pdf">Report</a> ·
  <a href="Slides_Team_Mangaussian.pdf">Slides</a> ·
  <a href="modeling_final_regular_checkups_4plus_improved_screener.ipynb">Notebook</a>
</p>

---

## Overview

This project examines **22,155 Type 2 diabetes care journeys** and **50,070 annual care records** from Stormont Vail Health (SVH) in Kansas. It investigates where recorded follow-up falls short of the project's benchmarks and develops a prototype for prioritizing outreach when care-team capacity is limited.

The analysis connects **data integration, longitudinal analysis, predictive modelling and resource planning**. Its proposed output, the **Journey Navigation Flag**, ranks annual records by their estimated likelihood of falling below a defined encounter-frequency benchmark.

This is a retrospective analytics project and screening prototype. Outreach recommendations were proposed; changes in patient outcomes were not measured.

## Key Findings

| Finding | Result | What it means |
| --- | ---: | --- |
| Type 2 diabetes journeys analysed | **22,155** | Patient-level histories used to examine recorded encounter patterns and first follow-up. |
| Journeys without recorded follow-up within 120 days | **61%** | Approximately six in ten journeys had no observed return encounter within the project's four-month window. |
| Median time to first follow-up | **91 days** | Among journeys with an observed return, half had their first follow-up within 91 days and half at or beyond 91 days. |
| Annual care records analysed | **50,070** | Each record represents one patient in one observed calendar year; patients can contribute multiple records. |
| Annual records meeting the four-encounter benchmark | **26.2%** | Approximately one in four records contained at least four diabetes-related encounters that year. |
| Median annual encounter count | **2** | Half of the annual records contained two or fewer diabetes-related encounters. |

**Recorded encounters reflect care visible in the SVH dataset.** An absent encounter does not establish that a patient received no care elsewhere.

### Two distinct measures

- **120-day follow-up:** A descriptive measure of whether a second diabetes-related encounter was recorded within 120 days of the first observed encounter.
- **Annual encounter frequency:** The modelling outcome. The improved screener identifies annual records with **fewer than four diabetes-related encounters**.

These measures answer different questions. The four-encounter threshold is a **project benchmark**, and an annual count does not establish that visits were evenly spaced.

## Analytical Approach

1. **Prepare and connect the data.** Load seven source tables covering encounters, patients, diagnoses, departments, providers, social determinants and census codes. Clean identifiers and dates, then enrich the focused diabetes encounter data with patient, department and social-risk information.
2. **Reconstruct care journeys.** Order encounters over time and calculate first follow-up intervals, encounter counts and operational journey categories.
3. **Build annual records.** Count distinct diabetes-related encounters within observed calendar years and construct calendar, first-encounter, patient-context and previous-observed-year features.
4. **Evaluate outreach priorities.** Compare classifiers, select a practical screening rule on validation data, and assess performance alongside the number of records flagged.

The notebook also includes coefficient interpretation, permutation importance and rule-based context labels to help communicate predictions. These labels describe patient context; they are not causal explanations.

## Predictive Modelling

### Target and model selection

The improved screener's positive class is **fewer than four recorded diabetes-related encounters in an observed year**.

The notebook first explores logistic regression and a decision tree, then compares four models for the improved screening task:

- Logistic regression
- Random forest
- Extra trees
- Histogram-based gradient boosting

**Histogram-based gradient boosting** achieved the highest validation average precision and was selected for the final screening rule.

The recorded experiment uses a stratified **60% training / 20% validation / 20% test split** of annual records: 30,042 for training, 10,014 for validation and 10,014 for testing. Model and screening-rule selection use the validation set. See the limitations below for the implications of splitting annual records rather than distinct patients.

### Recorded test results

The selected probability threshold was approximately **0.884**, chosen under an approximately 30% validation workload constraint.

| Metric | Result | Interpretation |
| --- | ---: | --- |
| ROC-AUC | **0.799** | Measures how well the model ranks records below the annual benchmark above those meeting it. |
| Average precision | **0.917** | Summarizes precision across recall levels; it is not an accuracy percentage. |
| Precision among flagged records | **94.9%** | About 95 of every 100 flagged records were below the annual encounter benchmark. |
| Recall of below-benchmark records | **38.2%** | The selected rule captured approximately 38 of every 100 records below the benchmark. |
| Share of records flagged | **29.7%** | The rule flagged just under three in ten annual records in the test set. |

Of **10,014 test records**, the rule flagged **2,975**: **2,824** were below the benchmark and **151** met it. It left **4,569** below-benchmark records unflagged.

Because **73.8%** of test records were already below the benchmark, precision should be interpreted against that baseline. The flagged group had approximately **1.29 times** the below-benchmark rate of a randomly selected group.

This illustrates a resource-allocation trade-off: a smaller outreach list can concentrate likely follow-up gaps while leaving many gaps outside the selected group.

## Proposed Application: Journey Navigation Flag

The prototype produces a ranked screening list with contextual labels, such as limited previous observed history, an inactive or unclear patient-portal status, or an emergency-based first encounter.

A care team could review flagged records to:

- Confirm whether follow-up has already occurred or is scheduled.
- Check transportation and appointment-access barriers.
- Offer MyChart or other digital-access support.
- Assess whether social-service referrals or care-navigation support would be useful.

For the Kansas community served by SVH, the project's contribution is an analytical basis for investigating follow-up gaps and planning outreach. Any improvement in access, continuity or health outcomes would need to be evaluated through a prospective implementation.

## Repository Guide

| File | Contents |
| --- | --- |
| [Analysis notebook](modeling_final_regular_checkups_4plus_improved_screener.ipynb) | Data preparation, journey construction, annual outcomes, model comparisons, screening evaluation and saved outputs. |
| [Project report](Report_Team_Mangaussian.pdf) | Concise summary of findings, the proposed navigation flag and recommendations. |
| [Presentation slides](Slides_Team_Mangaussian.pdf) | Visual explanation of the problem, findings and proposed outreach workflow. |
| [README.md](README.md) | Project overview, methods, results and reproduction notes. |

## Reviewing and Running the Analysis

The report, slides and saved notebook outputs can be reviewed without running the code. **The raw CSV files are not included in this repository.** Reproduction requires access to the original project data.

### Setup

Clone the repository and install the notebook's dependencies:

```bash
git clone https://github.com/sy0203/Diabetes-Care-and-Patient-Followup-Analysis.git
cd Diabetes-Care-and-Patient-Followup-Analysis
python -m pip install pandas numpy matplotlib seaborn scikit-learn jupyter ipykernel
```

Create a `data/` folder containing the expected inputs:

```text
encounters.csv
patients.csv
diagnosis.csv
departments.csv
providers.csv
social_determinants.csv
tigercensuscodes.csv
```

Open the notebook:

```bash
jupyter notebook modeling_final_regular_checkups_4plus_improved_screener.ipynb
```

Check `DATA_DIR` in the configuration cell, then run the notebook from top to bottom. The notebook writes processed tables and summaries to `data/processed/`. Dependency versions are not pinned in the repository.

## Limitations and Next Steps

- **Incomplete observation:** Care outside SVH is not captured. Journeys may extend beyond the available data window, and the annual table includes years with at least one observed diabetes-related encounter.
- **Unequal time at risk:** A patient first observed late in a calendar year has less opportunity to accumulate four encounters. A fixed follow-up window or exposure-adjusted outcome would help address this.
- **Patient overlap across splits:** Random splitting of annual records can place different years from the same patient in training and testing. Patient-grouped and chronological validation are needed to assess generalization more rigorously.
- **Feature timing:** Some social-risk features summarize the full observed record. They need to be restricted to information available at prediction time before treating the results as evidence of prospective screening performance.
- **Clinical and operational interpretation:** Encounter counts are a proxy for recorded utilization. Benchmark suitability, subgroup performance, missing social-risk information and the effects of outreach require further evaluation.

The reported metrics describe the saved retrospective experiment. They do not establish readiness for clinical deployment.

## Tools

**Python · pandas · NumPy · scikit-learn · Matplotlib · Jupyter Notebook**

## Team

**Team Mangaussian — Seyoung Yoo and Zihang Peng**  
2026 ASA DataFest
