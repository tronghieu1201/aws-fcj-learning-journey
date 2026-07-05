---
title: "Proposal"
date: 2026-05-14
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

## AI Content Generator Platform - Nền tảng Sáng tạo Nội dung Marketing AI trên AWS

---

### 1. Tổng quan dự án

**AI Content Generator Platform** là nền tảng SaaS thế hệ mới, hỗ trợ các doanh nghiệp vừa và nhỏ (SMB) tự động hóa quy trình sáng tạo nội dung Marketing bằng công nghệ Generative AI. Nền tảng kết hợp **AWS Cloud** và **Gemini API (Google AI)** để cung cấp giải pháp tạo nội dung đa dạng, có thể mở rộng và bảo mật cao.

| Tiêu chí | Giá trị |
| --- | --- |
| **Mô hình kinh doanh** | SaaS – Subscription theo tháng/năm |
| **Đối tượng khách hàng** | Doanh nghiệp vừa và nhỏ, Agency Marketing |
| **Công nghệ AI** | Gemini API (Google AI) qua AWS Lambda |
| **Hạ tầng** | AWS ap-southeast-1 / ap-southeast-2 |
| **Khả năng mở rộng** | Auto Scaling – từ 10 đến 10,000+ người dùng |
| **Tính sẵn sàng** | Multi-AZ, RDS Failover – 99.9% uptime |

---

### 2. Mục tiêu

#### 2.1 Mục tiêu dự án

| # | Mục tiêu | Chỉ số đo lường |
| --- | --- | --- |
| 1 | Xây dựng MVP hoàn chỉnh trong 4 tuần (Tuần 9–12) | Go-live cuối Tuần 12 |
| 2 | Đạt tính sẵn sàng cao cho hệ thống production | Uptime ≥ 99.9% |
| 3 | Tự động hóa luồng sinh nội dung AI end-to-end | Thời gian < 60 giây/bộ nội dung |
| 4 | Triển khai kiến trúc đạt chuẩn AWS Well-Architected | Đủ 6 trụ cột |
| 5 | Bảo mật toàn diện theo nguyên tắc Least Privilege | Không có quyền wildcard `*` |

#### 2.2 Giá trị mang lại

- **Tiết kiệm thời gian:** Quy trình tạo nội dung rút ngắn từ 3–5 ngày xuống còn **< 30 phút**
- **Tiết kiệm chi phí:** Giảm 60–80% so với thuê Copywriter/Agency (từ $500–3,000/tháng xuống $50–150/tháng)
- **Nhất quán thương hiệu:** Brand Persona đảm bảo đúng tone of voice dù tạo 10 hay 1,000 bài viết
- **Mở rộng dễ dàng:** Scale sang nhiều sản phẩm, thị trường, ngôn ngữ mà không tăng nhân lực tuyến tính

---

### 3. Vấn đề cần giải quyết

#### 3.1 Bối cảnh thị trường

Thị trường Marketing Digital tại Đông Nam Á tăng trưởng mạnh sau đại dịch COVID-19. Các doanh nghiệp SMB phải đối mặt với áp lực tạo nội dung đa kênh liên tục, trong khi nguồn lực sáng tạo có hạn và chi phí Copywriter/Agency ngày càng tăng cao.

#### 3.2 Các vấn đề cốt lõi

**Vấn đề 1 – Chi phí nhân lực cao**
Một SMB trung bình cần chi $500–$3,000/tháng cho nội dung marketing. Nguồn nhân lực sáng tạo chất lượng cao khan hiếm và tốn kém.

**Vấn đề 2 – Thiếu nhất quán thương hiệu**
Khi nhiều người hoặc agency cùng sản xuất nội dung, giọng văn và hình ảnh thương hiệu dễ bị phân kỳ, gây mất tin tưởng từ khách hàng cuối.

**Vấn đề 3 – Tốc độ sản xuất nội dung chậm**
Quy trình truyền thống từ brief → viết → duyệt → chỉnh sửa → đăng mất trung bình **3–7 ngày làm việc**, không đáp ứng được nhịp độ real-time marketing.

**Vấn đề 4 – Khó mở rộng quy mô**
Khi doanh nghiệp mở rộng sang nhiều sản phẩm hoặc thị trường mới, nhu cầu nội dung tăng gấp bội nhưng không thể tăng nhân lực theo tỷ lệ tuyến tính.

#### 3.3 Cơ hội

Sự phổ biến của LLM và khả năng tích hợp qua API mở ra cơ hội xây dựng nền tảng **tự động hóa nội dung** có khả năng hiểu ngữ cảnh thương hiệu, tùy biến theo đối tượng khách hàng, và xuất ra nhiều định dạng — tất cả trong một giao diện đơn giản, không cần kỹ năng kỹ thuật.

---

### 4. Kiến trúc giải pháp

#### 4.1 Tổng quan

Hệ thống triển khai trên AWS với kiến trúc nhiều lớp (multi-tier), phân tách rõ ràng giữa các tầng Edge, API, Compute, Queue, AI Worker và Data. Toàn bộ tài nguyên tính toán nằm trong **VPC (10.0.0.0/16)** trải rộng **2 Availability Zones**.

