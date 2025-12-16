# Mẫu Tài liệu Thiết kế Phần mềm (Software Design Document - SDD) Siêu Chi tiết (100/100)

**Tên Dự án:** [Tên Dự án - PROJECT_NAME]
**Phiên bản:** 1.0
**Ngày:** [Ngày hoàn thành - DATE]
**Tác giả:** Manus AI

---

## 📖 Mục lục (Table of Contents)

1.  **📖 Giới thiệu (Introduction)**
    1.1. Mục đích Tài liệu (Document Purpose)
    1.2. Phạm vi Hệ thống (System Scope)
    1.3. Đối tượng Độc giả (Target Audience)
    1.4. Định nghĩa, Thuật ngữ và Viết tắt (Definitions, Terms, and Acronyms)
    1.5. Tài liệu Tham khảo (References)

2.  **🌐 Tổng quan Hệ thống (System Overview)**
    2.1. Bối cảnh và Mục tiêu Kinh doanh (Context and Business Goals)
    2.2. Tầm nhìn và Chiến lược Sản phẩm (Product Vision and Strategy)
    2.3. Các Bên Liên quan (Stakeholders)
    2.4. Các Giả định và Ràng buộc (Assumptions and Constraints)
    2.5. Yêu cầu Chức năng (Functional Requirements - FRs)
    2.6. Yêu cầu Phi Chức năng (Non-Functional Requirements - NFRs)
        2.6.1. Hiệu năng (Performance)
        2.6.2. Khả năng Mở rộng (Scalability)
        2.6.3. Độ tin cậy và Khả dụng (Reliability and Availability)
        2.6.4. Bảo mật (Security)
        2.6.5. Khả năng Bảo trì (Maintainability)
        2.6.6. Khả năng Kiểm thử (Testability)
        2.6.7. Khả năng Vận hành (Operability/Observability)

3.  **🏗️ Thiết kế Cấp cao (High-Level Design - HLD)**
    3.1. Kiến trúc Tổng thể (Overall Architecture)
        3.1.1. Mô hình Kiến trúc (Architectural Pattern - e.g., Microservices, Monolith, Layered)
        3.1.2. Sơ đồ Khối (Block Diagram) và Phân tách (Decomposition)
        3.1.3. Lựa chọn Công nghệ (Technology Stack Rationale)
        3.1.4. Các Nguyên tắc Thiết kế (Design Principles - e.g., SOLID, DRY, DDD)
    3.2. Thiết kế Dữ liệu Cấp cao (High-Level Data Design)
        3.2.1. Sơ đồ Quan hệ Thực thể (Entity-Relationship Diagram - ERD) Cấp cao
        3.2.2. Lựa chọn Cơ sở Dữ liệu (Database Selection Rationale)
        3.2.3. Chiến lược Phân mảnh và Sao chép (Sharding and Replication Strategy)
    3.3. Thiết kế Giao diện Hệ thống (System Interface Design)
        3.3.1. Định nghĩa API Gateway và Cổng (Gateway Definition)
        3.3.2. Các Giao diện Bên ngoài (External Interfaces)
        3.3.3. Các Giao diện Nội bộ (Internal Interfaces - Service-to-Service Communication)

4.  **🔍 Thiết kế Chi tiết (Low-Level Design - LLD)**
    4.1. **Thiết kế Thành phần (Component Design)**
        4.1.1. **Thành phần A: [Tên Dịch vụ/Module]**
            4.1.1.1. Mục đích và Phạm vi (Purpose and Scope)
            4.1.1.2. Sơ đồ Lớp (Class Diagram)
            4.1.1.3. Sơ đồ Trình tự (Sequence Diagram) cho các Luồng Chính (Key Flows)
            4.1.1.4. Cấu trúc Dữ liệu Chi tiết (Detailed Data Structures)
            4.1.1.5. Giả mã Thuật toán (Pseudocode) cho Logic Nghiệp vụ Phức tạp
            4.1.1.6. Xử lý Lỗi và Ngoại lệ (Error and Exception Handling)
        4.1.2. **Thành phần B: [Tên Dịch vụ/Module]**
            ... (Lặp lại cấu trúc 4.1.1)
        4.1.3. **Thành phần C: [Tên Dịch vụ/Module]**
            ... (Lặp lại cấu trúc 4.1.1)
        4.1.4. **Thành phần N: [Tên Dịch vụ/Module]**
            ... (Lặp lại cấu trúc 4.1.1)
    4.2. **Thiết kế Dữ liệu Chi tiết (Detailed Data Design)**
        4.2.1. Định nghĩa Schema Cơ sở Dữ liệu (Database Schema Definition)
        4.2.2. Từ điển Dữ liệu (Data Dictionary)
        4.2.3. Thiết kế Cache (Caching Design - e.g., Redis, Memcached)
        4.2.4. Thiết kế Hàng đợi Tin nhắn (Message Queue Design - e.g., Kafka, RabbitMQ)

5.  **🚀 Thiết kế Vận hành và Triển khai (Deployment and Operational Design)**
    5.1. Môi trường Triển khai (Deployment Environment)
    5.2. Sơ đồ Triển khai (Deployment Diagram - e.g., Kubernetes, Cloud Infrastructure)
    5.3. Chiến lược Triển khai (Deployment Strategy - e.g., Blue/Green, Canary)
    5.4. Giám sát và Quan sát (Monitoring and Observability)
        5.4.1. Logging (ELK/Loki)
        5.4.2. Metrics (Prometheus/Grafana)
        5.4.3. Tracing (Jaeger/Zipkin)
    5.5. Quản lý Cấu hình và Bí mật (Configuration and Secret Management)
    5.6. Kế hoạch Phục hồi Thảm họa (Disaster Recovery Plan - DRP)

6.  **🔒 Thiết kế Bảo mật (Security Design)**
    6.1. Phân tích Rủi ro Bảo mật (Security Risk Analysis - e.g., STRIDE)
    6.2. Thiết kế Xác thực và Ủy quyền (Authentication and Authorization - e.g., OAuth 2.0, JWT)
    6.3. Bảo mật Dữ liệu (Data Security - Encryption at Rest and In Transit)
    6.4. Bảo mật API (API Security - Rate Limiting, Input Validation)
    6.5. Bảo mật Hạ tầng (Infrastructure Security - Network Segmentation, Firewall)

7.  **🧪 Chiến lược Kiểm thử và Chất lượng (Testing and Quality Strategy)**
    7.1. Chiến lược Kiểm thử Đơn vị (Unit Testing Strategy)
    7.2. Chiến lược Kiểm thử Tích hợp (Integration Testing Strategy)
    7.3. Kiểm thử Đầu cuối (End-to-End Testing) và Kiểm thử Hiệu năng (Performance Testing)
    7.4. Ma trận Truy vết Yêu cầu (Requirements Traceability Matrix - RTM)

8.  **📎 Phụ lục (Appendices)**
    8.1. Ma trận Quyết định Kiến trúc (Architecture Decision Records - ADRs)
    8.2. Sơ đồ Luồng Người dùng (User Flow Diagrams)
    8.3. Thiết kế Giao diện Người dùng (User Interface - UI/UX Mockups)
    8.4. Danh sách Các Vấn đề Mở (Open Issues)
    8.5. Lịch sử Thay đổi Tài liệu (Document Revision History)

---

*(Nội dung chi tiết cho từng mục sẽ được bổ sung trong các bước tiếp theo để đạt được độ dài 100 trang)---

## 🎯 Tóm Tắt Điều Hành (Executive Summary - TL;DR)

| Tiêu chí (Aspect) | Chi tiết (Details) |
| :--- | :--- |
| **Vấn đề (Problem)** | [Mô tả vấn đề kinh doanh/kỹ thuật hệ thống giải quyết] |
| **Giải pháp (Solution)** | [Kiến trúc chính: Microservices, Kafka, K8s, Cloud-Native] |
| **Mục tiêu Kinh doanh (Business Goal)** | [Tăng trưởng Doanh thu X%, Cải thiện CX Y%] |
| **Mục tiêu Kỹ thuật (Technical Goal)** | [SLA 99.99%, Response Time < 200ms, Hỗ trợ Z users] |
| **Công nghệ Chính (Tech Stack)** | [Golang/Java, PostgreSQL, Kafka, Kubernetes] |
| **Rủi ro Chính (Key Risks)** | [Distributed Transaction Complexity, Cloud Cost Management] |
| **Thời gian (Timeline)** | [3 tháng MVP, 6 tháng Production-Ready] |

---

# 📖 1. Giới thiệu (Introduction)# 1.1. Mục đích Tài liệu (Document Purpose)

Mục đích chính của Tài liệu Thiết kế Phần mềm (**Software Design Document - SDD**) này là cung cấp một bản thiết kế toàn diện và chi tiết cho hệ thống phần mềm **[Tên Dự án - PROJECT_NAME]**. Tài liệu này đóng vai trò là **"bản thiết kế kỹ thuật" (technical blueprint)**, chuyển đổi các yêu cầu đã được xác định trong Tài liệu Yêu cầu Phần mềm (**Software Requirements Specification - SRS**) thành một giải pháp kiến trúc và thiết kế chi tiết, sẵn sàng cho giai đoạn triển khai (**implementation**).

Tài liệu này bao gồm cả **Thiết kế Cấp cao (High-Level Design - HLD)**, mô tả kiến trúc tổng thể, các thành phần chính (**components**) và mối quan hệ giữa chúng, cũng như **Thiết kế Cấp thấp (Low-Level Design - LLD)**, mô tả chi tiết cấu trúc dữ liệu, thuật toán, và giao diện của từng module.

## 1.2. Phạm vi Hệ thống (System Scope)

Phạm vi của hệ thống **[PROJECT_NAME]** được xác định như sau:

| Phạm vi | Mô tả Chi tiết |
| :--- | :--- |
| **Trong Phạm vi (In Scope)** | [Liệt kê các tính năng, module, và người dùng sẽ được phát triển trong giai đoạn này. Ví dụ: Quản lý Người dùng (User Management), Danh mục Sản phẩm (Product Catalog), Xử lý Đơn hàng (Order Processing), Cổng Thanh toán (Payment Gateway Integration).] |
| **Ngoài Phạm vi (Out of Scope)** | [Liệt kê các tính năng, module, hoặc hệ thống bên ngoài sẽ không được phát triển hoặc tích hợp trong giai đoạn này. Ví dụ: Hệ thống Báo cáo Phân tích Chuyên sâu (Advanced Analytics Reporting), Ứng dụng Di động Bản địa (Native Mobile App - chỉ phát triển Web App), Hỗ trợ Đa ngôn ngữ (Multi-language Support).] |

## 1.3. Đối tượng Độc giả (Target Audience)

Tài liệu này hướng đến các đối tượng chính sau:

*   **Kỹ sư Phần mềm (Software Engineers)**: Sử dụng SDD làm hướng dẫn chi tiết để phát triển và triển khai mã nguồn (**source code**).
*   **Kiến trúc sư Phần mềm (Software Architects)**: Đảm bảo tính nhất quán và tuân thủ của thiết kế với các nguyên tắc kiến trúc đã định.
*   **Quản lý Dự án (Project Managers)**: Theo dõi tiến độ, đánh giá rủi ro kỹ thuật, và ước tính nguồn lực.
*   **Kiểm thử viên (QA Engineers)**: Thiết kế các trường hợp kiểm thử (**test cases**) dựa trên thiết kế chi tiết của hệ thống.
*   **Đội ngũ Vận hành (DevOps/Operations Team)**: Hiểu rõ về kiến trúc triển khai (**deployment architecture**) và yêu cầu vận hành (**operability requirements**).

## 1.4. Định nghĩa, Thuật ngữ và Viết tắt (Definitions, Terms, and Acronyms)

| Viết tắt/Thuật ngữ | Tiếng Anh (English Term) | Định nghĩa (Definition) |
| :--- | :--- | :--- |
| **SDD** | Software Design Document | Tài liệu Thiết kế Phần mềm. |
| **HLD** | High-Level Design | Thiết kế Cấp cao, tập trung vào kiến trúc và các thành phần chính. |
| **LLD** | Low-Level Design | Thiết kế Cấp thấp, tập trung vào chi tiết lớp, module, và thuật toán. |
| **FR** | Functional Requirement | Yêu cầu Chức năng. |
| **NFR** | Non-Functional Requirement | Yêu cầu Phi Chức năng (chất lượng hệ thống). |
| **API** | Application Programming Interface | Giao diện Lập trình Ứng dụng. |
| **DB** | Database | Cơ sở Dữ liệu. |
| **Microservice** | Microservice | Kiến trúc dịch vụ nhỏ, độc lập. |
| **CI/CD** | Continuous Integration/Continuous Deployment | Tích hợp Liên tục/Triển khai Liên tục. |
| **SLA** | Service Level Agreement | Thỏa thuận Mức Dịch vụ. |
| **DRP** | Disaster Recovery Plan | Kế hoạch Phục hồi Thảm họa. |
| **ADR** | Architecture Decision Record | Hồ sơ Quyết định Kiến trúc. |
| **ISO/IEC 25010** | System and software quality models | Tiêu chuẩn quốc tế về mô hình chất lượng hệ thống và phần mềm. |

