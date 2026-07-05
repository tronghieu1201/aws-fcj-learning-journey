---
title: "Proposal"
date: 2026-05-14
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

## AI Content Generator Platform - AI-Powered Marketing Content Creation Platform on AWS

---

### 1. Project Overview

**AI Content Generator Platform** is a next-generation SaaS platform that helps small and medium-sized businesses (SMBs) automate their marketing content creation process using Generative AI technology. The platform combines **AWS Cloud** and the **Gemini API (Google AI)** to deliver a content generation solution that is diverse, scalable, and highly secure.

| Criteria | Value |
| --- | --- |
| **Business Model** | SaaS – Monthly/Annual Subscription |
| **Target Customers** | Small and medium-sized businesses, Marketing Agencies |
| **AI Technology** | Gemini API (Google AI) via AWS Lambda |
| **Infrastructure** | AWS ap-southeast-1 / ap-southeast-2 |
| **Scalability** | Auto Scaling – from 10 to 10,000+ users |
| **Availability** | Multi-AZ, RDS Failover – 99.9% uptime |

---

### 2. Objectives

#### 2.1 Project Objectives

| # | Objective | Success Metric |
| --- | --- | --- |
| 1 | Build a complete MVP within 4 weeks (Week 9–12) | Go-live by end of Week 12 |
| 2 | Achieve high availability for the production system | Uptime ≥ 99.9% |
| 3 | Automate the end-to-end AI content generation flow | < 60 seconds per content set |
| 4 | Implement an architecture aligned with the AWS Well-Architected Framework | All 6 pillars covered |
| 5 | Comprehensive security following the Least Privilege principle | No wildcard `*` permissions |

#### 2.2 Value Delivered

- **Time savings:** Content creation cycle shortened from 3–5 days to **under 30 minutes**
- **Cost savings:** 60–80% reduction compared to hiring a Copywriter/Agency (from $500–3,000/month down to $50–150/month)
- **Brand consistency:** Brand Persona ensures the correct tone of voice whether generating 10 or 1,000 posts
- **Easy scalability:** Scale across multiple products, markets, and languages without linearly increasing headcount

---

### 3. Problem Statement

#### 3.1 Market Context

Southeast Asia's Digital Marketing market has grown strongly since the COVID-19 pandemic. SMBs face constant pressure to produce multi-channel content, while creative resources remain limited and Copywriter/Agency costs continue to rise.

#### 3.2 Core Problems

**Problem 1 – High labor costs**
The average SMB spends $500–$3,000/month on marketing content. High-quality creative talent is scarce and expensive.

**Problem 2 – Lack of brand consistency**
When multiple people or agencies produce content together, tone of voice and brand visuals tend to diverge, eroding end-customer trust.

**Problem 3 – Slow content production speed**
The traditional workflow from brief → writing → review → revision → publishing takes an average of **3–7 business days**, which cannot keep up with real-time marketing demands.

**Problem 4 – Difficulty scaling**
As a business expands into more products or new markets, content demand multiplies, but headcount cannot scale linearly to match.

#### 3.3 Opportunity

The rise of LLMs and their API integration capabilities create an opportunity to build a **content automation** platform that understands brand context, adapts to target audiences, and exports multiple formats — all through a simple interface that requires no technical skills.

---

### 4. Solution Architecture

#### 4.1 Overview

The system is deployed on AWS using a multi-tier architecture, with clear separation between the Edge, API, Compute, Queue, AI Worker, and Data layers. All compute resources reside within a **VPC (10.0.0.0/16)** spanning **2 Availability Zones**.

![Architecture](/aws-fcj-learning-journey/images/Proposal/architecture.png)

#### 4.2 Architecture Components

