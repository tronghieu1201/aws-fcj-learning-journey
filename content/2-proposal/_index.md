---
title: "Proposal"
date: 2026-06-12
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

## AI-Powered Marketing Content Creation Platform on AWS

### 1. Project Overview

**AI Content Generator Platform** is a next-generation SaaS platform that helps small and medium-sized businesses (SMBs) automate their marketing content creation workflows using Generative AI technology. The platform integrates **AWS Cloud** and **Gemini API (Google AI)** to deliver a highly scalable, secure, and diverse content generation solution.

| Criterion | Value |
| --- | --- |
| Business Model | SaaS — Monthly/Annual Subscription |
| Target Customers | Small and Medium-sized Businesses (SMBs), Marketing Agencies |
| AI Technology | Gemini API (Google AI) via AWS Lambda |
| AWS Region | ap-southeast-1 (Singapore) |
| Availability Zones | ap-southeast-1a and ap-southeast-1b |
| Scalability | Auto Scaling Group (10–10,000+ users) |
| High Availability | Multi-AZ Deployment, RDS Multi-AZ Failover (99.9% uptime) |

### 2. Objectives

#### 2.1 Project Objectives

| # | Objective | Key Metric |
| --- | --- | --- |
| 1 | Build a complete MVP within 4 weeks (Weeks 9–12) | Go-live by the end of Week 12 |
| 2 | Achieve high availability for the production system | Uptime ≥ 99.9% |
| 3 | Automate the end-to-end AI content generation workflow | Execution time < 60 seconds per content set |
| 4 | Deploy an architecture aligned with AWS Well-Architected Framework | Covers all 6 pillars |
| 5 | Comprehensive security based on the Principle of Least Privilege | No wildcard `*` permissions |

#### 2.2 Delivered Value

- **Time Savings:** Reduces content creation time from 3–5 days to **< 30 minutes**.
- **Cost Savings:** Decreases costs by 60–80% compared to hiring a Copywriter/Agency (from $500–$3,000/month down to $50–$150/month).
- **Brand Consistency:** Brand Persona ensures the correct tone of voice whether creating 10 or 1,000 articles.
- **Effortless Scaling:** Easily scales across multiple products, markets, and languages without a linear increase in headcount.

### 3. Problems to Solve

#### 3.1 Market Context

The Digital Marketing market in Southeast Asia has grown robustly post-COVID-19. SMBs face constant pressure to produce multi-channel content, while creative resources are limited and Copywriter/Agency costs continue to rise.

#### 3.2 Core Problems

**Problem 1 — High Labor Costs**
An average SMB needs to spend $500–$3,000/month on marketing content. High-quality creative talent is scarce and expensive.

**Problem 2 — Lack of Brand Consistency**
When multiple individuals or agencies produce content, the brand's voice and imagery easily diverge, leading to a loss of trust from end customers.

**Problem 3 — Slow Content Production Speed**
The traditional workflow from brief → writing → approval → editing → publishing takes an average of **3–7 business days**, failing to keep up with the pace of real-time marketing.

**Problem 4 — Scalability Challenges**
As businesses expand into new products or markets, content demand multiplies, but headcount cannot scale linearly.

#### 3.3 Opportunities

The widespread adoption of LLMs and open APIs offers an opportunity to build a **content automation platform**. This platform understands brand context, customizes output for target audiences, and exports to multiple formats—all within a simple interface requiring no technical skills.

### 4. Solution Architecture

### 4.1 Overview

The system is deployed on Amazon Web Services (AWS) using a Multi-tier Architecture and adheres to the AWS Well-Architected Framework. The entire infrastructure is provisioned within an Amazon VPC (10.0.***.***/16) in the ap-southeast-1 (Singapore) region, spanning two Availability Zones (ap-southeast-1a and ap-southeast-1b) to ensure High Availability and Fault Tolerance.

The React SPA Frontend is hosted on Amazon S3 Static Website and distributed through Amazon CloudFront. CloudFront utilizes two Origins:

