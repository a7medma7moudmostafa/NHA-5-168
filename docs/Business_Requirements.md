# Business Requirements Document

**Project:** AI Document Classification & Routing System  
**Team:** Ahmed Mahmoud Mostafa Abdelkarim, Abdelrahman Mohamed Ezz  
**Repository:** https://github.com/nhahub/NHA-5-168  
**Version:** 1.2 (Draft)

---

## 1. Project Overview

Organizations receive large volumes of incoming documents such as complaints, requests, and invoices. Today, a person reads each one and decides which department should handle it. This project builds an AI system that reads the text of an incoming document and recommends the department that should receive it, without manual sorting.

## 2. Problem Statement

- Manual sorting is slow and depends on the individual employee's judgment.
- Documents that reach the wrong department are forwarded again, which delays the response to the customer.
- As volume grows, more staff are needed only for sorting.

## 3. Business Objectives

Each objective answers one of the problems in Section 2. Because the project has no real organization behind it, business results such as "hours saved" cannot be measured directly. Each objective therefore has an indicator that can be measured inside the project.

### BO-1: Reduce the time needed to assign a document to a department

- **Problem addressed:** Manual sorting is slow.
- **How the system helps:** The recommendation is produced automatically within seconds of receiving the text, instead of waiting for a person to read it.
- **Indicator in this project:** Response time of the deployed service per document. This is a proxy: in a real organization most of the delay is the document waiting for a person to open it, which cannot be measured here.
- **Related requirements:** BR-3, BR-4.

### BO-2: Reduce documents sent to the wrong department

- **Problem addressed:** Misrouted documents are forwarded again and delay the customer.
- **How the system helps:** A model trained on labelled examples applies the same rule to every document, and it offers three candidate departments so that the correct one is rarely missing.
- **Indicator in this project:** Accuracy, macro F1, and top-3 accuracy on a held-out test set.
- **Related requirements:** BR-1, BR-2, BR-5.

### BO-3: Free staff from manual sorting

- **Problem addressed:** More volume requires more people only for sorting.
- **How the system helps:** Staff confirm a recommendation instead of reading and deciding from zero. Only uncertain documents need full attention.
- **Indicator in this project:** Share of test documents whose confidence is above the review threshold (documents that would not need full manual sorting).
- **Related requirements:** BR-2, BR-7.

### BO-4: Make uncertainty visible

- **Problem addressed:** Sorting quality depends on individual judgment, and errors are discovered late.
- **How the system helps:** Every recommendation carries a confidence score. Low-confidence documents are flagged for human review, so the system supports the employee and does not replace the final decision.
- **Indicator in this project:** Accuracy of high-confidence recommendations compared with low-confidence ones.
- **Related requirements:** BR-2, BR-7.

### BO-5: Base the choice of solution on evidence

- **Problem addressed:** A model chosen without comparison cannot be defended.
- **How the system helps:** A simple baseline and an advanced model are trained on the same data, measured with the same metrics, and recorded in an experiment tracker.
- **Indicator in this project:** A comparison table of all experiments, reproducible from the repository.
- **Related requirements:** BR-5, BR-6.

## 4. Scope

### 4.1 What the system receives and returns

| Item | Description |
|---|---|
| Input | The text of one document: its subject line and its body, in English. |
| Output 1 | The document type: Incident, Request, Problem, or Change. |
| Output 2 | The three most likely departments, in order, each with a confidence score. |
| Output 3 | A flag when the confidence is low and a person should review the document. |

### 4.2 How the document kinds in the project description map to the data

The project description mentions complaints, requests, and invoices. The chosen dataset contains support tickets written as emails. The mapping is:

| In the project description | In the dataset |
|---|---|
| Complaints | Tickets of type Incident or Problem |
| Requests | Tickets of type Request or Change |
| Invoices | Tickets about billing, handled by the Billing and Payments department |

**Limitation:** the dataset does not contain invoice documents themselves (tables of items and amounts). It contains messages about invoices and payments. This must be stated in the report.

### 4.3 Departments

The departments are the values of the `queue` field in the dataset, for example Technical Support, IT Support, Product Support, Customer Service, Billing and Payments, Returns and Exchanges, Sales and Pre-Sales, Human Resources, Service Outages and Maintenance, and General Inquiry. The final list is fixed after data exploration, and overlapping departments may be merged (see Section 9, R-1).

Merging departments raises accuracy but makes the recommendation less useful, because a merged "technical" department no longer tells the employee which team should act. Results are therefore reported for both the original and the merged list, and the merged version is described as an easier task, not a better system.

### 4.4 In scope

