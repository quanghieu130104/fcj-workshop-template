---
title: "Global Content Delivery"
date: 2026-07-09
weight: 7
chapter: false
pre: " <b> 5.7. </b> "
---

# Amazon CloudFront (CDN) Integration

In section 5.3, we uploaded our static website (Frontend) to **Amazon S3** but selected "Block all public access" to ensure maximum security. Therefore, users cannot access the website directly via the S3 link.

In this section, we will create an **Amazon CloudFront Distribution** (Content Delivery Network). CloudFront will act as the single, secure entry point allowing users worldwide to access the Frontend quickly and safely.

### 1. Initialize CloudFront Distribution

**Step 1: Initialize CloudFront Distribution**
1. Navigate to **Amazon CloudFront** → select **Create distribution**.
2. **Distribution name:** Enter `zopee-ecommerce`.
3. **Distribution type:** Select **Single website or app**.
4. **Description:** Leave blank or enter a description for the Distribution.
5. **Route 53 managed domain:**
   - If the domain `zopee.xyz` is already managed by Route 53 in the same AWS account, enter `zopee.xyz` and click **Check domain**.
   - If the domain is managed by another DNS provider, leave this blank and configure the domain after the Distribution is created.
6. Click **Next** to proceed to the Specify origin step.

![Cloudfront](/images/5-Workshop/5.7-cloudfront/cloudfront.png)

**Step 2: Specify Origin**
1. **Origin type:** Select **Elastic Load Balancer**.
2. **Origin:** Select the Application Load Balancer (`my-app-alb`) created in the previous step.
3. **Origin path:** Leave blank.

![Cloudfront](/images/5-Workshop/5.7-cloudfront/cloudfront1.png)

**Step 3: Enable Security**
1. **Enable security protections:** Select **Enable security protections**.
2. **Web Application Firewall (WAF):** Keep the default configuration recommended by CloudFront.
3. **Use monitor mode:** Do not select.
4. **Rate limiting:** Enable.
5. **SQL protections:** Enable.
6. **Protection against Layer 7 DDoS attacks:** Enable.
7. Click **Next** to proceed to the Get TLS certificate step.
8. **Origin settings:** Select **Use recommended origin settings**.
9. **Cache settings:** Select **Use recommended cache settings**.
10. Click **Next** to proceed to the next step.

![Cloudfront](/images/5-Workshop/5.7-cloudfront/cloudfront2.png)

**Step 4: Configure TLS Certificate**
1. Under **Available certificates**, select the ACM certificate created for the domain `zopee.xyz`.
2. Confirm the certificate includes the following domains:
   - `zopee.xyz`
   - `*.zopee.xyz` (Wildcard)
3. Ensure the certificate was issued by AWS Certificate Manager (ACM) in the US East (N. Virginia) - `us-east-1` region.
4. Click **Next** to proceed to the Review and create step.
5. Click **Create distribution**.

![Cloudfront](/images/5-Workshop/5.7-cloudfront/cloudfront3.png)

### 2. Grant CloudFront access to read the S3 Bucket (Bucket Policy)
After clicking Create, if CloudFront displays a prompt requiring you to update the S3 Bucket Policy, click Copy policy, then navigate to Amazon S3 → select your Frontend Bucket → Permissions → Bucket policy → Edit, paste the copied policy, and click Save changes.

![Cloudfront](/images/5-Workshop/5.7-cloudfront/cloudfront5.png)

If you missed the prompt, you can still retrieve and configure the Bucket Policy by following these steps:

Navigate to Amazon CloudFront → select the newly created Distribution.
Switch to the Origins tab and select the Origin pointing to your S3 Bucket - > Edit.
Check the Origin access control section:
If using Origin Access Control (OAC), CloudFront usually provides a Copy policy or View policy option to copy the Bucket Policy.
After updating the Bucket Policy, click Save changes.

If the Bucket Policy already grants access to CloudFront (s3:GetObject), no further action is required.

![Cloudfront](/images/5-Workshop/5.7-cloudfront/cloudfront4.png)

### 3. Access your E-commerce Website!
1. Go back to the **General** tab of your CloudFront Distribution.
2. You will see the **Distribution domain name** (e.g., `d123...cloudfront.net`).
3. The status (Last modified) will initially show as *Deploying*. Be patient and wait about 3-5 minutes.
4. Once deployment is complete, copy that domain name, paste it into your browser, and enjoy your highly secure and blazing-fast E-commerce website!

![Cloudfront](/images/5-Workshop/5.7-cloudfront/cloudfront6.png)
