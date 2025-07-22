# Triển khai Ứng dụng Web Full-Stack Có Khả Năng Mở Rộng Trên AWS

##### _Một kiến trúc ứng dụng web hiện đại, an toàn, có khả năng mở rộng cao, sử dụng container, tích hợp MongoDB Atlas và tự động hóa triển khai bằng GitHub Actions trên nền tảng AWS._

# Executive Summary

- Mục tiêu của dự án này là thiết kế và triển khai một kiến trúc hạ tầng ứng dụng web hiện đại, linh hoạt và an toàn dựa trên các dịch vụ gốc của AWS. Hệ thống hỗ trợ phát triển full-stack (frontend React và backend Node.js/Express) và tích hợp với cả MongoDB cài đặt trên EC2 lẫn MongoDB Atlas (dịch vụ database quản lý bởi MongoDB).

- Để đảm bảo hiệu năng và độ tin cậy, kiến trúc sử dụng các dịch vụ như ECS (Elastic Container Service), ALB (Application Load Balancer), Route 53, CloudFront, CloudWatch, và KMS. Quy trình triển khai tự động (CI/CD) được thiết lập thông qua GitHub Actions giúp tiết kiệm thời gian và giảm thiểu rủi ro khi cập nhật hệ thống.

- MongoDB Atlas đóng vai trò là hệ thống cơ sở dữ liệu chính có khả năng sao lưu, mở rộng và phân phối toàn cầu. Ngoài ra, cụm MongoDB EC2 truyền thống vẫn được giữ lại trong VPC để phục vụ thử nghiệm nội bộ hoặc dự phòng.

* Hệ thống đảm bảo:

* Mở rộng linh hoạt theo nhu cầu,

* Khả năng giám sát và cảnh báo,

* Bảo mật dữ liệu và truy cập cao,

* Tối ưu hoá chi phí,

* Triển khai tự động hoá liên tục và thống nhất.

# 1. Problem Statement

- Các ứng dụng web hiện đại thường gặp phải các thách thức sau:

* Khi lưu lượng người dùng tăng đột biến, hệ thống truyền thống thường không đáp ứng kịp dẫn đến treo hoặc lỗi.

* Dữ liệu và ứng dụng dễ bị tấn công nếu không có tường lửa ứng dụng web (WAF), mã hóa (KMS), và phân quyền truy cập hợp lý.

* Việc triển khai thủ công khiến lỗi dễ xảy ra, mất thời gian và không đảm bảo môi trường nhất quán.

* Tài nguyên bị cấp phát quá mức gây lãng phí, hoặc cấp phát thiếu gây thiếu hụt hiệu năng.

## Current Situation

- Kiến trúc nguyên khối (monolithic): Frontend và backend được triển khai chung khiến việc cập nhật, mở rộng hoặc bảo trì rất khó khăn.

- Triển khai thủ công: Không có CI/CD nên mỗi lần cập nhật cần can thiệp thủ công, dễ xảy ra lỗi và mất thời gian.

- MongoDB đơn lẻ: Được cài đặt trên EC2 mà không có khả năng backup, scaling hay failover.

- Không có hệ thống phân phối toàn cầu: Người dùng ở xa server gốc gặp phải độ trễ cao.

- Thiếu an toàn bảo mật: Không có WAF, không mã hóa dữ liệu, không phân tách vùng mạng công khai và riêng tư.

- Không giám sát hiệu quả: Phát hiện lỗi muộn, không có cơ chế thông báo tự động.

## Key Challenges

- Quản lý container và scaling động
  Việc cấu hình ECS (Elastic Container Service) để tự động scale container frontend/backend theo nhu cầu là một bài toán phức tạp, đòi hỏi kiến thức sâu về tài nguyên CPU, RAM và cấu hình task definition, service, và autoscaling policy.

- Thiết lập CI/CD an toàn và hiệu quả
  Thiết lập GitHub Actions không chỉ dừng ở việc tự động build và deploy, mà còn cần đảm bảo bảo mật key/token, giới hạn quyền truy cập, rollback khi lỗi, kiểm thử tự động và phân tách môi trường staging - production rõ ràng.

- Tích hợp MongoDB Atlas và cấu hình bảo mật mạng
  Kết nối an toàn từ container backend đến MongoDB Atlas yêu cầu phải cấu hình IP whitelist, môi trường VPC peering (nếu cần), và xử lý các vấn đề liên quan đến SSL, latency giữa vùng (region).

