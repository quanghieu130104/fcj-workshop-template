---
title: "Tên miền & SSL (Route 53, ACM)"
date: 2026-07-09
weight: 1
chapter: false
pre: " <b> 5.4.0. </b> "
---

# Khởi tạo Tên miền và Chứng chỉ bảo mật SSL

Trước khi chúng ta đi vào triển khai Backend và Application Load Balancer (ALB), hệ thống yêu cầu bạn phải có sẵn một **Chứng chỉ SSL (ACM)** để mã hóa kết nối HTTPS, nhằm đảm bảo an toàn cho ứng dụng và đáp ứng yêu cầu của Amazon Cognito. Để cấp phát SSL, bạn cần sở hữu một tên miền (Domain) và quản lý nó qua **Amazon Route 53**.

Bài này sẽ hướng dẫn bạn tạo một Hosted Zone trên Route 53 để quản lý tên miền và yêu cầu chứng chỉ SSL. *(Vì tài khoản AWS Free Tier không hỗ trợ mua tên miền trực tiếp trên Route 53, bạn cần đăng ký tên miền từ các nhà cung cấp bên ngoài như Hostinger, Namecheap, v.v. trước khi bắt đầu).*

### Bước 1: Tạo Hosted Zone trên Amazon Route 53
1. Truy cập dịch vụ **Amazon Route 53**.
2. Ở thanh menu bên trái, chọn **Hosted zones**.
3. Bấm **Create hosted zone**.
4. **Domain name:** Nhập tên miền bạn đã mua từ nhà cung cấp bên ngoài (ví dụ: `zopee.xyz`).
5. **Type:** Giữ nguyên **Public hosted zone**.
6. Bấm **Create hosted zone**.
7. Sau khi tạo xong, bạn sẽ thấy 4 địa chỉ máy chủ phân giải tên miền ở cột **Value/Route traffic to** của bản ghi loại **NS** (Name server).
8. Quay lại trang quản lý tên miền của nhà cung cấp bên ngoài (nơi bạn mua tên miền), tìm phần cấu hình **DNS / Name Servers (NS)**, và thay thế bằng 4 giá trị NS mà Route 53 vừa cấp. Đợi một thời gian ngắn để DNS cập nhật.

![Router53](/images/5-Workshop/5.4-backend-ecs/5.4.0-route53-acm/Route53.png)
![Router53](/images/5-Workshop/5.4-backend-ecs/5.4.0-route53-acm/Route53_1.png)

### Bước 2: Yêu cầu chứng chỉ SSL từ AWS Certificate Manager (ACM)
Khi tên miền đã sẵn sàng trong Hosted Zone, chúng ta cần tạo chứng chỉ SSL/TLS.
**QUAN TRỌNG:** Vì hệ thống của chúng ta có sử dụng CloudFront (ở phần 5.7), bạn **BẮT BUỘC** phải chuyển Region sang **US East (N. Virginia) - us-east-1** trước khi tạo chứng chỉ ACM! (CloudFront chỉ nhận ACM ở us-east-1).
![Region](/images/5-Workshop/5.4-backend-ecs/5.4.0-route53-acm/region.png)

1. Đảm bảo góc trên cùng bên phải màn hình AWS đang chọn Region **US East (N. Virginia) - us-east-1**.
2. Truy cập dịch vụ **AWS Certificate Manager (ACM)**.
3. Bấm **Request a certificate**.
4. Chọn **Request a public certificate**, bấm **Next**.
5. Ở mục **Fully qualified domain name**, nhập vào 2 tên miền sau:
   - `zopee.xyz` *(Thay bằng tên miền bạn vừa mua)*
   - Bấm **Add another name to this certificate**
   - `*.zopee.xyz` *(Dấu sao đại diện cho tất cả Sub-domains)*
6. **Validation method:** Chọn **DNS validation - recommended**.
7. Key algorithm: Giữ nguyên **RSA 2048**.
8. Bấm **Request**.

![ACM](/images/5-Workshop/5.4-backend-ecs/5.4.0-route53-acm/ACM.png)

### Bước 3: Xác thực chứng chỉ SSL (DNS Validation)
Mặc định chứng chỉ vừa tạo sẽ ở trạng thái *Pending validation*.
1. Bấm vào mã **Certificate ID** của chứng chỉ vừa tạo để xem chi tiết.
2. Ở mục **Domains**, bạn sẽ thấy các bản ghi CNAME cần được thêm vào DNS.
3. Vì tên miền của bạn đang được quản lý bởi Route 53, bạn chỉ cần bấm nút **Create records in Route 53**.
4. Một cửa sổ hiện lên liệt kê các bản ghi CNAME, bấm **Create records**.
5. Đợi khoảng 2 đến 5 phút, Route 53 sẽ tự động cập nhật bản ghi DNS và trạng thái của chứng chỉ trong ACM sẽ chuyển sang màu xanh **Issued**.

![ACM](/images/5-Workshop/5.4-backend-ecs/5.4.0-route53-acm/ACM1.png)


