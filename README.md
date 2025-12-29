# Real-Time-Credit-Card-Fraud-Detection-using-AWS
🛡️ Real-Time Credit Card Fraud Detection System (AWS Serverless)

A fully serverless, cloud-native system that simulates real-time credit card transaction ingestion, fraud scoring, alerting, and investigation, built entirely on AWS Free Tier–compatible services.

This project demonstrates how modern financial institutions design low-latency, highly reliable fraud detection pipelines using event-driven architectures.

📌 Project Motivation

Credit card fraud costs financial institutions billions annually. Traditional batch-based systems struggle to detect fraud in real time and fail under high transaction throughput.

This project simulates a production-grade real-time fraud detection pipeline that:

Processes transactions continuously

Applies fraud rules within milliseconds

Persists risk state and history

Alerts investigators instantly

Provides a read-only investigation dashboard

🏗️ High-Level Architecture

Ingestion → Scoring → Storage → Alerting → Investigation → Observability

EventBridge (Scheduler)
        ↓
Lambda (tx-generator)
        ↓
SQS Queue  ───→  SQS DLQ (safety)
        ↓
Lambda (score-decide)
        ↓
DynamoDB (fraud_table)
        ↓
SNS (Fraud Alerts)
        ↓
API Gateway
        ↓
Lambda (investigator-api)
        ↓
S3 Static Website (Investigator UI)


Observability:
CloudWatch Logs & Metrics + AWS X-Ray tracing across all Lambdas.

⚙️ AWS Services Used
Service	Purpose
EventBridge Scheduler	Simulates continuous transaction traffic
AWS Lambda	Serverless compute for ingestion, scoring, and APIs
Amazon SQS	Transaction buffering and decoupling
Amazon DynamoDB	Low-latency fraud state and transaction storage
Amazon SNS	Real-time fraud alerts
Amazon API Gateway	Exposes investigator APIs
Amazon S3	Hosts static investigator dashboard
CloudWatch	Logs, metrics, dashboards
AWS X-Ray	End-to-end request tracing
🧾 Transaction Schema

Each synthetic transaction mirrors a real credit-card authorization record:

{
  "txnId": "UUID",
  "userId": "U4019",
  "ts": "2025-12-07T02:16:30.610Z",
  "amount": 48.85,
  "currency": "USD",
  "merchant": "AMZ-MKT",
  "mcc": 5311,
  "channel": "ecom",
  "device": "ios-17",
  "geo": {
    "lat": 39.7149,
    "lon": -97.7046,
    "country": "IN"
  },
  "ip": "55.62.181.125",
  "label": 0
}

🧠 Fraud Detection Logic

Fraud decisions are made in real time using rule-based logic (ML-ready).

Current Rules Implemented

Z-Score Amount Anomaly (Z > 3)

Country Mismatch (home country ≠ transaction country)

Velocity Signals (1h / 24h windows)

Risk Aggregation

Decisions
Decision	Meaning
SAFE	Low risk
FLAG	Suspicious — needs review
HOLD	High risk — immediate action
🗄️ DynamoDB Data Model

Single-table design with GSI for efficient querying.

Primary Keys

pk: USER#<userId>

sk: TS#<timestamp>#TXN#<txnId>

Global Secondary Index (GSI)

gsi1pk: DECISION#FLAG | DECISION#HOLD | DECISION#SAFE

gsi1sk: <timestamp>#<txnId>

Enables fast retrieval of latest high-risk transactions.

🚨 Alerts

Amazon SNS sends alerts for FLAG and HOLD decisions.

Email subscription demonstrates investigator escalation flow.

🔍 Investigator Dashboard

A read-only S3-hosted web UI that allows investigators to:

View latest flagged/high-risk transactions

Inspect user, amount, decision, risk, reasons

View geolocation mismatch details

Refresh in real time via API Gateway

📊 Observability & Monitoring
CloudWatch Dashboards

SQS Queue Depth

Lambda Invocation Count

DynamoDB Write Throughput

Fraud Rate Over Time

AWS X-Ray

Full request path tracing:

Scheduler → tx-generator → SQS → score-decide → DynamoDB → API → UI

🔐 Security & Reliability

IAM least-privilege roles per Lambda

SQS DLQ for failed messages

No servers or long-running processes

Fully Free Tier–compatible setup

🎯 Success Criteria Achieved

End-to-end latency: < 500 ms (p95)

Real-time fraud detection

Decoupled, scalable architecture

Production-style observability

Fully functional investigation UI

🚀 How to Run

Deploy Lambdas (tx-generator, score-decide, investigator-api)

Create SQS queue + DLQ

Create DynamoDB table + GSI

Enable EventBridge Scheduler

Deploy API Gateway

Upload index.html to S3 static website

Enable CloudWatch + X-Ray

📌 Future Extensions

Replace rules with ML models (Isolation Forest / Autoencoder)

Add user feedback loop (confirm fraud)

Geo heatmaps & analytics dashboard

Authentication for investigator UI

👨‍💻 Author

Gaurav Dnyanesh Mahajan
MS in Data Science, University of Maryland
Focus: Cloud Computing, Data Engineering, ML Systems
