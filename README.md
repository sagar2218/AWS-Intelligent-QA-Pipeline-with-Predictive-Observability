# 🧠 AWS AI-Augmented QA & Predictive Observability Architecture

> **Automated Test Execution → AI-Driven Alerting → Predictive Monitoring**  
> Powered by Amazon ECS · Bedrock · SageMaker · AMP · AMG

---

## 📌 Overview

This architecture delivers an end-to-end, cloud-native quality assurance pipeline on AWS. It automates test execution inside containers, uses **Amazon Bedrock (Claude 3)** to enrich failure notifications with AI-generated root cause analysis, and leverages **Amazon SageMaker** to run anomaly detection and predictive failure models on top of your observability metrics — surfacing ML-powered forecasts directly in Grafana dashboards.

---

## 🏗️ Architecture Phases

### Phase 1 — Test Execution Layer

| Component | Purpose |
|---|---|
| **Selenium Test Suite** | Entry point — browser-based automated test scripts |
| **Docker Build** | Containerizes the test suite into a portable image |
| **Amazon ECR** | Stores and versions Docker images (Container Registry) |
| **AWS Fargate** | Serverless compute — runs ECS tasks without managing EC2 instances |
| **AWS ECS** | Orchestrates container execution of the test suite |
| **Test Results File** | Output artifact written to S3 upon test completion |

**Flow:**
```
Selenium Test Suite → Docker Build → ECR → Fargate + ECS → Test Results File (S3)
```

---

### Phase 2 — AWS Event Pipeline (Notification Layer)

| Component | Purpose |
|---|---|
| **S3 Bucket + EventBridge** | Detects test result file upload and triggers downstream pipeline |
| **AWS Lambda** | Event handler — orchestrates AI analysis and notification dispatch |
| **Amazon Bedrock (Claude 3)** | ✦ NEW — AI-powered test failure analysis and root cause generation |
| **Amazon SNS** | Publishes enriched notification to subscribed engineers |
| **Engineers / Subscribers** | Receive AI-enriched alerts via email, Slack, PagerDuty, etc. |
| **IAM Roles & Permissions** | Lambda execution role, SNS publish policy, Bedrock invoke policy |

**Flow:**
```
S3 Upload → EventBridge → Lambda → Bedrock (AI Analysis) → SNS → Engineers
```

**Bedrock Integration Detail:**
- Lambda reads the test result file from S3
- Invokes **Amazon Bedrock (Claude 3)** with the test output as context
- Bedrock returns: root cause analysis, failure pattern summary, severity classification
- Lambda publishes the AI-enriched report to SNS before notifying engineers

---

### Phase 3 — Observability & Monitoring Layer

#### VPC Environment

| Component | Purpose |
|---|---|
| **Application Servers** | Emit runtime metrics and logs |
| **Prometheus Server** | Scrapes and collects metrics from application servers |
| **CloudWatch Agent** | Ships logs and system metrics to AWS managed services |

#### AWS Managed Observability

| Component | Purpose |
|---|---|
| **Amazon Managed Prometheus (AMP)** | Stores and queries time-series metrics at scale |
| **Amazon Managed Grafana (AMG)** | Visualizes metrics and ML predictions via dashboards |
| **Amazon SageMaker** | ✦ NEW — ML inference endpoint for anomaly detection & failure prediction |
| **Predictive Dashboard (AMG Panel)** | Displays SageMaker forecasts alongside real-time metrics |
| **IAM Roles & Policies** | Authentication and authorization for AMP, AMG, and SageMaker |

**Flow:**
```
App Servers → Prometheus → AMP → SageMaker (ML Inference) → AMG Predictive Dashboard → Engineers
                 ↑
         CloudWatch Agent
```

**SageMaker Integration Detail:**
- AMP time-series metrics are consumed by a SageMaker inference endpoint
- Trained models perform anomaly detection and failure rate forecasting
- Predictions are pushed back to AMG as a custom data source
- Engineers see ML-powered forecasts alongside live metrics in a single Grafana view

---

## 🔐 Security & IAM

| Policy | Scope |
|---|---|
| Lambda Execution Role | S3 read, Bedrock invoke, SNS publish |
| SNS Publish Policy | Lambda → SNS topic |
| Bedrock Invoke Policy | Lambda → Claude 3 model endpoint |
| AMP / AMG Access Policy | Prometheus remote write, Grafana data source |
| SageMaker Invoke Policy | AMP → SageMaker endpoint |

All cross-service communication is secured via least-privilege IAM roles. No credentials are hardcoded; all permissions are managed through AWS Identity and Access Management.

---

## ✦ AI/ML Services Summary

### Amazon Bedrock
- **Model:** Claude 3 (via Bedrock API)
- **Trigger:** Lambda invocation on every test result upload
- **Input:** Raw test result artifact (stdout, failure logs, stack traces)
- **Output:** Root cause analysis, severity level, failure pattern summary
- **Consumer:** SNS notification payload sent to engineers