- Bảo mật tầng mạng và tầng ứng dụng
  Kết hợp WAF (Web Application Firewall) để chặn các cuộc tấn công như SQL Injection, DDoS, XSS không đơn giản vì cần thiết lập rule cụ thể, testing và tối ưu hiệu suất mạng khi có thêm lớp bảo vệ.

- Quản lý phân tách subnet công khai/riêng tư trong VPC
  Phân tách chính xác frontend - backend - database trong các subnet khác nhau, với route table phù hợp, là điều bắt buộc để tối ưu bảo mật và lưu lượng mạng. Tuy nhiên điều này dễ bị sai sót khi thiết kế sơ đồ VPC không đúng chuẩn.

## Stakeholder Impact

- Nhà phát triển (Developers)

  - Triển khai code nhanh chóng nhờ CI/CD pipeline.

  - Môi trường dev và production đồng bộ (nhờ Docker).

  - Dễ dàng kiểm thử, rollback và phân tách môi trường rõ ràng.

- Người dùng cuối (End Users)

  - Tốc độ truy cập nhanh hơn nhờ CDN (CloudFront) và tối ưu server.

  - Hệ thống ổn định, ít downtime, khả năng mở rộng theo lưu lượng.

  - Bảo mật tốt hơn, giảm nguy cơ bị tấn công hoặc mất dữ liệu.

## Business Consequences

- Giảm trải nghiệm người dùng
  Ứng dụng chậm, thường xuyên bị lỗi hoặc downtime sẽ khiến người dùng từ bỏ dịch vụ, làm giảm lượng truy cập và mất uy tín thương hiệu.

- Tăng rủi ro bảo mật và mất dữ liệu
  Không có WAF, không có KMS, và cấu hình MongoDB thủ công dễ dẫn đến rò rỉ thông tin khách hàng, bị tấn công DDoS hoặc mất dữ liệu không thể khôi phục.

- Không theo kịp nhu cầu tăng trưởng
  Hệ thống cũ không thể scale kịp theo nhu cầu tăng nhanh, dẫn đến nghẽn cổ chai, giảm doanh thu do mất khách hàng trong giờ cao điểm

# 2. Solution Architecture

## Architecture Overview

- Kiến trúc của hệ thống được xây dựng theo mô hình ứng dụng hiện đại sử dụng container (cloud-native container-based architecture) nhằm đảm bảo khả năng mở rộng, tính linh hoạt, độ tin cậy và bảo mật cao. Toàn bộ ứng dụng được chia thành ba lớp chính: lớp giao diện người dùng (frontend), lớp dịch vụ xử lý nghiệp vụ (backend), và lớp lưu trữ dữ liệu (database), mỗi thành phần đều được tách biệt, triển khai độc lập và giao tiếp thông qua các API bảo mật.Phần frontend được phát triển bằng ReactJS, được đóng gói trong container NGINX và triển khai trên dịch vụ ECS (Elastic Container Service) của AWS. Tất cả các truy cập từ phía người dùng sẽ đi qua CloudFront – dịch vụ phân phối nội dung (CDN) – nhằm tăng tốc độ tải trang và giảm tải cho hệ thống, sau đó được định tuyến bởi Application Load Balancer (ALB) đến các container frontend. Phần backend được xây dựng bằng Node.js và ExpressJS, cũng được container hóa và triển khai trên ECS. Backend chịu trách nhiệm xử lý logic nghiệp vụ, xác thực người dùng, và giao tiếp với cơ sở dữ liệu.Dữ liệu được lưu trữ trong MongoDB Atlas – một dịch vụ cơ sở dữ liệu NoSQL quản lý hoàn toàn (Database-as-a-Service) – cho phép mở rộng dễ dàng, có tính sẵn sàng cao và tích hợp sẵn các cơ chế backup. Ngoài ra, một cụm MongoDB nội bộ được triển khai trên các máy chủ EC2 trong subnet riêng tư đóng vai trò thử nghiệm hoặc dự phòng.

## AWS Services Used

