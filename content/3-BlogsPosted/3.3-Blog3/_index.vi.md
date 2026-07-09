---
title: "Blog 3"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 3.3. </b> "
---
# Bảo vệ dữ liệu trên AWS với AWS Backup

Trong quá trình vận hành hệ thống trên đám mây, dữ liệu luôn là tài sản quan trọng nhất của doanh nghiệp. Dù hạ tầng AWS có tính sẵn sàng cao, các rủi ro như xóa nhầm dữ liệu, lỗi cấu hình, tấn công ransomware hoặc sự cố từ người dùng vẫn có thể xảy ra. Vì vậy, xây dựng một chiến lược sao lưu (Backup) và khôi phục (Recovery) là yêu cầu không thể thiếu đối với mọi hệ thống.

AWS Backup là dịch vụ được AWS phát triển nhằm đơn giản hóa việc sao lưu và khôi phục dữ liệu trên nhiều dịch vụ AWS thông qua một giao diện quản lý tập trung.

---

# 1. AWS Backup là gì?

AWS Backup là dịch vụ managed giúp doanh nghiệp tự động hóa quá trình sao lưu dữ liệu của nhiều dịch vụ AWS mà không cần xây dựng hệ thống backup riêng.

Thay vì cấu hình backup cho từng dịch vụ riêng lẻ, quản trị viên chỉ cần tạo một Backup Plan và áp dụng cho các tài nguyên mong muốn.

AWS Backup hiện hỗ trợ nhiều dịch vụ như:

- Amazon EC2
- Amazon EBS
- Amazon RDS
- Amazon DynamoDB
- Amazon EFS
- Amazon FSx
- Amazon S3
- Amazon Aurora
- AWS Storage Gateway
- Amazon DocumentDB

Nhờ đó doanh nghiệp có thể quản lý toàn bộ chính sách sao lưu từ một nơi duy nhất.

---

# 2. Những tính năng nổi bật

AWS Backup cung cấp nhiều tính năng giúp việc quản lý dữ liệu trở nên đơn giản hơn.

### Backup Plan

Cho phép định nghĩa:

- Chu kỳ sao lưu
- Thời gian lưu trữ
- Backup định kỳ
- Lifecycle của bản sao lưu

### Backup Vault

Các bản sao lưu được lưu trong Backup Vault, nơi hỗ trợ:

- Mã hóa dữ liệu bằng AWS KMS
- Phân quyền truy cập
- Quản lý tập trung

### Cross-Region Backup

AWS Backup có thể sao lưu dữ liệu sang Region khác nhằm tăng khả năng khôi phục khi xảy ra thảm họa.

### Cross-Account Backup

Doanh nghiệp có thể lưu bản sao dữ liệu sang tài khoản AWS khác để tăng mức độ an toàn và đáp ứng yêu cầu Disaster Recovery.

---

# 3. Lợi ích khi triển khai

Việc sử dụng AWS Backup mang lại nhiều lợi ích trong thực tế.

### Giảm rủi ro mất dữ liệu

Ngay cả khi người dùng xóa nhầm dữ liệu hoặc hệ thống gặp sự cố, dữ liệu vẫn có thể được phục hồi nhanh chóng từ các bản backup.

### Đơn giản hóa quản trị

Quản trị viên không cần cấu hình backup riêng cho từng dịch vụ AWS.

Mọi chính sách đều được quản lý tập trung.

### Hỗ trợ tuân thủ

AWS Backup hỗ trợ các yêu cầu kiểm toán và tiêu chuẩn như:

- HIPAA
- PCI DSS
- ISO 27001
- SOC
- FedRAMP

### Tăng khả năng khôi phục sau thảm họa
Backup dữ liệu sang nhiều Region và nhiều tài khoản AWS giúp doanh nghiệp xây dựng chiến lược Disaster Recovery hiệu quả.

---

# 4. Một số lưu ý khi sử dụng

Để triển khai AWS Backup hiệu quả, doanh nghiệp nên lưu ý:

- Không lưu toàn bộ backup trong cùng một Region.
- Thiết lập thời gian lưu trữ phù hợp để tối ưu chi phí.
- Mã hóa Backup Vault bằng AWS KMS.
- Thường xuyên kiểm tra khả năng Restore thay vì chỉ tạo backup.
- Áp dụng Backup Policy thông qua AWS Organizations nếu quản lý nhiều tài khoản.

Việc kiểm thử quá trình khôi phục dữ liệu định kỳ giúp đảm bảo các bản sao lưu luôn sẵn sàng khi cần sử dụng.

---

# 5. Đánh giá cá nhân

Theo mình, AWS Backup là một trong những dịch vụ quan trọng nhưng thường bị bỏ qua khi thiết kế hệ thống trên AWS. Nhiều doanh nghiệp tập trung vào việc triển khai ứng dụng và mở rộng hạ tầng, nhưng chỉ khi xảy ra sự cố mới nhận thấy giá trị của một chiến lược sao lưu hiệu quả.

Việc sử dụng AWS Backup giúp giảm đáng kể công sức quản trị, đồng thời tăng khả năng bảo vệ dữ liệu và đáp ứng các yêu cầu về tuân thủ. Đối với các hệ thống production, đây là một dịch vụ nên được triển khai ngay từ giai đoạn thiết kế kiến trúc thay vì bổ sung sau khi hệ thống đi vào vận hành.

---

# Kết luận

AWS Backup giúp doanh nghiệp xây dựng chiến lược sao lưu và khôi phục dữ liệu một cách đơn giản, an toàn và hiệu quả. Với khả năng quản lý tập trung, tự động hóa chính sách backup, hỗ trợ nhiều dịch vụ AWS cùng các tính năng Cross-Region và Cross-Account Backup, dịch vụ này góp phần nâng cao khả năng bảo vệ dữ liệu và đảm bảo tính liên tục của hệ thống.

Trong bối cảnh dữ liệu ngày càng trở thành tài sản quan trọng, việc triển khai AWS Backup không chỉ giúp giảm thiểu rủi ro mà còn là nền tảng cho một kiến trúc đám mây an toàn và bền vững.

---

# Tài liệu tham khảo

- https://docs.aws.amazon.com/aws-backup/latest/devguide/whatisbackup.html
- https://aws.amazon.com/backup/
- https://docs.aws.amazon.com/aws-backup/latest/devguide/backup-plans.html
- https://docs.aws.amazon.com/aws-backup/latest/devguide/cross-region-backup.html
- https://docs.aws.amazon.com/aws-backup/latest/devguide/restoring-a-backup.html