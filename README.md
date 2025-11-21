# Churn Prediction Pipeline (AWS SageMaker)

This project implements a complete end-to-end **machine learning pipeline** using **Amazon SageMaker Pipelines** to train an XGBoost-based customer churn prediction model.  
The pipeline performs preprocessing, hyperparameter tuning, evaluation, conditional logic, model registration, and model creation.

Due to changes in the current SageMaker Studio environment, several steps from the original template (Clarify, Transform, Config) are no longer supported.  
This README reflects the updated, working pipeline.

---

## 1. Project Overview

The pipeline automates:

- Data preprocessing  
- Hyperparameter tuning (XGBoost)  
- Model evaluation  
- Conditional AUC gating  
- Model registry entry  
- Model creation  

**Updated:**  
- AUC checking and model registration are now **separate steps**  
- Clarify, ConfigFile, and Batch Transform steps were removed due to AWS environment limitations  
- Code was modernized for the latest SageMaker SDK

---

## 2. Pipeline Architecture

### ✅ A. Final Working Pipeline (Ends at Model Creation)

![Final Working Pipeline](<sandbox:/mnt/data/Screenshot%202025-11-14%20at%202.54.53 PM.png>)

### ❌ B. Original Full Pipeline (Clarify + Transform Failed)

![Failed Pipeline](<sandbox:/mnt/data/Screenshot%202025-11-14%20at%202.53.13 PM.png>)

---

## 3. Pipeline Steps (Updated with New Step Labels)

### **Step 1 — ChurnModelProcess (ProcessingStep)**  
Cleans and preprocesses the raw churn dataset.

- Loads CSV from S3  
- Encodes categorical variables  
- Splits into train, validation, test  
- Outputs processed files to S3 via `preprocess.py`

---

### **Step 2 — ChurnHyperParameterTuning (TuningStep)**  
Tunes XGBoost hyperparameters.

- Uses XGBoost 1.7-1 container  
- Hyperparameter ranges: `eta`, `alpha`, `min_child_weight`, `max_depth`  
- Objective metric: **validation AUC**  
- Produces best-model artifact in S3

---

### **Step 3 — ChurnEvalBestModel (ProcessingStep)**  
Evaluates the best model against the test dataset.

- Executes `evaluate.py`  
- Produces `evaluation.json`  
- AUC score extracted via PropertyFile

---

### **Step 4 — CheckAUCScoreChurnEvaluation (ConditionStep)**  
Controls whether the model should proceed to the next steps.

IF AUC > 0.75:
    continue pipeline
ELSE:
    stop pipeline

This step **only checks the AUC value** — it does **not** create or register the model.

---

### **Step 5 — RegisterChurnModel (RegisterModel)**  
Registers the model in the SageMaker Model Registry.

- Stores model data + metrics  
- Logs AUC score into Model Package metadata  
- Supports inference instances & batch transform instances  
- Creates or updates this package group:  
ChurnModelPackageGroup

---

### **Step 6 — ChurnCreateModel (ModelStep)**  
Creates a SageMaker Model object.

- Uses XGBoost inference container  
- References best model artifact  
- Prepares model for deployment (real-time or batch)

This is the final step of the working pipeline.

---

## 4. Execution Flow (Updated)

1. ChurnModelProcess
            ↓
2. ChurnHyperParameterTuning
            ↓
3. ChurnEvalBestModel
            ↓
4. CheckAUCScoreChurnEvaluation
         ├── AUC ≤ 0.75 → STOP
         └── AUC > 0.75 → continue
                    ↓
5. RegisterChurnModel
            ↓
6. ChurnCreateModel
            ↓
(end)

---

## 5. Removed Steps (Due to AWS Restrictions)

The original project contained steps that cannot run in the current SageMaker Studio environment.

| Step | Status | Reason |
|------|--------|--------|
| ClarifyProcessing | ❌ Removed | Clarify no longer supported in classic Studio |
| ChurnModelConfigFile | ❌ Removed | Relied on Clarify |
| ChurnTransform (Batch Inference) | ❌ Removed | Requires quota increase |

---

## 6. AWS Environment Limitations

### **1. Clarify Not Supported in Current Studio**  
- The class project was designed for an older Studio version  
- Current Studio cannot execute Clarify Processing jobs  
- AWS requires the new **SageMaker AI (Studio v3)** environment

➡️ **Clarify step fails consistently**

---

### **2. Batch Transform Requires Higher Service Quotas**  
- Batch Transform instance types not available by default  
- Quota must be increased via AWS Support ticket  
- Approval takes several days

➡️ **Transform step fails due to insufficient quota**

---

## 7. Final Status

| Step | Result |
|------|--------|
| Process Data | ✅ Succeeded |
| Hyperparameter Tuning | ✅ Succeeded |
| Evaluate Best Model | ✅ Succeeded |
| Check AUC Score | ✅ Succeeded |
| Register Model | ✅ Succeeded |
| Create Model | ✅ Succeeded |
| Clarify | ❌ Removed |
| ConfigFile | ❌ Removed |
| Batch Transform | ❌ Removed |

The pipeline now completes cleanly through **Model Creation**, which is the highest achievable step given Studio limitations.

---

## 8. Conclusion

The original GitHub project used outdated SageMaker code incompatible with the modern Studio environment.  
To make the pipeline runnable:

- Updated all code for the current SageMaker SDK  
- Debugged and modernized Processing, Tuning, Evaluation steps  
- Split **Check AUC** and **Register Model** into separate steps  
- Removed outdated/unusable steps  
- Successfully executed and validated the full working pipeline

The final pipeline is production-ready up through **model creation**, satisfying the learning objectives of the mini-project.

---

