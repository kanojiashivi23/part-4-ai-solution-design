# Part 4: AI Solution Design – Healthcare Medical Image Triage

## Overview

This repository contains the AI Solution Design submission for **Part 4** of the AI Business Solutions module. The selected domain is **Healthcare**, with a focus on automating **Medical Image Triage** using deep learning.

## Repository Structure

```
part-4-ai-solution-design/
│
├── README.md                        ← This file
├── solution_report.md               ← Full 8-task solution design report
└── diagrams/
    └── solution_architecture.png    ← System architecture diagram
```

## Domain

**Healthcare** — Medical Image Triage (X-ray / CT Scan Classification)

## AI Task Type

**Image Classification** using a CNN-based Transfer Learning model (ResNet-50)

## Quick Summary

| Component        | Detail                                      |
|------------------|---------------------------------------------|
| Problem          | Radiologists overwhelmed by high scan volumes|
| AI Solution      | Automated urgency triage of X-ray images    |
| Model            | Transfer learning on ResNet-50              |
| Key Metric       | Sensitivity ≥ 95%, review time reduced 40%  |
| Primary Risk     | Incorrect diagnosis / missed critical cases |
| Mitigation       | Human-in-the-loop for all flagged cases     |

## How to Navigate

1. Start with `solution_report.md` for the complete 8-task design document.
2. Refer to `diagrams/solution_architecture.png` for a visual overview of the system pipeline.

## Reference Files Used

- `ai_usecase_reference_catalog.csv` — domain and model selection reference
- `business_kpi_sample.csv` — baseline KPI data for business impact estimation
- `data_dictionary.md` — dataset field descriptions
