# AI Solution Design Report
## Domain: Healthcare — Medical Image Triage

**Module:** Part 4 – AI Solution Design  
**Prepared by:** AI Business Solutions Student  
**Date:** May 2026

---

## Task 1: Business Domain

**Selected Domain: Healthcare**

Healthcare is one of the most data-intensive and high-stakes domains for AI adoption. Hospitals and diagnostic centres generate millions of medical images daily — X-rays, CT scans, MRIs — and radiologists are expected to review each one accurately and promptly. AI-powered image triage directly addresses one of the most critical bottlenecks in modern healthcare delivery: the speed and accuracy of diagnostic imaging review.

---

## Task 2: Define the Business Problem

### What problem is being solved?

Radiology departments in hospitals face a growing backlog of medical images — particularly chest X-rays and CT scans — that must be reviewed and prioritised for urgency. Radiologists must manually examine every scan, determine whether it contains critical findings (e.g., pneumothorax, pulmonary embolism, tumour), and decide how quickly a patient needs further care.

This manual triage process is slow, error-prone under fatigue, and creates dangerous delays in identifying life-threatening conditions.

### Who are the users or stakeholders?

| Stakeholder          | Role                                                             |
|----------------------|------------------------------------------------------------------|
| Radiologists         | Primary end-users; receive AI-prioritised worklist              |
| Emergency physicians | Depend on rapid scan results for treatment decisions            |
| Hospital management  | Measure operational KPIs: throughput, turnaround time, errors   |
| Patients             | Receive faster diagnosis and earlier treatment                  |
| IT/Data team         | Integrate AI model into the PACS (imaging system)               |

### What is the current manual or traditional process?

Currently, X-rays and CT scans are added to a radiologist's worklist in the order they are received (first-in, first-out). The radiologist opens each scan in a PACS (Picture Archiving and Communication System), visually inspects it, and dictates a report. A reporting clerk or voice-recognition system transcribes the report, which is sent to the referring physician.

### Limitations of the current process

- **No automatic urgency prioritisation** — critical cases may wait behind routine ones
- **Radiologist fatigue** causes missed findings, especially during night shifts or peak periods
- **High scan volumes** exceed radiologist capacity in under-resourced hospitals
- **Inconsistent reporting quality** depending on experience level and shift timing
- **No decision support** to flag anomalous scans before radiologist review
- **Slow turnaround time** — average resolution time of 21–44 hours as seen in the KPI data

---

## Task 3: AI Task Type

**Selected AI Task Type: Image Classification**

### Classification Categories

The model classifies each medical image into one of three urgency tiers:

| Label | Description                                              |
|-------|----------------------------------------------------------|
| 0     | **Normal / Routine** — no critical findings detected     |
| 1     | **Significant** — abnormality present, non-urgent review |
| 2     | **Critical** — urgent finding requiring immediate action |

### Why Image Classification is Suitable

Medical image triage is fundamentally a **visual pattern recognition problem**. Each X-ray or CT scan is a structured image containing spatial features (shapes, densities, borders) that correlate with diagnoses. Image classification is the correct AI task type because:

- The input is a fixed-size image (or standardised image slice)
- The output is a discrete category (urgency tier)
- Labelled training data is available from historical radiologist reports
- The model needs to extract spatial features (e.g., white-out in pneumothorax), which CNNs are specifically designed to do
- It is not a regression problem (the output is categorical, not continuous)
- It is not detection (we do not need to localise the finding — only classify urgency)

---

## Task 4: Data Requirement Plan

### Type of Data Required

| Data Type             | Description                                                       |
|-----------------------|-------------------------------------------------------------------|
| Medical images        | Chest X-ray and CT scan images in DICOM format                   |
| Radiologist reports   | Historical text reports used to extract urgency labels           |
| Patient metadata      | Age, sex, ward type, referral source (structured, tabular)       |
| Scan metadata         | Modality, body part examined, image quality flags                |

### Structured vs Unstructured

| Category       | Data                            | Type         |
|----------------|---------------------------------|--------------|
| Primary input  | X-ray / CT scan images          | Unstructured |
| Labels source  | Radiologist report text         | Unstructured |
| Supplementary  | Patient and scan metadata       | Structured   |

### Input Features

- Pixel matrix of the image (pre-processed to 224×224 or 512×512 resolution)
- Normalised pixel intensity values (standardised against training mean/std)
- Optionally: patient age, sex as auxiliary inputs to the final classifier layer

### Target Variable / Labels

- **Label**: Urgency tier — {0: Normal, 1: Significant, 2: Critical}
- Labels are derived by mapping radiologist report text to urgency categories using a combination of keyword rules and clinical review

### Data Collection Method

- Extract historical DICOM images and paired reports from the hospital PACS
- Clinical informatics team maps free-text reports to urgency labels (semi-automated with clinician validation)
- Target a minimum of **50,000 labelled images** for initial training, with ongoing data collection for fine-tuning
- Use publicly available datasets for pre-training (e.g., NIH ChestX-ray14, CheXpert)