| Dịch vụ                                | Vai trò                                                                                   |
| -------------------------------------- | ----------------------------------------------------------------------------------------- |
| Amazon EC2                             | Lưu trữ MongoDB nội bộ (test/dev), có thể dùng làm fallback                               |
| Amazon ALB (Application Load Balancer) | Cân bằng tải và định tuyến tới container backend                                          |
| Amazon ECR                             | Container Registry nơi lưu trữ các phiên bản của các Docker Image được build từ CodeBuild |
| AWS VPC                                | Tạo các môi trường riêng biệt và độc lập cho các giai đoạn khác nhau                      |
| IAM                                    | Tạo các user, các role cho phù hợp cho các team phát triển và các quá trình               |
| AWS Code Pipeline                      | Điều phối CI/CD pipeline                                                                  |
| AWS Code Build                         | Build Docker image từ source code                                                         |
| AWS CodeDeploy                         | Deploy ECS task sau khi build                                                             |
| Amazon Route 53                        | Quản lý DNS và routing domain đến ALB                                                     |
| Application Load Balancer              | Điều phối các traffic đến các node trong cụm                                              |
| Amazon CloudWatch                      | Thu thập logs, metrics và tạo alarm                                                       |
| Amazon SNS                             | Gửi thông báo khi có sự kiện xảy ra                                                       |

## Component Design

- Frontend Service
  - Sử dụng ReactJS build với Vite, đóng gói vào Docker container.
  - Chạy trên ECS, expose port 5173 qua Nginx để phản hồi HTTP.
  - Được truy cập qua CloudFront → ALB → ECS container.
- Backend Service
  - Sử dụng Node.js + Express, đóng gói container và expose tại port 4000.
  - Tích hợp các route API, kết nối MongoDB Atlas qua URI bảo mật.
  - Được bảo vệ bằng ALB và có thể scale nhiều instance ECS nếu cần.
- Database Layer
  - MongoDB Atlas là database chính, hỗ trợ backup, cluster, scaling.
  - MongoDB EC2 (cài đặt nội bộ) được đặt trong subnet riêng tư, chủ yếu dùng thử nghiệm hoặc khôi phục.
  - Database không được truy cập trực tiếp từ frontend — chỉ backend có quyền truy vấn.
- CI/CD Pipeline
  - GitHub Actions triển khai:
    - Build frontend/backend,
    - Push Docker image lên ECR hoặc ECS trực tiếp,
    - Trigger deploy container mới vào ECS.
  - Toàn bộ secret/key được lưu trữ dưới GitHub Secrets và KMS.

## Security Architecture

- WAF lọc toàn bộ truy cập đến hệ thống, ngăn SQL Injection, XSS, bot traffic.
- KMS được tích hợp để mã hóa:
  - Secrets,
  - Connection string,
  - Thông tin nhạy cảm trong môi trường deploy.
- Phân tầng subnet mạng:
  - Public subnet chỉ chứa ALB, Internet Gateway và CloudFront.
  - Private subnet chứa ECS backend và EC2 database — không thể truy cập trực tiếp từ internet.
- IAM Roles và Policy: Giới hạn quyền truy cập từng dịch vụ cụ thể.
- Security Groups: Cấu hình rule chặt chẽ giữa các thành phần (ví dụ: chỉ ALB mới được gọi tới ECS port 4000).
- Logging + Monitoring: Mọi request và lỗi được log vào CloudWatch và cảnh báo qua SNS.

## Scalability Design

- Auto Scaling ECS Service: ECS có thể tự scale số lượng container backend/frontend dựa trên metric như CPU > 70% hoặc request count.
- ALB Load Balancer: Có thể scale ngang không giới hạn để phục vụ lưu lượng truy cập tăng cao.
- MongoDB Atlas: Cluster có khả năng scale vertical (RAM/CPU) hoặc horizontal (sharding).
- CloudFront: Cache nội dung frontend, giảm tải cho ECS.
- CI/CD nhanh chóng: Cho phép triển khai bản cập nhật liên tục mà không gây downtime.
- Stateless Container Design: Container không lưu trạng thái, giúp việc scale diễn ra dễ dàng và không bị mất phiên đăng nhập khi dịch vụ được nhân bản.

# 3. Technical Implementation

## Implementation Phases

- Giai đoạn 1: Thiết kế kiến trúc hệ thống
  - Lựa chọn công nghệ phù hợp (React, Node.js, MongoDB Atlas, Docker, ECS).
  - Vẽ sơ đồ kiến trúc mạng, phân chia subnet công khai/riêng tư trong VPC.
  - Thiết lập các tài nguyên AWS ban đầu: ECS Cluster, VPC, ALB, IAM, CloudWatch, GitHub Actions.
- Giai đoạn 2: Phát triển ứng dụng
  - Xây dựng giao diện người dùng (ReactJS) và backend API (Node.js/Express).
  - Thiết lập kết nối MongoDB Atlas, cấu hình kết nối qua URI an toàn.
  - Tạo Dockerfile cho từng thành phần, chuẩn hóa theo chuẩn multistage.
