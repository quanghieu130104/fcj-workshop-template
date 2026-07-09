---
title: "Blog 1"
date: 2026-06-04
weight: 1
chapter: false
pre: " <b> 3.1. </b> "
---

# Bảo Mật Đám Mây Với Amazon VPC Encryption Controls

Trong các hệ thống doanh nghiệp hiện đại, việc mã hóa dữ liệu khi truyền tải (*encryption in transit*) là một yêu cầu quan trọng để đáp ứng các tiêu chuẩn bảo mật như **HIPAA**, **PCI DSS**, **FedRAMP** hay **SOC 2**. Tuy nhiên, khi hạ tầng AWS phát triển với hàng trăm hoặc hàng nghìn tài nguyên, việc xác định lưu lượng nào đang được mã hóa và tài nguyên nào vẫn truyền dữ liệu dưới dạng *plaintext* trở nên rất khó khăn.

Để giải quyết bài toán này, AWS đã giới thiệu **Amazon VPC Encryption Controls** – một tính năng mới giúp doanh nghiệp giám sát, kiểm soát và thực thi việc mã hóa dữ liệu khi truyền tải giữa các tài nguyên AWS trong cùng hoặc giữa nhiều VPC trong một Region.

---

## Amazon VPC Encryption Controls là gì?

Amazon VPC Encryption Controls là một tính năng bảo mật cho phép:

- Giám sát trạng thái mã hóa của lưu lượng mạng trong VPC.
- Xác định các tài nguyên vẫn cho phép truyền dữ liệu không mã hóa.
- Thực thi chính sách bắt buộc mã hóa dữ liệu trong toàn bộ môi trường AWS.
- Đơn giản hóa quá trình kiểm toán và đáp ứng các yêu cầu tuân thủ.

Tính năng này tận dụng hai cơ chế mã hóa chính:

- **Mã hóa ở tầng ứng dụng (Application Layer Encryption)** như TLS.
- **Mã hóa phần cứng của AWS Nitro System**, được tích hợp trên các thế hệ instance hiện đại.

Ngoài Amazon EC2, AWS còn mở rộng khả năng mã hóa của Nitro cho nhiều dịch vụ khác như:

- Application Load Balancer (ALB)
- Network Load Balancer (NLB)
- AWS Fargate
- Amazon EKS
- Amazon ECS
- Amazon RDS
- Amazon OpenSearch Service
- Amazon MSK
- AWS Transit Gateway

Nhờ đó doanh nghiệp có thể triển khai mã hóa đồng nhất trên quy mô lớn mà không cần xây dựng hệ thống PKI phức tạp hoặc sử dụng nhiều giải pháp bảo mật bên thứ ba.

---

## Hai chế độ hoạt động chính

### 1. Monitor Mode – Quan sát trước khi thực thi

AWS khuyến nghị mọi VPC hiện hữu nên bắt đầu với **Monitor Mode**.

Ở chế độ này, VPC Encryption Controls **không chặn lưu lượng**, mà chỉ ghi nhận và đánh giá trạng thái mã hóa của các kết nối.

Một trường dữ liệu mới có tên **encryption-status** được bổ sung vào **VPC Flow Logs** với các giá trị:

| Giá trị | Ý nghĩa |
|----------|----------|
| 0 | Không mã hóa |
| 1 | Mã hóa bằng AWS Nitro |
| 2 | Mã hóa ở tầng ứng dụng (TLS) |
| 3 | Nitro + TLS |
| Không có giá trị | Không xác định |
Thông qua Flow Logs, quản trị viên có thể nhanh chóng xác định:

- Luồng dữ liệu nào đang được bảo vệ.
- Dịch vụ nào vẫn cho phép truyền dữ liệu dạng plaintext.
- Những tài nguyên cần nâng cấp trước khi thực thi chính sách mã hóa.

Ngoài ra, AWS còn cung cấp API:

```
GetVpcResourcesBlockingEncryptionEnforcement
```

API này giúp liệt kê các tài nguyên chưa đáp ứng yêu cầu mã hóa trước khi chuyển sang chế độ thực thi.

---

## Quá trình chuyển đổi sang môi trường mã hóa hoàn toàn

Sau khi đánh giá hệ thống bằng Monitor Mode, doanh nghiệp có thể tiến hành chuyển đổi hạ tầng.

### Các dịch vụ được AWS tự động nâng cấp

AWS tự động chuyển đổi hạ tầng nền sang AWS Nitro đối với:

- Application Load Balancer
- Network Load Balancer
- AWS Fargate
- Amazon EKS Control Plane