- Amazon S3 Static Website (React SPA)
- Amazon API Gateway (REST API)

Every API request passes through AWS WAF, Amazon API Gateway, and an Amazon Cognito Authorizer before being forwarded via VPC Link to an internal Application Load Balancer. The Application Load Balancer performs Health Checks and distributes requests to an Amazon EC2 Auto Scaling Group deployed across two Availability Zones.

**Architecture Diagram:**

![AI Content Generator Platform architecture](/aws-fcj-learning-journey/images/Proposal/proposal-architecture1.png)

#### 4.2 Architectural Components

| Layer | Service | Role |
| --- | --- | --- |
| Edge & Security | Amazon CloudFront | Global Content Delivery Network (CDN), caches static assets, and routes requests to two Origins: Amazon S3 Static Website and Amazon API Gateway. |
| | AWS WAF | Protects the application from SQL Injection, Cross-site Scripting (XSS), Layer 7 DDoS, and malicious bots before requests reach API Gateway. |
| API | Amazon API Gateway | Performs JWT Validation via Amazon Cognito User Pool Authorizer before forwarding requests to the Application Load Balancer. |
| | Amazon Cognito | Manages the User Pool, authenticates users with JWT Tokens, and acts as the Authorizer for Amazon API Gateway. |
| Compute | Application Load Balancer | Deployed across two Availability Zones; receives requests from Amazon API Gateway, performs Health Checks, and distributes load to the Amazon EC2 Auto Scaling Group. |
| | Amazon EC2 (Express API) | Handles all core system business logic, including user authentication, constructing prompts from Brand Personas, querying Amazon RDS PostgreSQL, and pushing AI Jobs to Amazon SQS. EC2 instances use an IAM Role to retrieve Database Credentials from AWS Secrets Manager. EC2 is deployed in Application Private Subnets with no Public IPs. |
| | Auto Scaling Group | Automatically scales the number of EC2 instances based on actual load. |
| Queue & AI Worker | Amazon SQS (Main Queue) | Receives AI jobs from EC2 for asynchronous processing. |
| | Amazon SQS (Dead Letter Queue) | Captures failed messages via a Redrive Policy. |
| | AWS Lambda (Node.js) | Lambda Worker deployed inside the VPC (VPC Attached) that **polls AI Jobs from the Amazon SQS Main Queue via Event Source Mapping**. It uses an IAM Role to retrieve the Gemini API Key from AWS Secrets Manager, accesses the Internet via NAT Gateway and Internet Gateway to invoke the Gemini API, and saves results to both Amazon S3 and Amazon RDS. |
| Network & Connectivity | NAT Gateway | Deployed as 1 NAT Gateway per AZ in Public Subnets, allowing **Amazon EC2 and AWS Lambda (deployed in Private Subnets)** to initiate outbound Internet connections without Public IPs. |
| | Internet Gateway | The Internet gateway for the VPC, enabling NAT Gateways to route traffic from EC2/Lambda to the Internet for external service integration (Gemini API, AWS public endpoints). |
| | VPC Endpoint for S3 | A dedicated Gateway Endpoint for Amazon S3, allowing AWS Lambda to save AI results to the Amazon S3 Export Bucket over the AWS internal network without routing through the Internet. |
| AI | Gemini API (Google AI) | LLM foundation model used to generate marketing content. |
| Data | Amazon RDS PostgreSQL (Multi-AZ) | Deployed in Database Private Subnets with a DB Subnet Group spanning two Availability Zones. The application only connects to the **RDS Primary** endpoint; data is synchronously replicated to the **RDS Standby** for High Availability (the Standby instance is not directly accessed by the application). |
| | Amazon S3 (Static Website) | Hosts the React Single Page Application (SPA) and serves as an Origin for Amazon CloudFront. |
| | Amazon S3 (Export Bucket) | Stores AI-generated files including PDFs, DOCX, Images, and other assets. |
| Observability | CloudWatch | Collects Logs, Metrics, and Alarms from Amazon EC2, AWS Lambda, and Amazon API Gateway for monitoring, alerting, and operational visibility. |
| Security | AWS Secrets Manager | Securely stores the Gemini API Key and Database Credentials. Amazon EC2 and AWS Lambda access these secrets via IAM Roles in compliance with the Principle of Least Privilege. |
| | AWS KMS | Manages encryption keys for Amazon RDS, Amazon S3, and AWS Secrets Manager to ensure Encryption at Rest. |
| | AWS IAM | Enforces the Principle of Least Privilege. EC2 and Lambda utilize dedicated IAM Roles; no static IAM access keys are used. |

