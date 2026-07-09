---
title: "Phân phối nội dung toàn cầu"
date: 2026-07-09
weight: 7
chapter: false
pre: " <b> 5.7. </b> "
---

# Tích hợp Amazon CloudFront (CDN)

Ở phần 5.3, chúng ta đã đưa bộ mã nguồn tĩnh (Frontend) lên **Amazon S3** nhưng đã chọn khóa hoàn toàn (Block all public access) để đảm bảo bảo mật. Do đó, người dùng không thể truy cập trực tiếp web qua link S3.

Trong phần này, chúng ta sẽ tạo một **Amazon CloudFront Distribution** (Mạng phân phối nội dung CDN). CloudFront sẽ đóng vai trò như cánh cửa duy nhất cho phép người dùng trên toàn thế giới truy cập Frontend một cách nhanh chóng và an toàn.

### 1. Khởi tạo CloudFront Distribution

**Bước 1: Khởi tạo CloudFront Distribution**
1. Truy cập dịch vụ **Amazon CloudFront** → chọn **Create distribution**.
2. **Distribution name:** Nhập `zopee-ecommerce`.
3. **Distribution type:** Chọn **Single website or app**.
4. **Description:** Có thể để trống hoặc nhập mô tả cho Distribution.
5. **Route 53 managed domain:**
   - Nếu tên miền `zopee.xyz` đã được quản lý bởi Route 53 trong cùng tài khoản AWS, nhập `zopee.xyz` và nhấn **Check domain**.
   - Nếu tên miền được quản lý bởi nhà cung cấp DNS khác, bỏ trống mục này và cấu hình tên miền sau khi Distribution được tạo.
6. Nhấn **Next** để chuyển sang bước Specify origin.

![Cloudfront](/images/5-Workshop/5.7-cloudfront/cloudfront.png)

**Bước 2: Chỉ định Origin**
1. **Origin type:** Chọn **Elastic Load Balancer**.
2. **Origin:** Chọn Application Load Balancer (`my-app-alb`) đã tạo ở bước trước.
3. **Origin path:** Để trống.

![Cloudfront](/images/5-Workshop/5.7-cloudfront/cloudfront1.png)

**Bước 3: Bật bảo mật**
1. **Enable security protections:** Chọn **Enable security protections**.
2. **Web Application Firewall (WAF):** Giữ cấu hình mặc định do CloudFront đề xuất.
3. **Use monitor mode:** Không chọn.
4. **Rate limiting:** Bật.
5. **SQL protections:** Bật.
6. **Protection against Layer 7 DDoS attacks:** Bật.
7. Nhấn **Next** để chuyển sang bước Get TLS certificate.
8. **Origin settings:** Chọn **Use recommended origin settings**.
9. **Cache settings:** Chọn **Use recommended cache settings**.
10. Nhấn **Next** để chuyển sang bước tiếp theo.

![Cloudfront](/images/5-Workshop/5.7-cloudfront/cloudfront2.png)

**Bước 4: Cấu hình chứng chỉ TLS**
1. Tại mục **Available certificates**, chọn chứng chỉ ACM đã tạo cho tên miền `zopee.xyz`.
2. Xác nhận chứng chỉ bao gồm các tên miền:
   - `zopee.xyz`
   - `*.zopee.xyz` (Wildcard)
3. Kiểm tra chứng chỉ được phát hành bởi AWS Certificate Manager (ACM) tại Region US East (N. Virginia) - `us-east-1`.
4. Nhấn **Next** để chuyển sang bước Review and create.
5. Bấm **Create distribution**.

![Cloudfront](/images/5-Workshop/5.7-cloudfront/cloudfront3.png)

### 2. Cấp quyền cho CloudFront đọc S3 Bucket (Bucket Policy)
Sau khi nhấn Create, nếu CloudFront hiển thị thông báo yêu cầu cập nhật S3 Bucket Policy, hãy bấm Copy policy, sau đó truy cập Amazon S3 → chọn Bucket Frontend → Permissions → Bucket policy → Edit, dán policy vừa sao chép và nhấn Save changes.

![Cloudfront](/images/5-Workshop/5.7-cloudfront/cloudfront5.png)

Nếu lỡ bỏ qua thông báo, bạn vẫn có thể lấy lại và cấu hình Bucket Policy theo các bước sau:

Truy cập Amazon CloudFront → chọn Distribution vừa tạo.
Chuyển sang tab Origins và chọn Origin trỏ đến Bucket S3 - > Edit.
Kiểm tra mục Origin access control:
Nếu đang sử dụng Origin Access Control (OAC), CloudFront thường có tùy chọn Copy policy hoặc View policy để sao chép Bucket Policy.
Sau khi cập nhật Bucket Policy, nhấn Save changes.

Nếu Bucket Policy đã chứa quyền cho CloudFront (s3:GetObject) thì không cần thực hiện thêm.

![Cloudfront](/images/5-Workshop/5.7-cloudfront/cloudfront4.png)

### 3. Truy cập trang web E-commerce của bạn!
1. Quay lại tab **General** của CloudFront Distribution.
2. Bạn sẽ thấy mục **Distribution domain name** (có dạng `d123...cloudfront.net`).
3. Trạng thái (Last modified) ban đầu sẽ là *Deploying*. Hãy kiên nhẫn đợi khoảng 3-5 phút.
4. Khi trạng thái hoàn tất, hãy copy link domain name đó, dán vào trình duyệt và chiêm ngưỡng thành quả trang web E-commerce cực kỳ bảo mật và tốc độ cao của bạn!

![Cloudfront](/images/5-Workshop/5.7-cloudfront/cloudfront6.png)