- Giai đoạn 3: Tạo quy trình CI/CD
  - Cấu hình GitHub Actions cho frontend và backend:
    - Tự động build, test, và deploy lên ECS.
    - Sử dụng secrets, environment matrix, rollback nếu thất bại.
- Giai đoạn 4: Tích hợp và triển khai AWS
  - Đưa image lên ECS, kết nối với ALB.
  - Cấu hình CloudFront + Route 53 + SSL/TLS
  - Bổ sung WAF, KMS và CloudWatch để đảm bảo bảo mật và giám sát.
- Giai đoạn 5: Kiểm thử hệ thống và tối ưu
  - Kiểm tra toàn bộ hệ thống từ frontend đến database.
  - Xác nhận khả năng scale, failover, và hiệu suất hoạt động.
- Giai đoạn 6: Chuyển giao và vận hành
  - Cung cấp tài liệu vận hành, quyền IAM cho các nhóm liên quan.
  - Giám sát qua CloudWatch và phản hồi cảnh báo SNS nếu có lỗi.

## Technical Requirements

- _Yêu cầu phần mềm & công nghệ:_
  - Frontend: ReactJS, Vite, Nginx, Docker
  - Backend: Node.js, Express.js, Docker
  - Cơ sở dữ liệu: MongoDB Atlas (Cloud), MongoDB EC2 (Dev/Backup)
  - Hệ thống: AWS (ECS, EC2, ALB, Route 53, CloudFront, S3, WAF, KMS)
  - CI/CD: GitHub Actions
  - Containerization: Docker (compose + ECS-compatible Dockerfile)
- _Yêu cầu hạ tầng:_
  - VPC có 2 AZ (Availability Zone), với public và private subnets
  - ECS Cluster (Fargate hoặc EC2 mode)
  - ALB với rules cho cả frontend (port 5173) và backend (port 4000)
  - IAM roles và policy chi tiết
  - CloudFront CDN + SSL (ACM)

## Development Approach

- Tách biệt frontend và backend:

  - Frontend hoạt động độc lập trên ECS, build riêng, chỉ giao tiếp với backend qua REST API.

  - Backend có thể được thay đổi mà không ảnh hưởng giao diện và ngược lại.

- API-first Design:

  - Các API backend được thiết kế theo nguyên tắc REST, có tài liệu rõ ràng và kiểm thử qua Postman.

- Infrastructure as Code (IaC) (nếu mở rộng):

  - Trong các giai đoạn nâng cao, Terraform hoặc CloudFormation có thể được sử dụng để quản lý hạ tầng.

- Tái sử dụng code & component:

  - Các component React được tách riêng cho từng trang.

  - Backend có các controller và middleware rõ ràng.

- Container hóa toàn bộ:

  - Docker được sử dụng để build và chạy cả frontend/backend.

  - Môi trường dev và production được đồng bộ hoàn toàn.

## Testing Strategy

- Kiểm thử đơn vị (Unit Testing)

  - Kiểm thử các hàm xử lý logic trong backend, từng component React.

  - Sử dụng Jest (React) và Mocha/Chai (Node.js).

- Kiểm thử tích hợp (Integration Testing)

  - Kiểm tra luồng xử lý giữa các module backend với MongoDB.

  - Test các route API, validate dữ liệu đầu vào/đầu ra.

- Kiểm thử end-to-end (E2E)

  - Sử dụng Cypress hoặc Selenium để giả lập người dùng thực tế.

  - Kiểm thử quy trình login, thao tác form, hiển thị dữ liệu từ database.

- Kiểm thử tải (Load Testing)

  - Sử dụng JMeter/K6 để kiểm tra backend chịu được bao nhiêu request/giây.

  - Theo dõi response time qua CloudWatch.

- Kiểm thử bảo mật (Security Testing)

  - Kiểm tra SQL Injection, XSS, CORS, CSRF bằng Burp Suite hoặc OWASP ZAP.

## Deployment Plan

- Chuẩn bị môi trường sản xuất (Production-ready)

  - Thiết lập ECS Cluster, ALB, Route 53, CloudFront, S3.

  - Cấp quyền IAM tối thiểu cần thiết cho ECS task/service.

- Build & Push Docker Image

  - Sử dụng GitHub Actions để tự động hóa build image.

  - Push image vào Amazon ECR hoặc deploy trực tiếp đến ECS service.

- Triển khai ECS Task

  - ECS nhận task definition mới và cập nhật service.

  - ALB sẽ tự động chuyển traffic sang container mới.