| Area | Included work |
|---|---|
| Data | Download, English filter, cleaning, duplicate removal, train/validation/test split. |
| Baseline model | TF-IDF features with classical machine-learning classifiers. |
| Advanced model | A pretrained Transformer from Hugging Face, fine-tuned on the data. |
| Evaluation | Same metrics for all models, error analysis, final model selection. |
| Experiment tracking | All runs recorded in MLflow. |
| Inference | One function that takes text and returns the outputs in 4.1. |
| Deployment | The model served on AWS and reachable through a web address. |
| Documentation | README, architecture diagram, final report. |

### 4.5 Out of scope

| Excluded | Reason | Status |
|---|---|---|
| Arabic and other languages | The dataset is used in English only; no suitable Arabic data in this domain was found. | Future work |
| Scanned documents and images (OCR) | No data for it, and it is a separate problem. | Future work |
| Sending the document into a department's mailbox or ticketing system | Requires integration with a real organization's systems. | Not planned |
| Writing a reply to the customer | A text-generation task, different from classification. | Not planned |
| User accounts and a full web application | The deliverable is a service (API); a simple demo page is optional. | Optional |
| Automatic retraining and data-drift monitoring | Beyond the "simple MLOps" level agreed for this project. | Future work |
| Predicting urgency (the `priority` field in the dataset) | Valuable for a business, but not part of the approved description. | Future work |
| Handling personal data in real documents | The dataset is synthetic, so it holds no real personal data. Real use would require removing or protecting names, account numbers, and similar details. | Future work |

## 5. Stakeholders and Users

The project has no real client. The roles below describe an **assumed scenario**: a medium-sized company with one shared inbox for incoming documents. They should be replaced if the course provides a specific case.

### 5.1 Stakeholders

| Stakeholder | Who they are | What they need from the system | How they use it |
|---|---|---|---|
| Intake employee | The person who sorts the shared inbox today. | A recommendation to confirm instead of sorting from zero; the kind of document (complaint-like or request-like) at a glance; a clear flag on uncertain documents. | Sees type, top-3 departments, and confidence; confirms or corrects. |
| Department staff | Employees in Billing, Technical Support, HR, and others. | To receive only documents that belong to them. | Indirect: they receive better-sorted documents. |
| Team lead / operations manager | Responsible for response times. | Fewer misrouted documents and visibility of how the system performs. | Reads the evaluation results and service monitoring. |
| Customer | The sender of the document. | A faster answer from the right department. | Indirect: never sees the system. |
| IT / integration developer | Connects the service to the company's tools. | A stable web address, a clear input and output format. | Calls the API. |
| Project team | Ahmed and Abdelrahman. | Clear scope and reproducible work. | Build, evaluate, deploy. |
| Course supervisor | Evaluates the project. | Evidence that the course requirements are met, including AWS. | Reviews the repository, report, and demo. |

### 5.2 Typical use

1. A customer sends an email to the company's shared inbox.
2. The text is sent to the system.
3. The system returns the type, three candidate departments, and confidence.
4. If confidence is high, the intake employee confirms the first department.
5. If confidence is low, the employee reads the document and chooses, helped by the three candidates.

### 5.3 Team responsibilities (proposed)

| Area | Abdelrahman | Ahmed |
|---|---|---|
| Data | Cleaning and splitting | Exploration and charts |
| Models | Transformer model | Baseline models |
| Tracking and evaluation | MLflow | Evaluation and comparison |
| AWS | Endpoint and Lambda | S3, API Gateway, CloudWatch |

## 6. Business Requirements

### 6.1 Alignment between the project description and the requirements

Every statement in the approved project description is covered by at least one requirement.

| Statement in the project description | Requirement |
|---|---|
| "reads and analyses the text of incoming documents (complaints, requests, invoices)" | BR-1 |
| "classifies them automatically" | BR-1 |
| "produces a recommendation to route them to the responsible department" | BR-2 |
| "without human intervention for manual sorting" | BR-2, BR-3, BR-7 |
| "dataset preparation and text preprocessing" | BR-5 (same prepared data for all models) |
| "a baseline model based on TF-IDF and classical machine learning" | BR-5 |
| "an advanced model using LSTM or pretrained Transformers from Hugging Face" | BR-5 |
| "models evaluated and experiments tracked with MLflow" | BR-5, BR-6 |
| "final model packaged for inference" | BR-3, BR-4 |
| "technical architecture of the project prepared" | BR-4, constraints C-3 and C-4 |
| Course condition: AWS must be part of the project | BR-4, constraint C-4 |

Note: "without human intervention" is read as removing manual sorting. A person still reviews low-confidence cases (BR-7), because the expected accuracy does not justify fully automatic routing.

