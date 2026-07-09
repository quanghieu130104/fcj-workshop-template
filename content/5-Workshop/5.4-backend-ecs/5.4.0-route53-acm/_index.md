---
title: "Domain & SSL (Route 53, ACM)"
date: 2026-07-09
weight: 1
chapter: false
pre: " <b> 5.4.0. </b> "
---

# Setting up Domain Name and SSL Certificate

Before diving into deploying the Backend and Application Load Balancer (ALB), the system requires you to have an **SSL Certificate (ACM)** ready to encrypt HTTPS connections. This ensures application security and fulfills Amazon Cognito's requirements. To provision an SSL certificate, you must own a custom domain and manage it via **Amazon Route 53**.

This guide will show you how to create a Hosted Zone in Route 53 to manage your domain and request an SSL certificate. *(Since AWS Free Tier accounts do not support direct domain registration on Route 53, you must register a domain from a third-party provider like Hostinger, Namecheap, etc., before starting).*

### Step 1: Create a Hosted Zone in Amazon Route 53
1. Navigate to the **Amazon Route 53** console.
2. In the left navigation menu, select **Hosted zones**.
3. Click **Create hosted zone**.
4. **Domain name:** Enter the custom domain you purchased from the third-party provider (e.g., `zopee.xyz`).
5. **Type:** Keep it as **Public hosted zone**.
6. Click **Create hosted zone**.
7. Once created, you will see 4 name server addresses in the **Value/Route traffic to** column of the **NS** (Name server) record.
8. Go back to your third-party domain registrar's dashboard, find the **DNS / Name Servers (NS)** configuration, and replace the existing nameservers with the 4 NS values provided by Route 53. Wait a short time for the DNS to propagate.

![Router53](/images/5-Workshop/5.4-backend-ecs/5.4.0-route53-acm/Route53.png)
![Router53](/images/5-Workshop/5.4-backend-ecs/5.4.0-route53-acm/Route53_1.png)

### Step 2: Request an SSL Certificate from AWS Certificate Manager (ACM)
Once your domain is active in the Hosted Zone, you need to create an SSL/TLS certificate.
**CRITICAL:** Because our architecture will use CloudFront (in section 5.7), you **MUST** switch your AWS Region to **US East (N. Virginia) - us-east-1** before requesting the ACM certificate! (CloudFront only accepts ACM certificates from us-east-1).
![Region](/images/5-Workshop/5.4-backend-ecs/5.4.0-route53-acm/region.png)

1. Ensure the AWS Region dropdown at the top-right corner is set to **US East (N. Virginia) - us-east-1**.
2. Navigate to the **AWS Certificate Manager (ACM)** console.
3. Click **Request a certificate**.
4. Select **Request a public certificate**, click **Next**.
5. In the **Fully qualified domain name** section, enter the following 2 names:
   - `zopee.xyz` *(Replace with your purchased domain)*
   - Click **Add another name to this certificate**
   - `*.zopee.xyz` *(The wildcard covers all subdomains)*
6. **Validation method:** Select **DNS validation - recommended**.
7. Key algorithm: Leave as **RSA 2048**.
8. Click **Request**.

![ACM](/images/5-Workshop/5.4-backend-ecs/5.4.0-route53-acm/ACM.png)

### Step 3: Validate the SSL Certificate (DNS Validation)
The newly requested certificate will initially have a *Pending validation* status.
1. Click on the **Certificate ID** to view its details.
2. Under the **Domains** section, you will see CNAME records that need to be added to your DNS.
3. Since your domain is managed by Route 53, simply click the **Create records in Route 53** button.
4. A prompt will appear listing the CNAME records. Click **Create records**.
5. Wait for about 2 to 5 minutes. Route 53 will automatically propagate the DNS records, and your certificate status in ACM will turn green and show as **Issued**.

![ACM](/images/5-Workshop/5.4-backend-ecs/5.4.0-route53-acm/ACM1.png)
