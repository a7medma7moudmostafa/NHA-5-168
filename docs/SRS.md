# Software Requirements Specification

**Project:** AI Document Classification & Routing System  
**Team:** Ahmed Mahmoud Mostafa Abdelkarim, Abdelrahman Mohamed Ezz  
**Repository:** https://github.com/nhahub/NHA-5-168  
**Version:** 1.1 (Draft)  
**Based on:** Business Requirements Document v1.2

---

## 1. Introduction

### 1.1 Purpose

The Business Requirements Document (BRD) explains why the system is built and what counts as success. This document specifies what the system must do (functional requirements) and how well it must do it (non-functional requirements), in a form that can be built and tested.

**Audience:** the project team and the course supervisor.

### 1.2 Conventions

- **FR-n:** functional requirement. **NFR-n:** non-functional requirement. **AT-n:** acceptance test.
- **BR-n, BO-n, C-n:** items from the BRD.
- "Shall" means mandatory. Each requirement traces back to at least one BRD item (Section 6).
- All requirements in this document are mandatory. BR-7 (human review of low-confidence results) is treated as Must; the BRD will be updated to match.

### 1.3 Decisions taken for this version

| ID | Decision | Reason |
|---|---|---|
| DEC-1 | Document type and department are predicted by **two separate models**, not one model with two outputs. The baselines follow the same rule: two classifiers on shared TF-IDF features. | Simpler to train, evaluate, and compare. Cost: the service loads two models. |
| DEC-2 | The advanced model is **DistilBERT** only. LSTM is not built. | The project description allows "LSTM or Transformers"; time is limited to about five weeks. LSTM is listed as future work. |
| DEC-3 | The cloud API is protected by an **API key**. | Without it, anyone with the address could use the service and spend the cloud credits. An API key is basic protection, not full user authentication; this is a stated limitation. |
| DEC-4 | The warm response-time target is **3 seconds** on the cloud. | Starting value; reviewed after the first deployment measurement. |
| DEC-5 | The starting confidence threshold is **0.5**, stored in `config.yaml`. The final value comes from the threshold analysis (FR-11). | The threshold is a business decision (BRD 7.4) and needs data before it is fixed. With about ten departments, the top confidence is often below 0.5 even when correct, and model confidence may not be calibrated; the analysis will check both. |
| DEC-6 | Experiments are tracked in a **local MLflow** during development. At the end, all runs and registered models are exported to **MLflow on SageMaker** with the mlflow-export-import tool. | Keeps development free of cloud cost while ending with one complete record on AWS. |

## 2. System Context

### 2.1 Context diagram

```
                     +--------------------------------------------+
  Hugging Face  ---> |                                            | <--- Client application
  (dataset)          |   AI Document Classification & Routing     | ---> type, top-3 departments,
                     |   System                                   |      review flag
  Project team  ---> |                                            |
  (train, deploy)    |   S3 | SageMaker | Lambda | API Gateway    | ---> CloudWatch (logs)
                     +--------------------------------------------+
```

### 2.2 Components

| Component | Role | Where it runs |
|---|---|---|
| Configuration (`config.yaml`) | Holds all paths, model settings, random seed, and threshold. | Repository |
| Data source | Public support-ticket dataset (external system). | Hugging Face |
| Storage | Holds the data splits and trained models used by the cloud. | Amazon S3 |
| Data pipeline | Loads, filters, cleans, de-duplicates, and splits the data. | Laptop |
| Baseline training | TF-IDF with Logistic Regression and Linear SVM. | Laptop |
| Transformer training | DistilBERT fine-tuning. | Free cloud notebook (Colab). A final run on SageMaker is optional, if GPU quota and credits allow; otherwise the Colab-trained model is uploaded to S3. |
| Experiment tracker | Records every run (DEC-6). | MLflow, local, then exported to SageMaker |
| Model registry | Holds the selected, versioned models; the link between training and deployment. | MLflow Model Registry |
| Inference function | `predict()`: text in, type and departments out. | Laptop and inside the SageMaker endpoint container |
| Local API | FastAPI service for testing at no cloud cost. | Laptop |
| Cloud API | API Gateway receives the request and checks the API key; Lambda validates the input and forwards it to the endpoint. Lambda does not run the model. | AWS |
| Model endpoint | Runs `predict()` with the deployed models. | SageMaker Serverless |
| Monitoring | Request logs and metrics. | CloudWatch (cloud), log file (local) |

### 2.3 Actors

| Actor | Interaction |
|---|---|
| Client application | Sends a document and receives the recommendation through the API. In the demo, this role is played by the demo client (OP-5). |
| Intake employee | Reads the recommendation and confirms or corrects it. Main beneficiary of the review flag, but works outside this system. |
| Supervisor / evaluator | Tests the service during the demo, using an API key given by the team. |
| ML developer (project team) | Prepares data, trains, evaluates, and registers models. |
| Deployment engineer (project team) | Deploys and monitors the service on AWS. |

