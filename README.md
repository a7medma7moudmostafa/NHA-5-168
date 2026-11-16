# 📄 AI Document Classification & Routing System

![Python](https://img.shields.io/badge/Python-3.9+-blue.svg)
![AWS](https://img.shields.io/badge/AWS-Cloud-orange.svg)
![MLflow](https://img.shields.io/badge/MLflow-Tracking-blue.svg)
![HuggingFace](https://img.shields.io/badge/HuggingFace-Transformers-yellow.svg)

## 📌 Project Overview
Organizations receive large volumes of incoming documents such as complaints, requests, and invoices. Manual sorting is slow, prone to errors, and scales poorly as document volume grows. 

This project builds an end-to-end **AI Document Classification & Routing System** that reads the text of an incoming document and automatically recommends the appropriate department to handle it. Low-confidence predictions are flagged for human review, ensuring a balanced approach between automation and human accuracy.

## 🏗️ System Architecture

```mermaid
flowchart TB

    subgraph ML["1. Machine Learning Pipeline"]
        A["Customer Support Dataset"]
        B["Data Preprocessing<br/>Cleaning & Tokenization"]
        C["Feature Extraction<br/>TF-IDF"]
        
        D["Baseline Models<br/>Logistic Regression<br/>Linear SVM"]
        E["Transformer Model<br/>DistilBERT Fine-Tuning"]
        
        F["Model Evaluation<br/>Accuracy / Precision / Recall / F1"]
        G["MLflow<br/>Experiment Tracking"]
        H["Trained Model Artifacts"]

        A --> B
        B --> C
        C --> D
        B --> E
        D --> F
        E --> F
        F --> G
        F --> H
    end

    subgraph AWS["2. AWS Cloud Deployment"]
        I["Amazon S3<br/>Model Artifacts"]
        J["Amazon SageMaker<br/>Serverless Endpoint"]
        
        H --> I
        I --> J
    end

    subgraph INFERENCE["3. Real-Time Inference & Routing"]
        K["User / Client Application"]
        L["Amazon API Gateway"]
        M["AWS Lambda<br/>Request Handler"]
        N["Document Classification"]
        O["Department Routing<br/>Top-3 Recommendations"]
        P{"Confidence Score<br/>Above Threshold?"}
        Q["Automatic Routing"]
        R["Human-in-the-Loop<br/>Manual Review"]
        S["Prediction Response<br/>Class + Department + Confidence"]

        K --> L
        L --> M
        M --> J
        J --> N
        N --> O
        O --> P
        P -->|Yes| Q
        P -->|No| R
        Q --> S
        R --> S
    end

    T["Amazon CloudWatch<br/>Logs & Monitoring"]

    M -.-> T
    J -.-> T

    style ML fill:#eaf2ff,stroke:#3973c6,stroke-width:2px
    style AWS fill:#fff2df,stroke:#d98b25,stroke-width:2px
    style INFERENCE fill:#e8f7ed,stroke:#29965b,stroke-width:2px

```
## 🎯 Key Features
* **Document Type Classification:** Automatically classifies documents into predefined types (e.g., Incident, Request, Problem, Change).
* **Smart Routing (Top-3 Departments):** Recommends the 3 most likely departments to handle the document, complete with confidence scores.
* **Human-in-the-Loop:** Flags uncertain or low-confidence documents for manual review.
* **Cloud-Native Inference:** Fully deployed on AWS using serverless architecture for scalable, cost-effective inference.

## 🛠️ Technology Stack
* **Data Processing:** Pandas, NLTK, spaCy
* **Baseline Modeling:** Scikit-learn (TF-IDF, Logistic Regression, Linear SVM)
* **Advanced Modeling:** Hugging Face Transformers (DistilBERT fine-tuning)
* **Experiment Tracking:** MLflow
* **Backend & API:** Python (FastAPI / Inference Script)
* **Cloud Deployment (AWS):** S3, SageMaker Serverless Endpoints, AWS Lambda, API Gateway, CloudWatch

## 📂 Repository Structure
```text
NHA-5-168/
├── README.md              # Project overview, setup instructions, and team info
├── requirements.txt       # Python dependencies
├── .gitignore             # Ignored files (data/, mlruns/, models/, etc.)
├── config.yaml            # Centralized configurations and paths
│
├── data/                  # Local data folder (Not uploaded to GitHub)
│   ├── raw/               # Raw downloaded datasets
│   └── processed/         # Cleaned and split datasets (train/val/test)
│
├── notebooks/             # Jupyter notebooks for experimentation
│   ├── 01_eda.ipynb       # Exploratory Data Analysis
│   ├── 02_baseline.ipynb  # TF-IDF & Classical ML experiments
│   └── 03_transformer.ipynb # DistilBERT experiments
│
├── src/                   # Production-ready source code
│   ├── preprocess.py      # Text cleaning, filtering, and data splitting
│   ├── train_baseline.py  # Baseline model training script
│   ├── train_transformer.py # Transformer fine-tuning script
│   ├── evaluate.py        # Evaluation metrics (Macro F1, Top-3, Confusion Matrix)
│   └── predict.py         # Final inference function
│
├── aws/                   # Cloud deployment scripts
│   ├── lambda_handler.py  # AWS Lambda function code
│   └── deploy_endpoint.py # SageMaker deployment script
│
└── docs/
    └── architecture.md    # System architecture diagrams and workflows

```

## 📸 System in Action

*(screenshots once the UI/Demo is ready)*

### 1. Classification Results & Top-3 Departments

*Shows the system predicting the document type and recommending the top 3 departments with confidence scores.*


### 2. Human Review Flag (Low Confidence)

*Demonstrates how the system flags a document for manual review when the confidence score falls below the accepted threshold.*


## 📊 Model Performance Comparison

All models are trained and evaluated on the exact same train/test splits. The table below highlights the actual final metrics achieved:

| Model | Accuracy | Precision | Recall | F1-Score |
| --- | --- | --- | --- | --- |
| **Logistic Regression** | TBD | TBD | TBD | TBD |
| **Linear SVM** | TBD | TBD | TBD | TBD |
| **DistilBERT (Advanced)** | TBD | TBD | TBD | TBD |

*(Note: Table values will be updated with actual numbers upon the completion of the training and evaluation phases.)*

## 🛠️ Technology Stack

* **Data Processing:** Pandas, NLTK, spaCy
* **Baseline Modeling:** Scikit-learn (TF-IDF, Logistic Regression, Linear SVM)
* **Advanced Modeling:** Hugging Face Transformers (DistilBERT fine-tuning)
* **Experiment Tracking:** MLflow
* **Backend & Local API:** Python, FastAPI (For local testing without AWS costs)
* **Cloud Deployment (AWS):** S3, SageMaker Serverless Endpoints, AWS Lambda, API Gateway, CloudWatch

## 🚀 How to Run the Project

You can run this project locally without incurring AWS costs, or deploy it fully to the cloud.

### 1. Local Development (Without AWS)

This mode uses a locally trained model and serves it via a local **FastAPI** application for testing.

**Step 1: Clone the repository & install dependencies**

```bash
git clone [https://github.com/nhahub/NHA-5-168.git](https://github.com/nhahub/NHA-5-168.git)
cd NHA-5-168
pip install -r requirements.txt

```

**Step 2: Data Preprocessing & Training**
*(Note: Requires placing the dataset in `data/raw/` first)*

```bash
python src/preprocess.py
python src/train_baseline.py

```

**Step 3: Run the Local API Server**

```bash
uvicorn src.predict:app --reload

```

You can now test predictions locally by sending POST requests to `http://localhost:8000/predict`.

### 2. Cloud Production (AWS Deployment)

For production, the local FastAPI server is replaced by native AWS services.

1. The trained model artifacts are uploaded to an **Amazon S3** bucket.
2. The deployment script creates a **SageMaker Serverless Endpoint**.
3. **AWS Lambda** and **API Gateway** are provisioned to expose the model securely over the internet.

## 👥 Team Members

This project is developed for the **Digital Egypt Pioneers Initiative (DEPI)**.

* **Ahmed Mahmoud Mostafa Abdelkarim** 
* **Abdelrahman Mohamed Ezz** 