![Architecture](/aws-fcj-learning-journey/images/Proposal/architecture.png)

#### 4.2 Các thành phần kiến trúc

| Tầng | Dịch vụ | Vai trò |
| --- | --- | --- |
| **Edge & Bảo mật** | Amazon CloudFront | CDN phân phối React SPA toàn cầu, cache nội dung tĩnh |
| | AWS WAF | Bảo vệ trước SQL Injection, XSS, Bot, DDoS layer 7 |
| **API** | Amazon API Gateway | Tiếp nhận request, rate limit, xác thực qua Cognito Authorizer |
| | Amazon Cognito | Xác thực và phân quyền người dùng (User Pool) |
| **Compute** | Application Load Balancer | Phân phối tải tới EC2, Health Check tự động, span 2 AZ |
| | Amazon EC2 (Express API) | Xử lý nghiệp vụ: xác thực, Brand Persona, Prompt, truy vấn RDS (đặt trong Private Subnet, không có Public IP, chỉ nhận traffic qua ALB) |
| | Auto Scaling Group | Tự động tăng/giảm số EC2 theo tải thực tế |
| **Queue & AI Worker** | Amazon SQS (Main Queue) | Nhận tác vụ AI từ EC2, xử lý bất đồng bộ |
| | AWS Lambda (Node.js) | Worker lấy job từ SQS, gọi Gemini API |
| **AI** | Gemini API (Google AI) | Mô hình LLM sinh nội dung marketing |
| **Data** | Amazon RDS PostgreSQL (Multi-AZ) | Lưu user, Brand Persona, lịch sử chiến dịch, trạng thái job |
| | Amazon S3 | Lưu file xuất (PDF, DOCX, HTML), logo, assets thương hiệu |
| **Observability** | CloudWatch | Logs, Metrics, Alarms toàn hệ thống |
| | AWS X-Ray | Distributed tracing – theo dõi request end-to-end |
| **Bảo mật** | AWS Secrets Manager | Lưu API Key Gemini và thông tin nhạy cảm |
| | AWS KMS | Mã hóa data at-rest (RDS, S3) |
| | AWS IAM | Phân quyền Least Privilege cho mọi dịch vụ |

#### 4.3 Luồng xử lý chính

1. User → React SPA → CloudFront
2. CloudFront → WAF (kiểm tra SQL Injection, XSS, Bot)
3. WAF → API Gateway (xác thực Cognito, rate limit)
4. API Gateway → ALB → EC2 (Auto Scaling Group)
5. EC2: xác thực user, xây dựng Prompt từ Brief + Brand Persona, truy vấn RDS → đẩy job → SQS Main Queue
6. Lambda Worker: lấy job từ SQS, lấy API Key từ Secrets Manager → gọi Gemini API
7. Gemini API sinh nội dung
8. Lambda: lưu file → S3, cập nhật trạng thái → RDS
9. EC2 trả kết quả về UI → User chỉnh sửa & xuất file

#### 4.4 AWS Well-Architected Framework

| Trụ cột | Giải pháp áp dụng |
| --- | --- |
| **Operational Excellence** | CloudWatch Alarms, X-Ray Tracing, CI/CD tự động |
| **Security** | WAF, IAM Least Privilege, Secrets Manager, KMS, Private Subnet |
| **Reliability** | ALB + Auto Scaling, RDS Multi-AZ Failover |
| **Performance Efficiency** | CloudFront CDN, Lambda Serverless, RDS Query Optimization |
| **Cost Optimization** | Lambda pay-per-use, S3 Lifecycle Policy, Auto Scaling theo nhu cầu |
| **Sustainability** | Chỉ cấp phát tài nguyên theo nhu cầu thực, tắt môi trường Dev ngoài giờ |

---

### 5. Timeline

| Tuần | Mục tiêu | Công việc chính | Milestone |
| --- | --- | --- | --- |
| **9** | Hạ tầng AWS hoàn chỉnh | VPC + Subnet + IAM; RDS Multi-AZ + Secrets Manager; ALB + EC2 ASG; CloudFront + WAF + API Gateway + CI/CD | Ping test CloudFront → API GW → ALB → EC2 → RDS thành công |
| **10** | Backend nghiệp vụ + UI cơ bản | Auth API (JWT + Cognito); CRUD Brand Persona; Prompt engine từ Brief + Persona; React Dashboard (UI v1.0) | Đăng nhập, tạo Persona, nhập Brief, nhận Prompt tự động |
| **11** | Luồng AI Worker end-to-end | SQS pipeline; Lambda → Gemini API → S3 + RDS; Export PDF/DOCX/HTML; Load Testing & tối ưu | Brief → AI sinh nội dung → Xuất PDF trong < 60 giây |
| **12** | Hardening & Production | Security audit (WAF, IAM, pentest); UAT + thu feedback; Fix bug + hoàn thiện docs/runbook; Go-Live + onboard | Production live, monitoring 24/7, khách hàng đầu tiên onboard |

---

