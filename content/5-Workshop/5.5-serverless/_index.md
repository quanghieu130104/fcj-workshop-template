---
title: "Serverless Event-Driven Integration"
date: 2026-07-08
weight: 5
chapter: false
pre: " <b> 5.5. </b> "
---

One of the greatest advantages of cloud computing is the ability to process events asynchronously (Asynchronous Event-Driven). Instead of forcing the Backend server (EC2) to wait for a cumbersome order confirmation email to be sent (which slows down API response times), we will offload this task to Serverless services.

Specifically, when a new order is saved into the **Orders** table in DynamoDB, an event stream will automatically trigger **AWS Lambda**. The Lambda function will read the order details and use **Amazon SES** to send an email to the customer.

### Step 1: Verify Sending Email with Amazon SES
For AWS to allow you to send emails, you must verify an email address that you own.
1. Navigate to the **Amazon SES** (Simple Email Service) console.
2. Select **Identities** > Click **Create identity**.
3. **Identity type:** Select **Email address**.
4. **Email address:** Enter your personal email address.
5. Click **Create identity**.
6. Open your email inbox, find the email from AWS, and click the confirmation link. *Note: Because SES is in Sandbox mode by default, you can only send emails TO addresses that have also been verified.*

![SES](/images/5-Workshop/5.5-serverless/SES.png)

### Step 2: Enable DynamoDB Streams for the Orders table
1. Navigate to **DynamoDB** > **Tables**.
2. Click on the `Orders` table
3. Switch to the **Exports and streams** tab.
4. Scroll down to the **DynamoDB stream details** section, click **Turn on**.
5. Select **New and old images** (To retrieve both the newly created order data and the old data).
6. Click **Turn on stream**.

![DynamoDB_Stream](/images/5-Workshop/5.5-serverless/DynamoDB_Stream.png)

### Step 3: Create an AWS Lambda Function
1. Navigate to **AWS Lambda** > **Functions** > Click **Create function**.
2. Select **Author from scratch**.
3. **Function name:** Name it, e.g., `OrderEmailWorker`.
4. **Runtime:** Select `Node.js 24.x`.
5. Enable **Additional settings** > Enable **custom execution role**.
    - 5.1. **Custom execution role:** Select the `lambda-dynamodb-ses-role` you created in section 5.2.
    - **Save**.

6. Click **Create function**.

![Lambda](/images/5-Workshop/5.5-serverless/Lambda.png)

- **In the Lambda configuration tab**
7. **Trigger:** click **+ Add trigger**.
8. **Trigger configuration:** Search and select **DynamoDB**.
9. **DynamoDB table:** Select the `Orders` table.
10. Leave other parameters (Batch size = 1, Starting position = Latest) as default.
11. Click **Add**.

![Trigger](/images/5-Workshop/5.5-serverless/Trigger.png)

![Trigger](/images/5-Workshop/5.5-serverless/Trigger1.png)

![Trigger](/images/5-Workshop/5.5-serverless/Trigger2.png)

### Step 4: Write the Lambda Source Code
In the **Lambda** interface you just created, switch to the **Code** tab. Prepare the Lambda source code including the `index.js` and `template.js` files, then install dependencies using `npm install`.

Package the entire Lambda project directory (including `index.js`, `template.js`, `package.json`, `package-lock.json`, and `node_modules`) into a **.zip** file.

In the **Code source** section, select **Upload from** → **.zip file**, upload the `.zip` file you just created, and click **Deploy** to update the Lambda source code.

 `index.js` File
