---
title: "Đề xuất dự án"
date: 2026-06-12
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

## Nền tảng Sáng tạo Nội dung Marketing AI trên AWS

### 1. Tổng quan dự án

**AI Content Generator Platform** là nền tảng SaaS thế hệ mới, hỗ trợ các doanh nghiệp vừa và nhỏ (SMB) tự động hóa quy trình sáng tạo nội dung Marketing bằng công nghệ Generative AI. Nền tảng kết hợp **AWS Cloud** và **Gemini API (Google AI)** để cung cấp giải pháp tạo nội dung đa dạng, có thể mở rộng và bảo mật cao.

| Tiêu chí | Giá trị |
| --- | --- |
| Mô hình kinh doanh | SaaS — Subscription theo tháng/năm |
| Đối tượng khách hàng | Doanh nghiệp vừa và nhỏ (SMB), Marketing Agency |
| Công nghệ AI | Gemini API (Google AI) qua AWS Lambda |
| AWS Region | ap-southeast-1 (Singapore) |
| Availability Zones | ap-southeast-1a và ap-southeast-1b |
| Khả năng mở rộng | Auto Scaling Group (10–10,000+ users) |
| Tính sẵn sàng | Multi-AZ Deployment, RDS Multi-AZ Failover (99.9% uptime) |

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

- **Tiết kiệm thời gian:** Quy trình tạo nội dung rút ngắn từ 3–5 ngày xuống còn **< 30 phút**.
- **Tiết kiệm chi phí:** Giảm 60–80% so với thuê Copywriter/Agency (từ $500–3,000/tháng xuống $50–150/tháng).
- **Nhất quán thương hiệu:** Brand Persona đảm bảo đúng tone of voice dù tạo 10 hay 1,000 bài viết.
- **Mở rộng dễ dàng:** Scale sang nhiều sản phẩm, thị trường, ngôn ngữ mà không tăng nhân lực tuyến tính.

### 3. Vấn đề cần giải quyết

#### 3.1 Bối cảnh thị trường

Thị trường Marketing Digital tại Đông Nam Á tăng trưởng mạnh sau đại dịch COVID-19. Các doanh nghiệp SMB phải đối mặt với áp lực tạo nội dung đa kênh liên tục, trong khi nguồn lực sáng tạo có hạn và chi phí Copywriter/Agency ngày càng tăng cao.

#### 3.2 Các vấn đề cốt lõi

**Vấn đề 1 — Chi phí nhân lực cao**
Một SMB trung bình cần chi $500–$3,000/tháng cho nội dung marketing. Nguồn nhân lực sáng tạo chất lượng cao khan hiếm và tốn kém.

**Vấn đề 2 — Thiếu nhất quán thương hiệu**
Khi nhiều người hoặc agency cùng sản xuất nội dung, giọng văn và hình ảnh thương hiệu dễ bị phân kỳ, gây mất tin tưởng từ khách hàng cuối.

**Vấn đề 3 — Tốc độ sản xuất nội dung chậm**
Quy trình truyền thống từ brief → viết → duyệt → chỉnh sửa → đăng mất trung bình **3–7 ngày làm việc**, không đáp ứng được nhịp độ real-time marketing.

**Vấn đề 4 — Khó mở rộng quy mô**
Khi doanh nghiệp mở rộng sang nhiều sản phẩm hoặc thị trường mới, nhu cầu nội dung tăng gấp bội nhưng không thể tăng nhân lực theo tỷ lệ tuyến tính.

#### 3.3 Cơ hội

Sự phổ biến của LLM và khả năng tích hợp qua API mở ra cơ hội xây dựng nền tảng **tự động hóa nội dung** có khả năng hiểu ngữ cảnh thương hiệu, tùy biến theo đối tượng khách hàng, và xuất ra nhiều định dạng — tất cả trong một giao diện đơn giản, không cần kỹ năng kỹ thuật.

### 4. Kiến trúc giải pháp

### 4.1 Tổng quan

Hệ thống được triển khai trên Amazon Web Services (AWS) theo kiến trúc nhiều lớp (Multi-tier Architecture) và tuân thủ AWS Well-Architected Framework. Toàn bộ hạ tầng được triển khai trong một Amazon VPC (10.0.0.0/16) thuộc Region ap-southeast-1 (Singapore), trải rộng trên hai Availability Zones (ap-southeast-1a và ap-southeast-1b) nhằm đảm bảo High Availability và Fault Tolerance.

Frontend React SPA được lưu trữ trên Amazon S3 Static Website và phân phối thông qua Amazon CloudFront. CloudFront sử dụng hai Origins:

- Amazon S3 Static Website (React SPA)
- Amazon API Gateway (REST API)

Mọi API request đều đi qua AWS WAF, Amazon API Gateway và Amazon Cognito Authorizer trước khi được chuyển tiếp qua VPC Link tới Internal Application Load Balancer. Application Load Balancer thực hiện Health Check và phân phối request tới Amazon EC2 Auto Scaling Group triển khai trên hai Availability Zones.

**Sơ đồ kiến trúc:**

![AI Content Generator Platform architecture](/aws-fcj-learning-journey/images/Proposal/proposal-architecture1.png)

#### 4.2 Các thành phần kiến trúc

| Tầng | Dịch vụ | Vai trò |
| --- | --- | --- |
| Edge & Bảo mật | Amazon CloudFront | Phân phối nội dung toàn cầu (CDN), cache nội dung tĩnh và định tuyến request tới hai Origins gồm Amazon S3 Static Website và Amazon API Gateway. |
| | AWS WAF | Bảo vệ ứng dụng khỏi SQL Injection, Cross-site Scripting (XSS), Layer 7 DDoS và các bot độc hại trước khi request tới API Gateway. |
| API | Amazon API Gateway | Thực hiện JWT Validation thông qua Amazon Cognito User Pool Authorizer trước khi chuyển request tới Application Load Balancer. |
| | Amazon Cognito | Quản lý User Pool, xác thực người dùng bằng JWT Token và đóng vai trò Authorizer cho Amazon API Gateway. |
| Compute | Application Load Balancer | Triển khai trên hai Availability Zones, nhận request từ Amazon API Gateway, thực hiện Health Check và phân phối tải tới Amazon EC2 Auto Scaling Group. |
| | Amazon EC2 (Express API) | Xử lý toàn bộ nghiệp vụ của hệ thống, bao gồm xác thực người dùng, xây dựng Prompt từ Brand Persona, truy vấn Amazon RDS PostgreSQL và gửi AI Job vào Amazon SQS. EC2 sử dụng IAM Role để truy cập AWS Secrets Manager lấy Database Credentials. EC2 được triển khai trong Application Private Subnets và không có Public IP. |
| | Auto Scaling Group | Tự động tăng/giảm số EC2 theo tải thực tế. |
| Queue & AI Worker | Amazon SQS (Main Queue) | Nhận tác vụ AI từ EC2, xử lý bất đồng bộ. |
| | Amazon SQS (Dead Letter Queue) | Nhận message lỗi qua Redrive Policy. |
| | AWS Lambda (Node.js) | Lambda Worker được triển khai trong VPC (VPC Attached), **poll AI Job từ Amazon SQS Main Queue qua Event Source Mapping**, sử dụng IAM Role để truy cập AWS Secrets Manager lấy Gemini API Key, truy cập Internet thông qua NAT Gateway và Internet Gateway để gọi Gemini API và lưu kết quả về Amazon S3 cũng như Amazon RDS. |
| Mạng & Kết nối | NAT Gateway | Triển khai 1 NAT Gateway/AZ trong Public Subnet, cho phép **Amazon EC2 và AWS Lambda (triển khai trong Private Subnets)** khởi tạo kết nối ra Internet mà không cần Public IP. |
| | Internet Gateway | Cổng kết nối Internet của VPC, cho phép NAT Gateway chuyển tiếp traffic từ EC2/Lambda ra Internet để gọi các dịch vụ bên ngoài (Gemini API, AWS public endpoints). |
| | VPC Endpoint for S3 | Gateway Endpoint riêng cho Amazon S3, cho phép AWS Lambda ghi kết quả AI vào Amazon S3 Export Bucket qua mạng nội bộ AWS, không đi qua Internet. |
| AI | Gemini API (Google AI) | Mô hình LLM sinh nội dung marketing. |
| Data | Amazon RDS PostgreSQL (Multi-AZ) | Triển khai trong Database Private Subnets với DB Subnet Group trải rộng trên hai Availability Zones. Ứng dụng chỉ kết nối tới endpoint của **RDS Primary**; dữ liệu được đồng bộ tự động sang **RDS Standby** nhằm đảm bảo High Availability — Standby không được ứng dụng truy cập trực tiếp. |
| | Amazon S3 (Static Website) | Lưu trữ React Single Page Application (SPA) và đóng vai trò Origin cho Amazon CloudFront. |
| | Amazon S3 (Export Bucket) | Lưu trữ các file PDF, DOCX, Images và các tài nguyên (assets) được sinh ra từ AI. |
| Observability | CloudWatch | Thu thập Logs, Metrics và Alarms từ Amazon EC2, AWS Lambda và Amazon API Gateway nhằm phục vụ Monitoring, Alerting và Operational Visibility. |
| Security | AWS Secrets Manager | Lưu trữ Gemini API Key và Database Credentials. Amazon EC2 và AWS Lambda truy cập Secrets thông qua IAM Roles theo nguyên tắc Least Privilege. |
| | AWS KMS | Quản lý khóa mã hóa cho Amazon RDS, Amazon S3 và AWS Secrets Manager nhằm bảo vệ dữ liệu ở trạng thái lưu trữ (Encryption at Rest). |
| | AWS IAM | Áp dụng nguyên tắc Least Privilege. EC2 và Lambda sử dụng IAM Roles riêng biệt, không sử dụng Access Key tĩnh. |