#### 4.3 Main Execution Flow

1. Users access the application via Amazon CloudFront.
2. CloudFront retrieves the React SPA from Amazon S3 Static Website.
3. CloudFront routes API requests to Amazon API Gateway.
4. Amazon API Gateway validates the JWT via the Amazon Cognito Authorizer.
5. Amazon API Gateway forwards the request through a VPC Link to the internal Application Load Balancer.
6. The Application Load Balancer performs Health Checks and distributes the request to the Amazon EC2 Auto Scaling Group.
7. Amazon EC2 executes business logic, queries Amazon RDS PostgreSQL, retrieves Database Credentials from AWS Secrets Manager using its IAM Role, and pushes an AI Job to the Amazon SQS Main Queue.
8. The AWS Lambda Worker polls AI Jobs from the Amazon SQS Main Queue using Event Source Mapping (Lambda actively pulls the job; SQS does not push it).
9. AWS Lambda retrieves the Gemini API Key from AWS Secrets Manager and initiates an outbound connection via the NAT Gateway.
10. AWS Lambda saves the AI-generated results to the Amazon S3 Export Bucket via the Amazon VPC Endpoint for S3.
11. AWS Lambda updates the job status in Amazon RDS PostgreSQL (**Primary**); data is automatically replicated to the RDS Standby (the application does not connect directly to the Standby).
12. The NAT Gateway forwards traffic through the Internet Gateway to the Internet, allowing AWS Lambda to send the Prompt to the Gemini API (Google AI) and receive the generated content.
13. Amazon EC2 queries the results from Amazon RDS and returns the response to Amazon API Gateway, CloudFront, and the React SPA.

#### 4.4 AWS Well-Architected Framework

| Pillar | Applied Solution |
| --- | --- |
| Operational Excellence | CloudWatch, CI/CD |
| Security | AWS WAF, IAM Least Privilege, Secrets Manager, KMS, Private Subnets |
| Reliability | Application Load Balancer, Auto Scaling Group, Multi-AZ RDS, NAT Gateway, SQS DLQ, VPC Endpoint for S3 |
| Performance Efficiency | CloudFront, Lambda, RDS Optimization |
| Cost Optimization | Auto Scaling, Lambda Pay-per-use, S3 Lifecycle |
| Sustainability | Scaling on demand, turning off Dev environment outside of business hours |

### 5. Timeline

| Week | Target | Key Tasks | Milestone |
| --- | --- | --- | --- |
| 9 | Complete AWS Infrastructure | Design architecture diagram + VPC + Subnet + IAM; Multi-AZ RDS + Secrets Manager; ALB + EC2 ASG; CloudFront + WAF + API Gateway + CI/CD | Ping test: CloudFront → API Gateway → Application Load Balancer → EC2 → RDS |
| 10 | Core Backend + Basic UI | Auth API (JWT + Cognito); CRUD Brand Persona; Prompt engine from Brief + Persona; React Dashboard (UI v1.0) | Successful login, Persona creation, Brief submission, automated Prompt generation |
| 11 | End-to-End AI Worker Flow | SQS pipeline; Lambda → Gemini API → S3 + RDS; Export to PDF/DOCX/HTML; Load Testing & Optimization | Brief → AI content generation → Export PDF in < 60 seconds |
| 12 | Hardening & Production | Security audit (WAF, IAM, pentest); UAT + user feedback; Bug fixing + finalize docs/runbook; Go-Live + Onboarding | Production live, 24/7 monitoring, first customer onboarded |