```javascript
"use strict";
Object.defineProperty(exports, "__esModule", { value: true });
exports.handler = void 0;
const util_dynamodb_1 = require("@aws-sdk/util-dynamodb");
const client_ses_1 = require("@aws-sdk/client-ses");
const template_1 = require("./template");
const ses = new client_ses_1.SESClient({ region: process.env.AWS_REGION || "ap-southeast-1" });
const handler = async (event, context) => {
    const requestId = context.awsRequestId;
    console.info(`[LambdaRequestId: ${requestId}] Starting processing of ${event.Records.length} records.`);
    try {
        for (const record of event.Records) {
            // We only care about new orders being created (INSERT events)
            if (record.eventName !== "INSERT") {
                console.info(`[LambdaRequestId: ${requestId}] Skipping record with eventName: ${record.eventName}`);
                continue;
            }
            const dynamodbImage = record.dynamodb?.NewImage;
            if (!dynamodbImage) {
                console.warn(`[LambdaRequestId: ${requestId}] Record has no NewImage data. Skipping.`);
                continue;
            }
            // Convert DynamoDB AttributeMap to normal Javascript Object
            const order = (0, util_dynamodb_1.unmarshall)(dynamodbImage);
            const orderId = order.id || order.Id;
            const customerEmail = order.customerEmail;
            const items = (order.items || []);
            const totalAmount = Number(order.totalAmount || 0);
            console.info(`[LambdaRequestId: ${requestId}] Processing order: ${orderId} for customer email: ${customerEmail}`);
            if (!customerEmail) {
                console.warn(`[LambdaRequestId: ${requestId}] Order ${orderId} is missing customerEmail. Cannot send confirmation email.`);
                continue;
            }
            const sourceEmail = process.env.SOURCE_EMAIL;
            if (!sourceEmail) {
                throw new Error("SOURCE_EMAIL environment variable is not set.");
            }
            // Generate HTML Template
            const htmlBody = (0, template_1.getOrderConfirmationHtml)(orderId, items, totalAmount);
            // Construct SendEmailCommand
            const sendEmailCmd = new client_ses_1.SendEmailCommand({
                Source: sourceEmail,
                Destination: {
                    ToAddresses: [customerEmail],
                },
                Message: {
                    Subject: {
                        Charset: "UTF-8",
                        Data: `[Zopee] Order Confirmation #${orderId}`,
                    },
                    Body: {
                        Html: {
                            Charset: "UTF-8",
                            Data: htmlBody,
                        },
                    },
                },
            });
            console.info(`[LambdaRequestId: ${requestId}] Sending email via SES to: ${customerEmail} from: ${sourceEmail}`);
            const sesResponse = await ses.send(sendEmailCmd);
            console.info(`[LambdaRequestId: ${requestId}] Email sent successfully for Order: ${orderId}. SES MessageId: ${sesResponse.MessageId}`);
        }
        console.info(`[LambdaRequestId: ${requestId}] Finished processing all records.`);
    }
    catch (error) {
        console.error(`[LambdaRequestId: ${requestId}] Error processing database stream records:`, error);
        // Throw error so AWS Lambda registers this invocation as failed and triggers retry logic
        throw error;
    }
};
exports.handler = handler;
```


`template.js` File
```javascript
"use strict";
Object.defineProperty(exports, "__esModule", { value: true });
exports.getOrderConfirmationHtml = getOrderConfirmationHtml;
function getOrderConfirmationHtml(orderId, items, totalAmount) {
    const formatPrice = (price) => {
        return new Intl.NumberFormat("vi-VN", {
            style: "currency",
            currency: "VND",
        }).format(price);
    };
    const itemsHtml = items
        .map((item) => `
      <tr style="border-bottom: 1px solid #e0e0e0;">
        <td style="padding: 12px 0; color: #333333; font-size: 14px;">
          <strong>${item.name}</strong>
          <div style="font-size: 12px; color: #666666; margin-top: 4px;">
            Quantity: ${item.quantity}
          </div>
        </td>
        <td style="padding: 12px 0; text-align: right; color: #333333; font-size: 14px; font-weight: 600;">
          ${formatPrice(item.price * item.quantity)}
        </td>
      </tr>
    `)
        .join("");
    return `
    <!DOCTYPE html>
    <html lang="en">
    <head>
      <meta charset="UTF-8">
      <meta name="viewport" content="width=device-width, initial-scale=1.0">
      <title>Order Confirmation from Zopee</title>
    </head>
    <body style="margin: 0; padding: 0; background-color: #f4f7f6; font-family: 'Segoe UI', Roboto, Helvetica, Arial, sans-serif; -webkit-font-smoothing: antialiased;">
      <table width="100%" border="0" cellspacing="0" cellpadding="0" style="background-color: #f4f7f6; padding: 20px 0;">
        <tr>
          <td align="center">
            <table width="600" border="0" cellspacing="0" cellpadding="0" style="background-color: #ffffff; border-radius: 12px; overflow: hidden; box-shadow: 0 4px 15px rgba(0,0,0,0.05); border: 1px solid #e2e8f0;">
              <!-- Header -->
              <tr>
                <td style="background: linear-gradient(135deg, #ff4e2e 0%, #ff7a00 100%); padding: 30px; text-align: center;">
                  <h1 style="color: #ffffff; margin: 0; font-size: 28px; font-weight: 800; letter-spacing: 0.5px;">ZOPEE</h1>
                  <p style="color: rgba(255,255,255,0.9); margin: 5px 0 0 0; font-size: 14px; font-weight: 500;">Thank you for shopping at Zopee!</p>
                </td>
              </tr>
              <!-- Body -->
              <tr>
                <td style="padding: 30px;">
                  <h2 style="color: #1a202c; margin: 0 0 10px 0; font-size: 20px; font-weight: 700;">Order Confirmation Successful 🎉</h2>
                  <p style="color: #4a5568; line-height: 1.6; margin: 0 0 25px 0; font-size: 14px;">
                    Dear customer, your order has been successfully recorded in the Zopee system. We will promptly process and hand it over to the shipping provider.
                  </p>
                  
                  <!-- Order Info -->
                  <table width="100%" border="0" cellspacing="0" cellpadding="0" style="background-color: #f8fafc; border-radius: 8px; padding: 20px; margin-bottom: 25px; border: 1px solid #edf2f7;">
                    <tr>
                      <td style="padding-bottom: 8px; color: #718096; font-size: 13px;">Order ID:</td>
                      <td style="padding-bottom: 8px; color: #1a202c; font-size: 13px; font-weight: 700; text-align: right;">${orderId}</td>
                    </tr>
                    <tr>
                      <td style="color: #718096; font-size: 13px;">Order Date:</td>
                      <td style="color: #1a202c; font-size: 13px; font-weight: 500; text-align: right;">${new Date().toLocaleDateString("en-US")}</td>
                    </tr>
                  </table>

                  <!-- Items Table -->
                  <h3 style="color: #2d3748; margin: 0 0 15px 0; font-size: 16px; font-weight: 700; border-bottom: 2px solid #edf2f7; padding-bottom: 8px;">Product Details</h3>
                  <table width="100%" border="0" cellspacing="0" cellpadding="0" style="margin-bottom: 25px;">
                    ${itemsHtml}
                    <!-- Total -->
                    <tr>
                      <td style="padding: 20px 0 0 0; font-size: 16px; font-weight: 700; color: #1a202c;">Total:</td>
                      <td style="padding: 20px 0 0 0; font-size: 20px; font-weight: 800; color: #ff4e2e; text-align: right;">
                        ${formatPrice(totalAmount)}
                      </td>
                    </tr>
                  </table>

                  <p style="color: #718096; font-size: 12px; line-height: 1.5; margin: 0; text-align: center; border-top: 1px solid #edf2f7; padding-top: 20px;">
                    If you have any questions, please contact Zopee customer support by replying to this email.
                  </p>
                </td>
              </tr>
              <!-- Footer -->
              <tr>
                <td style="background-color: #f7fafc; padding: 20px; text-align: center; border-top: 1px solid #edf2f7;">
                  <p style="margin: 0; color: #a0aec0; font-size: 12px;">© ${new Date().getFullYear()} Zopee Corporation. All rights reserved.</p>
                </td>
              </tr>
            </table>
          </td>
        </tr>
      </table>
    </body>
    </html>
  `;
}

```
Click **Deploy** to save the code.