#### 4.3 Luồng xử lý chính

1. User truy cập ứng dụng thông qua Amazon CloudFront.
2. CloudFront lấy React SPA từ Amazon S3 Static Website.
3. CloudFront chuyển API request tới Amazon API Gateway.
4. Amazon API Gateway xác thực JWT bằng Amazon Cognito Authorizer.
5. Amazon API Gateway chuyển request qua VPC Link tới Internal Application Load Balancer.
6. Application Load Balancer thực hiện Health Check và phân phối request tới Amazon EC2 Auto Scaling Group.
7. Amazon EC2 xác thực nghiệp vụ, truy vấn Amazon RDS PostgreSQL, lấy Database Credentials từ AWS Secrets Manager thông qua IAM Role và gửi AI Job vào Amazon SQS Main Queue.
8. AWS Lambda Worker poll AI Job từ Amazon SQS Main Queue thông qua Event Source Mapping (Lambda chủ động nhận job, không phải SQS đẩy job sang Lambda).
9. AWS Lambda lấy Gemini API Key từ AWS Secrets Manager và khởi tạo kết nối ra ngoài thông qua NAT Gateway.
10. AWS Lambda lưu kết quả AI vào Amazon S3 Export Bucket thông qua Amazon VPC Endpoint for S3.
11. AWS Lambda cập nhật trạng thái Job vào Amazon RDS PostgreSQL (**Primary**); dữ liệu được đồng bộ tự động sang RDS Standby (ứng dụng không kết nối trực tiếp tới Standby).
12. NAT Gateway chuyển tiếp traffic qua Internet Gateway ra Internet để AWS Lambda gửi Prompt tới Gemini API (Google AI) và nhận nội dung AI trả về.
13. Amazon EC2 truy vấn kết quả từ Amazon RDS và trả response về Amazon API Gateway, CloudFront và React SPA.

#### 4.4 AWS Well-Architected Framework

| Trụ cột | Giải pháp áp dụng |
| --- | --- |
| Operational Excellence | CloudWatch, CI/CD |
| Security | AWS WAF, IAM Least Privilege, Secrets Manager, KMS, Private Subnets |
| Reliability | Application Load Balancer, Auto Scaling Group, Multi-AZ RDS, NAT Gateway, SQS DLQ, VPC Endpoint for S3 |
| Performance Efficiency | CloudFront, Lambda, RDS Optimization |
| Cost Optimization | Auto Scaling, Lambda Pay-per-use, S3 Lifecycle |
| Sustainability | Scale theo nhu cầu, tắt môi trường Dev ngoài giờ |

### 5. Timeline

| Tuần | Mục tiêu | Công việc chính | Milestone |
| --- | --- | --- | --- |
| 9 | Hạ tầng AWS hoàn chỉnh | Vẽ sơ đồ kiến trúc + VPC + Subnet + IAM; RDS Multi-AZ + Secrets Manager; ALB + EC2 ASG; CloudFront + WAF + API Gateway + CI/CD | Ping test CloudFront → API Gateway → Application Load Balancer → EC2 → RDS |
| 10 | Backend nghiệp vụ + UI cơ bản | Auth API (JWT + Cognito); CRUD Brand Persona; Prompt engine từ Brief + Persona; React Dashboard (UI v1.0) | Đăng nhập, tạo Persona, nhập Brief, nhận Prompt tự động |
| 11 | Luồng AI Worker end-to-end | SQS pipeline; Lambda → Gemini API → S3 + RDS; Export PDF/DOCX/HTML; Load Testing & tối ưu | Brief → AI sinh nội dung → Xuất PDF trong < 60 giây |
| 12 | Hardening & Production | Security audit (WAF, IAM, pentest); UAT + thu feedback; Fix bug + hoàn thiện docs/runbook; Go-Live + onboard | Production live, monitoring 24/7, khách hàng đầu tiên onboard |

### 6. Ngân sách