### Data Quality Risks

| Risk                                | Description                                                                 | Mitigation                                              |
|-------------------------------------|-----------------------------------------------------------------------------|---------------------------------------------------------|
| Label noise                         | Inconsistent urgency assignment across different radiologists               | Multi-annotator consensus labelling; inter-rater review |
| Class imbalance                     | Critical cases are rare compared to routine scans                           | Oversampling, class-weighted loss function              |
| Image quality variation             | Different scanners, exposure settings, patient positioning                  | DICOM normalisation; augmentation during training       |
| Missing metadata                    | Some older records lack age or referral details                             | Imputation; model trained to work without aux features  |
| Privacy and de-identification risk  | Images may contain embedded patient info in DICOM headers                  | Mandatory de-identification before ingestion            |

---

## Task 5: Model Recommendation

**Recommended Model: Transfer Learning on ResNet-50 (CNN-based)**

### Architecture Choice

| Option              | Consideration                                                         |
|---------------------|-----------------------------------------------------------------------|
| Feed-forward NN     | Cannot process spatial image structure; not suitable                  |
| Standard CNN        | Suitable, but requires large dataset to train from scratch           |
| **ResNet-50 (TL)**  | ✅ Pre-trained on ImageNet; fine-tuned on medical images; best balance|
| Vision Transformer  | High performance but requires very large data and compute budget     |
| RNN / LSTM          | Designed for sequences, not 2D image grids; not applicable           |

### Why ResNet-50 with Transfer Learning

**Transfer learning** allows the model to leverage features already learned from millions of natural images (edges, textures, shapes), then fine-tune the upper layers on labelled medical images. This is the industry-standard approach for medical imaging tasks because:

- **Data efficiency**: Works well with 10,000–100,000 labelled images — far fewer than training from scratch
- **Proven architecture**: ResNet-50's residual connections prevent vanishing gradients in deep networks
- **Established in literature**: ResNet-based models underpin leading medical imaging benchmarks (CheXNet, etc.)
- **Practical deployment**: Model size is manageable for hospital IT infrastructure
- **Interpretability support**: Grad-CAM visualisations can highlight which regions influenced the prediction

### Training Strategy

1. Load ResNet-50 pre-trained on ImageNet
2. Freeze the first 3 residual blocks (feature extraction layers)
3. Replace the final fully connected layer with a 3-class softmax head
4. Fine-tune on the labelled hospital dataset with class-weighted cross-entropy loss
5. Apply data augmentation: horizontal flip, rotation ±10°, brightness jitter

---

## Task 6: Evaluation Plan

### Technical Metrics

| Metric             | Target       | Why it Matters                                                  |
|--------------------|--------------|------------------------------------------------------------------|
| **Sensitivity / Recall** | ≥ 95% for Critical class | Missing a critical case is the highest-risk failure |
| **Specificity**    | ≥ 85%        | Avoid over-triggering urgent alerts (alert fatigue)             |
| **AUC-ROC**        | ≥ 0.92       | Overall discriminative performance across thresholds            |
| **F1-score (macro)**| ≥ 0.88      | Balanced measure across all three urgency tiers                 |
| **Top-1 Accuracy** | ≥ 90%        | Gross overall correctness                                        |

### Business Metrics

Using the `business_kpi_sample.csv` baseline data as a reference:

| KPI                           | Current Baseline (Avg)   | Target After AI Deployment |
|-------------------------------|--------------------------|----------------------------|
| Average resolution time (hrs) | ~30 hours                | ≤ 18 hours (40% reduction) |
| Manual processing hours/month | ~453 hours               | ≤ 280 hours (38% reduction)|
| Error rate (%)                | ~6.8%                    | ≤ 4.0%                     |
| Customer satisfaction score   | 7.1 / 10                 | ≥ 8.0 / 10                 |
| Critical case escalation time | Not tracked              | < 30 minutes from scan     |

### Possible Failure Cases

- Model misclassifies a critical pneumothorax as "routine" — patient delay in treatment
- Model over-flags routine scans as critical — radiologist alert fatigue reduces trust in AI
- Model performs poorly on paediatric or rare-condition images underrepresented in training
- Adversarial image artefacts (e.g., pacemaker shadow) confuse the model

### Human Review and Validation Process

- **All Critical (Class 2) predictions** are sent immediately to the radiologist worklist; AI flag is shown as advisory only
- **Radiologist must confirm or override** every AI triage decision before clinical action
- Monthly **model audit**: radiologist committee reviews a random 5% sample of AI predictions
- **Override tracking**: all radiologist overrides logged for continuous model retraining
- Initial **shadow mode deployment** for 3 months before clinical integration — AI runs in parallel with existing workflow; predictions not shown to radiologists until validation thresholds are met

---

## Task 7: Responsible AI Considerations