## 3. Functional Requirements

### 3.1 Data

| ID | Requirement | Traces to |
|---|---|---|
| FR-1 | The system shall load the dataset from a path defined in `config.yaml`, which may be a local path or an S3 path. | BR-6, C-4 |
| FR-2 | The system shall keep English records only, join `subject` and `body` into one `text` field, and drop the `answer` field. | BR-1, BR-2 |
| FR-3 | The system shall clean the text, remove duplicate texts before splitting, and split the data 80% / 10% / 10% (train / validation / test), stratified by department, with a fixed random seed. | BR-5, BR-6 |
| FR-4 | The system shall save the three splits and record their row counts and a checksum, so that every model is trained and tested on identical data. | BR-5, BR-6 |

### 3.2 Training

| ID | Requirement | Traces to |
|---|---|---|
| FR-5 | The system shall train TF-IDF + Logistic Regression and TF-IDF + Linear SVM for each target (document type and department). | BR-5 |
| FR-6 | The system shall fine-tune DistilBERT for each target. | BR-5 |
| FR-7 | All settings (model names, hyperparameters, paths, random seed, threshold) shall be read from `config.yaml` and not written in the code. | BR-6 |

### 3.3 Evaluation and Tracking

| ID | Requirement | Traces to |
|---|---|---|
| FR-8 | The system shall compute, on the test set, accuracy, macro F1, top-3 accuracy, per-class precision, recall and F1, and a confusion matrix, and shall compare them with the majority-class baseline and the most-frequent top-3 baseline. | BR-5 |
| FR-9 | Every training run shall be logged to MLflow with its parameters, metrics, data checksum, and trained model. | BR-6 |
| FR-10 | The selected model for each target shall be registered in the MLflow Model Registry with a version number. | BR-5, BR-6 |
| FR-11 | The system shall report, on the validation set, the accuracy of auto-routed documents and the share of documents auto-routed, for a range of thresholds. | BR-7 |

### 3.4 Inference

| ID | Requirement | Traces to |
|---|---|---|
| FR-12 | A function `predict(subject, body)` shall return the document type with its confidence, the three most likely departments with their confidences in descending order, a review flag, and the model version. | BR-1, BR-2, BR-7 |
| FR-13 | The review flag shall be true when the confidence of the first department is below the threshold set in `config.yaml`. | BR-7 |
| FR-14 | Empty or non-text input shall return a clear error message. Text longer than the model's input limit shall be truncated, not rejected. | BR-1 |

### 3.5 Service

| ID | Requirement | Traces to |
|---|---|---|
| FR-15 | A local FastAPI service shall expose `POST /predict` and `GET /health`, and shall call the same `predict()` function. | BR-4 |
| FR-16 | The cloud service shall expose `POST /predict` through API Gateway, Lambda, and a SageMaker Serverless endpoint, using the same request and response format as the local service. | BR-4, C-4 |
| FR-17 | Every request shall be logged with its time, latency, predicted department, confidence, and review flag: to CloudWatch in the cloud and to a log file locally. The document text shall not be logged. | BR-3, BR-7 |
| FR-18 | The cloud service shall reject requests without a valid API key. | BR-4 |

## 4. Interface Specification

### 4.1 Request

`POST /predict`, content type `application/json`.

```json
{
  "subject": "Charged twice for my subscription",
  "body": "Hello, I noticed two payments on my card this month..."
}
```

| Field | Type | Required | Rule |
|---|---|---|---|
| subject | string | No | May be empty. |
| body | string | Yes | Must contain at least one non-space character. |

### 4.2 Response

```json
{
  "type": { "label": "Problem", "confidence": 0.81 },
  "departments": [
    { "name": "Billing and Payments", "confidence": 0.72 },
    { "name": "Customer Service", "confidence": 0.15 },
    { "name": "General Inquiry", "confidence": 0.06 }
  ],
  "needs_review": false,
  "model_version": "3"
}
```

Values above are illustrative.

### 4.3 Status codes

| Code | Meaning |
|---|---|
| 200 | Prediction returned. |
| 400 | Invalid input, for example an empty body. |
| 403 | Missing or invalid API key (cloud only). |
| 500 | Internal error while running the model. |

## 5. Non-functional Requirements