#### 6.1 Chi phí build project

| Dịch vụ | Cấu hình | Chi phí/tháng |
| --- | --- | --- |
| Amazon EC2 | t3.medium × 2 (On-Demand, Auto Scaling min=2, 1/AZ) | ~$60 |
| Amazon RDS PostgreSQL | db.t3.micro, Multi-AZ | ~$50 |
| NAT Gateway | Production: 2 NAT Gateway · Development: 1 NAT Gateway | ~$64 |
| CloudFront | 100 GB transfer | ~$8 |
| API Gateway | 1M requests | ~$3–4/tháng (tùy lưu lượng) |
| CloudWatch | Logs + Metrics cơ bản | ~$5 |
| AWS Lambda | ~100,000 invocations, 512MB | ~$1 |
| Amazon SQS | ~500,000 requests | ~$0.20 |
| Amazon S3 | 50 GB | ~$2 |
| AWS Secrets Manager | 2 Secrets (Gemini API Key, Database Credentials) | ~$2/tháng |
| AWS KMS | 1 Customer Managed KMS Key (dùng cho RDS, S3, Secrets Manager) | ~$1 |
| Amazon Cognito | < 50,000 MAU (Free Tier) | $0 |
| **Tổng AWS/tháng** | | **~$196** |

#### Gemini API (Google AI)

| Giai đoạn | Số lượng requests | Chi phí ước tính |
| --- | --- | --- |
| Dev/Staging | ~1,000/tháng | ~$1–5/tháng |

#### 6.2 Chiến lược tối ưu chi phí

- **Reserved Instances:** Đặt trước EC2 và RDS 1 năm → tiết kiệm 30–40%.
- **Lambda Serverless:** AI Worker chỉ tính phí khi có job, không tốn tiền idle.
- **CloudFront caching:** Giảm số request tới EC2 và tải bandwidth.
- **S3 Lifecycle Policy:** Tự động chuyển file > 90 ngày sang S3 Glacier.
- **Auto Scaling:** Thu nhỏ về min instances ngoài giờ cao điểm.
- **Spot Instances cho Dev:** Tiết kiệm 60–70% so với On-Demand.

### 7. Rủi ro

#### 7.1 Ma trận rủi ro

| Rủi ro | Khả năng | Ảnh hưởng |
| --- | --- | --- |
| Gemini API downtime/throttle | Trung bình | Cao |
| Chi phí Gemini API vượt ngân sách | Cao | Trung bình |
| Lỗ hổng bảo mật dữ liệu người dùng | Thấp | Rất cao |
| Một EC2 instance riêng lẻ gặp sự cố (không phải toàn hệ thống — đã có Auto Scaling Group đa AZ tự thay thế) | Trung bình | Thấp |
| RDS failover chậm | Thấp | Trung bình |
| Lambda timeout khi Gemini chậm | Trung bình | Trung bình |
| Chi phí AWS vượt ước tính | Trung bình | Thấp |
| Trễ tiến độ Phase 3 (AI Integration) | Trung bình | Thấp |

#### 7.2 Biện pháp giảm thiểu

- AWS WAF bảo vệ hệ thống khỏi SQL Injection, XSS và Bot.
- Amazon API Gateway sử dụng Cognito Authorizer để xác thực JWT.
- Application Load Balancer nhận request từ Amazon API Gateway và phân phối lưu lượng tới Amazon EC2 Auto Scaling Group trên hai Availability Zones — một EC2 instance gặp sự cố sẽ được ASG tự động thay thế mà không ảnh hưởng toàn hệ thống.
- Amazon EC2, AWS Lambda và Amazon RDS đều nằm trong Private Subnets.
- AWS Secrets Manager lưu Database Credentials và Gemini API Key.
- AWS KMS mã hóa Amazon RDS, Amazon S3 và AWS Secrets Manager.
- NAT Gateway và Internet Gateway giới hạn chiều kết nối ra Internet, chỉ phục vụ EC2/Lambda gọi các dịch vụ bên ngoài cần thiết (Gemini API, AWS public endpoints).
- Amazon VPC Endpoint for S3 cho phép AWS Lambda ghi dữ liệu vào Amazon S3 qua mạng nội bộ AWS, không đi qua Internet công cộng.
- IAM Roles áp dụng nguyên tắc Least Privilege.
- Toàn bộ kết nối sử dụng TLS 1.2 trở lên.

---

**Tài liệu tham khảo:**

- AWS Well-Architected — <https://aws.amazon.com/architecture/well-architected/>
- Gemini API Docs — <https://ai.google.dev/docs>
- Amazon SQS Best Practices — <https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/best-practices.html>