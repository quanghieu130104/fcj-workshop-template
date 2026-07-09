---
title: "Blog 2"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 3.2. </b> "
---
# Web Search Trên Amazon Bedrock AgentCore: AI Agent Giờ Đã Tự Tra Cứu Được Web Mà Dữ Liệu Không Rời Khỏi AWS

Chào mọi người, trong quá trình tìm hiểu về Agentic AI trên AWS, mình thấy AWS vừa công bố tính năng **Web Search trên Amazon Bedrock AgentCore** đạt trạng thái General Availability. Đây là công cụ tìm kiếm web được quản lý hoàn toàn (fully managed), cho phép AI agent tự tra cứu thông tin mới nhất trên Internet mà toàn bộ truy vấn vẫn nằm trong môi trường AWS của khách hàng. Mình xin tổng hợp và chia sẻ các điểm cốt lõi để cộng đồng cùng tham khảo.

## 1. Amazon Bedrock AgentCore là gì?

Amazon Bedrock AgentCore là nền tảng của AWS dành cho việc xây dựng, triển khai và vận hành AI agent ở quy mô production. Thay vì để developer tự lo phần hạ tầng (runtime, memory, identity, quan sát hành vi agent...), AgentCore cung cấp sẵn các dịch vụ này dưới dạng managed service và hoạt động với mọi framework mã nguồn mở phổ biến như Strands Agents, LangGraph, CrewAI hay LlamaIndex, với bất kỳ foundation model nào.

Một vấn đề cố hữu của AI agent là **kiến thức của model bị "đóng băng" tại thời điểm huấn luyện**. Agent tư vấn tài chính không biết tin tức sáng nay, agent hỗ trợ khách hàng không biết chính sách vừa thay đổi tuần trước. Trước đây, muốn agent tra cứu được web, developer phải tự tích hợp API search của bên thứ ba — và đó chính là chỗ tính năng mới này nhắm đến.

## 2. Điểm mới của tính năng

Web Search được cung cấp dưới dạng một **connector target dựng sẵn trên AgentCore Gateway**, giao tiếp qua chuẩn Model Context Protocol (MCP). Agent chỉ cần gửi truy vấn bằng ngôn ngữ tự nhiên, Web Search sẽ trả về các đoạn trích (snippet) liên quan nhất kèm URL nguồn, tiêu đề và ngày đăng — đủ ngữ liệu để model suy luận và tạo ra câu trả lời có căn cứ (grounded) kèm trích dẫn.

![Kiến trúc Web Search trên AgentCore](images/agentcore-websearch-architecture.png)

Các đặc điểm kỹ thuật nổi bật:

- **Xây trên hạ tầng search của chính Amazon**: cùng nền tảng tìm kiếm đang vận hành cho Alexa+, Amazon Quick và Kiro, được tối ưu cho agentic retrieval — trả về các đoạn trích giá trị cao thay vì danh sách link thô, giúp "nhiều trí tuệ hơn trên mỗi token".
- **Grounding đa nguồn**: kết hợp web index công khai với Amazon Knowledge Graph — dữ liệu thực thể có cấu trúc và các thông tin đã xác minh, kể cả dữ liệu thời gian thực như giá cổ phiếu hay tỷ số thể thao.
- **Zero data egress**: prompt của người dùng và truy vấn tìm kiếm không bị gửi ra nhà cung cấp search bên ngoài AWS — điểm khác biệt lớn nhất so với việc tự tích hợp Google/Bing API.
- **Chuẩn MCP**: vì đi qua Gateway theo giao thức mở, tính năng này hoạt động với mọi framework agent, không khóa chặt vào một SDK riêng.

## 3. Ứng dụng cho thực tế

So với cách làm cũ, giá trị vận hành là rất rõ ràng:

![So sánh tự tích hợp vs managed Web Search](images/agentcore-websearch-before-after.png)