| ID | Category | Requirement | Measure |
|---|---|---|---|
| NFR-1 | Performance | Warm requests shall be answered quickly. Cold-start time shall be measured and reported separately. | p95 of warm requests ≤ 3 s on the cloud and ≤ 1 s locally. |
| NFR-2 | Model quality | Models shall beat the reference baselines. | Macro F1 above the majority-class baseline; top-3 accuracy above the most-frequent top-3 baseline. |
| NFR-3 | Reproducibility | Rerunning training from the repository shall reproduce the recorded results. | Identical metrics for the baselines; difference ≤ 1 percentage point for DistilBERT. |
| NFR-4 | Cost | The solution shall stay inside the free cloud credits. | Budget alert at USD 10; endpoint deleted after the demo. |
| NFR-5 | Security | No secrets in the repository; least-privilege IAM roles; HTTPS only; request throttling on API Gateway. | Repository check and AWS settings review. |
| NFR-6 | Privacy | Document text shall not be stored in logs. | Inspection of CloudWatch logs. |
| NFR-7 | Portability | The same `predict()` shall run locally, in FastAPI, and on SageMaker. Switching between local and S3 storage shall need only a configuration change. | The same request gives the same response in both environments. |
| NFR-8 | Resources | Baselines shall train on the development laptop (16 GB RAM, CPU only). DistilBERT shall train within one free Colab session. | Training time recorded in MLflow. |
| NFR-9 | Maintainability | Code shall be organized in `src/` modules, follow PEP 8, and be runnable from the README instructions by either team member. | Code review on every pull request. |
| NFR-10 | Testability | Unit tests shall cover preprocessing, `predict()`, and input validation. | All tests pass before every merge to `main`. |
| NFR-11 | Availability | The service shall be available during the demonstration. No uptime guarantee is given beyond it. | Successful live demo. |
| NFR-12 | Compliance | The dataset shall be credited (CC BY-NC 4.0), not redistributed, and used for education only. | Dataset section in the README. |

## 6. Traceability Matrix

| BRD item | Covered by |
|---|---|
| BR-1 Return document type | FR-2, FR-12, FR-14 |
| BR-2 Top-3 departments with confidence | FR-2, FR-12 |
| BR-3 Fast response | FR-17, NFR-1 |
| BR-4 Available through a web address | FR-15, FR-16, FR-18 |
| BR-5 Compare baseline and advanced model | FR-3 – FR-6, FR-8, FR-10, NFR-2 |
| BR-6 Experiments recorded and reproducible | FR-1, FR-3, FR-4, FR-7, FR-9, FR-10, NFR-3 |
| BR-7 Flag low confidence for review | FR-11, FR-12, FR-13, FR-17 |
| C-1 Dataset license | NFR-12 |
| C-2 Laptop without GPU | NFR-8 |
| C-3 Cloud budget | NFR-4 |
| C-4 AWS part of the solution | FR-1, FR-16 |

## 7. Acceptance Tests

| ID | Covers | Steps | Pass when |
|---|---|---|---|
| AT-1 | FR-12 | Send a clear billing email. | Billing and Payments is among the three departments. |
| AT-2 | FR-13 | Send a vague text such as "hello, I need help". | `needs_review` is true. |
| AT-3 | FR-14 | Send an empty body. | Status 400 with an error message. |
| AT-4 | FR-14 | Send a body longer than the model limit. | Status 200; the text is truncated. |
| AT-5 | FR-16, NFR-7 | Send the same request locally and to the cloud. | Both responses are identical. |
| AT-6 | FR-17, NFR-6 | Send requests and read the logs. | Logs show department, confidence and latency, and contain no document text. |
| AT-7 | FR-18 | Call the cloud API without a key. | Status 403. |
| AT-8 | FR-3 | Compare train and test texts. | No text appears in both. |
| AT-9 | FR-8, NFR-2 | Run the evaluation. | Report shows all metrics and both reference baselines. |
| AT-10 | NFR-3 | Rerun baseline training from a fresh clone. | Metrics match the recorded MLflow run. |

## 8. Open Points

| ID | Point | When it is closed |
|---|---|---|
| OP-1 | Final department list, and whether overlapping departments are merged. | After data exploration (week 1). |
| OP-2 | Final confidence threshold. | After the threshold analysis (FR-11, week 3). |
| OP-3 | Final response-time target. | After the first deployment test (week 4). |
| OP-4 | SageMaker Serverless memory and size limits for two DistilBERT models. | Checked in the design document. |
| OP-5 | Demo client: a command-line call (curl or Postman) or a simple web page. | Before the deployment phase (week 4). |

## 9. Future Work

- LSTM model as a third comparison point (DEC-2).
- Single model with two outputs for type and department (DEC-1).
- Urgency prediction from the `priority` field.
- Arabic support.
- Automatic retraining and data-drift monitoring.

## 10. Approval and Change Log

| Role | Name | Date | Signature |
|---|---|---|---|
| Team member | Ahmed Mahmoud Mostafa Abdelkarim | | |
| Team member | Abdelrahman Mohamed Ezz | | |
| Supervisor | | | |

| Version | Change |
|---|---|
| 1.0 | First draft, based on BRD v1.2 and decisions DEC-1 to DEC-5. |
| 1.1 | Review of Sections 1 and 2: audience added; all requirements mandatory (BR-7 as Must); DEC-1, DEC-3, DEC-5 clarified; DEC-6 (MLflow) added; context diagram added; components completed (configuration, data source, S3, model registry, endpoint); SageMaker training made optional; `predict()` location corrected; actors expanded; local logging added to FR-17; OP-5 added. |
