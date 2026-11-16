# 📄 AI Document Classification & Routing System

![Python](https://img.shields.io/badge/Python-3.9+-blue.svg)
![AWS](https://img.shields.io/badge/AWS-Cloud-orange.svg)
![MLflow](https://img.shields.io/badge/MLflow-Tracking-blue.svg)
![HuggingFace](https://img.shields.io/badge/HuggingFace-Transformers-yellow.svg)

## 📌 Project Overview
Organizations receive large volumes of incoming documents such as complaints, requests, and invoices. Manual sorting is slow, prone to errors, and scales poorly as document volume grows. 

This project builds an end-to-end **AI Document Classification & Routing System** that reads the text of an incoming document and automatically recommends the appropriate department to handle it. Low-confidence predictions are flagged for human review, ensuring a balanced approach between automation and human accuracy.

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

## 📊 Dataset

We utilize the `customer-support-tickets` dataset (by Tobias Bück), available on both Kaggle and Hugging Face.

* **Note:** Due to the `CC BY-NC 4.0` license restrictions and file size, the dataset is **not** included in this repository.
* To reproduce the results, please download the dataset directly from Hugging Face or Kaggle and place it in the `data/raw/` directory.

## 🚀 Getting Started (Local Development)

1. **Clone the repository:**
```bash
git clone [https://github.com/nhahub/NHA-5-168.git](https://github.com/nhahub/NHA-5-168.git)
cd NHA-5-168

```


2. **Install dependencies:**
```bash
pip install -r requirements.txt

```


3. **Run Preprocessing:**
```bash
python src/preprocess.py

```


4. **Train Baseline Model:**
```bash
python src/train_baseline.py

```


5. **View MLflow Tracking:**
```bash
mlflow ui

```



## ☁️ AWS Cloud Architecture

The final model is packaged and deployed on AWS for production use:

1. **API Gateway** receives the user's document text.
2. **AWS Lambda** acts as a bridge, formatting the request and forwarding it.
3. **SageMaker Serverless Endpoint** runs the inference script and returns the predicted Type and Top-3 Departments.
4. **CloudWatch** monitors system performance, response times, and logs.

## 👥 Team Members

This project is developed as the graduation project for the **Digital Egypt Pioneers Initiative (DEPI)**.

* **Ahmed Mahmoud Mostafa Abdelkarim** 
* **Abdelrahman Mohamed Ezz** 

```