### 1. Bias in Data

Medical imaging datasets historically underrepresent women, elderly patients, and patients from lower-income regions (who may have different disease presentations or inferior scan quality due to older equipment). A model trained predominantly on high-quality scans from large urban hospitals may perform poorly in rural clinics.

**Mitigation**: Stratify training data by age, sex, and scan equipment type. Test model performance disaggregated across demographic subgroups before deployment. Partner with diverse hospital sites for data collection.

### 2. Incorrect Predictions

False negatives (missing a critical case) are more dangerous than false positives in this context. A single missed pneumothorax or pulmonary embolism could be fatal.

**Mitigation**: Set decision threshold conservatively to maximise sensitivity at the cost of some specificity. Mandate human radiologist sign-off for every image. Never allow the AI to generate or dispatch a clinical report autonomously.

### 3. Privacy Concerns

Medical images are among the most sensitive personal data. DICOM files contain embedded metadata (patient name, DOB, hospital number). A data breach would violate HIPAA (US), GDPR (EU), or applicable national health data regulations.

**Mitigation**: Full de-identification of all DICOM metadata before ingestion into the AI pipeline. Data stored in encrypted, access-controlled hospital infrastructure. No patient data transmitted to external cloud services without explicit regulatory approval and patient consent.

### 4. Over-Reliance on AI

Clinical staff may begin to defer to AI classifications without critical thinking, reducing their own diagnostic vigilance over time. This is sometimes called "automation bias."

**Mitigation**: Training for radiologists on AI limitations before deployment. System UI designed to show AI as "advisory" (not diagnostic). Regular simulation exercises where radiologists review cases without AI assistance.

### 5. Impact on Users (Radiologists)

Radiologists may fear that the AI system is replacing their role, leading to resistance and reduced engagement.

**Mitigation**: Position AI explicitly as a triage support tool, not a diagnostic replacement. Involve radiologist representatives in the design and validation process from the start. Demonstrate that the tool reduces administrative burden, not clinical responsibility.

### 6. Need for Human Oversight

Given the life-critical nature of the domain, this system must never operate in a fully autonomous mode.

**Mitigation**: Regulatory approval process (CE marking in EU, FDA 510(k) in the US) required before clinical deployment. Ongoing clinical governance board oversight. Clear escalation pathways if the model confidence score falls below a threshold.

---

## Task 8: Final Solution Summary

---

### One-Page Solution Summary

**Problem**
Radiology departments are overwhelmed by high volumes of medical images (X-rays, CT scans). The current first-in, first-out review workflow means critical cases — such as pneumothorax or pulmonary embolism — may wait hours before a radiologist reviews them. Average resolution time is approximately 30 hours. Error rates average 6.8%, and manual processing consumes over 450 staff hours per month.

**Proposed AI Solution**
Deploy a deep learning image classification system that automatically assigns each incoming scan to one of three urgency tiers (Normal, Significant, Critical) immediately upon receipt. The AI-generated triage score is used to **re-sort the radiologist's worklist** so that critical cases surface first. The radiologist retains full decision-making authority; the AI provides an advisory signal only.

**Required Data**
- 50,000+ labelled chest X-ray and CT scan images (DICOM format)
- Urgency labels derived from historical radiologist reports
- Patient metadata (age, sex, ward) for auxiliary model input
- Ongoing override data from deployed system for continuous retraining

**Model Recommendation**
ResNet-50 with transfer learning (fine-tuned on labelled hospital imaging data). The model is pre-trained on ImageNet and adapted to the three-class urgency classification task. Grad-CAM visualisations provide interpretable heatmaps for radiologist review.

**Expected Business Impact**

| KPI                        | Before AI | After AI (Target) |
|----------------------------|-----------|-------------------|
| Avg resolution time        | ~30 hrs   | ≤ 18 hrs          |
| Manual processing hrs/mo   | ~453 hrs  | ≤ 280 hrs         |
| Error rate                 | 6.8%      | ≤ 4.0%            |
| Customer satisfaction      | 7.1 / 10  | ≥ 8.0 / 10        |
| Critical case alert time   | Not tracked | < 30 min       |

**Risks and Mitigation Plan**

| Risk                          | Mitigation                                                        |
|-------------------------------|-------------------------------------------------------------------|
| Missed critical diagnoses     | Conservative threshold; mandatory human sign-off                 |
| Demographic bias              | Stratified training data; subgroup performance audits            |
| Patient data privacy breach   | DICOM de-identification; encrypted on-premise storage            |
| Automation bias               | Radiologist training; UI labels AI as advisory only              |
| Regulatory non-compliance     | CE/FDA approval process before clinical deployment               |
| Model degradation over time   | Monthly performance audits; override-triggered retraining        |

---

*Report prepared for Part 4 – AI Business Solutions Module. All KPI baselines sourced from `business_kpi_sample.csv`. Domain and model references sourced from `ai_usecase_reference_catalog.csv`.*