### 6.2 Requirements

| ID | Requirement | Objective | Priority | Accepted when |
|---|---|---|---|---|
| BR-1 | The system shall accept the text of a document and return its type. This is the "classification" named in the project description: it separates complaint-like documents (Incident, Problem) from request-like ones (Request, Change). | BO-2 | Must | A test document returns one of the four types. |
| BR-2 | The system shall return the three most likely departments, each with a confidence score. | BO-2, BO-3, BO-4 | Must | A test document returns three ordered departments with scores. |
| BR-3 | The system shall return its recommendation within a few seconds. | BO-1 | Must | Response time is measured on the deployed service and reported; the exact limit is set after the first deployment test. The first request after an idle period is slower on a serverless service and is reported separately. |
| BR-4 | The system shall be available through a web address on AWS so that any application can use it. | BO-1 | Must | A request sent to the public address returns a result during the demo. |
| BR-5 | The team shall compare a classical baseline with a pretrained language model on the same data and justify the final choice with measured results. | BO-5 | Must | The report contains a comparison table and a reasoned choice. |
| BR-6 | All experiments shall be recorded so that results can be reproduced and compared. | BO-5 | Must | Every reported number matches a recorded MLflow run. |
| BR-7 | Low-confidence results shall be flagged for human review. | BO-3, BO-4 | Should | Results below the threshold carry a review flag; the threshold is chosen as described in Section 7.4. |

The cloud budget and the use of AWS are fixed limits, not requirements, and are listed as constraints C-3 and C-4 in Section 8.2.

## 7. Success Criteria and Baseline

### 7.1 Baseline

Results mean nothing without a reference point. Three references are used:

| Reference | What it is | Purpose |
|---|---|---|
| Majority-class baseline | Always predicting the most common department. | The minimum any model must beat. |
| Most-frequent top-3 baseline | Always suggesting the three most common departments. | The minimum for top-3 accuracy. A few departments hold most of the documents, so this simple rule already scores high; top-3 results are only meaningful compared with it. |
| Classical baseline | TF-IDF with Logistic Regression or Linear SVM. | The reference the advanced model must justify itself against. |

### 7.2 External reference points

Public projects on the same dataset report, for department prediction with classical models, accuracy of about 65-67%, and top-3 accuracy of about 90%. These figures come from other people's repositories and were not reproduced by this team. They are used only to set realistic expectations.

### 7.3 Success criteria

Targets are proposals. They are confirmed with the supervisor after the baseline results exist.

| Measure | Proposed target | Objective |
|---|---|---|
| Department macro F1 | Clearly above the majority-class baseline; reported for every model | BO-2 |
| Department top-3 accuracy | About 85% or higher, and clearly above the most-frequent top-3 baseline | BO-2 |
| Share of documents above the confidence threshold | Reported together with the accuracy of those documents; target set after the baseline | BO-3 |
| Document type accuracy | Above the majority-class baseline; value set after the baseline | BO-2 |
| Advanced model vs classical baseline | Difference measured and explained, whether or not the advanced model wins | BO-5 |
| High-confidence accuracy | Higher than overall accuracy | BO-4 |
| Response time | A few seconds per request, measured on the deployed service | BO-1 |
| Reproducibility | A team member can rerun training from the repository and get the recorded results | BO-5 |
| Cloud cost | Inside the free credits | C-3 |

The project is considered successful if the service works end to end in the demo, the comparison is complete and honest, and the limits of the results are explained. A high accuracy number alone is not the goal.

### 7.4 Choosing the confidence threshold

The threshold in BR-7 is a business decision, not a technical one. It balances two costs:

| Threshold | Effect |
|---|---|
| Higher | Fewer wrong recommendations reach the departments, but more documents go back to manual review (weakens BO-3). |
| Lower | Less manual review, but more misrouted documents (weakens BO-2). |

The right balance depends on which is more expensive for the organization: a document sent to the wrong department, or an employee reviewing one more document. This project has no real organization and so no real cost figures. The team will therefore show, on the validation data, how accuracy and the share of automatically handled documents change with the threshold, and propose one value with its reasoning.

## 8. Assumptions and Constraints

### 8.1 Assumptions (believed true, not guaranteed)

| ID | Assumption | If it is wrong |
|---|---|---|
| A-1 | The dataset stays publicly available during the project. | A local copy is kept by the team (not published). |
| A-2 | Labels in the dataset are good enough to learn from. | Overlapping departments are merged. |
| A-3 | Synthetic tickets are similar enough to real ones for a course project. | Stated as a limitation; no claim about real-world performance. |
| A-4 | The cloud account allows the services needed (S3, SageMaker, Lambda, API Gateway). | Training is done outside AWS and only deployment stays on AWS. |
| A-5 | A free cloud notebook with a GPU is available for training. | A smaller model or fewer training rounds are used. |