Quá trình này diễn ra:

- Không gây downtime.
- Không cần thay đổi cấu hình.
- Không yêu cầu thao tác thủ công.

### Các dịch vụ cần nâng cấp thủ công

Một số tài nguyên vẫn cần khách hàng chủ động chuyển đổi, bao gồm:

- EC2 thế hệ cũ
- Auto Scaling Groups
- Amazon RDS
- Amazon ElastiCache
- Amazon Redshift
- Amazon ECS sử dụng EC2
- Amazon OpenSearch Service
- Amazon EMR

Nếu các tài nguyên này đang sử dụng instance không hỗ trợ Nitro, doanh nghiệp cần:

- Chuyển sang các dòng EC2 hiện đại hỗ trợ AWS Nitro.
- Hoặc triển khai TLS ở tầng ứng dụng.

---

## Enforce Mode – Thực thi chính sách mã hóa

Sau khi hoàn tất quá trình đánh giá và chuyển đổi, doanh nghiệp có thể kích hoạt **Enforce Mode**.

Đây là giai đoạn AWS bắt đầu thực thi chính sách bảo mật.

Khi bật Enforce Mode:

- Không thể tạo EC2 thế hệ cũ không hỗ trợ AWS Nitro.
- Không thể triển khai tài nguyên cho phép lưu lượng không mã hóa.
- Chỉ các kết nối đã được mã hóa mới được phép hoạt động trong VPC.

Nhờ đó, toàn bộ lưu lượng nội bộ luôn được bảo vệ mà không phụ thuộc hoàn toàn vào việc người dùng có cấu hình TLS hay không.

---

## Các trường hợp ngoại lệ được hỗ trợ

Trong thực tế, một số tài nguyên cần giao tiếp với bên ngoài hạ tầng AWS và không thể được quản lý hoàn toàn bởi VPC Encryption Controls.

AWS cho phép cấu hình **Exclusion** đối với các tài nguyên như:

- Internet Gateway
- NAT Gateway
- Egress-only Internet Gateway
- Virtual Private Gateway
- VPC Peering
- AWS Lambda trong VPC
- VPC Lattice
- Amazon EFS

Các ngoại lệ này vẫn được ghi nhận đầy đủ trong quá trình kiểm toán, giúp doanh nghiệp chứng minh khả năng tuân thủ khi thực hiện audit.

---
## Lợi ích đối với doanh nghiệp

### Tăng cường bảo mật

- Đảm bảo mọi lưu lượng nội bộ đều được mã hóa.
- Giảm nguy cơ rò rỉ dữ liệu trong quá trình truyền tải.

### Đơn giản hóa vận hành

Doanh nghiệp không cần:

- Xây dựng hệ thống PKI riêng.
- Quản lý số lượng lớn chứng chỉ số.
- Triển khai nhiều giải pháp bảo mật khác nhau.

### Hỗ trợ kiểm toán và tuân thủ

VPC Encryption Controls cung cấp:

- Báo cáo trạng thái mã hóa.
- VPC Flow Logs phục vụ kiểm toán.
- Bằng chứng đáp ứng các tiêu chuẩn như HIPAA, PCI DSS và FedRAMP.

### Tận dụng AWS Nitro

Khả năng mã hóa được thực hiện ở cấp phần cứng nên gần như không ảnh hưởng đến hiệu năng của ứng dụng.

---

## Kết luận

Amazon VPC Encryption Controls là một bước tiến quan trọng trong việc đơn giản hóa bảo mật mạng trên AWS. Thay vì phải tự xây dựng hệ thống giám sát và kiểm soát mã hóa phức tạp, doanh nghiệp có thể sử dụng giải pháp tích hợp sẵn để:

- Theo dõi trạng thái mã hóa của toàn bộ lưu lượng mạng.
- Xác định các điểm yếu bảo mật.
- Thực thi chính sách mã hóa một cách tập trung.
- Đáp ứng các yêu cầu tuân thủ nghiêm ngặt.

Nếu doanh nghiệp đang hướng tới các chứng nhận như **HIPAA**, **PCI DSS** hoặc **FedRAMP**, việc triển khai **Monitor Mode** là bước khởi đầu phù hợp để đánh giá mức độ an toàn của hạ tầng trước khi chuyển sang **Enforce Mode**.

---

## Tài liệu tham khảo

- https://docs.aws.amazon.com/vpc/latest/userguide/vpc-encryption-controls.html
- https://aws.amazon.com/vi/blogs/aws/introducing-vpc-encrypt…ce-encryption-in-transit-within-and-across-vpcs-in-a-region/
