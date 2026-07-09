---
title: "DynamoDB & S3 Buckets"
date: 2026-07-08
weight: 3
chapter: false
pre: " <b> 5.3. </b> "
---

# Database and Storage Initialization

In an E-commerce system, data is the most valuable asset. In this step, we will set up **Amazon DynamoDB** as a high-speed NoSQL database for our Backend and **Amazon S3** to store static assets (images, frontend code).

---

### 1. Initialize 13 Amazon DynamoDB Tables

Our system design splits data into 13 distinct tables.
To create a table, navigate to the **DynamoDB** service on the AWS Console and create the tables sequentially using the following standard configuration:

**General Configuration for ALL tables:**
- **Capacity Mode:** Choose `On-demand` (Avoid Provisioned to prevent being charged continuous fixed rates when there is no traffic).
- **Point-in-Time Recovery (PITR):** Choose `Enable` (Turn on continuous backups to be able to restore data to any second).

**List of tables to create:**
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

### 2. Initialize Amazon S3 Buckets

Next, we initialize 2 Buckets in the **Amazon S3** service to store static data.
Navigate to Amazon S3 and click **Create bucket**. *Note: Bucket names on AWS must be globally unique (no one else can have your exact name), so append unique project suffixes to the name.*

**Bucket 1: Uploads Bucket (Example: `my-app-uploads`)**

![Upload image](/images/5-Workshop/5.3-database-storage/S3Fe_static.png)

Used to store product images or user avatars uploaded and processed by the Backend.

**Note when choosing Global namespace**:

- **The bucket name must be globally unique across all of AWS.**
- If the bucket name is already used by another AWS account, you must change it.
- We recommend adding a unique suffix to avoid conflicts, for example:
    `my-app-uploads-2026`
    `my-app-uploads-e-commerce`

**Configuration**:
- **Block all public access:** Turn ON (The Backend will handle generating presigned-URLs to view images, ensuring absolute data security).
- **Bucket Versioning:** Turn ON (Keeps versions of files in case of accidental overwrites).

![Upload image](/images/5-Workshop/5.3-database-storage/S3Fe_static.png)

![Upload image](/images/5-Workshop/5.3-database-storage/S3Fe_static1.png)

**Bucket 2: Frontend Static Bucket (Example: `my-app-fe-static`)**
Used to store the static source code of the Frontend (React/Next.js) after it has been built (the `dist/client/*` directory).

**Note when choosing Global namespace**:

- **The bucket name must be globally unique across all of AWS.**
- If the name already exists, you must change it to something else, for example:
    `my-app-fe-static-2026`
    `my-app-fe-static-e-commerce`

**Configuration**:
- **Block all public access:** Turn ON. (The Frontend website will not be accessed directly via the S3 link; it will route through the CloudFront CDN network using the Origin Access Control - OAC method).
- **Bucket Versioning:** Turn ON.

![Upload image](/images/5-Workshop/5.3-database-storage/S3Upload.png)

![Upload image](/images/5-Workshop/5.3-database-storage/S3Upload1.png)