### 6. Budget

#### 6.1 Project Build Costs

| Service | Configuration | Cost/Month |
| --- | --- | --- |
| Amazon EC2 | t3.medium × 2 (On-Demand, Auto Scaling min=2, 1/AZ) | ~$60 |
| Amazon RDS PostgreSQL | db.t3.micro, Multi-AZ | ~$50 |
| NAT Gateway | Production: 2 NAT Gateways · Development: 1 NAT Gateway | ~$64 |
| CloudFront | 100 GB Data Transfer | ~$8 |
| API Gateway | 1M requests | ~$3–4/month (based on traffic) |
| CloudWatch | Basic Logs + Metrics | ~$5 |
| AWS Lambda | ~100,000 invocations, 512MB | ~$1 |
| Amazon SQS | ~500,000 requests | ~$0.20 |
| Amazon S3 | 50 GB | ~$2 |
| AWS Secrets Manager | 2 Secrets (Gemini API Key, Database Credentials) | ~$2/month |
| AWS KMS | 1 Customer Managed KMS Key (used for RDS, S3, Secrets Manager) | ~$1 |
| Amazon Cognito | < 50,000 MAU (Free Tier) | $0 |
| **Total AWS/Month** | | **~$196** |

#### Gemini API (Google AI)

| Phase | Number of Requests | Estimated Cost |
| --- | --- | --- |
| Dev/Staging | ~1,000/month | ~$1–5/month |

#### 6.2 Cost Optimization Strategy

- **Reserved Instances:** Committing to a 1-year term for EC2 and RDS saves 30–40%.
- **Lambda Serverless:** The AI Worker only incurs charges during execution, eliminating idle costs.
- **CloudFront Caching:** Minimizes backend requests to EC2 and reduces bandwidth consumption.
- **S3 Lifecycle Policy:** Automatically transitions files older than 90 days to S3 Glacier.
- **Auto Scaling:** Scales down to the minimum instance count outside of peak hours.
- **Spot Instances for Dev:** Saves 60–70% compared to On-Demand pricing.

### 7. Risks

#### 7.1 Risk Matrix

| Risk | Likelihood | Impact |
| --- | --- | --- |
| Gemini API downtime/throttling | Medium | High |
| Gemini API costs exceeding budget | High | Medium |
| User data security vulnerability | Low | Critical |
| Individual EC2 instance failure (mitigated by Multi-AZ Auto Scaling Group auto-replacement) | Medium | Low |
| Slow RDS failover | Low | Medium |
| Lambda timeout during slow Gemini responses | Medium | Medium |
| AWS costs exceeding estimation | Medium | Low |
| Schedule delay in Phase 3 (AI Integration) | Medium | Low |

#### 7.2 Mitigation Measures

- AWS WAF protects the system against SQL Injection, XSS, and Bots.
- Amazon API Gateway uses Cognito Authorizer for JWT validation.
- The Application Load Balancer receives requests from Amazon API Gateway and distributes traffic to the Amazon EC2 Auto Scaling Group across two Availability Zones—ensuring a single instance failure is automatically resolved by ASG without impacting system availability.
- Amazon EC2, AWS Lambda, and Amazon RDS are all located in Private Subnets.
- AWS Secrets Manager stores Database Credentials and Gemini API Key.
- AWS KMS encrypts Amazon RDS, Amazon S3, and AWS Secrets Manager.
- NAT Gateways and Internet Gateways restrict outbound connections, limiting access only to necessary external endpoints (Gemini API, AWS public endpoints).
- Amazon VPC Endpoint for S3 allows AWS Lambda to write data to Amazon S3 via the AWS internal network, avoiding the public Internet.
- IAM Roles enforce the Principle of Least Privilege.
- All connections utilize TLS 1.2 or higher.

---

**References:**

- AWS Well-Architected — <https://aws.amazon.com/architecture/well-architected/>
- Gemini API Docs — <https://ai.google.dev/docs>
- Amazon SQS Best Practices — <https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/best-practices.html>
