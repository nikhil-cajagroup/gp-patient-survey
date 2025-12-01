# GP Patient Survey – AWS Ingestion

This repository contains code to ingest the **GP Patient Survey** results into an AWS data lake.

The GP Patient Survey asks patients about their experience of GP practice services, including access, appointments, communication, and overall satisfaction.  [oai_citation:23‡NHS England](https://www.england.nhs.uk/statistics/statistical-work-areas/gp-patient-survey/?utm_source=chatgpt.com)

---

## Source dataset

- **Name:** GP Patient Survey  
- **Owner:** NHS England (survey conducted by Ipsos)  [oai_citation:24‡NHS England](https://www.england.nhs.uk/statistics/statistical-work-areas/gp-patient-survey/?utm_source=chatgpt.com)  
- **Official info page:** GP Patient Survey statistics / survey website.  
- **Licence:** Typically OGL – check licence notes on the data download pages.

The survey includes:

- Patient responses about appointment access, practice opening times, and online services.  
- Experience of care and communication with clinicians.  
- Some questions about wider health, pharmacy, and dentistry usage.  [oai_citation:25‡NHS England](https://www.england.nhs.uk/statistics/statistical-work-areas/gp-patient-survey/?utm_source=chatgpt.com)  

---

## Geographic coverage & granularity

- **Coverage:** England  
- **Granularity:**  
  - GP practice  
  - ICB / CCG  
  - Region  
  - National  

---

## Time coverage & refresh

- Generally **annual** survey with results published once per year.  [oai_citation:26‡NHS England Digital](https://digital.nhs.uk/data-and-information/publications/statistical/nhse-gp-patient-survey-results/2025?utm_source=chatgpt.com)  

---

## This project’s data model

- **Bronze layer**  
  - Raw CSV/Excel survey outputs as published (practice and higher-level tables).  
  - Per-year folders or partitions.  

- **Silver layer**  
  - Cleaned tables such as:
    - `fact_gp_patient_survey_scores` – practice-level percentage scores and denominators.  
    - `dim_question` – mapping of question codes to full text and themes (access, continuity, communication, etc.).  

---

## Repository contents

- `Untitled.ipynb` – initial ingestion notebook (to be renamed, e.g. `bronze_gp_patient_survey.ipynb`).  

---

## How to use

1. Download the latest GP Patient Survey CSV/Excel files from the official results site.  [oai_citation:27‡NHS England Digital](https://digital.nhs.uk/data-and-information/publications/statistical/nhse-gp-patient-survey-results/2025?utm_source=chatgpt.com)  
2. Run the Bronze notebook to ingest the files to S3.  
3. Build Silver tables for use in dashboards (e.g. patient experience vs workforce, appointments, or outcomes).

---

## Attribution

Credit NHS England and Ipsos for the GP Patient Survey, following the licence guidance on the official site.  [oai_citation:28‡NHS England](https://www.england.nhs.uk/statistics/statistical-work-areas/gp-patient-survey/?utm_source=chatgpt.com)
