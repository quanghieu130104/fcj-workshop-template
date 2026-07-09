---
title: "DynamoDB & S3 Buckets"
date: 2026-07-08
weight: 3
chapter: false
pre: " <b> 5.3. </b> "
---

# Khởi tạo Cơ sở dữ liệu và Lưu trữ

Trong hệ thống E-commerce, dữ liệu là tài sản quý giá nhất. Ở bước này, chúng ta sẽ thiết lập **Amazon DynamoDB** làm cơ sở dữ liệu NoSQL tốc độ cao cho Backend và **Amazon S3** để lưu trữ các tài nguyên tĩnh (hình ảnh, frontend).

---

### 1. Khởi tạo 13 Bảng Amazon DynamoDB

Hệ thống của chúng ta sử dụng thiết kế phân rã dữ liệu thành 13 bảng riêng biệt. 
Để tạo bảng, hãy truy cập vào dịch vụ **DynamoDB** trên AWS Console và lần lượt tạo các bảng với cấu hình chuẩn như sau:

**Cấu hình chung cho TẤT CẢ các bảng:**
- **Capacity Mode (Chế độ thanh toán):** Chọn `On-demand` (Tránh chọn Provisioned để không bị tính phí duy trì liên tục khi không có truy cập).
- **Point-in-Time Recovery (PITR):** Chọn `Enable` (Bật tính năng sao lưu liên tục để có thể khôi phục dữ liệu ở bất kỳ giây nào).

**Danh sách các bảng cần tạo:**
1. **`users_table`** (Partition key: `id` - String)
2. **`product_table`** (Partition key: `id` - String)
3. **`product_variant_table`** (Partition key: `productId` - String, Sort key: `variantId` - String)
4. **`category_table`** (Partition key: `id` - String)
5. **`cart_item_table`** (Partition key: `userId` - String, Sort key: `productId` - String)
6. **`payment_table`** (Partition key: `id` - String)
7. **`shipment_table`** (Partition key: `id` - String)
8. **`inventory_table`** (Partition key: `productId` - String)
9. **`inventory_movement_table`** (Partition key: `id` - String, Sort key: `timestamp` - Number)
10. **`review_table`** (Partition key: `productId` - String, Sort key: `id` - String)
11. **`promotion_table`** (Partition key: `id` - String)
12. **`support_tickets`** (Partition key: `id` - String)
13. **`order_table`** (Partition key: `id` - String, Sort key: `createdAt` - Number)

![Create Table](/images/5-Workshop/5.3-database-storage/Createtable.png)

![Enable PITR](/images/5-Workshop/5.3-database-storage/PITR.png)

![13 Tables Created](/images/5-Workshop/5.3-database-storage/13table.png)

---

### 2. Khởi tạo Amazon S3 Buckets

Tiếp theo, chúng ta khởi tạo 2 Buckets trên dịch vụ **Amazon S3** để chứa dữ liệu tĩnh.
Truy cập Amazon S3 và bấm **Create bucket**. *Lưu ý: Tên bucket trên AWS phải là duy nhất trên toàn cầu (không ai được trùng tên với bạn), hãy thêm các hậu tố đặc trưng của dự án vào tên.*

**Bucket 1: Bucket chứa ảnh Upload (Ví dụ: `my-app-uploads`)**

![Upload image](/images/5-Workshop/5.3-database-storage/S3Fe_static.png)

Dùng để chứa hình ảnh sản phẩm hoặc avatar người dùng do Backend xử lý tải lên.

**Lưu ý khi chọn Global namespace**:

- **Bucket phải có tên duy nhất trên toàn bộ AWS.**
- Nếu tên bucket đã được tài khoản AWS khác sử dụng, cần đổi sang tên khác.
- Khuyến nghị đặt tên có hậu tố riêng để tránh trùng, ví dụ:
    `my-app-uploads-2026`
    `my-app-uploads-e-commerce`

**Cấu hình**:
- **Block all public access:** Tích BẬT (Backend sẽ đảm nhiệm việc tạo presigned-URL để xem ảnh, đảm bảo bảo mật tuyệt đối cho dữ liệu).
- **Bucket Versioning:** BẬT (Lưu lại các phiên bản của file để phòng trường hợp vô tình ghi đè nhầm).

![Upload image](/images/5-Workshop/5.3-database-storage/S3Fe_static.png)

![Upload image](/images/5-Workshop/5.3-database-storage/S3Fe_static1.png)

**Bucket 2: Bucket chứa Frontend (Ví dụ: `my-app-fe-static`)**
Dùng để chứa bộ mã nguồn tĩnh của Frontend (React/Next.js) sau khi đã build (thư mục `dist/client/*`).

**Lưu ý khi chọn Global namespace**:

- **Bucket phải có tên duy nhất trên toàn AWS.**
- Nếu tên đã tồn tại, cần đổi sang tên khác, ví dụ:
    `my-app-fe-static-2026`
    `my-app-fe-static-e-commerce`

**Cấu hình**:
- **Block all public access:** Tích BẬT. (Trang web Frontend sẽ không truy cập trực tiếp bằng link S3 mà phải đi vòng qua mạng CDN CloudFront bằng phương thức Origin Access Control - OAC).
- **Bucket Versioning:** BẬT.

![Upload image](/images/5-Workshop/5.3-database-storage/S3Upload.png)

![Upload image](/images/5-Workshop/5.3-database-storage/S3Upload1.png)
