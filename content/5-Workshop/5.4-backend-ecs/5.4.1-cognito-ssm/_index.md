---
title: "Cognito & SSM Parameter Store"
date: 2026-07-08
weight: 1
chapter: false
pre: " <b> 5.4.1. </b> "
---

# 1. Initialize Amazon Cognito User Pool

Amazon Cognito will act as the authentication "guard" before users are allowed to call our Backend APIs. We will integrate Cognito directly into our Load Balancer in the next step.

### Step 1: Create User Pool
- On the **Amazon Cognito** screen, select **User pools**.
- Click **Create user pool**.

![Create User Pool](/images/5-Workshop/5.4-backend-ecs/5.4.1-cognito-ssm/Cognito0.png)

### Step 2: Basic Configuration
Fill in the parameters as follows:
- **Application type:** Select **Traditional web application**.
- **Name your application:** Enter any name, e.g., `My web app`.
- **Options for sign-in identifiers:** Check **Email** and **Username**.
- **Self-registration:** Check **Enable self-registration**.
- **Required attributes for sign-up:** Check **Email** and **name**.
- Finally, click **Create user directory**.

![Cognito Step 1](/images/5-Workshop/5.4-backend-ecs/5.4.1-cognito-ssm/Cognito1.png)

![Cognito Step 2](/images/5-Workshop/5.4-backend-ecs/5.4.1-cognito-ssm/Cognito2.png)

### Step 3: Setup Resources
- On the **Set up resources for your application** screen.
- Under the **Quick setup guide**, select **NodeJS** as the language -> Then click **Go to overview**.

![Cognito Setup Resources 1](/images/5-Workshop/5.4-backend-ecs/5.4.1-cognito-ssm/Cognito3.png)

![Cognito Setup Resources 2](/images/5-Workshop/5.4-backend-ecs/5.4.1-cognito-ssm/Cognito4.png)

### Step 4: Configure App Client & Hosted UI
To integrate with the Application Load Balancer (ALB), you must configure the Hosted UI.
Navigate to the newly created User Pool, switch to the **App integration** tab, and scroll down to **App clients**.
Click **Create app client**
- **Application type:** Select **Traditional web application**.
- **Name your application:** Enter any name, e.g., `My web app`.
- **Add a return URL**: Enter `Your domain name`
- Click **Create app client**

![Cognito Setup Resources 2](/images/5-Workshop/5.4-backend-ecs/5.4.1-cognito-ssm/Cognito5.png)

Click on your App Client -> **Edit**:

![Cognito Setup Resources 2](/images/5-Workshop/5.4-backend-ecs/5.4.1-cognito-ssm/Cognito6.png)

- Under **Auth flows**, ensure you have selected all 3 permissions: `ALLOW_USER_PASSWORD_AUTH`, `ALLOW_REFRESH_TOKEN_AUTH`, `ALLOW_USER_SRP_AUTH`.

![Cognito Setup Resources 2](/images/5-Workshop/5.4-backend-ecs/5.4.1-cognito-ssm/Cognito7.png)

### Step 5: Create Cognito Domain
- Still in the **App integration** tab, locate the **Domain** section.
- Register a default Cognito **Domain Name** (or use your custom domain).

---

# 2. Store Environment Variables (SSM Parameter Store)

Instead of hardcoding sensitive information (like JWT secrets, payment API Keys) directly into the Backend source code, we will securely store them in **AWS Systems Manager (SSM) Parameter Store**.

Navigate to **Systems Manager > Parameter Store**, and click **Create parameter** to create the following environment variables one by one:

*Note: You must select **SecureString** as the Type so AWS encrypts the content.*

1. Name: `/app/env/jwt_secret` | Type: `SecureString` | Value: *(Enter your secret code)*
2. Name: `/app/env/PAYOS_CLIENT_ID` | Type: `SecureString` | Value: *(Key of PayOS payment gateway)*
3. Name: `/app/env/PAYOS_API_KEY` | Type: `SecureString` | Value: *(API Key of PayOS payment gateway)*
4. Name: `/app/env/PAYOS_CHECKSUM_KEY` | Type: `SecureString` | Value: *(Checksum Key of PayOS)*

![SSM Parameter Store](/images/5-Workshop/5.4-backend-ecs/5.4.1-cognito-ssm/SSM.png)

*💡 Thanks to the `ecsInstanceRole` (with the `Zopee-Backend-Permission` Policy) we set up in section 5.2, the Backend Node.js code will have the authority to automatically fetch these environment variables during startup!*