- Kiểm thử hậu triển khai

  - Truy cập thử nghiệm frontend và backend từ môi trường production.

  - Kiểm tra CloudWatch logs để phát hiện lỗi phát sinh.

- Giám sát và tối ưu sau triển khai

  - Cấu hình cảnh báo SNS khi CPU, memory vượt ngưỡng.

  - Theo dõi lượng request qua CloudFront và ALB để điều chỉnh scaling.

# 4. Timeline & Milestones

## Project Timeline

| Tuần     | Giai đoạn                            | Hoạt động cụ thể                                                    | Thời gian |
| -------- | ------------------------------------ | ------------------------------------------------------------------- | --------- |
| Tuần 1-2 | Phân tích & Thiết kế kiến trúc       | - Thu thập yêu cầu, chọn công nghệ, vẽ sơ đồ VPC, thiết kế hệ thống | 2 tuần    |
| Tuần 3–4 | Phát triển frontend/backend          | - Phát triển frontend/backend                                       | 2 tuần    |
| Tuần 5   | Thiết lập MongoDB Atlas & Docker hóa | Cấu hình cơ sở dữ liệu, viết Dockerfile, kiểm thử local             | 1 tuần    |
| Tuần 6   | Thiết lập hạ tầng AWS                | Tạo VPC, ECS Cluster, ALB, Route 53, CloudFront, KMS, WAF           | 1 tuần    |
| Tuần 7   | Thiết lập CI/CD với GitHub Actions   | Viết workflow cho frontend/backend, kiểm thử build/deploy           | 5 ngày    |
| Tuần 8   | Kiểm thử toàn hệ thống               | Kiểm thử đơn vị, tích hợp, bảo mật và tải hệ thống                  | 3 ngày    |
| Tuần 9   | Triển khai production & nghiệm thu   | Deploy chính thức, chuyển giao hệ thống, hướng dẫn sử dụng          | 1 tuần    |

## Key Milestones

| Mốc quan trọng                         | Thời gian dự kiến | Kết quả mong đợi                                                |
| -------------------------------------- | ----------------- | --------------------------------------------------------------- |
| ✅ Hoàn tất thiết kế kiến trúc         | Cuối tuần 1       | Sơ đồ kiến trúc, lựa chọn công nghệ, sơ đồ VPC                  |
| ✅ Hoàn tất frontend/backend MVP       | Cuối tuần 3       | Giao diện cơ bản hoạt động + API CRUD mẫu                       |
| ✅ Hoàn tất Docker hóa & MongoDB Atlas | Giữa tuần 5       | Có thể chạy frontend/backend bằng Docker, kết nối DB thành công |
| ✅ Cấu hình xong toàn bộ hạ tầng AWS   | Cuối tuần 6       | ECS Cluster hoạt động, có thể truy cập qua ALB                  |
| ✅ CI/CD hoạt động ổn định             | Cuối tuần 7       | Code push → deploy ECS tự động                                  |
| ✅ Triển khai production               | Cuối tuần 9       | Ứng dụng hoạt động thực tế, có tài liệu hướng dẫn               |

## Dependencies

- Triển khai ECS và VPC phụ thuộc vào thiết kế kiến trúc hoàn chỉnh:
  Không thể cấu hình mạng hoặc cluster nếu sơ đồ VPC chưa thống nhất.

- CI/CD phụ thuộc vào Docker hóa backend/frontend:
  GitHub Actions cần Dockerfile chuẩn để build container đúng.

- Kết nối MongoDB Atlas phụ thuộc vào cấu hình mạng (IP whitelist, SSL):
  Cần xác định IP hoặc sử dụng peering cho phép backend truy cập.

- Thời gian nghiệm thu phụ thuộc vào việc kiểm thử hoàn chỉnh và không có lỗi nghiêm trọng:
  Nếu phát sinh lỗi lớn ở tuần 8, việc nghiệm thu có thể phải lùi lại.

## Resource Allocation

| Vai trò            | Số lượng | Nhiệm vụ cụ thể                                        |
| ------------------ | -------- | ------------------------------------------------------ |
| DevOps Engineer    | 1–2      | Cấu hình AWS, ECS, CI/CD, CloudWatch, WAF              |
| Backend Developer  | 2        | Viết mã microservices, Dockerfile, Kubernetes manifest |
| QA/Tester          | 1–2      | Viết test case, kiểm thử đơn vị, kiểm thử tải          |
| Project Manager    | 1        | Lên kế hoạch, theo dõi tiến độ, kết nối giữa các nhóm  |
| Frontend Developer | 1-2      | Thiết kế giao diện, xây dựng React UI, cấu hình NGINX  |