### 8.2 Constraints (fixed limits)

| ID | Constraint |
|---|---|
| C-1 | Dataset license is CC BY-NC 4.0: educational, non-commercial use, with attribution. The data file is not stored in the repository. |
| C-2 | Development hardware: a laptop with 16 GB RAM and no dedicated GPU. |
| C-3 | Cloud budget: the free credits of a new AWS account. |
| C-4 | AWS must be part of the solution (course condition). |
| C-5 | Team of two, about five weeks. |
| C-6 | English text only. |

## 9. Risks

| ID | Risk | Likelihood | Impact | Mitigation | Owner |
|---|---|---|---|---|---|
| R-1 | Overlapping department labels cap the accuracy. | High | Medium | Report macro F1 and top-3; test a merged-department version; human review for low confidence. | Both |
| R-2 | The advanced model is not better than the baseline. | Medium | Low | Treated as a finding and explained, not hidden. | Abdelrahman |
| R-3 | Synthetic data does not reflect real customer language. | High | Medium | Stated as a limitation; no real-world claims. | Both |
| R-4 | Cloud credits run out before the demo. | Low | High | Budget alert from day one; serverless deployment; delete the endpoint after the demo. | Abdelrahman |
| R-5 | No GPU quota on the new cloud account. | Medium | Medium | Check in week 1; request quota early or train on a free notebook service. | Abdelrahman |
| R-6 | The same ticket appears in training and test data. | Medium | High | Remove duplicates before splitting. | Abdelrahman |
| R-7 | Both members edit the same files. | Medium | Low | Separate branches and pull requests. | Both |
| R-8 | Deployment problems appear late. | Medium | High | One-hour AWS check in week 1; deployment starts in week 4, not in the last days. | Both |

## 10. Deliverables

| # | Deliverable | Content | Planned week |
|---|---|---|---|
| D-1 | Repository setup | License, README, folder structure, configuration file. | 1 |
| D-2 | Prepared data | Cleaning and splitting code, and instructions to recreate the data (data not published). | 1 |
| D-3 | Data exploration | Notebook with label distributions and text lengths. | 1 |
| D-4 | Baseline models | Training code and recorded results. | 2 |
| D-5 | Advanced model | Training code and recorded results. | 3 |
| D-6 | Evaluation report | Comparison table, error analysis, final model choice. | 3 |
| D-7 | Inference function | Text in; type, top-3 departments, confidence out. | 3 |
| D-8 | Deployed service | Model on AWS behind a public web address, with monitoring. | 4 |
| D-9 | Architecture diagram | Components and data flow. | 5 |
| D-10 | Final report and live demo | Results, limitations, future work. | 5 |

## 11. Glossary

| Term | Meaning |
|---|---|
| Classification | Assigning a document to one category from a fixed list. |
| Routing | Deciding which department should handle a document. |
| Baseline | A simple reference result that a better model must beat. |
| Accuracy | The share of documents for which the first recommendation is correct. |
| Top-3 accuracy | The share of documents for which the correct department is among the three suggestions. |
| Macro F1 | A quality score that gives every department equal weight, so small departments count as much as large ones. |
| Confidence score | A number from the model showing how sure it is about a recommendation. |
| Threshold | The confidence value below which a document is sent for human review. |
| Held-out test set | Documents kept aside and never used in training, used only for the final measurement. |
| Synthetic data | Data generated artificially, not collected from real customers. |
| Serverless | A cloud service that runs only when a request arrives and is billed per use. |

## 12. Next Documents

After this document is approved, the next steps in the workflow are:

1. Functional and non-functional requirements (technical detail of what the system must do and how well).
2. System design and architecture.
3. Implementation, testing, deployment, and maintenance plan.

## 13. Approval and Change Log

| Role | Name | Date | Signature |
|---|---|---|---|
| Team member | Ahmed Mahmoud Mostafa Abdelkarim | | |
| Team member | Abdelrahman Mohamed Ezz | | |
| Supervisor | | | |

| Version | Change |
|---|---|
| 1.0 | First draft. |
| 1.1 | Objectives, scope, stakeholders, requirements, and later sections expanded; alignment table added. |
| 1.2 | Third baseline for top-3 accuracy; success criterion for BO-3; confidence-threshold section; cloud budget moved from requirements to constraints; urgency and personal data added to out of scope; glossary and approval added. |