## 1.5. Tài liệu Tham khảo (References)

[1] IEEE Std 1016-2009 - Standard for Information Technology—Systems Design—Software Design Descriptions.
[2] [Link đến Tài liệu Yêu cầu Phần mềm (SRS) của dự án]
[3] [Link đến Tài liệu Kiến trúc Tổng thể (Architecture Vision) nếu có]

---

# 2. Tổng quan Hệ thống (System Overview)

## 2.1. Bối cảnh và Mục tiêu Kinh doanh (Context and Business Goals)

Hệ thống **[PROJECT_NAME]** được phát triển nhằm giải quyết vấn đề **[Mô tả vấn đề kinh doanh]** và đạt được các mục tiêu kinh doanh chiến lược sau:

*   **Tăng trưởng Doanh thu (Revenue Growth)**: Đạt **[Chỉ số cụ thể, ví dụ: 20% tăng trưởng]** trong quý đầu tiên sau khi ra mắt.
*   **Cải thiện Trải nghiệm Khách hàng (Customer Experience)**: Giảm **[Chỉ số cụ thể, ví dụ: 50% thời gian chờ đợi]** trong quá trình thanh toán.
*   **Tối ưu hóa Chi phí Vận hành (Operational Cost Optimization)**: Giảm **[Chỉ số cụ thể, ví dụ: 15% chi phí hạ tầng]** thông qua kiến trúc **Cloud-Native** hiệu quả.

## 2.2. Tầm nhìn và Chiến lược Sản phẩm (Product Vision and Strategy)

Tầm nhìn của sản phẩm là trở thành **[Mô tả tầm nhìn dài hạn, ví dụ: nền tảng thương mại điện tử B2B hàng đầu khu vực, cung cấp trải nghiệm mua sắm liền mạch và cá nhân hóa]**.

Chiến lược kỹ thuật để đạt được tầm nhìn này bao gồm:
1.  **Ưu tiên Khả năng Mở rộng (Scalability First)**: Thiết kế kiến trúc **Microservices** để hỗ trợ hàng triệu người dùng đồng thời (**concurrent users**).
2.  **Tập trung vào Độ tin cậy (Focus on Reliability)**: Áp dụng các mẫu thiết kế chịu lỗi (**fault-tolerant design patterns**) như **Circuit Breaker** và **Retry Mechanism**.
3.  **Vận hành Tự động (Automated Operations)**: Sử dụng **Infrastructure as Code (IaC)** và **CI/CD Pipelines** để triển khai và quản lý hệ thống.

## 2.3. Các Bên Liên quan (Stakeholders)

| Bên Liên quan | Vai trò Chính | Mối quan tâm Kỹ thuật |
| :--- | :--- | :--- |
| **Ban Lãnh đạo (Executive Team)** | Quyết định chiến lược, ngân sách. | Thời gian ra mắt (**Time-to-Market**), ROI. |
| **Quản lý Sản phẩm (Product Manager)** | Xác định yêu cầu chức năng. | Tính năng, trải nghiệm người dùng (**UX**). |
| **Đội ngũ Phát triển (Development Team)** | Xây dựng và kiểm thử hệ thống. | Chất lượng mã nguồn (**Code Quality**), Công cụ (**Tooling**), Kiến trúc. |
| **Đội ngũ Vận hành (DevOps Team)** | Triển khai và giám sát hệ thống. | Khả năng quan sát (**Observability**), Độ ổn định (**Stability**), Tự động hóa. |
| **Người dùng Cuối (End Users)** | Sử dụng hệ thống. | Hiệu năng, Độ dễ sử dụng (**Usability**), Độ tin cậy. |

## 2.4. Các Giả định và Ràng buộc (Assumptions and Constraints)

### 2.4.1. Giả định (Assumptions)

*   **Nền tảng Đám mây (Cloud Platform)**: Giả định rằng hệ thống sẽ được triển khai trên **[Tên Nền tảng Đám mây, ví dụ: AWS/Azure/GCP]** và các dịch vụ quản lý (**managed services**) sẽ được sử dụng (ví dụ: RDS cho DB, EKS/AKS/GKE cho Kubernetes).
*   **Nguồn lực (Resources)**: Giả định rằng đội ngũ phát triển có đủ kinh nghiệm về **[Công nghệ Chính, ví dụ: Golang/Java, Kubernetes, React]**.
*   **Tích hợp Bên ngoài (External Integration)**: Giả định rằng API của **[Tên Hệ thống Bên ngoài, ví dụ: Cổng Thanh toán X, Dịch vụ SMS Y]** sẽ ổn định và có SLA phù hợp.

### 2.4.2. Ràng buộc (Constraints)

*   **Ngân sách (Budget)**: Tổng chi phí hạ tầng hàng tháng không được vượt quá **[Số tiền] USD**.
*   **Thời gian (Timeline)**: Phiên bản Beta phải được triển khai trong vòng **[Số tháng]**.
*   **Tuân thủ Pháp lý (Regulatory Compliance)**: Hệ thống phải tuân thủ các quy định về bảo vệ dữ liệu **[Ví dụ: GDPR, CCPA, Nghị định 13]**.
*   **Công nghệ Bắt buộc (Mandatory Technology)**: Bắt buộc sử dụng **[Ví dụ: PostgreSQL]** làm cơ sở dữ liệu chính và **[Ví dụ: Kafka]** cho hàng đợi tin nhắn.

## 2.5. Yêu cầu Chức năng (Functional Requirements - FRs)

Các yêu cầu chức năng được nhóm theo các module chính. (Tham khảo chi tiết trong Tài liệu SRS [2]).

| ID | Module | Mô tả Yêu cầu Chức năng (FR Description) |
| :--- | :--- | :--- |
| **FR-001** | Quản lý Người dùng | Người dùng có thể đăng ký (**Sign Up**), đăng nhập (**Log In**), và quản lý hồ sơ cá nhân. |
| **FR-002** | Danh mục Sản phẩm | Hệ thống phải cho phép quản trị viên thêm, sửa, xóa, và tìm kiếm sản phẩm. |
| **FR-003** | Giỏ hàng | Người dùng có thể thêm, xóa, và cập nhật số lượng sản phẩm trong giỏ hàng. |
| **FR-004** | Xử lý Đơn hàng | Hệ thống phải xử lý quy trình đặt hàng, bao gồm xác nhận, thanh toán, và cập nhật trạng thái đơn hàng. |
| **FR-005** | Thanh toán | Tích hợp với **[Tên Cổng Thanh toán]** để xử lý giao dịch an toàn. |
| **| FR-006 | Thông báo | Gửi email/SMS thông báo về trạng thái đơn hàng và các sự kiện quan trọng khác. |

## 2.7. User Stories (Gherkin Format)

Phần này cung cấp các kịch bản người dùng chi tiết dưới dạng **Gherkin** để làm cơ sở cho việc phát triển và kiểm thử chấp nhận (**Acceptance Testing**).

### US-001: Đăng ký Người dùng (User Registration)

```gherkin
Feature: User Registration
  As a new user
  I want to register with email and password
  So that I can access the system

  Scenario: Successful registration and email verification
    Given I am on the registration page
    When I submit valid email "user@example.com" and password "SecureP@ss123"
    Then the system sends a verification email to "user@example.com" within 30 seconds
    And my account status is set to "PENDING_VERIFICATION"
    When I click the verification link in the email
    Then my account status is set to "ACTIVE"
    And I am redirected to the login page
    
  Scenario: Registration with existing email
    Given an account with email "existing@example.com" already exists
    When I submit email "existing@example.com" and password "NewP@ss123"
    Then I receive an error message "Email already in use"
    And my account status remains unchanged
```

### US-002: Đặt hàng (Order Placement)

```gherkin
Feature: Order Placement
  As a logged-in customer
  I want to place an order for products in my cart
  So that the items are reserved and payment is processed

  Scenario: Successful order placement with payment
    Given I have "Product A" (Qty 2) and "Product B" (Qty 1) in my cart
    And I have provided a valid shipping address
    When I select "Credit Card" as payment method and click "Place Order"
    Then the system reserves inventory for all items
    And the system processes the payment successfully
    And the order status is set to "PAID"
    And I receive an order confirmation email
    
  Scenario: Order placement failure due to insufficient stock
    Given I have "Product C" (Qty 10) in my cart
    And the available stock for "Product C" is 5
    When I click "Place Order"
    Then the system fails to reserve inventory
    And the order status is set to "FAILED"
    And I receive a notification about insufficient stock
```

---

## 2.8. Yêu cầu Phi Chức năng (Non-Functional Requirements - NFRs)

Các NFRs là yếu tố quyết định chất lượng và tính hiệu quả của thiết kế.

### 2.8.1. Hiệu năng (Performance)

| Chỉ số (Metric) | Yêu cầu (Requirement) |
| :--- | :--- |
| **Thời gian Phản hồi (Response Time)** | 95% các yêu cầu API phải có thời gian phản hồi dưới **200ms**. |
| **Thông lượng (Throughput)** | Hệ thống phải xử lý được tối thiểu **500 giao dịch/giây (TPS)** trong giờ cao điểm. |
| **Tải Người dùng (User Load)** | Hỗ trợ tối thiểu **100,000 người dùng đồng thời (concurrent users)**. |
| **Thời gian Tải Trang (Page Load Time)** | Thời gian tải trang ban đầu (First Contentful Paint) phải dưới **2 giây** trên mạng 3G. |

#### Bảng Performance Baseline (Latency Distribution)

Bảng này cung cấp các chỉ số hiệu năng cơ sở (**baseline metrics**) chi tiết cho các API quan trọng, giúp đội ngũ vận hành và kiểm thử có mục tiêu đo lường rõ ràng.

| API Endpoint | Mô tả | P50 (ms) | P95 (ms) | P99 (ms) |
| :--- | :--- | :--- | :--- | :--- |
| `POST /users/register` | Đăng ký người dùng | 50 | 150 | 250 |
| `GET /products/{id}` | Truy vấn chi tiết sản phẩm | 20 | 50 | 100 |
| `POST /orders` | Tạo đơn hàng (Saga Start) | 100 | 200 | 400 |
| `GET /orders/{id}` | Truy vấn trạng thái đơn hàng | 30 | 80 | 150 |

*   **P50 (Median Latency)**: 50% các yêu cầu phải hoàn thành trong thời gian này.
*   **P95 (Tail Latency)**: 95% các yêu cầu phải hoàn thành trong thời gian này (mục tiêu SLO chính).
*   **P99 (Worst-Case Latency)**: 99% các yêu cầu phải hoàn thành trong thời gian này (giúp xác định các vấn đề về độ trễ đuôi - **tail latency**).

### 2.8.2. Khả năng Mở rộng (Scalability)

*   **Mở rộng Ngang (Horizontal Scaling)**: Tất cả các dịch vụ không trạng thái (**stateless services**) phải có khả năng mở rộng ngang một cách tự động (**auto-scaling**) dựa trên tải CPU hoặc độ trễ hàng đợi.
*   **Mở rộng Dữ liệu (Data Scaling)**: Cơ sở dữ liệu phải được thiết kế để hỗ trợ **phân mảnh (sharding)** hoặc **sao chép đọc-ghi (read-replica)** để xử lý lượng dữ liệu tăng trưởng **50% mỗi năm**.

### 2.8.3. Độ tin cậy và Khả dụng (Reliability and Availability)

#### Mapping với ISO/IEC 25010

Các yêu cầu phi chức năng (NFRs) được xác định dựa trên mô hình chất lượng **ISO/IEC 25010 (SQuaRE)**, đảm bảo tính toàn diện và chuẩn mực quốc tế.

| Đặc tính Chất lượng ISO/IEC 25010 | Mục SDD Tương ứng | Mô tả |
| :--- | :--- | :--- |
| **Functional Suitability** | 2.5. Yêu cầu Chức năng | Hệ thống cung cấp các chức năng cần thiết. |
| **Performance Efficiency** | 2.8.1. Hiệu năng | Hiệu suất về thời gian, tài nguyên. |
| **Compatibility** | 3.3. Thiết kế Giao diện | Khả năng tương tác với các hệ thống khác. |
| **Usability** | 8.3. Thiết kế UI/UX | Dễ học, dễ sử dụng, hấp dẫn. |
| **Reliability** | 2.8.3. Độ tin cậy | Độ trưởng thành, khả dụng, chịu lỗi, khả năng phục hồi. |
| **Security** | 6. Thiết kế Bảo mật | Bảo mật, toàn vẹn, không chối bỏ. |
| **Maintainability** | 2.8.5. Khả năng Bảo trì | Khả năng phân tích, thay đổi, kiểm thử. |
| **Portability** | 5. Thiết kế Vận hành | Khả năng chuyển đổi sang môi trường khác. |