### Amazon SageMaker
- **Type:** Real-time inference endpoint
- **Model:** Anomaly detection + failure prediction (time-series)
- **Input:** Prometheus metrics streamed from AMP
- **Output:** Anomaly scores, failure probability forecasts
- **Consumer:** Amazon Managed Grafana (custom data source panel)

---

## 🛠️ Technology Stack

| Layer | Technologies |
|---|---|
| Test Automation | Selenium, Docker, Amazon ECR |
| Container Orchestration | AWS Fargate, Amazon ECS |
| Eventing | Amazon S3, Amazon EventBridge, AWS Lambda |
| AI / ML | Amazon Bedrock (Claude 3), Amazon SageMaker |
| Notification | Amazon SNS |
| Metrics Collection | Prometheus, CloudWatch Agent |
| Metrics Storage | Amazon Managed Prometheus (AMP) |
| Visualization | Amazon Managed Grafana (AMG) |
| Security | AWS IAM (Roles, Policies) |
| Networking | AWS VPC |

---

## 🔄 End-to-End Data Flow

```
┌─────────────────────────────────────────────────────────┐
│  PHASE 1 — TEST EXECUTION                               │
│  Selenium → Docker → ECR → Fargate/ECS → Results (S3)  │
└──────────────────────┬──────────────────────────────────┘
                       │ S3 Upload triggers EventBridge
┌──────────────────────▼──────────────────────────────────┐
│  PHASE 2 — EVENT PIPELINE                               │
│  Lambda → Bedrock AI Analysis → SNS → Engineers         │
└──────────────────────┬──────────────────────────────────┘
                       │ Metrics & logs flow continuously
┌──────────────────────▼──────────────────────────────────┐
│  PHASE 3 — OBSERVABILITY                                │
│  Prometheus/CW → AMP → SageMaker → AMG Dashboard        │
└─────────────────────────────────────────────────────────┘
```

---

## 📂 Repository Structure (Suggested)

```
├── tests/
│   └── selenium/              # Selenium test suites
├── docker/
│   └── Dockerfile             # Test container definition
├── infra/
│   ├── ecs/                   # ECS task definitions
│   ├── lambda/                # Lambda function code (event handler + Bedrock call)
│   ├── sns/                   # SNS topic configuration
│   ├── sagemaker/             # SageMaker model artifacts and endpoint config
│   └── iam/                   # IAM role and policy definitions
├── observability/
│   ├── prometheus/            # Prometheus scrape configs
│   ├── grafana/               # AMG dashboard JSON exports
│   └── cloudwatch/            # CloudWatch agent config
└── README.md
```

---

## 🚀 Getting Started

### Prerequisites
- AWS CLI configured with appropriate permissions
- Docker installed locally
- Python 3.9+ (for Lambda and SageMaker)
- Terraform or AWS CDK (recommended for IaC)

### Deployment Steps

1. **Build & push Docker image**
   ```bash
   docker build -t selenium-tests ./docker
   aws ecr get-login-password | docker login --username AWS --password-stdin <ECR_URI>
   docker tag selenium-tests:latest <ECR_URI>/selenium-tests:latest
   docker push <ECR_URI>/selenium-tests:latest
   ```

2. **Deploy ECS task definition**
   ```bash
   aws ecs register-task-definition --cli-input-json file://infra/ecs/task-definition.json
   ```

3. **Deploy Lambda function**
   ```bash
   cd infra/lambda
   zip -r function.zip .
   aws lambda update-function-code --function-name test-event-handler --zip-file fileb://function.zip
   ```

4. **Deploy SageMaker endpoint**
   ```bash
   aws sagemaker create-endpoint --endpoint-name qe-anomaly-detector \
     --endpoint-config-name qe-anomaly-config
   ```

5. **Configure Prometheus remote write to AMP**
   ```yaml
   remote_write:
     - url: https://aps-workspaces.<region>.amazonaws.com/workspaces/<id>/api/v1/remote_write
       sigv4:
         region: <region>
   ```

---

## 📊 Key Benefits

| Capability | Without AI/ML | With Bedrock + SageMaker |
|---|---|---|
| Failure Notifications | Raw log dump | AI root cause + severity summary |
| Anomaly Detection | Manual threshold rules | ML-trained model, continuous learning |
| Engineer Response Time | High — requires log analysis | Low — actionable insight delivered instantly |
| Failure Prediction | Reactive only | Proactive — forecast before outage |
| Dashboard Intelligence | Historical metrics only | Real-time + ML forecast overlay |

---

## 📄 License

This architecture is provided for reference and educational purposes.  
Adapt freely for your organization's AWS environment.

---

*Architecture designed for AWS cloud-native workloads using serverless, managed, and AI/ML services.*
