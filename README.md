# Fetch → Analyze → Evaluate: A Reusable Agent Design Pattern (Strands Agents on AWS)

This repository implements a **behavioral agent design pattern** for clinical analytics called **Fetch → Analyze → Evaluate**.

---

## Pattern Overview

### 1. Fetch (data access)
- A single entry tool reads data from one configured source (HTTP servlet, local file, or S3 object; PDF/DOCX/TXT supported).

### 2. Analyzer Agent (LLM reasoning)
- Fetches from configured source (URL / file / S3) using fetch data tool
- Normalizes raw text (splits rows by delimiter if present)
- Produces a JSON that conforms to an **OUTPUT CONTRACT**

### 3. Evaluator Agent (plan-driven LLM-as-Judge)
- Loads output from analyzer agent through JSONL and optional clinician JSON
- Auto-selects an evaluation mode:
  - **Engagement vs Clinician** – compare categories/rationales
  - **SMART Goals Rubric** – S/M/A/R/T + clarity scoring
- Emits scores per case **+** overall means

---

## What’s Configurable (per use case)

The **Analyzer agent** is configurable so you can repurpose it without editing core logic:

- `provider` and `model` (Bedrock vs SageMaker endpoint)
- `DATA_SOURCE` – one of: HTTP/HTTPS URL, local file path, or `s3://bucket/key`
- `ROW_DELIM` – e.g., `"@"` for row-splitting; ignored for free text
- `DATA_LOG_FILE` – where formatted fetched data is appended for audit
- `OUTPUT_JSONL` – where Analyzer outputs are appended
- In **`ANALYZER_PROMPT`**:
  - `DEFAULT TASK` – replace entire text to change what the Analyzer does
  - `DEFAULT OUTPUT CONTRACT` – replace with your JSON schema for outputs

> 💡 You can fully repurpose the Analyzer by editing only the **DEFAULT TASK** and **DEFAULT OUTPUT CONTRACT** strings.

---

## System Requirements

- **AWS account** with permissions  
- **Domain:** Create a domain in Amazon SageMaker AI to run SageMaker Studio notebook  
- **Kernel:** Python 3 (`ipykernel`)  
- **Instance:** `ml.t3.medium` (sufficient for orchestration; model inference billed separately)  

---

## Python Packages (from `requirements.txt`)
```python
aiohttp==3.12.13
boto3==1.38.39
botocore==1.38.39
sagemaker==2.247.0
litellm==1.72.2
strands-agents==0.1.6
strands-agents-builder==0.1.2
strands-agents-tools==0.1.4
joblib==1.5.1
requests==2.32.4
uv==0.7.13```


---

## IAM Permissions

- `AmazonBedrockFullAccess` – Bedrock model invocation (if `provider="BEDROCK"`)
- `AmazonSageMakerFullAccess` – SageMaker runtime invoke endpoint (if `provider="SAGEMAKER"`)
- `AmazonS3FullAccess` – S3 read (if using `s3://…`)

---

## Bedrock Models Setup and Configuration

If using Bedrock models for Strands Agents:

1. Navigate to the **Amazon Bedrock** console (IAM User)
2. In the left navigation pane, choose **Model access**
3. Select the region you want to use (e.g., `us-east-1`)
4. Find and select the checkboxes for models you want to use:

**Examples:**
- Anthropic Claude 3.7 Sonnet  
- Anthropic Claude 3.5 Sonnet v2  
- Anthropic Claude 3.5 Haiku  
- DeepSeek-R1  
- Mistral Large (24.02)  

---

## How-to (from scratch on AWS)

1. Set IAM user to include **S3 + Bedrock or SageMaker endpoint invoke** permissions.
2. Open the AWS Console and launch **SageMaker AI Studio** in your target region (e.g., `us-east-1`).
3. Open **JupyterLab**, create a notebook with Python 3 (`ipykernel`) on `ml.t3.medium`.
4. Clone this repo or copy the notebook & `requirements.txt` files into Studio.
5. Install requirements using the `requirements.txt` file. **Restart kernel**.
6. Pick your model provider (Bedrock or SageMaker) and ensure region matches availability.
7. Configure data source (URL/file/S3) and `ROW_DELIM`.
8. Edit Analyzer PROMPT: adjust **DEFAULT TASK** + **DEFAULT OUTPUT CONTRACT** for your use case.
9. Run Analyzer to produce outputs in `OUTPUT_JSONL`.
10. Add `clinician_evaluation.json` for engagement scoring (if data is available).
11. Run Evaluator; inspect `evaluator_runs.jsonl` for scores and aggregated metrics.

---