| Layer | Service | Role |
| --- | --- | --- |
| **Edge & Security** | Amazon CloudFront | CDN distributing the React SPA globally, caching static content |
| | AWS WAF | Protects against SQL Injection, XSS, Bots, and Layer 7 DDoS |
| **API** | Amazon API Gateway | Receives requests, rate limiting, authentication via Cognito Authorizer |
| | Amazon Cognito | User authentication and authorization (User Pool) |
| **Compute** | Application Load Balancer | Distributes traffic to EC2, automatic Health Check, spans 2 AZs |
| | Amazon EC2 (Express API) | Handles business logic: authentication, Brand Persona, Prompt building, RDS queries (deployed in a Private Subnet with no Public IP, receiving traffic only via the ALB) |
| | Auto Scaling Group | Automatically scales EC2 count based on actual load |
| **Queue & AI Worker** | Amazon SQS (Main Queue) | Receives AI tasks from EC2, processed asynchronously |
| | AWS Lambda (Node.js) | Worker that pulls jobs from SQS and calls the Gemini API |
| **AI** | Gemini API (Google AI) | LLM model generating marketing content |
| **Data** | Amazon RDS PostgreSQL (Multi-AZ) | Stores users, Brand Personas, campaign history, job status |
| | Amazon S3 | Stores exported files (PDF, DOCX, HTML), logos, brand assets |
| **Observability** | CloudWatch | Logs, Metrics, Alarms across the whole system |
| | AWS X-Ray | Distributed tracing – end-to-end request tracking |
| **Security** | AWS Secrets Manager | Stores the Gemini API Key and other sensitive information |
| | AWS KMS | Encrypts data at rest (RDS, S3) |
| | AWS IAM | Least Privilege permissions for every service |

#### 4.3 Main Processing Flow

1. User → React SPA → CloudFront
2. CloudFront → WAF (checks for SQL Injection, XSS, Bots)
3. WAF → API Gateway (Cognito authentication, rate limiting)
4. API Gateway → ALB → EC2 (Auto Scaling Group)
5. EC2: authenticates the user, builds the Prompt from the Brief + Brand Persona, queries RDS → pushes the job → SQS Main Queue
6. Lambda Worker: pulls the job from SQS, retrieves the API Key from Secrets Manager → calls the Gemini API
7. Gemini API generates the content
8. Lambda: saves the file → S3, updates status → RDS
9. EC2 returns the result to the UI → user edits & exports the file

#### 4.4 AWS Well-Architected Framework

| Pillar | Applied Solution |
| --- | --- |
| **Operational Excellence** | CloudWatch Alarms, X-Ray Tracing, automated CI/CD |
| **Security** | WAF, IAM Least Privilege, Secrets Manager, KMS, Private Subnet |
| **Reliability** | ALB + Auto Scaling, RDS Multi-AZ Failover |
| **Performance Efficiency** | CloudFront CDN, Lambda Serverless, RDS Query Optimization |
| **Cost Optimization** | Lambda pay-per-use, S3 Lifecycle Policy, demand-based Auto Scaling |
| **Sustainability** | Resources provisioned only as actually needed; Dev environments shut down outside working hours |

---

### 5. Timeline

| Week | Goal | Key Activities | Milestone |
| --- | --- | --- | --- |
| **9** | Complete AWS infrastructure | VPC + Subnets + IAM; RDS Multi-AZ + Secrets Manager; ALB + EC2 ASG; CloudFront + WAF + API Gateway + CI/CD | Successful ping test CloudFront → API GW → ALB → EC2 → RDS |
| **10** | Backend business logic + basic UI | Auth API (JWT + Cognito); Brand Persona CRUD; Prompt engine from Brief + Persona; React Dashboard (UI v1.0) | Login, create Persona, submit Brief, receive auto-generated Prompt |
| **11** | End-to-end AI Worker flow | SQS pipeline; Lambda → Gemini API → S3 + RDS; PDF/DOCX/HTML export; Load Testing & optimization | Brief → AI-generated content → PDF export in under 60 seconds |
| **12** | Hardening & Production | Security audit (WAF, IAM, pentest); UAT + feedback collection; bug fixes + docs/runbook completion; Go-Live + onboarding | Production live, 24/7 monitoring, first customer onboarded |

---

### 6. Budget

#### 6.1 Build Project Cost