### 6. Ngân sách

#### 6.1 Chi phí build project

| Dịch vụ | Cấu hình | Chi phí/tháng |
| --- | --- | --- |
| Amazon EC2 | t3.medium × 2 (On-Demand, Auto Scaling min=2, 1/AZ) | ~$60 |
| Amazon RDS PostgreSQL | db.t3.micro, Multi-AZ | ~$50 |
| NAT Gateway | 2 AZ | ~$64 |
| CloudFront | 100 GB transfer | ~$8 |
| API Gateway | 1M requests | ~$3.50 |
| CloudWatch | Logs + Metrics cơ bản | ~$5 |
| AWS Lambda | ~100,000 invocations, 512MB | ~$1 |
| Amazon SQS | ~500,000 requests | ~$0.20 |
| Amazon S3 | 50 GB | ~$2 |
| AWS Secrets Manager | 1 secret (Gemini API Key) | ~$1 |
| AWS KMS | 1 CMK (mã hóa RDS + S3) | ~$1 |
| Amazon Cognito | < 50,000 MAU (Free Tier) | $0 |
| **Tổng AWS/tháng** | | **~$196** |

> 💡 **Ghi chú:** ElastiCache Redis sẽ được bổ sung ở Phase 2 khi số lượng người dùng đồng thời vượt 200+, nhằm giảm tải cho RDS và tối ưu response time.

#### Gemini API (Google AI)

| Giai đoạn | Số lượng requests | Chi phí ước tính |
| --- | --- | --- |
| Dev/Staging | ~1,000/tháng | ~$1–5/tháng |

#### 6.2 Chiến lược tối ưu chi phí

- **Reserved Instances:** Đặt trước EC2 và RDS 1 năm → tiết kiệm 30–40%
- **Lambda Serverless:** AI Worker chỉ tính phí khi có job, không tốn tiền idle
- **CloudFront caching:** Giảm số request tới EC2 và tải bandwidth
- **S3 Lifecycle Policy:** Tự động chuyển file > 90 ngày sang S3 Glacier
- **Auto Scaling:** Thu nhỏ về min instances ngoài giờ cao điểm
- **Spot Instances cho Dev:** Tiết kiệm 60–70% so với On-Demand

---

### 7. Rủi ro

#### 7.1 Ma trận rủi ro

| Rủi ro | Khả năng | Ảnh hưởng |
| --- | --- | --- |
| Gemini API downtime/throttle | Trung bình | Cao |
| Chi phí Gemini API vượt ngân sách | Cao | Trung bình |
| Lỗ hổng bảo mật dữ liệu người dùng | Thấp | Rất cao |
| EC2 instance failure (không Multi-AZ) | Thấp | Cao |
| RDS failover chậm | Thấp | Trung bình |
| Lambda timeout khi Gemini chậm | Trung bình | Trung bình |
| Chi phí AWS vượt ước tính | Trung bình | Thấp |
| Trễ tiến độ Phase 3 (AI Integration) | Trung bình | Thấp |

#### 7.2 Biện pháp giảm thiểu

**Gemini API Downtime/Throttling:**

- Lambda retry với Exponential Backoff (3 lần, max 5 phút)
- CloudWatch Alarm khi số lượng job lỗi tăng bất thường
- Fallback: tích hợp OpenAI API hoặc Amazon Bedrock (Phase 2+)

**Chi phí Gemini API Vượt ngân sách:**

- Giới hạn số lần gọi AI theo plan (Free: 10/tháng, Basic: 100, Pro: unlimited)
- Rate limiting tại API Gateway (requests/minute per user)
- Google Cloud Budget Alert + AWS Cost Anomaly Detection

**Lỗ hổng bảo mật dữ liệu:**

- EC2, Lambda, RDS nằm trong Private Subnet (không có Public IP)
- KMS mã hóa data at-rest (RDS + S3); toàn bộ kết nối qua TLS 1.2+
- Secrets Manager thay thế hoàn toàn environment variables cho credentials
- WAF chặn SQL Injection, XSS, bad bots; CloudTrail audit log toàn bộ API calls
- Review IAM permissions định kỳ 90 ngày/lần

**Lambda Timeout khi Gemini Phản hồi Chậm:**

- Lambda timeout = 270 giây (có buffer so với 300 giây max)
- Mỗi job chỉ sinh 1 loại nội dung (không gom nhiều loại vào 1 Lambda call)
- Streaming response từ Gemini API khi có thể
- Timeout → job tự động retry với priority cao hơn

**Trễ Tiến độ Phase 3 (AI Integration):**

- Buffer 1 tuần (Tuần 12) dành cho Performance Testing và bug fix
- Prototype Lambda + Gemini API độc lập từ Tuần 2, song song với Phase 1
- Dùng Gemini API Sandbox (miễn phí) trong giai đoạn phát triển

---

>Tài liệu tham khảo:
>
>[AWS Well-Architected](https://aws.amazon.com/architecture/well-architected/)
>
>[Gemini API Docs](https://ai.google.dev/docs)
>
>[Amazon SQS Best Practices](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/best-practices.html)
