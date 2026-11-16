# 📄 AI Document Classification & Routing System

![Python](https://img.shields.io/badge/Python-3.9+-blue.svg)
![AWS](https://img.shields.io/badge/AWS-Cloud-orange.svg)
![MLflow](https://img.shields.io/badge/MLflow-Tracking-blue.svg)
![HuggingFace](https://img.shields.io/badge/HuggingFace-Transformers-yellow.svg)

## 📌 Project Overview
Organizations receive large volumes of incoming documents such as complaints, requests, and invoices. Manual sorting is slow, prone to errors, and scales poorly as document volume grows. 

This project builds an end-to-end **AI Document Classification & Routing System** that reads the text of an incoming document and automatically recommends the appropriate department to handle it. Low-confidence predictions are flagged for human review, ensuring a balanced approach between automation and human accuracy.

## 🏗️ Visual Architecture
![System Architecture](docs/architecture.png)
*(Note: Replace with actual architecture diagram)*

**Cloud Request Flow:**
1. **Client/User** sends the document text via an HTTP request.
2. **AWS API Gateway** receives the request and triggers **AWS Lambda**.
3. **AWS Lambda** parses the payload and invokes the **SageMaker Serverless Endpoint**.
4. The model inside SageMaker processes the text and returns the predicted Type, Top-3 Departments, Confidence Scores, and a Human Review Flag back through the Lambda to the user.

## 📸 System in Action
*(Note: Replace placeholders with actual screenshots once the UI/Demo is ready)*

### 1. Classification Results & Top-3 Departments
*Shows the system predicting the document type and recommending the top 3 departments with confidence scores.*
![Results Screenshot](docs/results_demo.png)

### 2. Human Review Flag (Low Confidence)
*Demonstrates how the system flags a document for manual review when the confidence score falls below the accepted threshold.*
![Human Review Screenshot](docs/human_review_demo.png)

## 📊 Model Performance Comparison
All models are trained and evaluated on the exact same train/test splits. The table below highlights the actual final metrics achieved:

| Model | Accuracy | Precision | Recall | F1-Score |
| :--- | :---: | :---: | :---: | :---: |
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
This mode uses a pre-trained/locally trained model and serves it via a local **FastAPI** application for testing.

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