| Service | Configuration | Monthly Cost |
| --- | --- | --- |
| Amazon EC2 | t3.medium × 2 (On-Demand, Auto Scaling min=2, 1/AZ) | ~$60 |
| Amazon RDS PostgreSQL | db.t3.micro, Multi-AZ | ~$50 |
| NAT Gateway | 2 AZ | ~$64 |
| CloudFront | 100 GB transfer | ~$8 |
| API Gateway | 1M requests | ~$3.50 |
| CloudWatch | Basic Logs + Metrics | ~$5 |
| AWS Lambda | ~100,000 invocations, 512MB | ~$1 |
| Amazon SQS | ~500,000 requests | ~$0.20 |
| Amazon S3 | 50 GB | ~$2 |
| AWS Secrets Manager | 1 secret (Gemini API Key) | ~$1 |
| AWS KMS | 1 CMK (encrypts RDS + S3) | ~$1 |
| Amazon Cognito | < 50,000 MAU (Free Tier) | $0 |
| **Total AWS/month** | | **~$196** |

> 💡 **Note:** ElastiCache Redis will be added in Phase 2 once concurrent users exceed 200+, to offload RDS and optimize response time.

#### Gemini API (Google AI)

| Phase | Number of Requests | Estimated Cost |
| --- | --- | --- |
| Dev/Staging | ~1,000/month | ~$1–5/month |

#### 6.2 Cost Optimization Strategy

- **Reserved Instances:** Reserve EC2 and RDS for 1 year → save 30–40%
- **Lambda Serverless:** AI Worker is billed only when a job runs, no idle cost
- **CloudFront caching:** Reduces requests reaching EC2 and bandwidth load
- **S3 Lifecycle Policy:** Automatically moves files older than 90 days to S3 Glacier
- **Auto Scaling:** Scales down to minimum instances outside peak hours
- **Spot Instances for Dev:** Saves 60–70% compared to On-Demand

---

### 7. Risks

#### 7.1 Risk Matrix

| Risk | Likelihood | Impact |
| --- | --- | --- |
| Gemini API downtime/throttling | Medium | High |
| Gemini API cost exceeding budget | High | Medium |
| User data security breach | Low | Very High |
| EC2 instance failure (not Multi-AZ) | Low | High |
| Slow RDS failover | Low | Medium |
| Lambda timeout due to slow Gemini response | Medium | Medium |
| AWS cost exceeding estimate | Medium | Low |
| Phase 3 (AI Integration) schedule delay | Medium | Low |

#### 7.2 Mitigation Measures

**Gemini API Downtime/Throttling:**

- Lambda retry with Exponential Backoff (3 attempts, max 5 minutes)
- CloudWatch Alarm when failed job count rises abnormally
- Fallback: integrate OpenAI API or Amazon Bedrock (Phase 2+)

**Gemini API Cost Exceeding Budget:**

- Limit AI calls per plan (Free: 10/month, Basic: 100, Pro: unlimited)
- Rate limiting at the API Gateway (requests/minute per user)
- Google Cloud Budget Alert + AWS Cost Anomaly Detection

**User Data Security Breach:**

- EC2, Lambda, and RDS reside in a Private Subnet (no Public IP)
- KMS encrypts data at rest (RDS + S3); all connections use TLS 1.2+
- Secrets Manager fully replaces environment variables for credentials
- WAF blocks SQL Injection, XSS, and bad bots; CloudTrail audit-logs all API calls
- IAM permissions reviewed every 90 days

**Lambda Timeout from Slow Gemini Responses:**

- Lambda timeout = 270 seconds (with buffer below the 300-second max)
- Each job generates only one content type (multiple types are never batched into a single Lambda call)
- Streaming responses from the Gemini API where possible
- On timeout, the job automatically retries with higher priority

**Phase 3 (AI Integration) Schedule Delay:**

- 1-week buffer (Week 12) reserved for Performance Testing and bug fixing
- Lambda + Gemini API prototyped independently from Week 2, in parallel with Phase 1
- Use the free Gemini API Sandbox during development

---

>References:
>
>[AWS Well-Architected](https://aws.amazon.com/architecture/well-architected/)
>
>[Gemini API Docs](https://ai.google.dev/docs)
>
>[Amazon SQS Best Practices](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/best-practices.html)