# 5. Budget Estimation

## Infrastructure Costs

| Dịch vụ AWS                              | Mục đích sử dụng                       | Thông số                                         | Chi phí ước tính hàng tháng (USD) |
| ---------------------------------------- | -------------------------------------- | ------------------------------------------------ | --------------------------------- |
| EC2 (EKS Worker Nodes)                   | Chạy container ứng dụng                | 1 EC2 t3.medium (2 vCPU, 4GB RAM) x $0.0416/giờ  | 37                                |
| NAT Gateway                              | Truy cập internet cho subnet riêng tư  | 2 NAT Gateway x $0.045/giờ + băng thông          | 130                               |
| Amazon Route 53                          | DNS cho ứng dụng                       | 1 hosted zone                                    | 0.50                              |
| Amazon ECR                               | Lưu trữ Docker image                   | 100GB / tháng                                    | 10                                |
| Amazon EKS (không bao gồm EC2/NodeGroup) | Cluster control plane                  | 1 cluster - 4 nodes                              | 73                                |
| Amazon ECS - Fargate                     | Test container task                    | 10 task/ngày và duration 1 giờ                   | 37.49                             |
| AWS CodeBuild                            | Build Image/Container                  | 300 build/tháng                                  | 24                                |
| S3 (tùy chọn)                            | Lưu trữ build artifact/log             | 100 GB/ tháng                                    | 2.5                               |
| Amazon CloudWatch                        | Ghi log, theo dõi metric, gửi cảnh báo | báo Dùng cho Container Insights, dashboard, logs | 3                                 |

## Operational Costs

| Dịch vụ AWS        | Mục đích sử dụng                   | Thông số          | Chi phí ước tính hàng tháng (USD) |
| ------------------ | ---------------------------------- | ----------------- | --------------------------------- |
| CloudWatch Metrics | Theo dõi CPU, memory, logs ECS/EKS | 10 metrics /tháng | 3                                 |
| CloudWatch Logs    | Logs ECS, EKS, ứng dụng            | 20GB/tháng        | 14.09                             |
| CloudWatch Alarms  | Cảnh báo CPU, memory, error        | 20 alarm /tháng   | 2                                 |

**Monthly cost**: 19.09 USD
**12 months cost**: 229.08 USD
_Các dịch vụ khác có chi phí hàng tháng không đáng kể hoặc không tính phí_

## ROI Analysis

## 🏗️ Investment (Chi phí hạ tầng ban đầu - không bao gồm nhân công)

| Hạng mục chi phí   | Mô tả                                                                                 | Ước tính chi phí hàng tháng (USD) |
| ------------------ | ------------------------------------------------------------------------------------- | --------------------------------- |
| Hạ tầng AWS        | Chi phí cho cụm EKS, EC2, NAT Gateway, ECR, CodePipeline, Route53 và các LoadBalancer | 180~200                           |
| Chi phí monitoring | CloudWatch và SNS                                                                     | 35                                |

**Tổng chi phí**: 200~235 USD

## 💡 Return (Lợi ích mang lại)

- ⏱️ Rút ngắn chu kỳ phát hành sản phẩm:
  Từ trung bình 1 tuần xuống còn vài giờ với CI/CD

- 📈 Tăng khả năng mở rộng:
  Hỗ trợ auto scaling, sẵn sàng cho hàng triệu người dùng

- 🌐 Hỗ trợ IPv6 đầy đủ:
  Giúp tương thích với các thiết bị và mạng hiện đại, nhất là tại các thị trường quốc tế

- 💼 Giảm thiểu chi phí vận hành dài hạn:
  Nhờ tự động hóa, logging thông minh và cảnh báo sớm

- 🔒 Nâng cao bảo mật:
  IAM Role tách biệt, CI/CD audit log và network private subnet

# 6. Risk Assessment

## Risk Matrix