- **Bỏ được toàn bộ "plumbing"**: không cần đăng ký vendor search riêng, không cần tự viết logic orchestration, xác thực, retry và billing cho việc tìm kiếm.
- **Đáp ứng yêu cầu tuân thủ**: với các ngành như tài chính, y tế, bảo hiểm — nơi việc gửi prompt của người dùng ra dịch vụ bên ngoài là rào cản pháp lý — kiến trúc zero egress giúp bài toán governance đơn giản đi đáng kể.
- **Câu trả lời có trích dẫn**: URL nguồn và ngày đăng đi kèm mỗi kết quả giúp giảm ảo giác (hallucination) và tăng độ tin cậy khi agent trả lời về thông tin thời sự.
- **Use case điển hình**: agent nghiên cứu thị trường cần tin tức mới, chatbot hỗ trợ cần tra thông số sản phẩm vừa ra mắt, agent tạo nội dung cần bám theo sự kiện đang diễn ra.

Một khách hàng thực tế được AWS dẫn chứng là Gen Digital (công ty mẹ của Norton): họ dùng Web Search để sản phẩm Norton Revamp gợi ý nội dung xây dựng thương hiệu cá nhân dựa trên những gì đang thực sự diễn ra, và đánh giá cao việc truy vấn không rời khỏi môi trường AWS tin cậy của họ.

## 4. Hạn chế và lưu ý triển khai

- **Vùng khả dụng còn hẹp**: tại thời điểm GA, Web Search mới có ở region US East (N. Virginia). Anh em ở Việt Nam dùng ap-southeast-1 cần theo dõi lộ trình mở rộng region, hoặc chấp nhận gọi cross-region với độ trễ cao hơn.
- **Không thay thế Knowledge Base**: Web Search giải quyết bài toán kiến thức công khai và thời sự; với dữ liệu nội bộ doanh nghiệp, vẫn cần kết hợp với Bedrock Knowledge Bases (RAG) — hai mảnh ghép này bổ trợ chứ không loại trừ nhau.
- **Chi phí theo mức sử dụng**: mô hình pay-as-you-go không cần cam kết trước, nhưng agent có thể tự quyết định tìm kiếm nhiều lần trong một phiên, nên cần giới hạn số vòng lặp tool-use và giám sát chi phí như mọi workload agentic khác.
- **Vẫn cần đánh giá chất lượng**: kết quả có trích dẫn giúp kiểm chứng dễ hơn, nhưng agent dùng nguồn web công khai thì bài toán lọc nguồn kém chất lượng vẫn thuộc trách nhiệm thiết kế của bạn (có thể kết hợp AgentCore Evaluations và Guardrails).

## 5. Đánh giá cá nhân

Theo mình, đây là mảnh ghép mà những ai từng tự build agent "biết search" sẽ thấy giá trị ngay lập tức. Phần khó của tính năng này chưa bao giờ là gọi một API tìm kiếm — mà là chuỗi việc không tên phía sau: quản lý key, parse kết quả, xếp hạng lại, và đau đầu nhất là giải trình với bộ phận bảo mật về việc dữ liệu người dùng đi đâu. Việc AWS gói tất cả vào một connector target chuẩn MCP, chạy trên chính hạ tầng search của Amazon, là cách tiếp cận "đúng bài" cho môi trường doanh nghiệp.

Điểm mình sẽ theo dõi tiếp là chất lượng kết quả tìm kiếm so với các search API chuyên dụng, và tốc độ mở rộng region — vì với ứng dụng chat thời gian thực, một vòng tra cứu xuyên bán cầu có thể ảnh hưởng trải nghiệm rõ rệt.

## Kết luận

Web Search trên Amazon Bedrock AgentCore là bước tiến quan trọng giúp AI agent thoát khỏi giới hạn kiến thức tại thời điểm huấn luyện mà không đánh đổi bảo mật dữ liệu. Với cách tích hợp qua chuẩn MCP, grounding đa nguồn kèm trích dẫn và kiến trúc zero data egress, đây sẽ là lựa chọn mặc định đáng cân nhắc cho bất kỳ ai đang đưa agent từ demo lên production trên AWS. Mình tin trong thời gian tới, "agent có khả năng tra cứu web an toàn" sẽ trở thành tiêu chuẩn chứ không còn là tính năng cộng thêm.

Hy vọng những tóm tắt này giúp các bạn có cái nhìn nhanh về hướng đi của AWS trong mảng Agentic AI!

**Nguồn tham khảo:**
- https://aws.amazon.com/blogs/aws/announcing-web-search-on-a…ore-ground-your-ai-agents-in-current-accurate-web-knowledge/
- https://aws.amazon.com/blogs/machine-learning/new-in-amazon…build-agents-with-broader-knowledge-and-continuous-learning/