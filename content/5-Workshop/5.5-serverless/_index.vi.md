---
title: "Tích hợp Serverless Event-Driven"
date: 2026-07-08
weight: 5
chapter: false
pre: " <b> 5.5. </b> "
---


Một trong những ưu điểm lớn nhất của kiến trúc đám mây là khả năng xử lý sự kiện không đồng bộ (Asynchronous Event-Driven). Thay vì bắt máy chủ Backend (EC2) phải chờ đợi việc gửi Email xác nhận đơn hàng rườm rà (làm giảm hiệu năng phản hồi API), chúng ta sẽ đẩy công việc này cho các dịch vụ Serverless.

Cụ thể, khi có đơn hàng mới được lưu vào bảng **Orders** trong DynamoDB, một dòng sự kiện (Stream) sẽ tự động kích hoạt **AWS Lambda**. Lambda sẽ đọc thông tin đơn hàng và dùng **Amazon SES** để gửi email cho khách hàng.

### Bước 1: Xác thực Email gửi đi với Amazon SES
Để AWS cho phép bạn gửi email, bạn cần xác thực một địa chỉ email sở hữu.
1. Truy cập dịch vụ **Amazon SES** (Simple Email Service).
2. Chọn **Identities** > Bấm **Create identity**.
3. **Identity type:** Chọn **Email address**.
4. **Email address:** Nhập địa chỉ email cá nhân của bạn.
5. Bấm **Create identity**.
6. Mở hộp thư email của bạn, tìm email từ AWS và bấm vào link xác nhận. *Lưu ý: Do SES đang ở chế độ Sandbox (mặc định), bạn chỉ có thể gửi email ĐẾN các địa chỉ cũng đã được xác thực.*

![SES](/images/5-Workshop/5.5-serverless/SES.png)

### Bước 2: Bật DynamoDB Streams cho bảng Orders
1. Truy cập dịch vụ **DynamoDB** > **Tables**.
2. Bấm vào bảng `Orders`
3. Chuyển sang tab **Exports and streams**.
4. Kéo xuống phần **DynamoDB stream details**, bấm **Turn on**.
5. Chọn **New and old images** (Để lấy cả dữ liệu đơn hàng mới tạo và dữ liệu cũ).
6. Bấm **Turn on stream**.

![DynamoDB_Stream](/images/5-Workshop/5.5-serverless/DynamoDB_Stream.png)

### Bước 3: Tạo AWS Lambda Function
1. Truy cập dịch vụ **AWS Lambda** > **Functions** > Bấm **Create function**.
2. Chọn **Author from scratch**.
3. **Function name:** Đặt tên, ví dụ `OrderEmailWorker`.
4. **Runtime:** Chọn `Node.js 24.x`.
5. Bật **Additional settings** > Bật **custom execution role**.
    - 5.1. **Custom execution role:** Chọn `lambda-dynamodb-ses-role` mà bạn đã tạo ở bài 5.2.
    - **Save**.

6. Bấm **Create function**.

![Lambda](/images/5-Workshop/5.5-serverless/Lambda.png)

- **Tại tab configuration của lambda**
7. **Trigger:** bấm **+ Add trigger**.
8. **Trigger configuration:** Tìm và chọn **DynamoDB**.
9. **DynamoDB table:** Chọn bảng `Orders`.
10. Các thông số khác (Batch size = 1, Starting position = Latest) giữ nguyên mặc định.
11. Bấm **Add**.

![Trigger](/images/5-Workshop/5.5-serverless/Trigger.png)

![Trigger](/images/5-Workshop/5.5-serverless/Trigger1.png)

![Trigger](/images/5-Workshop/5.5-serverless/Trigger2.png)

### Bước 4: Viết mã nguồn cho Lambda
Tại giao diện hàm **Lambda** vừa tạo, chuyển sang tab **Code**. Chuẩn bị mã nguồn Lambda gồm các tệp `index.js` và `template.js`, sau đó cài đặt các thư viện phụ thuộc bằng `npm install`.

Đóng gói toàn bộ thư mục dự án Lambda (bao gồm `index.js`, `template.js`, `package.json`, `package-lock.json` và `node_modules`) thành tệp **.zip**.

Trong phần **Code source**, chọn **Upload from** → **.zip file**, tải tệp `.zip` vừa tạo lên và nhấn **Deploy** để cập nhật mã nguồn cho Lambda.

 File `index.js`
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
                        Data: `[Zopee] Xác nhận đơn hàng thành công #${orderId}`,
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


File `template.js`
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
            Số lượng: ${item.quantity}
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
    <html lang="vi">
    <head>
      <meta charset="UTF-8">
      <meta name="viewport" content="width=device-width, initial-scale=1.0">
      <title>Xác Nhận Đơn Hàng từ Zopee</title>
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
                  <p style="color: rgba(255,255,255,0.9); margin: 5px 0 0 0; font-size: 14px; font-weight: 500;">Cảm ơn bạn đã mua sắm tại Zopee!</p>
                </td>
              </tr>
              <!-- Body -->
              <tr>
                <td style="padding: 30px;">
                  <h2 style="color: #1a202c; margin: 0 0 10px 0; font-size: 20px; font-weight: 700;">Xác Nhận Đơn Hàng Thành Công 🎉</h2>
                  <p style="color: #4a5568; line-height: 1.6; margin: 0 0 25px 0; font-size: 14px;">
                    Chào quý khách, đơn hàng của quý khách đã được lưu nhận trên hệ thống của Zopee. Chúng tôi sẽ nhanh chóng xử lý và bàn giao cho đơn vị vận chuyển.
                  </p>
                  
                  <!-- Order Info -->
                  <table width="100%" border="0" cellspacing="0" cellpadding="0" style="background-color: #f8fafc; border-radius: 8px; padding: 20px; margin-bottom: 25px; border: 1px solid #edf2f7;">
                    <tr>
                      <td style="padding-bottom: 8px; color: #718096; font-size: 13px;">Mã đơn hàng:</td>
                      <td style="padding-bottom: 8px; color: #1a202c; font-size: 13px; font-weight: 700; text-align: right;">${orderId}</td>
                    </tr>
                    <tr>
                      <td style="color: #718096; font-size: 13px;">Ngày đặt hàng:</td>
                      <td style="color: #1a202c; font-size: 13px; font-weight: 500; text-align: right;">${new Date().toLocaleDateString("vi-VN")}</td>
                    </tr>
                  </table>

                  <!-- Items Table -->
                  <h3 style="color: #2d3748; margin: 0 0 15px 0; font-size: 16px; font-weight: 700; border-bottom: 2px solid #edf2f7; padding-bottom: 8px;">Chi tiết sản phẩm</h3>
                  <table width="100%" border="0" cellspacing="0" cellpadding="0" style="margin-bottom: 25px;">
                    ${itemsHtml}
                    <!-- Total -->
                    <tr>
                      <td style="padding: 20px 0 0 0; font-size: 16px; font-weight: 700; color: #1a202c;">Tổng cộng:</td>
                      <td style="padding: 20px 0 0 0; font-size: 20px; font-weight: 800; color: #ff4e2e; text-align: right;">
                        ${formatPrice(totalAmount)}
                      </td>
                    </tr>
                  </table>

                  <p style="color: #718096; font-size: 12px; line-height: 1.5; margin: 0; text-align: center; border-top: 1px solid #edf2f7; padding-top: 20px;">
                    Nếu bạn có bất kỳ câu hỏi nào, vui lòng liên hệ với bộ phận hỗ trợ khách hàng của Zopee bằng cách trả lời email này.
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
Nhấn **Deploy** để lưu lại code.