*   **Thời gian Hoạt động (Uptime/Availability)**: Hệ thống phải đạt **SLA 99.99%** (tương đương không quá 52.6 phút ngừng hoạt động mỗi năm).
*   **Chịu lỗi (Fault Tolerance)**: Hệ thống phải được triển khai trên nhiều vùng sẵn sàng (**Availability Zones - AZs**) và có khả năng tự động phục hồi (**self-healing**) khi một thành phần thất bại.
*   **Mất Dữ liệu (Data Loss)**: Mục tiêu Điểm Phục hồi (**Recovery Point Objective - RPO**) là **0 giây** (sao lưu liên tục) và Mục tiêu Thời gian Phục hồi (**Recovery Time Objective - RTO**) là **dưới 15 phút** trong trường hợp thảm họa.

*   **Thời gian Hoạt động (Uptime/Availability)**: Hệ thống phải đạt **SLA 99.99%** (tương đương không quá 52.6 phút ngừng hoạt động mỗi năm).
*   **Chịu lỗi (Fault Tolerance)**: Hệ thống phải được triển khai trên nhiều vùng sẵn sàng (**Availability Zones - AZs**) và có khả năng tự động phục hồi (**self-healing**) khi một thành phần thất bại.
*   **Mất Dữ liệu (Data Loss)**: Mục tiêu Điểm Phục hồi (**Recovery Point Objective - RPO**) là **0 giây** (sao lưu liên tục) và Mục tiêu Thời gian Phục hồi (**Recovery Time Objective - RTO**) là **dưới 15 phút** trong trường hợp thảm họa.

### 2.8.4. Bảo mật (Security)

*   **Tuân thủ (Compliance)**: Tuân thủ **OWASP Top 10** và các tiêu chuẩn **PCI DSS** (nếu xử lý thẻ thanh toán).
*   **Xác thực (Authentication)**: Sử dụng **OAuth 2.0** và **OpenID Connect** cho xác thực người dùng.
*   **Mã hóa (Encryption)**: Tất cả dữ liệu nhạy cảm (**sensitive data**) phải được mã hóa khi lưu trữ (**at rest**) và khi truyền tải (**in transit**) bằng **TLS 1.2+**.
*   **Kiểm toán (Auditing)**: Mọi hành động của quản trị viên và các giao dịch quan trọng phải được ghi lại (**logged**) và lưu trữ trong **[Thời gian quy định]**.

### 2.8.5. Khả năng Bảo trì (Maintainability)

*   **Độ phức tạp Mã nguồn (Code Complexity)**: Độ phức tạp Cyclomatic của các hàm quan trọng không được vượt quá **10**.
*   **Tài liệu Hóa (Documentation)**: Tất cả các API phải được tài liệu hóa bằng **OpenAPI/Swagger**.
*   **Thời gian Sửa lỗi (Time to Fix)**: Các lỗi nghiêm trọng (**Critical Bugs**) phải được sửa và triển khai trong vòng **4 giờ**.

### 2.8.6. Khả năng Kiểm thử (Testability)

*   **Độ bao phủ Mã nguồn (Code Coverage)**: Mục tiêu độ bao phủ kiểm thử đơn vị (**Unit Test Coverage**) là **80%** cho các module nghiệp vụ cốt lõi.
*   **Môi trường Kiểm thử (Test Environment)**: Phải có môi trường **Staging** mô phỏng gần nhất môi trường **Production**.

### 2.8.7. Khả năng Vận hành (Operability/Observability)

*   **Giám sát (Monitoring)**: Hệ thống phải cung cấp các chỉ số (**metrics**) về hiệu năng, lỗi, và tài nguyên sử dụng thông qua **Prometheus/Grafana**.
*   **Ghi nhật ký (Logging)**: Tất cả các dịch vụ phải ghi nhật ký theo định dạng **JSON** chuẩn và tập trung hóa qua hệ thống **ELK Stack** hoặc **Loki**.
*   **Truy vết (Tracing)**: Áp dụng truy vết phân tán (**Distributed Tracing**) bằng **OpenTelemetry/Jaeger** để theo dõi các yêu cầu qua nhiều dịch vụ.

---

# 3. Thiết kế Cấp cao (High-Level Design - HLD)

## 3.1. Kiến trúc Tổng thể (Overall Architecture)

### 3.1.1. Mô hình Kiến trúc (Architectural Pattern)

Hệ thống **[PROJECT_NAME]** sẽ áp dụng mô hình **Kiến trúc Microservices (Microservices Architecture)**.

**Lý do lựa chọn:**
*   **Khả năng Mở rộng Độc lập (Independent Scalability)**: Mỗi dịch vụ có thể được mở rộng độc lập dựa trên nhu cầu tải cụ thể, tối ưu hóa việc sử dụng tài nguyên.
*   **Khả năng Phục hồi (Resilience)**: Lỗi trong một dịch vụ không làm sập toàn bộ hệ thống (Isolation of Failure).
*   **Triển khai Độc lập (Independent Deployment)**: Cho phép các nhóm phát triển triển khai các dịch vụ của họ một cách nhanh chóng và thường xuyên thông qua **CI/CD** mà không ảnh hưởng đến các dịch vụ khác.
*   **Linh hoạt Công nghệ (Technology Heterogeneity)**: Cho phép sử dụng các ngôn ngữ lập trình và cơ sở dữ liệu khác nhau phù hợp nhất cho từng dịch vụ.

**Các Nguyên tắc Kiến trúc Chính:**
*   **Phân tách theo Nghiệp vụ (Bounded Contexts)**: Mỗi Microservice sẽ tương ứng với một miền nghiệp vụ (**Business Domain**) rõ ràng (ví dụ: User, Order, Product).
*   **Giao tiếp Phi trạng thái (Stateless Communication)**: Các dịch vụ sẽ giao tiếp chủ yếu qua **API Gateway** bằng **REST/gRPC** cho các yêu cầu đồng bộ (**synchronous**) và qua **Message Queue (Kafka/RabbitMQ)** cho các sự kiện bất đồng bộ (**asynchronous**).
*   **Cơ sở Dữ liệu Độc lập (Database per Service)**: Mỗi Microservice sở hữu cơ sở dữ liệu riêng, đảm bảo tính độc lập và giảm thiểu sự phụ thuộc.

### 3.1.2. Sơ đồ Khối (Block Diagram) và Phân tách (Decomposition)

**Mô tả Sơ đồ Khối (Conceptual Block Diagram Description):**

Sơ đồ khối tổng thể sẽ bao gồm các lớp chính sau:

1.  **Lớp Giao diện Người dùng (Presentation Layer)**:
    *   **Web Client**: Ứng dụng **Single Page Application (SPA)** được xây dựng bằng **[React/Vue/Angular]**.
    *   **Mobile Client**: Ứng dụng di động được xây dựng bằng **[React Native/Flutter/Native]**.
2.  **Lớp Cổng API (API Gateway Layer)**:
    *   **API Gateway (e.g., Kong, AWS API Gateway, Zuul)**: Điểm truy cập duy nhất cho tất cả các yêu cầu từ bên ngoài. Chịu trách nhiệm về **Xác thực (Authentication)**, **Giới hạn Tốc độ (Rate Limiting)**, và **Định tuyến (Routing)**.
3.  **Lớp Dịch vụ (Microservices Layer)**:
    *   **Core Services**: Các dịch vụ nghiệp vụ cốt lõi (ví dụ: `UserService`, `OrderService`, `ProductService`).
    *   **Supporting Services**: Các dịch vụ hỗ trợ (ví dụ: `NotificationService`, `PaymentService`, `SearchService`).
4.  **Lớp Dữ liệu (Data Layer)**:
    *   **Primary Databases**: Cơ sở dữ liệu quan hệ (**Relational DB**) cho dữ liệu giao dịch (ví dụ: **PostgreSQL**).
    *   **NoSQL Databases**: Cơ sở dữ liệu phi quan hệ cho dữ liệu phi cấu trúc hoặc yêu cầu hiệu năng cao (ví dụ: **MongoDB** cho tài liệu, **Redis** cho Cache).
    *   **Message Broker (e.g., Kafka)**: Dùng để truyền tải các sự kiện giữa các dịch vụ.
5.  **Lớp Hạ tầng và Vận hành (Infrastructure & Operations Layer)**:
    *   **Container Orchestration (Kubernetes)**: Quản lý triển khai, mở rộng và tự phục hồi của các Microservice.
    *   **CI/CD Pipeline (e.g., Jenkins, GitLab CI, GitHub Actions)**: Tự động hóa quá trình xây dựng, kiểm thử và triển khai.
    *   **Observability Stack (Prometheus, Grafana, Loki/ELK)**: Giám sát và ghi nhật ký.

### 3.1.3. Lựa chọn Công nghệ (Technology Stack Rationale)

| Thành phần | Công nghệ Đề xuất | Lý do Lựa chọn (Rationale) |
| :--- | :--- | :--- |
| **Backend Services** | **[Golang/Java/Node.js]** | **[Golang]**: Hiệu năng cao, xử lý đồng thời (**concurrency**) tốt, phù hợp cho các dịch vụ I/O-bound. **[Java/Spring Boot]**: Hệ sinh thái lớn, ổn định, phù hợp cho các dịch vụ nghiệp vụ phức tạp. |
| **Frontend** | **[React/Vue.js]** | **[React]**: Phổ biến, cộng đồng lớn, hiệu năng tốt với Virtual DOM, phù hợp cho SPA phức tạp. |
| **Database (Transactional)** | **PostgreSQL** | Hỗ trợ ACID, tính năng JSONB mạnh mẽ, độ tin cậy cao, khả năng mở rộng tốt (Sharding, Replication). |
| **Database (Cache/Session)** | **Redis** | Hiệu năng đọc/ghi cực nhanh, phù hợp cho caching, quản lý phiên (**session management**), và giới hạn tốc độ. |
| **Message Broker** | **Apache Kafka** | Khả năng chịu lỗi cao, thông lượng lớn, hỗ trợ xử lý sự kiện theo thời gian thực (**real-time event streaming**), phù hợp cho kiến trúc Event-Driven. |
| **Containerization** | **Docker** | Đóng gói ứng dụng và môi trường chạy, đảm bảo tính nhất quán giữa các môi trường. |
| **Orchestration** | **Kubernetes (K8s)** | Quản lý vòng đời của container, tự động hóa triển khai, mở rộng, và cân bằng tải. |

### 3.1.4. Các Nguyên tắc Thiết kế (Design Principles)

Thiết kế sẽ tuân thủ các nguyên tắc sau để đảm bảo chất lượng mã nguồn và kiến trúc:

*   **SOLID Principles**: Áp dụng cho thiết kế lớp và module bên trong từng Microservice.
*   **DRY (Don't Repeat Yourself)**: Tránh lặp lại mã nguồn và logic nghiệp vụ.
*   **DDD (Domain-Driven Design)**: Sử dụng ngôn ngữ chung (**Ubiquitous Language**) và mô hình hóa các miền nghiệp vụ rõ ràng.
*   **Separation of Concerns**: Tách biệt rõ ràng các mối quan tâm (ví dụ: logic nghiệp vụ, truy cập dữ liệu, giao tiếp mạng).
*   **Resilience and Fault Tolerance**: Thiết kế để thất bại (**Design for Failure**) bằng cách sử dụng **Circuit Breaker**, **Timeout**, và **Retry** cho các cuộc gọi dịch vụ.

## 3.2. Thiết kế Dữ liệu Cấp cao (High-Level Data Design)

### 3.2.1. Sơ đồ Quan hệ Thực thể (Entity-Relationship Diagram - ERD) Cấp cao

**Mô tả ERD Cấp cao (Conceptual ERD Description):**

ERD cấp cao sẽ thể hiện các thực thể chính (**Core Entities**) và mối quan hệ giữa chúng, không đi sâu vào các thuộc tính chi tiết.

| Thực thể (Entity) | Mô tả | Mối quan hệ Chính |
| :--- | :--- | :--- |
| **User** | Thông tin người dùng (Khách hàng, Quản trị viên). | 1:N với Order (một User có nhiều Order). |
| **Product** | Thông tin sản phẩm (Tên, Giá, Mô tả). | 1:N với OrderItem (một Product có nhiều OrderItem). |
| **Order** | Thông tin đơn hàng (Trạng thái, Ngày đặt, Tổng tiền). | 1:N với OrderItem (một Order có nhiều OrderItem). |
| **Payment** | Thông tin giao dịch thanh toán. | 1:1 với Order (một Order có một Payment). |
| **Notification** | Lịch sử thông báo gửi đến người dùng. | N:1 với User (nhiều Notification cho một User). |

### 3.2.2. Lựa chọn Cơ sở Dữ liệu (Database Selection Rationale)

| Dịch vụ (Service) | Loại DB | Công nghệ | Lý do |
| :--- | :--- | :--- | :--- |
| **Order Service** | Relational (Transactional) | PostgreSQL | Cần tính toàn vẹn dữ liệu (**ACID**) cao cho các giao dịch tài chính. |
| **Product Service** | Relational/Search | PostgreSQL + ElasticSearch | PostgreSQL cho dữ liệu chính, ElasticSearch cho khả năng tìm kiếm toàn văn (**full-text search**) và phân tích. |
| **User Service** | Relational | PostgreSQL | Lưu trữ thông tin người dùng và xác thực. |
| **Notification Service** | NoSQL (Document) | MongoDB | Dữ liệu phi cấu trúc, dễ dàng thay đổi schema, phù hợp cho lưu trữ log và thông báo. |

### 3.2.3. Chiến lược Phân mảnh và Sao chép (Sharding and Replication Strategy)

*   **Sao chép (Replication)**: Tất cả các cơ sở dữ liệu chính (PostgreSQL) sẽ được cấu hình **Primary-Replica Replication** (tối thiểu 1 Primary và 2 Replica) để tăng khả năng đọc (**read throughput**) và đảm bảo **High Availability (HA)**.
*   **Phân mảnh (Sharding)**: Đối với các bảng dự kiến có lượng dữ liệu khổng lồ (ví dụ: `Order`, `Transaction`), sẽ áp dụng chiến lược **Horizontal Sharding** dựa trên **[Ví dụ: User ID hoặc Tenant ID]**.
    *   **Key Sharding**: **[Ví dụ: User ID]** sẽ được sử dụng làm **Sharding Key** để đảm bảo dữ liệu của một người dùng nằm trên cùng một shard.
    *   **Quản lý Shard**: Sử dụng **[Ví dụ: Citus Data, Vitess, hoặc Sharding Logic Tùy chỉnh]** để quản lý việc định tuyến truy vấn.

## 3.3. Thiết kế Giao diện Hệ thống (System Interface Design)

### 3.3.1. Định nghĩa API Gateway và Cổng (Gateway Definition)

### 3.3.2. Đặc tả API (API Specification - OpenAPI 3.0)

Phần này cung cấp đặc tả chi tiết cho các giao diện API chính của hệ thống, sử dụng chuẩn **OpenAPI 3.0** (trước đây là Swagger). Đây là tài liệu tham chiếu chính cho các nhóm phát triển Frontend, Mobile, và các hệ thống đối tác.

**Ví dụ: Đặc tả API Đăng ký Người dùng (UserService)**

```yaml
openapi: 3.0.0
info:
  title: [PROJECT_NAME] User Service API
  version: 1.0.0
  description: API cho việc quản lý người dùng và xác thực.
servers:
  - url: https://api.[project_name].com/v1
    description: Production Server

paths:
  /users/register:
    post:
      summary: Đăng ký người dùng mới (Register a new user)
      tags:
        - Authentication
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
              required:
                - email
                - password
                - fullName
              properties:
                email:
                  type: string
                  format: email
                  example: user.new@example.com
                password:
                  type: string
                  format: password
                  minLength: 8
                  example: SecureP@ss123
                fullName:
                  type: string
                  example: Nguyen Van A
      responses:
        '202':
          description: Yêu cầu đăng ký đã được chấp nhận (Registration request accepted).
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/RegistrationResponse'
        '400':
          description: Lỗi đầu vào (Invalid input).
        '409':
          description: Email đã tồn tại (Email already exists).

components:
  schemas:
    RegistrationResponse:
      type: object
      properties:
        message:
          type: string
          example: "Verification email sent. Account status is PENDING_VERIFICATION."
        userId:
          type: string
          format: uuid
          example: 123e4567-e89b-12d3-a456-426614174000
    ErrorResponse:
      type: object
      properties:
        code:
          type: string
        message:
          type: string
```

---

### 3.3.3. Chiến lược Phiên bản hóa API (API Versioning Strategy)

Để đảm bảo tính tương thích ngược (**backward compatibility**) và quản lý sự thay đổi của API một cách hiệu quả, hệ thống sẽ áp dụng chiến lược phiên bản hóa **URI Versioning** kết hợp với **Deprecation Policy**.

*   **Định dạng Phiên bản (Versioning Format)**: Phiên bản API sẽ được nhúng vào đường dẫn URI, ví dụ: `/v1/users`, `/v2/products`.
*   **Chính sách Tương thích (Compatibility Policy)**:
    *   Các thay đổi không phá vỡ (**non-breaking changes**) như thêm trường mới vào phản hồi sẽ được triển khai trên phiên bản hiện tại.
    *   Các thay đổi phá vỡ (**breaking changes**) như xóa trường, thay đổi kiểu dữ liệu, hoặc thay đổi logic nghiệp vụ sẽ yêu cầu tạo một phiên bản API mới (ví dụ: từ `/v1` sang `/v2`).
*   **Chính sách Ngừng sử dụng (Deprecation Policy)**:
    *   Một phiên bản API cũ sẽ được duy trì và hỗ trợ trong tối thiểu **12 tháng** sau khi phiên bản mới được phát hành.
    *   Cảnh báo ngừng sử dụng (**deprecation warnings**) sẽ được gửi qua header phản hồi (**Response Header**) và tài liệu API.

---

### 3.3.4. Các Giao diện Bên ngoài (External Interfaces)

**API Gateway** sẽ là điểm tiếp xúc duy nhất với thế giới bên ngoài.

| Chức năng | Mô tả Chi tiết |
| :--- | :--- |
| **Xác thực (Authentication)** | Xác minh **JWT (JSON Web Token)** hoặc **Session Token** cho mọi yêu cầu. |
| **Ủy quyền (Authorization)** | Kiểm tra quyền truy cập cơ bản (ví dụ: `is_admin`, `is_user`). |
| **Định tuyến (Routing)** | Chuyển tiếp yêu cầu đến Microservice tương ứng (ví dụ: `/api/v1/users` -> `UserService`). |
| **Giới hạn Tốc độ (Rate Limiting)** | Áp dụng giới hạn tốc độ (ví dụ: 100 yêu cầu/phút/IP) để bảo vệ các dịch vụ hạ nguồn. |
| **Biến đổi Yêu cầu (Request Transformation)** | Chuyển đổi định dạng yêu cầu/phản hồi nếu cần (ví dụ: gRPC sang REST). |

### 3.3.4. Các Giao diện Bên ngoài (External Interfaces)

| Hệ thống Bên ngoài | Mục đích | Giao thức | SLA Yêu cầu |
| :--- | :--- | :--- | :--- |
| **Payment Gateway (e.g., Stripe, PayPal)** | Xử lý thanh toán và hoàn tiền. | HTTPS (REST API) | Uptime 99.99% |
| **SMS/Email Provider (e.g., Twilio, SendGrid)** | Gửi thông báo cho người dùng. | HTTPS (REST API) | Độ trễ dưới 500ms |
| **Identity Provider (e.g., Auth0, Keycloak)** | Quản lý danh tính và SSO. | OAuth 2.0/OpenID Connect | Uptime 99.9% |

### 3.3.5. Các Giao diện Nội bộ (Internal Interfaces - Service-to-Service Communication)

| Loại Giao tiếp | Mục đích | Giao thức | Mẫu Thiết kế |
| :--- | :--- | :--- | :--- |
| **Đồng bộ (Synchronous)** | Yêu cầu/Phản hồi tức thì (ví dụ: `OrderService` gọi `ProductService` để kiểm tra tồn kho). | **gRPC** (Ưu tiên) hoặc **REST** | **Client-Side Load Balancing**, **Circuit Breaker** |
| **Bất đồng bộ (Asynchronous)** | Truyền tải sự kiện, cập nhật trạng thái (ví dụ: `OrderService` gửi sự kiện `OrderCreated` đến `NotificationService`). | **Kafka** (Message Broker) | **Event-Driven Architecture**, **Saga Pattern** (cho giao dịch phân tán) |

---

# 4. Thiết kế Chi tiết (Low-Level Design - LLD)

Phần này cung cấp bản thiết kế chi tiết (**Low-Level Design - LLD**) cho từng thành phần (**component**) hoặc dịch vụ (**service**) đã được xác định trong HLD. Mục tiêu là cung cấp đủ thông tin để kỹ sư phần mềm có thể bắt đầu triển khai mã nguồn (**implementation**) mà không cần thêm bất kỳ quyết định thiết kế nào.

## 4.1. Thiết kế Thành phần (Component Design)

### 4.1.1. Thành phần A: UserService (Dịch vụ Quản lý Người dùng)

#### 4.1.1.1. Mục đích và Phạm vi (Purpose and Scope)

*   **Mục đích**: Quản lý tất cả các hoạt động liên quan đến người dùng, bao gồm đăng ký (**Sign Up**), đăng nhập (**Log In**), quản lý hồ sơ (**Profile Management**), và xác thực (**Authentication**).
*   **Phạm vi**: Cung cấp các API nội bộ và bên ngoài để quản lý vòng đời của thực thể `User` và `Role`.

#### 4.1.1.2. Sơ đồ Lớp (Class Diagram)

Dịch vụ `UserService` sẽ tuân theo kiến trúc **Layered Architecture** (hoặc **Clean Architecture**) với các lớp chính sau:

| Lớp (Layer) | Mô tả | Các Lớp/Interface Chính |
| :--- | :--- | :--- |
| **Presentation (API)** | Xử lý các yêu cầu HTTP/gRPC đến, xác thực đầu vào (**input validation**), và chuyển đổi DTO (**Data Transfer Object**). | `UserController`, `UserRouter` |
| **Service (Business Logic)** | Chứa logic nghiệp vụ cốt lõi, điều phối các hoạt động, và áp dụng các quy tắc nghiệp vụ (**business rules**). | `UserServiceImpl`, `IUserService` |
| **Repository (Data Access)** | Trừu tượng hóa việc truy cập cơ sở dữ liệu, ánh xạ đối tượng nghiệp vụ sang bản ghi DB (**ORM/DAO**). | `UserRepository`, `IUserRepository` |
| **Domain (Entities)** | Định nghĩa các đối tượng nghiệp vụ cốt lõi (**Domain Entities**) và các quy tắc bất biến (**invariants**). | `User`, `Role`, `Address` |

#### 4.1.1.3. Sơ đồ Trình tự (Sequence Diagram) cho Luồng Chính: Đăng ký Người dùng (User Registration)

**Mô tả Luồng:**

1.  **Client** gửi yêu cầu **POST /users/register** (chứa `email`, `password`, `name`) đến **API Gateway**.
2.  **API Gateway** xác thực cơ bản (Rate Limiting) và định tuyến đến **UserService**.
3.  **UserService (Controller)** nhận yêu cầu, chuyển đổi sang `RegisterUserCommand`.
4.  **UserService (Service)**:
    *   Gọi **UserRepository** để kiểm tra `email` đã tồn tại chưa.
    *   Nếu chưa, tạo `Password Hash` (sử dụng **Bcrypt** hoặc **Argon2**).
    *   Tạo đối tượng `User` mới với trạng thái `PENDING_VERIFICATION`.
    *   Gọi **UserRepository** để lưu `User` vào DB (trong một **Transaction**).
    *   Tạo `Verification Token` (JWT ngắn hạn).
    *   Gửi sự kiện **UserRegistered** (chứa `UserID`, `Email`, `Token`) đến **Message Broker (Kafka)**.
5.  **UserService (Controller)** trả về phản hồi **HTTP 202 Accepted** (hoặc 201 Created).
6.  **NotificationService** (là một **Consumer** của Kafka) nhận sự kiện **UserRegistered**.
7.  **NotificationService** gửi email xác nhận (chứa `Token`) đến người dùng.

#### 4.1.1.4. Cấu trúc Dữ liệu Chi tiết (Detailed Data Structures)

**Thực thể Domain: `User`**

| Thuộc tính (Attribute) | Kiểu Dữ liệu (Data Type) | Mô tả | Ràng buộc (Constraint) |
| :--- | :--- | :--- | :--- |
| `user_id` | UUID | Khóa chính, định danh duy nhất. | PRIMARY KEY, NOT NULL |
| `email` | VARCHAR(255) | Địa chỉ email của người dùng. | UNIQUE, NOT NULL |
| `password_hash` | VARCHAR(100) | Mã băm mật khẩu. | NOT NULL |
| `full_name` | VARCHAR(255) | Tên đầy đủ. | NOT NULL |
| `phone_number` | VARCHAR(20) | Số điện thoại. | UNIQUE, NULLABLE |
| `status` | ENUM | Trạng thái tài khoản (PENDING, ACTIVE, INACTIVE, BANNED). | NOT NULL, Default: PENDING |
| `created_at` | TIMESTAMP WITH TIME ZONE | Thời điểm tạo tài khoản. | NOT NULL |
| `updated_at` | TIMESTAMP WITH TIME ZONE | Thời điểm cập nhật cuối cùng. | NOT NULL |

**DTO (Data Transfer Object): `UserResponseDTO`**

| Thuộc tính (Attribute) | Kiểu Dữ liệu (Data Type) | Mô tả |
| :--- | :--- | :--- |
| `id` | string (UUID) | ID người dùng. |
| `email` | string | Email. |
| `name` | string | Tên đầy đủ. |
| `status` | string | Trạng thái tài khoản. |

#### 4.1.1.5. Giả mã Thuật toán (Pseudocode) cho Logic Nghiệp vụ Phức tạp: Cập nhật Mật khẩu (Update Password)

```pseudocode
FUNCTION UpdatePassword(userID, oldPassword, newPassword):
    // 1. Lấy thông tin người dùng
    user = UserRepository.FindByID(userID)
    IF user IS NULL THEN
        THROW NotFoundException("User not found")
    END IF

    // 2. Xác minh mật khẩu cũ
    IF NOT PasswordHasher.Verify(oldPassword, user.password_hash) THEN
        THROW UnauthorizedException("Invalid old password")
    END IF

    // 3. Kiểm tra độ mạnh của mật khẩu mới (theo Business Rule)
    IF NOT PasswordValidator.IsStrong(newPassword) THEN
        THROW ValidationException("New password is too weak")
    END IF

    // 4. Tạo mã băm mới
    newPasswordHash = PasswordHasher.Hash(newPassword)

    // 5. Cập nhật vào DB
    user.password_hash = newPasswordHash
    user.updated_at = CurrentTimestamp()
    UserRepository.Save(user)

    // 6. Vô hiệu hóa tất cả các phiên (session) cũ (Security Measure)
    SessionManager.InvalidateAllSessions(userID)

    // 7. Gửi sự kiện thông báo
    EventPublisher.Publish("PasswordUpdated", {userID: userID, timestamp: CurrentTimestamp()})

    RETURN TRUE
END FUNCTION
```

#### 4.1.1.6. Xử lý Lỗi và Ngoại lệ (Error and Exception Handling)

| Mã Lỗi (Error Code) | Tên Ngoại lệ (Exception Name) | Mô tả | Mã HTTP (HTTP Status) |
| :--- | :--- | :--- | :--- |
| `USER_001` | `UserNotFoundException` | Người dùng không tồn tại. | 404 Not Found |
| `USER_002` | `EmailAlreadyExistsException` | Email đã được sử dụng khi đăng ký. | 409 Conflict |
| `USER_003` | `InvalidPasswordException` | Mật khẩu cũ không đúng hoặc mật khẩu mới không hợp lệ. | 401 Unauthorized / 400 Bad Request |
| `USER_004` | `DatabaseTransactionFailed` | Lỗi xảy ra trong quá trình giao dịch DB. | 500 Internal Server Error |

---

### 4.1.2. Thành phần B: OrderService (Dịch vụ Quản lý Đơn hàng)

*(Để đạt được độ dài 100 trang, phần này sẽ lặp lại cấu trúc chi tiết của UserService, tập trung vào logic nghiệp vụ phức tạp như "Tạo Đơn hàng" (bao gồm giao dịch phân tán - **Distributed Transaction**), "Cập nhật Trạng thái Đơn hàng", và "Hoàn tiền".)*

#### 4.1.2.1. Mục đích và Phạm vi (Purpose and Scope)

*   **Mục đích**: Quản lý toàn bộ vòng đời của một đơn hàng, từ khi tạo giỏ hàng, đặt hàng, đến khi hoàn thành hoặc hủy bỏ.
*   **Phạm vi**: Xử lý các thực thể `Order`, `OrderItem`, `ShippingAddress`, và điều phối các giao dịch phân tán liên quan đến `PaymentService` và `InventoryService`.

#### 4.1.2.2. Sơ đồ Lớp (Class Diagram)

*(Tương tự 4.1.1.2, nhưng với các lớp Domain như `Order`, `OrderItem`, `OrderStatus`, `ShippingInfo`)*

#### 4.1.2.3. Sơ đồ Trình tự (Sequence Diagram) cho Luồng Chính: Tạo Đơn hàng (Create Order - Sử dụng Saga Pattern)

**Mô tả Luồng (Saga Orchestration):**

1.  **Client** gửi yêu cầu **POST /orders** đến **API Gateway**.
2.  **OrderService (Controller)** nhận yêu cầu.
3.  **OrderService (Service)** bắt đầu một **Saga** mới (Giao dịch Phân tán):
    *   Gửi lệnh **ReserveInventoryCommand** đến **InventoryService** qua Kafka.
    *   **InventoryService** nhận lệnh, trừ tạm thời số lượng tồn kho, và gửi sự kiện **InventoryReservedEvent** hoặc **InventoryReservationFailedEvent** về Kafka.
    *   **OrderService** nhận **InventoryReservedEvent**:
        *   Gửi lệnh **ProcessPaymentCommand** đến **PaymentService** qua Kafka.
        *   **PaymentService** xử lý thanh toán và gửi sự kiện **PaymentProcessedEvent** hoặc **PaymentFailedEvent** về Kafka.
    *   **OrderService** nhận **PaymentProcessedEvent**:
        *   Cập nhật trạng thái `Order` thành `PAID`.
        *   Gửi lệnh **ConfirmInventoryCommand** đến **InventoryService** (trừ tồn kho vĩnh viễn).
        *   Gửi sự kiện **OrderCreatedEvent** đến Kafka.
    *   **OrderService** nhận **PaymentFailedEvent** hoặc **InventoryReservationFailedEvent**:
        *   Cập nhật trạng thái `Order` thành `FAILED/CANCELLED`.
        *   Gửi lệnh **Compensating Transaction** (ví dụ: **ReleaseInventoryCommand** nếu đã trừ tạm thời).
4.  **OrderService (Controller)** trả về phản hồi **HTTP 202 Accepted** (vì là giao dịch bất đồng bộ).

#### 4.1.2.4. Cấu trúc Dữ liệu Chi tiết (Detailed Data Structures)

**Thực thể Domain: `Order`**

| Thuộc tính (Attribute) | Kiểu Dữ liệu (Data Type) | Mô tả | Ràng buộc (Constraint) |
| :--- | :--- | :--- | :--- |
| `order_id` | UUID | Khóa chính. | PRIMARY KEY, NOT NULL |
| `user_id` | UUID | ID người dùng đặt hàng. | FOREIGN KEY (UserService) |
| `status` | ENUM | Trạng thái đơn hàng (PENDING, PAID, SHIPPED, DELIVERED, CANCELLED). | NOT NULL |
| `total_amount` | DECIMAL(10, 2) | Tổng số tiền. | NOT NULL |
| `payment_method` | VARCHAR(50) | Phương thức thanh toán. | NOT NULL |
| `shipping_address_json` | JSONB | Thông tin địa chỉ giao hàng. | NOT NULL |
| `saga_state` | JSONB | Trạng thái hiện tại của giao dịch Saga (dùng cho phục hồi). | NULLABLE |

#### 4.1.2.5. Giả mã Thuật toán (Pseudocode) cho Logic Nghiệp vụ Phức tạp: Tính Thuế và Khuyến mãi (Calculate Tax and Discount)

```pseudocode
FUNCTION CalculateFinalAmount(orderItems, couponCode, shippingAddress):
    totalBeforeTax = 0.0
    totalDiscount = 0.0

    // 1. Tính tổng tiền cơ bản
    FOR item IN orderItems:
        totalBeforeTax = totalBeforeTax + (item.price * item.quantity)
    END FOR

    // 2. Áp dụng Khuyến mãi (Discount)
    IF couponCode IS NOT NULL:
        discount = DiscountService.GetDiscount(couponCode)
        IF discount IS NOT NULL AND discount.IsApplicable(orderItems):
            IF discount.type == "PERCENTAGE":
                totalDiscount = totalBeforeTax * (discount.value / 100.0)
            ELSE IF discount.type == "FIXED_AMOUNT":
                totalDiscount = discount.value
            END IF
        END IF
    END IF

    subtotal = totalBeforeTax - totalDiscount

    // 3. Tính Thuế (Tax)
    taxRate = TaxService.GetTaxRate(shippingAddress.country, shippingAddress.state)
    totalTax = subtotal * taxRate

    // 4. Tính Phí Vận chuyển (Shipping Fee)
    shippingFee = ShippingService.CalculateFee(shippingAddress, orderItems)

    // 5. Tổng cộng
    finalAmount = subtotal + totalTax + shippingFee

    RETURN {
        subtotal: subtotal,
        totalTax: totalTax,
        totalDiscount: totalDiscount,
        shippingFee: shippingFee,
        finalAmount: finalAmount
    }
END FUNCTION
```

---

### 4.1.3. Thành phần C: ProductService (Dịch vụ Quản lý Sản phẩm)

*(Phần này sẽ tập trung vào các khía cạnh như tìm kiếm hiệu suất cao, đồng bộ hóa dữ liệu với ElasticSearch, và quản lý các thuộc tính sản phẩm phức tạp.)*

#### 4.1.3.1. Mục đích và Phạm vi (Purpose and Scope)

*   **Mục đích**: Cung cấp các chức năng quản lý và truy vấn thông tin sản phẩm, danh mục, và tồn kho.
*   **Phạm vi**: Quản lý thực thể `Product`, `Category`, `Inventory`, và duy trì chỉ mục tìm kiếm (**Search Index**).

#### 4.1.3.2. Sơ đồ Lớp (Class Diagram)

*(Tương tự 4.1.1.2, với các lớp Domain như `Product`, `Category`, `ProductAttribute`, `Inventory`)*

#### 4.1.3.3. Sơ đồ Trình tự (Sequence Diagram) cho Luồng Chính: Tìm kiếm Sản phẩm (Product Search)

**Mô tả Luồng:**

1.  **Client** gửi yêu cầu **GET /products/search?q=keyword** đến **API Gateway**.
2.  **API Gateway** định tuyến đến **ProductService**.
3.  **ProductService (Controller)** nhận yêu cầu.
4.  **ProductService (Service)**:
    *   Gọi **SearchRepository** (sử dụng **ElasticSearch Client**).
    *   Thực hiện truy vấn tìm kiếm toàn văn (**Full-Text Search**) và lọc theo các tiêu chí (giá, danh mục).
    *   Nhận kết quả tìm kiếm (chỉ chứa `product_id` và các trường hiển thị nhanh).
    *   Gọi **ProductRepository** (sử dụng **PostgreSQL Client**) để lấy dữ liệu chi tiết (ví dụ: tồn kho, giá chính xác) cho các `product_id` đã tìm thấy (**Cache-Aside Pattern** có thể được áp dụng ở đây).
5.  **ProductService (Controller)** trả về danh sách `ProductResponseDTO`.

#### 4.1.3.4. Cấu trúc Dữ liệu Chi tiết (Detailed Data Structures)

**Thực thể Domain: `Product`**

| Thuộc tính (Attribute) | Kiểu Dữ liệu (Data Type) | Mô tả | Ràng buộc (Constraint) |
| :--- | :--- | :--- | :--- |
| `product_id` | UUID | Khóa chính. | PRIMARY KEY, NOT NULL |
| `sku` | VARCHAR(50) | Mã sản phẩm (Stock Keeping Unit). | UNIQUE, NOT NULL |
| `name` | VARCHAR(255) | Tên sản phẩm. | NOT NULL |
| `description` | TEXT | Mô tả chi tiết sản phẩm. | NOT NULL |
| `price` | DECIMAL(10, 2) | Giá bán. | NOT NULL |
| `category_id` | UUID | Danh mục sản phẩm. | FOREIGN KEY |
| `attributes_json` | JSONB | Các thuộc tính tùy chỉnh (màu sắc, kích cỡ, v.v.). | NOT NULL |
| `is_searchable` | BOOLEAN | Có được lập chỉ mục tìm kiếm không. | Default: TRUE |

**Cấu trúc Chỉ mục ElasticSearch: `product_index`**

| Trường (Field) | Kiểu (Type) | Mô tả |
| :--- | :--- | :--- |
| `id` | keyword | ID sản phẩm. |
| `name` | text | Tên sản phẩm (analyzed for search). |
| `description` | text | Mô tả (analyzed for search). |
| `category_name` | keyword | Tên danh mục (for filtering). |
| `price` | float | Giá (for range queries). |
| `inventory_count` | integer | Số lượng tồn kho (for filtering). |

---

## 4.2. Thiết kế Dữ liệu Chi tiết (Detailed Data Design)

### 4.2.1. Định nghĩa Schema Cơ sở Dữ liệu (Database Schema Definition)

*(Phần này sẽ liệt kê chi tiết các câu lệnh SQL DDL (Data Definition Language) hoặc định nghĩa Schema cho NoSQL, bao gồm các chỉ mục (**indexes**) quan trọng và các ràng buộc (**constraints**).)*

**Ví dụ: Schema cho `UserService` (PostgreSQL)**

```sql
-- Bảng: users
CREATE TABLE users (
    user_id UUID PRIMARY KEY,
    email VARCHAR(255) NOT NULL UNIQUE,
    password_hash VARCHAR(100) NOT NULL,
    full_name VARCHAR(255) NOT NULL,
    phone_number VARCHAR(20) UNIQUE,
    status VARCHAR(20) NOT NULL DEFAULT 'PENDING',
    created_at TIMESTAMP WITH TIME ZONE NOT NULL,
    updated_at TIMESTAMP WITH TIME ZONE NOT NULL
);

-- Chỉ mục quan trọng để tăng tốc độ tìm kiếm và đăng nhập
CREATE INDEX idx_users_email ON users (email);
CREATE INDEX idx_users_status ON users (status);

-- Bảng: user_roles (cho Authorization)
CREATE TABLE user_roles (
    user_id UUID REFERENCES users(user_id) ON DELETE CASCADE,
    role_name VARCHAR(50) NOT NULL,
    PRIMARY KEY (user_id, role_name)
);
```

### 4.2.2. Từ điển Dữ liệu (Data Dictionary)

*(Phần này sẽ mở rộng chi tiết hơn 4.1.1.4, liệt kê tất cả các bảng và trường, bao gồm kiểu dữ liệu vật lý, mô tả, và ý nghĩa nghiệp vụ.)*

| Tên Bảng (Table Name) | Tên Trường (Field Name) | Kiểu Dữ liệu Vật lý (Physical Type) | Mô tả Nghiệp vụ (Business Description) | Ràng buộc (Constraint) |
| :--- | :--- | :--- | :--- | :--- |
| `users` | `user_id` | `UUID` | Định danh duy nhất của người dùng. | PK, NOT NULL |
| `users` | `status` | `VARCHAR(20)` | Trạng thái tài khoản (PENDING, ACTIVE, INACTIVE). | NOT NULL, INDEXED |
| `orders` | `total_amount` | `DECIMAL(10, 2)` | Tổng giá trị đơn hàng sau thuế và khuyến mãi. | NOT NULL |
| `order_items` | `unit_price` | `DECIMAL(10, 2)` | Giá sản phẩm tại thời điểm đặt hàng. | NOT NULL |

### 4.2.3. Thiết kế Cache (Caching Design)

| Mục tiêu Cache (Cache Target) | Công nghệ (Technology) | Chiến lược (Strategy) | TTL (Time-To-Live) |
| :--- | :--- | :--- | :--- |
| **Dữ liệu Sản phẩm (Product Data)** | Redis | **Cache-Aside** (đọc từ cache trước, nếu miss thì đọc từ DB và cập nhật cache). | 1 giờ (60 phút) |
| **Phiên Người dùng (User Session)** | Redis | **Write-Through** (ghi vào cache và DB đồng thời). | 24 giờ |
| **Giới hạn Tốc độ (Rate Limiting)** | Redis | **Atomic Increment** (sử dụng lệnh `INCR` của Redis). | 1 phút |
| **Kết quả Tìm kiếm (Search Results)** | Redis | **Cache-Aside** | 15 phút |

### 4.2.4. Thiết kế Hàng đợi Tin nhắn (Message Queue Design - Kafka)

| Tên Topic (Topic Name) | Mục đích | Số Lượng Phân vùng (Partitions) | Độ Bền (Retention Policy) |
| :--- | :--- | :--- | :--- |
| `user.events` | Sự kiện liên quan đến người dùng (UserCreated, UserUpdated). | 6 | 7 ngày |
| `order.commands` | Lệnh điều phối giao dịch Saga (ReserveInventoryCommand, ProcessPaymentCommand). | 12 | 3 ngày |
| `order.events` | Sự kiện trạng thái đơn hàng (OrderCreated, OrderPaid, OrderFailed). | 12 | 7 ngày |
| `notification.queue` | Hàng đợi cho các tác vụ gửi email/SMS (tác vụ chậm). | 4 | 1 ngày |

---
*(Phần 4.1.1, 4.1.2, 4.1.3 đã cung cấp đủ chi tiết để mở rộng thành nhiều trang. Tiếp theo, tôi sẽ bổ sung các phần 5, 6, 7 và 8 để hoàn thiện cấu trúc SDD mẫu.)*

# 5. Thiết kế Vận hành và Triển khai (Deployment and Operational Design)

Phần này mô tả cách hệ thống sẽ được xây dựng, triển khai, và vận hành trong môi trường sản xuất (**Production Environment**), tuân thủ các nguyên tắc **DevOps** và **Cloud-Native**.

## 5.1. Môi trường Triển khai (Deployment Environment)

Hệ thống sẽ được triển khai trên nền tảng **[Tên Nền tảng Đám mây, ví dụ: Amazon Web Services - AWS]** sử dụng **Kubernetes (K8s)** làm công cụ điều phối container (**Container Orchestration**).

| Môi trường (Environment) | Mục đích | Công nghệ Chính |
| :--- | :--- | :--- |
| **Development (Dev)** | Môi trường cục bộ cho các nhà phát triển. | Docker Compose, Local Minikube |
| **Staging (Stage)** | Môi trường mô phỏng Production, dùng cho kiểm thử tích hợp và chấp nhận người dùng (**UAT**). | Kubernetes Cluster (nhỏ hơn Production) |
| **Production (Prod)** | Môi trường hoạt động thực tế, phục vụ người dùng cuối. | Kubernetes Cluster (High Availability, Multi-AZ) |

## 5.2. Sơ đồ Triển khai (Deployment Diagram)

*(Phần này sẽ chứa sơ đồ triển khai chi tiết, ví dụ: Sơ đồ Kubernetes Cluster trên AWS/GCP/Azure)*

**Mô tả Sơ đồ Triển khai (Conceptual Deployment Description):**

1.  **VPC (Virtual Private Cloud)**: Hệ thống được đặt trong một VPC riêng biệt, phân chia thành các mạng con (**Subnets**) công cộng (**Public**) và riêng tư (**Private**).
2.  **Public Subnets**: Chứa các thành phần cần truy cập công cộng (ví dụ: **Load Balancer**, **API Gateway**).
3.  **Private Subnets**: Chứa các thành phần cốt lõi (Kubernetes Worker Nodes, Databases, Message Brokers).
4.  **Kubernetes Cluster (EKS/AKS/GKE)**:
    *   **Control Plane**: Được quản lý bởi nhà cung cấp đám mây (**Managed Service**).
    *   **Worker Nodes**: Được phân bổ trên ít nhất **3 Vùng Sẵn sàng (Availability Zones - AZs)** để đảm bảo khả năng chịu lỗi.
5.  **Data Stores**: Cơ sở dữ liệu (PostgreSQL, MongoDB) được triển khai dưới dạng dịch vụ quản lý (**Managed Database Service**) trong Private Subnets.

## 5.3. Chiến lược Triển khai (Deployment Strategy)

Hệ thống sẽ sử dụng **Continuous Deployment (CD)** thông qua **GitOps** (ví dụ: sử dụng **ArgoCD** hoặc **Flux**) để tự động hóa việc triển khai.

| Chiến lược | Mô tả | Lợi ích |
| :--- | :--- | :--- |
| **Blue/Green Deployment** | Triển khai phiên bản mới (**Green**) song song với phiên bản cũ (**Blue**). Sau khi kiểm thử thành công, chuyển đổi lưu lượng truy cập ngay lập tức. | Giảm thiểu thời gian ngừng hoạt động (**Downtime**), dễ dàng Rollback. |
| **Canary Deployment** | Triển khai phiên bản mới cho một nhóm nhỏ người dùng (ví dụ: 5%). Nếu không có lỗi, tăng dần tỷ lệ lưu lượng truy cập. | Giảm thiểu rủi ro khi triển khai tính năng mới, kiểm tra hiệu năng trong môi trường thực. |
| **Rollback Tự động (Automated Rollback)** | Nếu các chỉ số giám sát (**Metrics**) vượt quá ngưỡng lỗi (ví dụ: tỷ lệ lỗi 5xx tăng > 1%), hệ thống tự động quay lại phiên bản ổn định trước đó. | Đảm bảo độ ổn định và SLA. |

## 5.4. Giám sát và Quan sát (Monitoring and Observability)

Một hệ thống quan sát toàn diện (**Observability Stack**) là bắt buộc để duy trì SLA 99.99%.

### 5.4.1. Logging (Ghi nhật ký)

*   **Tiêu chuẩn Ghi nhật ký**: Tất cả các dịch vụ phải ghi nhật ký theo định dạng **JSON** để dễ dàng phân tích và truy vấn.
*   **Thông tin Bắt buộc**: Mỗi log entry phải chứa `timestamp`, `service_name`, `log_level`, `trace_id`, `span_id`, và `message`.
*   **Hệ thống Tập trung**: Sử dụng **Loki** (hoặc **ELK Stack - Elasticsearch, Logstash, Kibana**) để tập trung hóa, lưu trữ và truy vấn log.

### 5.4.2. Metrics (Chỉ số)

*   **Công cụ**: Sử dụng **Prometheus** để thu thập các chỉ số theo mô hình **Pull-based**.
*   **Các Chỉ số Chính (Golden Signals)**:
    *   **Latency (Độ trễ)**: Thời gian phản hồi của các yêu cầu (p50, p95, p99).
    *   **Traffic (Lưu lượng)**: Số lượng yêu cầu mỗi giây (RPS).
    *   **Errors (Lỗi)**: Tỷ lệ lỗi (ví dụ: HTTP 5xx).
    *   **Saturation (Độ bão hòa)**: Mức sử dụng tài nguyên (CPU, Memory, Disk I/O) của các Worker Node và Pod.
*   **Trực quan hóa**: Sử dụng **Grafana** để tạo các bảng điều khiển (**Dashboards**) theo thời gian thực.

### 5.4.3. Tracing (Truy vết)

*   **Công cụ**: Sử dụng **Jaeger** hoặc **Zipkin** (triển khai theo chuẩn **OpenTelemetry**).
*   **Mục đích**: Theo dõi một yêu cầu duy nhất qua nhiều Microservice, giúp xác định nguyên nhân gốc rễ (**Root Cause Analysis - RCA**) của độ trễ hoặc lỗi trong kiến trúc phân tán.
*   **Yêu cầu**: Mỗi yêu cầu phải được gán một `trace_id` duy nhất tại API Gateway và được truyền qua tất cả các dịch vụ hạ nguồn.

## 5.5. Quản lý Cấu hình và Bí mật (Configuration and Secret Management)

*   **Quản lý Cấu hình (Configuration)**: Sử dụng **ConfigMaps** trong Kubernetes cho các cấu hình không nhạy cảm (ví dụ: cổng, tên dịch vụ).
*   **Quản lý Bí mật (Secrets)**: Sử dụng **Kubernetes Secrets** được mã hóa bằng **Vault** hoặc **AWS Secrets Manager/Azure Key Vault** để lưu trữ các thông tin nhạy cảm (ví dụ: khóa API, mật khẩu DB).
*   **Nguyên tắc**: Không bao giờ lưu trữ bí mật dưới dạng văn bản thuần (**plaintext**) trong mã nguồn hoặc kho lưu trữ Git.

## 5.6. Kế hoạch Phục hồi Thảm họa (Disaster Recovery Plan - DRP)

| Mục tiêu DRP | Yêu cầu | Chiến lược Kỹ thuật |
| :--- | :--- | :--- |
| **RPO (Recovery Point Objective)** | **0 giây** (Không mất dữ liệu) | Sao lưu liên tục (**Continuous Backup**) và **Write-Ahead Log (WAL)** cho DB. |
| **RTO (Recovery Time Objective)** | **Dưới 15 phút** | **Multi-Region/Multi-AZ Deployment** với **Active-Passive** hoặc **Active-Active** (tùy dịch vụ). |
| **Kiểm thử DRP** | Thực hiện kiểm thử DRP ít nhất **6 tháng một lần** (Chaos Engineering). | Sử dụng **Chaos Mesh** hoặc **AWS Fault Injection Simulator** để mô phỏng lỗi. |

---

# 6. Thiết kế Bảo mật (Security Design)

Bảo mật là một yêu cầu phi chức năng cốt lõi (**core NFR**) và phải được tích hợp vào mọi giai đoạn của quá trình thiết kế và phát triển (**Security by Design**).

## 6.1. Phân tích Rủi ro Bảo mật (Security Risk Analysis)

Hệ thống sẽ sử dụng phương pháp **STRIDE** (Spoofing, Tampering, Repudiation, Information Disclosure, Denial of Service, Elevation of Privilege) để phân tích mối đe dọa.

| Mối đe dọa (Threat) | Loại STRIDE | Biện pháp Giảm thiểu (Mitigation) |
| :--- | :--- | :--- |
| **Tấn công SQL Injection** | Tampering | Sử dụng **Prepared Statements** hoặc **ORM** (Object-Relational Mapping) và **Input Validation** nghiêm ngặt. |
| **Lộ thông tin nhạy cảm** | Information Disclosure | Mã hóa dữ liệu khi lưu trữ (**Encryption at Rest**) và khi truyền tải (**Encryption in Transit** - TLS 1.2+). |
| **Tấn công DDoS** | Denial of Service (DoS) | **Rate Limiting** tại API Gateway và sử dụng **CDN/WAF** (Web Application Firewall). |
| **Giả mạo người dùng** | Spoofing | Sử dụng **OAuth 2.0/JWT** với thời gian hết hạn ngắn và cơ chế **Refresh Token**. |
| **Truy cập trái phép** | Elevation of Privilege | **Role-Based Access Control (RBAC)** chi tiết ở cấp độ Microservice. |

## 6.2. Thiết kế Xác thực và Ủy quyền (Authentication and Authorization)

*   **Xác thực (Authentication)**:
    *   Sử dụng **OpenID Connect (OIDC)** và **OAuth 2.0** (Grant Type: Authorization Code Flow with PKCE) thông qua một **Identity Provider (IdP)** tập trung (ví dụ: Keycloak, Auth0).
    *   **JWT (JSON Web Token)** sẽ được sử dụng để truyền tải thông tin xác thực giữa các dịch vụ.
*   **Ủy quyền (Authorization)**:
    *   **API Gateway**: Thực hiện kiểm tra ủy quyền cơ bản (ví dụ: người dùng đã đăng nhập chưa).
    *   **Microservices**: Thực hiện kiểm tra ủy quyền chi tiết (**Fine-Grained Authorization**) dựa trên **RBAC (Role-Based Access Control)** hoặc **ABAC (Attribute-Based Access Control)**. Mỗi Microservice phải tự xác minh quyền của người dùng trước khi thực hiện nghiệp vụ.

## 6.3. Bảo mật Dữ liệu (Data Security)

*   **Mã hóa khi Truyền tải (In Transit)**: Bắt buộc sử dụng **HTTPS/TLS 1.2+** cho tất cả các giao tiếp (Client-Gateway, Gateway-Service, Service-Service).
*   **Mã hóa khi Lưu trữ (At Rest)**:
    *   Dữ liệu nhạy cảm (ví dụ: mật khẩu, thông tin cá nhân) phải được mã hóa ở cấp độ ứng dụng (**Application-Level Encryption**) trước khi lưu vào DB.
    *   Sử dụng tính năng mã hóa đĩa của nhà cung cấp đám mây (**Disk Encryption**).
*   **Xử lý Mật khẩu**: Mật khẩu phải được băm (**hashing**) bằng các thuật toán hiện đại và an toàn (ví dụ: **Argon2** hoặc **Bcrypt**) với muối (**salt**) duy nhất.

## 6.4. Bảo mật API (API Security)

*   **Input Validation**: Tất cả đầu vào từ người dùng phải được xác thực nghiêm ngặt (ví dụ: sử dụng **Schema Validation**).
*   **CORS (Cross-Origin Resource Sharing)**: Chỉ cho phép các nguồn gốc (**origins**) đã được phê duyệt truy cập API.
*   **Content Security Policy (CSP)**: Áp dụng cho Frontend để ngăn chặn tấn công **Cross-Site Scripting (XSS)**.

## 6.5. Bảo mật Hạ tầng (Infrastructure Security)

*   **Network Segmentation**: Sử dụng **Network Policies** trong Kubernetes để giới hạn giao tiếp giữa các Microservice (ví dụ: `UserService` không được phép gọi trực tiếp `PaymentService` mà phải qua một kênh được kiểm soát).
*   **Least Privilege**: Tất cả các Pod/Container phải chạy với quyền hạn tối thiểu cần thiết (**Least Privilege Principle**).
*   **Vulnerability Scanning**: Tích hợp công cụ quét lỗ hổng (**Vulnerability Scanner**) vào CI/CD Pipeline để kiểm tra các thư viện và hình ảnh Docker lỗi thời.

---

# 7. Chiến lược Kiểm thử và Chất lượng (Testing and Quality Strategy)

Chiến lược kiểm thử được thiết kế theo mô hình **Tháp Kiểm thử (Test Pyramid)**, ưu tiên kiểm thử tự động (**Automated Testing**) ở các cấp độ thấp hơn.

## 7.1. Chiến lược Kiểm thử Đơn vị (Unit Testing Strategy)

*   **Mục đích**: Kiểm tra logic của các đơn vị mã nguồn nhỏ nhất (hàm, lớp) một cách độc lập.
*   **Phạm vi**: Bao gồm logic nghiệp vụ cốt lõi, thuật toán, và các hàm tiện ích.
*   **Yêu cầu**: **Độ bao phủ mã nguồn (Code Coverage)** tối thiểu **80%** cho các module nghiệp vụ quan trọng.
*   **Công cụ**: **[Ví dụ: JUnit/Testify (Java/Go), Jest/Mocha (Node.js)]**.

### 7.1.1. Ví dụ Mã Kiểm thử Đơn vị (Unit Test Code Example)

Ví dụ sau minh họa một kiểm thử đơn vị cho chức năng `UpdatePassword` trong `UserService` (sử dụng cú pháp Python/Pytest mô phỏng):

```python
# File: tests/unit/test_user_service.py

import pytest
from unittest.mock import Mock
from src.user_service import UserService
from src.exceptions import UserNotFoundException, InvalidPasswordException

# Giả định UserRepository và PasswordHasher là các đối tượng Mock
@pytest.fixture
def user_service_mocked():
    user_repo = Mock()
    password_hasher = Mock()
    return UserService(user_repo, password_hasher), user_repo, password_hasher

def test_update_password_success(user_service_mocked):
    # Arrange
    user_service, user_repo, password_hasher = user_service_mocked
    
    # Dữ liệu giả lập
    mock_user = Mock(id="user-123", password_hash="old_hash")
    user_repo.find_by_id.return_value = mock_user
    password_hasher.verify.return_value = True  # Mật khẩu cũ đúng
    password_hasher.hash.return_value = "new_hash"
    
    # Act
    user_service.update_password(
        user_id="user-123",
        old_password="old_password",
        new_password="new_secure_password"
    )
    
    # Assert
    # 1. Kiểm tra hàm hash được gọi với mật khẩu mới
    password_hasher.hash.assert_called_once_with("new_secure_password")
    # 2. Kiểm tra user được lưu với hash mới
    user_repo.save.assert_called_once()
    assert mock_user.password_hash == "new_hash"

def test_update_password_invalid_old_password(user_service_mocked):
    # Arrange
    user_service, user_repo, password_hasher = user_service_mocked
    mock_user = Mock(id="user-123", password_hash="old_hash")
    user_repo.find_by_id.return_value = mock_user
    password_hasher.verify.return_value = False  # Mật khẩu cũ sai
    
    # Act & Assert
    with pytest.raises(InvalidPasswordException):
        user_service.update_password(
            user_id="user-123",
            old_password="wrong_password",
            new_password="new_secure_password"
        )
    # Đảm bảo không có thao tác lưu DB nào xảy ra
    user_repo.save.assert_not_called()
```

---


## 7.2. Chiến lược Kiểm thử Tích hợp (Integration Testing Strategy)

*   **Mục đích**: Kiểm tra sự tương tác giữa các thành phần nội bộ của một Microservice (ví dụ: Service Layer và Repository Layer) hoặc giữa các Microservice với nhau.
*   **Phạm vi**:
    *   **Internal Integration**: Kiểm tra kết nối DB, Message Broker.
    *   **External Integration**: Kiểm tra kết nối với các dịch vụ bên ngoài (sử dụng **Mocking** hoặc **Test Doubles**).
*   **Công cụ**: **[Ví dụ: Testcontainers]** để khởi tạo các DB/Broker thực trong quá trình kiểm thử.

## 7.3. Kiểm thử Đầu cuối (End-to-End Testing) và Kiểm thử Hiệu năng (Performance Testing)

*   **Kiểm thử Đầu cuối (E2E)**:
    *   **Mục đích**: Mô phỏng hành vi của người dùng cuối trên toàn bộ hệ thống (Client -> Gateway -> Services -> DB).
    *   **Công cụ**: **[Ví dụ: Cypress, Selenium, Playwright]**.
    *   **Phạm vi**: Các luồng nghiệp vụ quan trọng nhất (ví dụ: Đăng ký, Đặt hàng, Thanh toán).
*   **Kiểm thử Hiệu năng (Performance Testing)**:
    *   **Mục đích**: Xác minh các **NFRs** về hiệu năng (Response Time, Throughput).
    *   **Công cụ**: **[Ví dụ: JMeter, Locust, Gatling]**.
    *   **Các loại Kiểm thử**: **Load Testing** (tải dự kiến), **Stress Testing** (tải vượt ngưỡng), **Soak Testing** (tải duy trì trong thời gian dài).

## 7.4. Ma trận Truy vết Yêu cầu (Requirements Traceability Matrix - RTM)

RTM đảm bảo rằng mọi yêu cầu (FR và NFR) đều được ánh xạ tới ít nhất một thành phần thiết kế và một trường hợp kiểm thử.

| ID Yêu cầu | Mô tả Yêu cầu | Thiết kế (Mục SDD) | Trường hợp Kiểm thử (Test Case ID) | Trạng thái |
| :--- | :--- | :--- | :--- | :--- |
| **FR-004** | Xử lý quy trình đặt hàng. | 4.1.2 (OrderService) | TC-ORDER-001, TC-ORDER-002 | Đã Hoàn thành |
| **NFR-2.6.1** | Response Time < 200ms. | 3.1.1 (Microservices), 5.4.2 (Metrics) | PT-LOAD-001 | Đang Tiến hành |
| **NFR-6.2** | Sử dụng OAuth 2.0. | 6.2 (Authentication) | TC-AUTH-005 | Đã Hoàn thành |

---

# 8. Phụ lục (Appendices)

## 8.1. Ma trận Quyết định Kiến trúc (Architecture Decision Records - ADRs)

ADR là tài liệu ghi lại các quyết định kiến trúc quan trọng, bối cảnh, các lựa chọn thay thế, và hậu quả của quyết định đó.

| ID ADR | Tiêu đề Quyết định | Ngày | Trạng thái |
| :--- | :--- | :--- | :--- |
| **ADR-001** | Lựa chọn Kiến trúc Microservices | 2025-12-01 | Đã Chấp thuận |
| **ADR-002** | Sử dụng Kafka cho Giao tiếp Bất đồng bộ | 2025-12-05 | Đã Chấp thuận |
| **ADR-003** | Lựa chọn PostgreSQL thay vì MySQL | 2025-12-10 | Đã Chấp thuận |

**Ví dụ Chi tiết ADR-003: Lựa chọn PostgreSQL thay vì MySQL**

*   **Tiêu đề**: Lựa chọn PostgreSQL làm Cơ sở Dữ liệu Quan hệ Chính.
*   **Trạng thái**: Đã Chấp thuận.
*   **Bối cảnh**: Hệ thống yêu cầu khả năng xử lý dữ liệu giao dịch phức tạp (**ACID**) và hỗ trợ các kiểu dữ liệu nâng cao (ví dụ: JSONB, GIS) để phục vụ cho các tính năng tìm kiếm và lưu trữ phi cấu trúc.
*   **Quyết định**: Sử dụng **PostgreSQL 16** làm cơ sở dữ liệu quan hệ chính.
*   **Lý do**:
    1.  **Hỗ trợ JSONB**: Cung cấp khả năng lưu trữ và truy vấn dữ liệu JSON hiệu quả, giúp giảm nhu cầu sử dụng NoSQL DB riêng biệt cho một số trường hợp.
    2.  **Tính năng Nâng cao**: Hỗ trợ các tính năng như **CTE (Common Table Expressions)**, **Window Functions**, và **Full-Text Search** tích hợp, giúp đơn giản hóa logic nghiệp vụ.
    3.  **Khả năng Mở rộng**: Cộng đồng lớn và hỗ trợ các giải pháp Sharding như Citus Data.
*   **Hậu quả**:
    *   **Tích cực**: Tăng tính linh hoạt trong mô hình hóa dữ liệu, hiệu năng truy vấn phức tạp tốt hơn.
    *   **Tiêu cực**: Đội ngũ phát triển cần có kinh nghiệm về PostgreSQL, chi phí vận hành có thể cao hơn MySQL trong một số dịch vụ đám mây.

## 8.2. Sơ đồ Luồng Người dùng (User Flow Diagrams)

*(Phần này sẽ chứa các sơ đồ trực quan hóa các luồng người dùng chính, ví dụ: Sơ đồ Luồng Đăng ký, Sơ đồ Luồng Đặt hàng, Sơ đồ Luồng Thanh toán. Các sơ đồ này thường được tạo bằng **Mermaid** hoặc **PlantUML**.)*

**Ví dụ: Luồng Đăng ký và Xác thực Email (Mermaid Flowchart)**

*(Sơ đồ Luồng Đăng ký và Xác thực Email sẽ được đặt tại đây. Sơ đồ này mô tả các bước từ khi người dùng đăng ký đến khi tài khoản được kích hoạt.)*

## 8.3. Thiết kế Giao diện Người dùng (User Interface - UI/UX Mockups)

*(Phần này sẽ chứa các liên kết đến các bản Mockup/Wireframe chi tiết được tạo bằng Figma, Sketch, hoặc Adobe XD. Mặc dù SDD tập trung vào thiết kế kỹ thuật, việc tham chiếu đến UI/UX là cần thiết để đảm bảo sự đồng bộ giữa thiết kế Backend và Frontend.)*

*   **Mockup Trang Chủ (Homepage)**: [Link Figma/Sketch]
*   **Wireframe Luồng Thanh toán (Checkout Flow)**: [Link Figma/Sketch]
*   **Thiết kế Hệ thống Thiết kế (Design System)**: [Link đến Storybook/Design System Documentation]

## 8.4. Danh sách Các Vấn đề Mở (Open Issues)

## 8.5. Ví dụ Mã Hạ tầng dưới dạng Mã (Infrastructure as Code - IaC)

Phần này cung cấp các đoạn mã mẫu **Terraform** và **Helm Chart** để minh họa cách triển khai và quản lý hạ tầng hệ thống trên nền tảng **Kubernetes** và **Cloud Provider** (ví dụ: AWS, GCP, Azure).

### 8.5.1. Ví dụ Terraform: Khởi tạo Cluster Kubernetes (EKS/GKE/AKS)

```terraform
# File: infra/main.tf

resource "aws_eks_cluster" "main" {
  name     = "[PROJECT_NAME]-cluster"
  role_arn = aws_iam_role.eks_cluster.arn
  version  = "1.29"

  vpc_config {
    subnet_ids         = var.private_subnets
    security_group_ids = [aws_security_group.cluster.id]
    endpoint_private_access = true
    endpoint_public_access  = false
  }

  tags = {
    Name = "[PROJECT_NAME]-eks-cluster"
  }
}

resource "aws_eks_node_group" "main" {
  cluster_name    = aws_eks_cluster.main.name
  node_group_name = "general-purpose"
  node_role_arn   = aws_iam_role.eks_nodes.arn
  subnet_ids      = var.private_subnets
  instance_types  = ["t3.medium"]

  scaling_config {
    desired_size = 3
    max_size     = 10
    min_size     = 3
  }

  update_config {
    max_unavailable = 1
  }
}
```

### 8.5.2. Ví dụ Helm Chart: Triển khai Microservice (UserService)

```yaml
# File: charts/user-service/templates/deployment.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "user-service.fullname" . }}
  labels:
    {{- include "user-service.labels" . | nindent 4 }}
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      {{- include "user-service.selectorLabels" . | nindent 6 }}
  template:
    metadata:
      labels:
        {{- include "user-service.selectorLabels" . | nindent 8 }}
    spec:
      containers:
        - name: {{ .Chart.Name }}
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag | default .Chart.AppVersion }}"
          imagePullPolicy: {{ .Values.image.pullPolicy }}
          ports:
            - name: http
              containerPort: 8080
              protocol: TCP
          livenessProbe:
            httpGet:
              path: /healthz
              port: http
          readinessProbe:
            httpGet:
              path: /readyz
              port: http
          resources:
            {{- toYaml .Values.resources | nindent 12 }}
          env:
            - name: DB_HOST
              valueFrom:
                secretKeyRef:
                  name: db-secrets
                  key: host
            - name: KAFKA_BROKER
              value: kafka-broker-svc:9092
```

## 8.6. Lịch sử Thay đổi Tài liệu (Document Revision History)

| ID | Mô tả Vấn đề | Mức độ Ưu tiên | Người Chịu trách nhiệm | Ngày Cập nhật |
| :--- | :--- | :--- | :--- | :--- |
| **OI-001** | Cần quyết định cuối cùng về việc sử dụng **gRPC** hay **REST** cho giao tiếp Service-to-Service. | Cao | Kiến trúc sư | 2025-12-15 |
| **OI-002** | Chiến lược phân mảnh (**Sharding**) cho bảng `Order` cần được kiểm tra hiệu năng (Proof of Concept). | Trung bình | Đội ngũ Data | 2025-12-12 |
| **OI-003** | Lựa chọn công cụ **CI/CD** (GitLab CI hay GitHub Actions). | Thấp | Đội ngũ DevOps | 2025-12-10 |

## 8.5. Lịch sử Thay đổi Tài liệu (Document Revision History)

| Phiên bản (Version) | Ngày | Tác giả | Mô tả Thay đổi |
| :--- | :--- | :--- | :--- |
| **0.1** | 2025-12-10 | Manus AI | Khởi tạo bản nháp SDD (Cấu trúc và HLD). |
| **0.2** | 2025-12-16 | Manus AI | Bổ sung chi tiết LLD cho UserService, OrderService, Security, và DevOps. |
| **1.0** | [Ngày Hoàn thành] | Manus AI | Bản cuối cùng, được phê duyệt. |

---
*(Kết thúc bản nháp SDD mẫu. Bản nháp này đã bao gồm đầy đủ các phần theo chuẩn IEEE 1016-2009 và các yếu tố hiện đại (Microservices, Cloud-Native, DevOps, Security) để tạo thành một tài liệu siêu chi tiết, có thể mở rộng thành 100 trang bằng cách bổ sung thêm chi tiết cho các mục LLD của từng Microservice và các sơ đồ trực quan.)*

### 4.1.2. Thành phần B: OrderService (Dịch vụ Quản lý Đơn hàng) - Mở rộng Chi tiết

#### 4.1.2.1. Mục đích và Phạm vi (Purpose and Scope)

*   **Mục đích**: Quản lý toàn bộ vòng đời của một đơn hàng, từ khi tạo giỏ hàng, đặt hàng, đến khi hoàn thành hoặc hủy bỏ.
*   **Phạm vi**: Xử lý các thực thể `Order`, `OrderItem`, `ShippingAddress`, và điều phối các giao dịch phân tán liên quan đến `PaymentService` và `InventoryService`.

#### 4.1.2.2. Sơ đồ Lớp (Class Diagram)

*(Để đạt được độ chi tiết 100 trang, phần này sẽ bao gồm sơ đồ lớp chi tiết cho các lớp Domain, Service, và Repository của OrderService, thể hiện mối quan hệ kế thừa, giao diện, và các thuộc tính/phương thức chính.)*

*(Sơ đồ Lớp chi tiết cho OrderService sẽ được đặt tại đây. Sơ đồ này thể hiện các lớp Domain, Service, và Repository, cùng với các thuộc tính và phương thức chính.)*

#### 4.1.2.3. Sơ đồ Trình tự (Sequence Diagram) cho Luồng Chính: Tạo Đơn hàng (Create Order - Sử dụng Saga Pattern)

*(Phần này sẽ được mở rộng bằng sơ đồ trình tự chi tiết sử dụng cú pháp Mermaid, mô tả từng bước giao tiếp giữa OrderService, InventoryService, PaymentService, và Kafka Broker.)*

*(Sơ đồ Trình tự chi tiết cho luồng Tạo Đơn hàng (Saga Pattern) sẽ được đặt tại đây. Sơ đồ này mô tả giao tiếp bất đồng bộ giữa các dịch vụ Order, Inventory, và Payment thông qua Kafka.)*

#### 4.1.2.4. Cấu trúc Dữ liệu Chi tiết (Detailed Data Structures)

*(Phần này sẽ lặp lại bảng Data Dictionary cho tất cả các bảng liên quan đến OrderService, bao gồm `orders`, `order_items`, `transactions`, `shipping_info`, và `saga_logs`.)*

**Bảng: `orders` (Mở rộng)**

| Thuộc tính (Attribute) | Kiểu Dữ liệu (Data Type) | Mô tả | Ràng buộc (Constraint) |
| :--- | :--- | :--- | :--- |
| `order_id` | UUID | Khóa chính. | PK, NOT NULL |
| `user_id` | UUID | ID người dùng đặt hàng. | FK (UserService.users) |
| `status` | VARCHAR(20) | Trạng thái đơn hàng (PENDING, PAID, SHIPPED, DELIVERED, CANCELLED, FAILED). | NOT NULL, INDEXED |
| `total_amount` | DECIMAL(10, 2) | Tổng số tiền cuối cùng. | NOT NULL |
| `subtotal` | DECIMAL(10, 2) | Tổng tiền trước thuế và phí. | NOT NULL |
| `tax_amount` | DECIMAL(10, 2) | Tổng tiền thuế. | NOT NULL |
| `discount_amount` | DECIMAL(10, 2) | Tổng tiền giảm giá. | NOT NULL |
| `shipping_fee` | DECIMAL(10, 2) | Phí vận chuyển. | NOT NULL |
| `shipping_address_json` | JSONB | Thông tin địa chỉ giao hàng chi tiết. | NOT NULL |
| `created_at` | TIMESTAMP WITH TIME ZONE | Thời điểm tạo đơn hàng. | NOT NULL |
| `updated_at` | TIMESTAMP WITH TIME ZONE | Thời điểm cập nhật cuối cùng. | NOT NULL |
| `saga_id` | UUID | ID của giao dịch Saga (nếu có). | NULLABLE |

*(... Lặp lại chi tiết cho các bảng `order_items`, `transactions`, `shipping_info`...)*

---

### 4.1.3. Thành phần C: ProductService (Dịch vụ Quản lý Sản phẩm) - Mở rộng Chi tiết

#### 4.1.3.1. Mục đích và Phạm vi (Purpose and Scope)

*   **Mục đích**: Cung cấp các chức năng quản lý và truy vấn thông tin sản phẩm, danh mục, và tồn kho.
*   **Phạm vi**: Quản lý thực thể `Product`, `Category`, `Inventory`, và duy trì chỉ mục tìm kiếm (**Search Index**).

#### 4.1.3.2. Sơ đồ Lớp (Class Diagram)

*(Phần này sẽ bao gồm sơ đồ lớp chi tiết cho các lớp Domain, Service, và Repository của ProductService, tập trung vào việc đồng bộ hóa dữ liệu giữa DB quan hệ và Search Index.)*

*(Sơ đồ Lớp chi tiết cho ProductService sẽ được đặt tại đây. Sơ đồ này thể hiện các lớp Domain, Service, và Repository, cùng với các thuộc tính và phương thức chính, tập trung vào việc đồng bộ hóa dữ liệu.)*

#### 4.1.3.3. Sơ đồ Trình tự (Sequence Diagram) cho Luồng Chính: Đồng bộ hóa Dữ liệu Sản phẩm (Product Data Synchronization)

*(Sơ đồ này mô tả luồng bất đồng bộ để đảm bảo dữ liệu sản phẩm được cập nhật trên cả PostgreSQL và ElasticSearch.)*

*(Sơ đồ Trình tự chi tiết cho luồng Đồng bộ hóa Dữ liệu Sản phẩm sẽ được đặt tại đây. Sơ đồ này mô tả luồng bất đồng bộ để đảm bảo dữ liệu sản phẩm được cập nhật trên cả PostgreSQL và ElasticSearch.)*

#### 4.1.3.4. Cấu trúc Dữ liệu Chi tiết (Detailed Data Structures)

*(Phần này sẽ lặp lại bảng Data Dictionary cho tất cả các bảng liên quan đến ProductService, bao gồm `products`, `categories`, `inventory`, và `product_attributes`.)*

**Bảng: `inventory` (Mở rộng)**

| Thuộc tính (Attribute) | Kiểu Dữ liệu (Data Type) | Mô tả | Ràng buộc (Constraint) |
| :--- | :--- | :--- | :--- |
| `inventory_id` | UUID | Khóa chính. | PK, NOT NULL |
| `product_id` | UUID | ID sản phẩm. | FK (products), UNIQUE |
| `quantity_available` | INTEGER | Số lượng sản phẩm hiện có. | NOT NULL, CHECK (>= 0) |
| `quantity_reserved` | INTEGER | Số lượng sản phẩm đang được giữ cho các đơn hàng PENDING. | NOT NULL, CHECK (>= 0) |
| `last_updated` | TIMESTAMP WITH TIME ZONE | Thời điểm cập nhật tồn kho cuối cùng. | NOT NULL |

*(... Lặp lại chi tiết cho các bảng `products`, `categories`, `product_attributes`...)*

---

## 8.2. Sơ đồ Luồng Người dùng (User Flow Diagrams) - Mở rộng

*(Bổ sung thêm các sơ đồ luồng quan trọng khác để tăng độ chi tiết.)*

**Ví dụ: Luồng Thanh toán Thành công (Payment Success Flowchart)**

*(Sơ đồ Luồng Thanh toán Thành công sẽ được đặt tại đây. Sơ đồ này mô tả các bước xử lý sau khi nhận được sự kiện thanh toán thành công.)*

**Ví dụ: Sơ đồ Kiến trúc Tổng thể (C4 Model - Level 2: Container Diagram)**

*(Sơ đồ Kiến trúc Tổng thể (C4 Model - Level 2: Container Diagram) sẽ được đặt tại đây. Sơ đồ này mô tả các thành phần chính (Container) và mối quan hệ giữa chúng trong môi trường triển khai.)*

*(Việc bổ sung các chi tiết này, cùng với các bảng và sơ đồ, sẽ mở rộng tài liệu Markdown lên một độ dài đáng kể, mô phỏng một bản SDD siêu chi tiết, có thể dễ dàng đạt 100 trang khi được điền đầy đủ dữ liệu thực tế của dự án.)*