| Rủi ro                                                              | Mức độ ảnh hưởng | Khả năng xảy ra | Mức độ ưu tiên | Ghi chú                                                                                                                                    |
| ------------------------------------------------------------------- | ---------------- | --------------- | -------------- | ------------------------------------------------------------------------------------------------------------------------------------------ |
| ⚠️ EC2 hoặc Container Service bị quá tải                            | Cao              | Trung bình      | Cao            | Ứng dụng chạy trên EKS hoặc EC2 nếu không cấu hình auto-scaling hoặc giám sát tải sẽ dễ gặp bottleneck vào giờ cao điểm.                   |
| ⚠️ Rò rỉ dữ liệu từ MongoDB hoặc MongoDB Atlas                      | Cao              | Thấp            | Cao            | Nếu không cấu hình kết nối SSL/TLS hoặc đặt IAM policies sai, dữ liệu người dùng có thể bị truy cập trái phép qua internet.                |
| ⚠️ Lỗi trong pipeline CI/CD tự động đẩy code lỗi lên production     | Trung bình       | Cao             | Cao            | Nếu CodePipeline không có bước kiểm thử/staging trước khi deploy production, code lỗi có thể gây downtime hệ thống.                        |
| ⚠️ AWS SNS hoặc CloudWatch không cảnh báo kịp thời                  | Trung bình       | Trung bình      | Trung bình     | Nếu cảnh báo cấu hình sai (ví dụ alarm sai ngưỡng, hoặc SNS topic không hoạt động), đội ngũ không thể phản ứng kịp thời với sự cố.         |
| ⚠️Dịch vụ AWS (EKS, EC2, ALB, Route53) ngừng hoạt động đột ngột     | Rất cao          | Thấp            | Trung bình     | Rất hiếm khi xảy ra nhưng nếu có (do AWS outage, lỗi cấu hình vùng), toàn hệ thống có thể không khả dụng trong vài giờ.                    |
| ⚠️ Lỗi cấu hình NAT Gateway/Internet Gateway dẫn đến mất kết nối    | Cao              | Thấp            | Trung bình     | Thiết lập routing/NAT Gateway sai khiến ứng dụng không thể truy cập Internet hoặc dịch vụ bên ngoài (như MongoDB Atlas hoặc CodePipeline). |
| ⚠️Tấn công DDoS hoặc injection qua ALB/WAF không hoạt động hiệu quả | Rất cao          | Trung bình      | Cao            | Nếu WAF không cấu hình đúng hoặc thiếu rule bảo vệ, hệ thống có thể bị tấn công làm nghẽn mạng, hoặc bị chèn mã độc vào request đầu vào.   |

## Mitigation Strategies

| Rủi ro                           | Biện pháp giảm thiểu                                                                           |
| -------------------------------- | ---------------------------------------------------------------------------------------------- |
| Quá tải EC2/Container            | - Auto-scaling cho ECS/EKS<br>- Giám sát CloudWatch để cảnh báo trước<br>                      |
| Pipeline CI/CD bị gián đoạn      | - Thiết lập thông báo lỗi trong CodePipeline<br>- Thêm approval step thủ công trước production |
| Rò rỉ dữ liệu MongoDB            | - Mã hóa kết nối TLS <br>- Quản lý IAM/IAM Role nghiêm ngặt <br> -Bật audit logging            |
| AWS dịch vụ ngừng hoạt động      | - Triển khai đa AZ<br>- Backup định kỳ vào S3 hoặc RDS snapshot                                |
| Lỗi cảnh báo CloudWatch hoặc SNS | - Thiết lập test alert định kỳ<br>- Xác minh qua nhiều kênh cảnh báo (email + SNS)             |
| Tấn công mạng/WAF không hiệu quả | - Cấu hình WAF với rule phù hợp<br>- Dùng AWS Shield và KMS cho mã hóa lưu trữ                 |

## Contingency Plans

| Rủi ro                      | Kế hoạch ứng phó khi xảy ra sự cố                                              |
| --------------------------- | ------------------------------------------------------------------------------ |
| EC2 bị crash hoặc quá tải   | - Kích hoạt auto-scaling<br>- Có template EC2 AMI để khôi phục nhanh           |
| MongoDB Atlas mất kết nối   | - Sử dụng cluster dạng multi-region (nếu trả phí) <br> - Bật backup tự động    |
| DDoS làm nghẽn hệ thống     | - Kết hợp CloudFront caching <br> - AWS Shield Advanced (nếu cần hiệu quả cao) |
| Code lỗi đẩy lên production | - Rollback bằng image cũ trong ECR<br>- Dùng CodePipeline version history      |
| NAT Gateway gặp sự cố       | - Sử dụng nhiều NAT Gateway ở mỗi AZ<br>- Kiểm tra định kỳ routing table       |
| WAF rule sai                | - Gắn rule theo IP reputation <br> -Luôn có rule "count only" để test trước    |

# 7. Expected Outcomes

## 🎯 Success Metrics

- Tỷ lệ uptime hệ thống ≥ 99.9%: Đảm bảo ứng dụng hoạt động ổn định thông qua kiến trúc HA (High Availability).

- Thời gian triển khai (CI/CD) ≤ 3 phút/build: Tối ưu hóa pipeline với CodeBuild, ECR và CodePipeline.

- Giảm 30% chi phí truy cập mạng ra Internet: Nhờ tận dụng IPv6 không cần NAT Gateway cho một số trường hợp.

- 100% các pod container tuân thủ chuẩn bảo mật (non-root, TLS, IAM role): Kiểm soát quyền hạn và kết nối an toàn giữa các dịch vụ.

- Thời gian khôi phục khi có sự cố (RTO) ≤ 15 phút: Qua giám sát CloudWatch + cảnh báo SNS.

- Chạy benchmark hiệu năng tăng ≥ 20% so với hệ thống cũ IPv4-only: Nhờ mở rộng băng thông và giảm tải NAT

## 💼 Business Benefits

- Giảm thiểu downtime sản phẩm, giúp nâng cao trải nghiệm khách hàng và giữ chân người dùng.

- Tăng tốc độ ra mắt sản phẩm mới nhờ quy trình CI/CD tự động hóa và rollback an toàn.

- Tối ưu chi phí hạ tầng AWS, đặc biệt giảm chi phí data egress qua NAT Gateway nhờ dual-stack.

- Tuân thủ tiêu chuẩn bảo mật doanh nghiệp, giúp dễ dàng vượt qua kiểm toán nội bộ và bên thứ ba.

- Khả năng mở rộng đa vùng, đa khu vực, dễ dàng phục vụ người dùng quốc tế (hỗ trợ tốt hơn với IPv6).

## 🧠 Technical Improvements

- Chuyển từ monolithic sang microservices container hóa dễ quản lý, dễ scale từng thành phần.

- Tích hợp giám sát tập trung (CloudWatch, Container Insights) giúp chủ động xử lý lỗi, tối ưu hiệu suất.

- Tăng khả năng recover nhờ IaC và GitOps, có thể tái dựng hệ thống hạ tầng nhanh từ mã nguồn.

- Quản lý quyền truy cập chặt chẽ theo nguyên tắc least privilege thông qua IAM roles và policies.

- Hỗ trợ mạng kép IPv4/IPv6 giúp hệ thống hiện đại, bền vững và sẵn sàng cho tương lai.

## 🌱 Long-term Value

- Sẵn sàng tích hợp AI/ML hoặc Big Data về sau nhờ kiến trúc nền tảng mở rộng, hiện đại.

- Khả năng vận hành tự động cao, giảm phụ thuộc vào con người, dễ bảo trì về lâu dài.

- Khả năng chuyển vùng hoặc đa cloud (vendor-agnostic) khi cần mở rộng hoặc tối ưu chi phí.

- Cơ sở hạ tầng tương thích với các tiêu chuẩn cloud-native hiện đại, dễ dàng áp dụng CNCF tools như Prometheus, ArgoCD, Fluent Bit...

- Góp phần xây dựng văn hóa DevOps bền vững trong tổ chức, cải thiện năng suất đội ngũ kỹ thuật.

# Appendices

## A. Technical Specifications

## B. Cost Calculations

Các chi phí được tính toán dựa trên AWS Region Singapore (ap-southeast-1), các services được tính toán dựa trên các cấu hình đề xuất ở phần **5.Budget Estimation**
**Monthly cost**: 200~230 USD
**12 months cost**: 2,200 ~ 2,500 USD

> Region Singapore có mức phí trung bình, tùy theo các yêu cầu về vị trí cũng như độ trễ có thể sử dụng các region khác có thể ảnh hưởng đến chi phí tổng 8-10%

## C. Architecture Diagrams

**Kiến trúc hạ tầng đề xuất**
![Architecture](/submissions/nguyen-thanh-long/project-proposal/WorkShop_Architecture.png)

## D. References

1.  [Deploy MERN Stack](https://medium.com/@integrationninjas/deploy-mern-stack-on-aws-ec2-with-github-actions-ssl-setup-fc4702b76055)
2.  [AWS VPC IPv6 Support](https://docs.aws.amazon.com/vpc/latest/userguide/what-is-amazon-vpc.html)
3.  [AWS CodePipeline User Guide](https://docs.aws.amazon.com/codepipeline/latest/userguide/welcome.html)
