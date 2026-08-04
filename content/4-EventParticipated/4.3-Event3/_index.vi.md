---
title: "Event 3 - AgentForge Deep Dive"
date: 2026-07-21
weight: 3
chapter: false
pre: " <b> 4.3. </b> "
---

# AgentForge Deep Dive – Day 1

## Tổng quan sự kiện

**AgentForge Deep Dive** là chuỗi workshop chuyên sâu hướng dẫn người tham dự tìm hiểu và xây dựng các hệ thống Agentic AI có khả năng triển khai trong môi trường thực tế bằng **Amazon Bedrock AgentCore**.

Khác với một chatbot chỉ tiếp nhận câu hỏi và tạo phản hồi, một hệ thống Agentic AI có thể phân tích mục tiêu, lập kế hoạch, sử dụng công cụ, truy xuất dữ liệu và thực hiện hành động thay cho người dùng trong phạm vi được cho phép.

Sự kiện kết hợp giữa phần trình bày lý thuyết và các bài thực hành có hướng dẫn. Qua đó, người tham dự không chỉ tìm hiểu cách AI Agent hoạt động mà còn được chuẩn bị môi trường phát triển, xây dựng một Agent cơ bản và từng bước tiếp cận quy trình triển khai Agent lên AWS.

![Giới thiệu sự kiện AgentForge Deep Dive](/images/4-EventParticipated/4.3-Event3/01-agentforge-introduction.png)

*Các diễn giả giới thiệu AgentForge và mục tiêu xây dựng hệ thống Agentic AI sẵn sàng cho môi trường thực tế bằng Amazon Bedrock AgentCore.*

## Lộ trình ba ngày của AgentForge

AgentForge được thiết kế thành chương trình kéo dài ba ngày. Mỗi ngày kết hợp giữa lý thuyết và Hands-on Lab với mức độ chuyên sâu tăng dần.

![Lộ trình ba ngày của AgentForge](/images/4-EventParticipated/4.3-Event3/02-three-day-agenda.png)

*Lộ trình tổng quan của chương trình AgentForge trong ba ngày.*

### Day 1 – Nền tảng Agentic AI

Nội dung của ngày đầu tiên tập trung vào:

- Giới thiệu Agentic AI.
- Tìm hiểu Amazon Bedrock AgentCore ở mức nền tảng.
- Tìm hiểu AgentCore Runtime, Gateway và Identity.
- Xây dựng và triển khai một AI Agent cơ bản.
- Kết nối Agent với công cụ và nguồn kiến thức bên ngoài.
- Tìm hiểu hướng tích hợp giao diện web và xác thực người dùng.

### Day 2 – Mở rộng khả năng của Agent

Theo lộ trình được giới thiệu, Day 2 tập trung vào:

- AgentCore Memory.
- AgentCore Evaluations.
- AgentCore Observability.
- AgentCore Registry.
- AgentCore Tools và các cơ chế tối ưu hóa.
- Theo dõi hiệu suất và bổ sung bộ nhớ để cá nhân hóa Agent.

### Day 3 – DevOps và bảo mật

Day 3 dự kiến tập trung vào:

- Trường hợp sử dụng DevOps với Amazon Bedrock AgentCore.
- Các phương pháp xây dựng Agentic System.
- Sử dụng AgentCore Policy để bảo vệ các lời gọi công cụ.
- Các bài thực hành mở rộng.

Trong phạm vi sự kiện này, em mới tham gia và hoàn thành nội dung của **Day 1**. Nội dung Day 2 và Day 3 được giới thiệu trong lộ trình nhưng chưa nằm trong phần thực hành của em.

## Phần 1 – Kiến thức nền tảng về Agentic AI

### Agentic AI là gì?

Agentic AI được giới thiệu là các hệ thống phần mềm có khả năng tự chủ một phần hoặc toàn phần, sử dụng AI để **suy luận, lập kế hoạch và hoàn thành nhiệm vụ** thay cho con người hoặc một hệ thống khác.

![Khái niệm Agentic AI](/images/4-EventParticipated/4.3-Event3/03-what-is-agentic-ai.png)

*Agentic AI sử dụng AI để suy luận, lập kế hoạch và hoàn thành nhiệm vụ với mức độ tự chủ khác nhau.*

Một Agent có thể tiếp nhận mục tiêu từ người dùng, phân tích yêu cầu, lựa chọn công cụ phù hợp và thực hiện nhiều bước để tạo ra kết quả cuối cùng.

Tuy nhiên, mức độ tự chủ của Agent phải được thiết kế phù hợp với từng trường hợp sử dụng. Không phải hệ thống nào cũng cần hoặc nên hoạt động hoàn toàn tự động.

### Các thành phần cơ bản của một AI Agent

Một AI Agent cơ bản có thể gồm những thành phần sau:

- **Goal:** Mục tiêu mà Agent cần hoàn thành.
- **LLM:** Thành phần trung tâm hỗ trợ hiểu yêu cầu và suy luận.
- **Tools:** Các công cụ hoặc API mà Agent có thể sử dụng.
- **Memory:** Bộ nhớ giúp lưu lại thông tin hoặc ngữ cảnh.
- **Data:** Dữ liệu cần thiết cho quá trình xử lý.
- **Actions:** Những hành động Agent có thể thực hiện.
- **Observability:** Khả năng theo dõi hoạt động và kết quả của Agent.
- **Guardrails:** Các quy tắc giúp giới hạn và kiểm soát hành vi của Agent.
- **User Interaction:** Kênh tương tác giữa người dùng với Agent.

![Các thành phần cơ bản của một AI Agent](/images/4-EventParticipated/4.3-Event3/04-basic-agent-components.png)

*Một Agent kết hợp mục tiêu, bộ nhớ, dữ liệu, công cụ và mô hình ngôn ngữ để thực hiện hành động.*

Qua sơ đồ này, em hiểu rằng mô hình ngôn ngữ chỉ là một thành phần của hệ thống. Để xây dựng một Agent có thể hoạt động trong thực tế, cần bổ sung công cụ, dữ liệu, bộ nhớ, cơ chế giám sát và các giới hạn an toàn.

### Các mức độ tự chủ

Diễn giả giới thiệu các mức độ tự chủ khác nhau của hệ thống AI:

1. **Simple Assistants:** Trả lời câu hỏi và thực hiện các tác vụ đơn giản trong một bước.
2. **Deterministic Agents:** Hoạt động theo quy trình và kế hoạch đã được xác định trước.
3. **Autonomous Agents:** Có khả năng lập kế hoạch trong thời gian chạy và thực hiện nhiệm vụ gồm nhiều bước.
4. **Agentic Virtual Workers:** Phối hợp nhiều Agent, làm việc trong thời gian dài và mô phỏng cách phối hợp của một nhóm con người.

![Các mức độ tự chủ của hệ thống AI](/images/4-EventParticipated/4.3-Event3/05-autonomy-gradient.png)

*Các mức độ tự chủ tăng dần từ trợ lý đơn giản đến Agentic Virtual Workers.*

Một bài học quan trọng em tiếp thu được là không phải lúc nào cũng cần xây dựng Agent hoàn toàn tự chủ. Mức độ tự chủ nên được lựa chọn dựa trên yêu cầu nghiệp vụ, mức độ rủi ro và khả năng kiểm soát của hệ thống.

### Vòng lặp hoạt động của Agent

Phần tiếp theo giới thiệu vòng lặp Agent cơ bản trong **Strands Agents**.

![Vòng lặp hoạt động cơ bản của Agent](/images/4-EventParticipated/4.3-Event3/06-strands-agent-loop.png)

*Vòng lặp xử lý giữa người dùng, Agent, mô hình và các công cụ.*

Quy trình hoạt động có thể được mô tả như sau:

1. Người dùng gửi prompt đến Agent.
2. Agent chuyển thông tin cần thiết đến mô hình.
3. Mô hình phân tích yêu cầu và lựa chọn công cụ nếu cần.
4. Agent thực thi công cụ được lựa chọn.
5. Kết quả từ công cụ được đưa trở lại mô hình.
6. Quá trình tiếp tục cho đến khi Agent có đủ thông tin.
7. Agent trả về câu trả lời cuối cùng cho người dùng.

Qua phần này, em hiểu rõ hơn rằng AI Agent không chỉ gửi một prompt đến mô hình rồi nhận câu trả lời. Agent có thể thực hiện nhiều vòng suy luận và sử dụng công cụ trước khi tạo ra kết quả cuối cùng.

## Phần 2 – Tổng quan về Amazon Bedrock AgentCore

### Vì sao Agent cần môi trường vận hành?

Khi một Agent được đưa vào sử dụng thực tế, hệ thống cần có một môi trường an toàn để:

- Chạy mã nguồn của Agent.
- Kết nối với mô hình AI.
- Truy cập công cụ và API.
- Sử dụng dữ liệu có cấu trúc và Knowledge Base.
- Quản lý bộ nhớ và ngữ cảnh.
- Kiểm soát quyền truy cập.
- Theo dõi hoạt động và lỗi.
- Mở rộng tài nguyên khi số lượng yêu cầu tăng.

Amazon Bedrock AgentCore cung cấp các thành phần hỗ trợ đưa Agent từ môi trường phát triển lên môi trường vận hành thực tế.

![Các thành phần của Amazon Bedrock AgentCore](/images/4-EventParticipated/4.3-Event3/07-agentcore-components.png)

*Amazon Bedrock AgentCore cung cấp môi trường vận hành, ngữ cảnh, công cụ, tối ưu hóa và các cơ chế quản lý Agent.*

Các thành phần được giới thiệu gồm:

- **AgentCore Runtime:** Môi trường vận hành Agent.
- **AgentCore Gateway:** Kết nối Agent với API, công cụ và tài nguyên.
- **AgentCore Identity:** Quản lý danh tính và xác thực.
- **AgentCore Memory:** Cung cấp bộ nhớ cho Agent.
- **AgentCore Observability:** Theo dõi hoạt động và hiệu suất.
- **AgentCore Evaluations:** Đánh giá chất lượng hoạt động của Agent.
- **AgentCore Registry:** Quản lý các Agent và tài nguyên liên quan.
- **AgentCore Policy:** Kiểm soát quyền và hành động của Agent.
- **Browser và Code Interpreter:** Các công cụ hỗ trợ Agent tương tác với trình duyệt hoặc thực thi mã.

### AgentCore Runtime

AgentCore Runtime cung cấp môi trường để triển khai và vận hành mã nguồn Agent. Agent có thể được đóng gói bằng container hoặc tệp triển khai và sau đó đưa lên Runtime.

Runtime có thể hỗ trợ nhiều kiểu giao tiếp khác nhau:

- **HTTP:** Giao tiếp request/response thông thường.
- **MCP:** Kết nối Agent với công cụ thông qua Model Context Protocol.
- **A2A:** Giao tiếp giữa Agent với Agent.
- **AG-UI:** Giao tiếp giữa Agent và giao diện người dùng.

![Các giao thức được AgentCore Runtime hỗ trợ](/images/4-EventParticipated/4.3-Event3/08-agentcore-runtime-protocols.png)

*AgentCore Runtime hỗ trợ nhiều hình thức giao tiếp giữa ứng dụng, công cụ, Agent và người dùng.*

Khả năng hỗ trợ nhiều giao thức giúp AgentCore Runtime phù hợp với nhiều kiến trúc ứng dụng khác nhau, từ Agent đơn giản đến hệ thống có nhiều Agent và nhiều công cụ.

### AgentCore Identity

AgentCore Identity hỗ trợ quản lý danh tính và xác thực cho Agent. Khi Agent cần truy cập API hoặc tài nguyên bên ngoài, hệ thống phải xác định Agent đang hoạt động dưới danh tính nào và được phép sử dụng tài nguyên nào.

Các mô hình xác thực được giới thiệu gồm:

- AWS Credentials với chữ ký SigV4.
- OAuth 2.0 dành cho giao tiếp giữa các dịch vụ.
- OAuth 2.0 cho trường hợp Agent thay mặt người dùng truy cập tài nguyên.
- Tích hợp với Amazon Cognito để xác thực người dùng.

Nhờ đó, Agent có thể truy cập tài nguyên với quyền phù hợp thay vì lưu trực tiếp thông tin đăng nhập hoặc sử dụng quyền quá rộng.

### AgentCore Gateway

AgentCore Gateway đóng vai trò là lớp kết nối an toàn giữa Agent với API, công cụ và tài nguyên bên ngoài.

![AgentCore Gateway cung cấp quyền truy cập an toàn](/images/4-EventParticipated/4.3-Event3/09-agentcore-gateway-secure-access.png)

*AgentCore Gateway quản lý kết nối và xác thực khi Agent truy cập các công cụ bên ngoài.*

Gateway có thể kết nối Agent với:

- REST API sử dụng OpenAPI schema.
- AWS Lambda function.
- MCP server.
- Các công cụ và tài nguyên nội bộ.
- API bên ngoài yêu cầu API key hoặc OAuth token.

Gateway phối hợp với AgentCore Identity để quản lý thông tin xác thực, token và quyền truy cập. Điều này giúp hạn chế việc đặt thông tin nhạy cảm trực tiếp trong mã nguồn của Agent.

## Phần 3 – Hands-on Lab trong Day 1

Sau phần lý thuyết, người tham dự chuyển sang **Hands-on Lab**. Diễn giả hướng dẫn từng bước từ chuẩn bị môi trường đến xây dựng một AI Agent cơ bản.

Bài thực hành sử dụng **Kiro**, một môi trường phát triển có tích hợp AI hỗ trợ lập trình. Thông qua cách tiếp cận thường được gọi là **vibe coding**, người dùng có thể mô tả yêu cầu bằng ngôn ngữ tự nhiên để AI hỗ trợ tạo mã nguồn, giải thích cấu trúc dự án và đề xuất cách triển khai.

Bài thực hành minh họa việc xây dựng một **Returns & Refunds Assistant** bằng **Strands Agents** và **Amazon Bedrock AgentCore**.

### Chuẩn bị môi trường phát triển

Trước khi thực hành, người tham dự được hướng dẫn chuẩn bị các công cụ cần thiết:

| Công cụ | Phiên bản tối thiểu | Mục đích |
|---|---:|---|
| Node.js | 20 trở lên | Chạy AgentCore CLI và các công cụ được phân phối qua npm |
| Python | 3.12 trở lên | Chạy mã nguồn Strands Agent và Lambda handler |
| AgentCore CLI | Phiên bản mới nhất | Khởi tạo, chạy thử và triển khai Agent |
| AWS CDK | Phiên bản 2 | Hỗ trợ khai báo và triển khai hạ tầng AWS |
| uv | Phiên bản mới nhất | Quản lý package và môi trường Python |
| AWS CLI | Phiên bản 2 | Cấu hình và tương tác với tài nguyên AWS |
| Kiro | Phiên bản phù hợp | Hỗ trợ phát triển mã nguồn bằng AI |

Người tham dự cũng được hướng dẫn kiểm tra phiên bản của từng công cụ để bảo đảm môi trường đã sẵn sàng.

Các thông tin xác thực AWS được cấu hình trong môi trường làm việc để Agent có thể gọi Amazon Bedrock và các API của AgentCore. Những thông tin như Access Key, Secret Access Key và Session Token chỉ được sử dụng trong phiên thực hành và không được đưa trực tiếp vào mã nguồn hoặc commit lên GitHub.

### Sử dụng Kiro để hỗ trợ lập trình

Trong Hands-on Lab, Kiro được sử dụng để hỗ trợ quá trình phát triển dự án. Người tham dự có thể nhập yêu cầu bằng ngôn ngữ tự nhiên, sau đó AI phân tích yêu cầu và hỗ trợ:

- Tạo cấu trúc thư mục dự án.
- Tạo các tệp mã nguồn ban đầu.
- Cài đặt hoặc đề xuất những package cần thiết.
- Giải thích vai trò của từng thành phần.
- Phát hiện và đề xuất sửa lỗi.
- Hỗ trợ viết prompt cho Agent.
- Hướng dẫn chạy thử ứng dụng.

Điều này giúp giảm thời gian phải viết thủ công các phần mã nguồn lặp lại. Tuy nhiên, người lập trình vẫn cần kiểm tra mã do AI tạo ra, hiểu luồng xử lý và xác nhận rằng kết quả phù hợp với yêu cầu.

### Xây dựng AI Assistant cơ bản

Sau khi chuẩn bị môi trường, người tham dự được hướng dẫn tạo một AI Assistant cơ bản bằng Strands Agents.

Luồng tương tác cơ bản gồm:

1. Người dùng nhập prompt hoặc câu hỏi.
2. Ứng dụng chuyển prompt đến Agent.
3. Agent sử dụng mô hình được cung cấp thông qua Amazon Bedrock.
4. Mô hình phân tích nội dung và tạo phản hồi.
5. Agent trả kết quả về giao diện hoặc terminal.
6. Người dùng tiếp tục nhập prompt để duy trì quá trình tương tác.

Trong bài thực hành, Agent được định hướng trở thành một **Returns & Refunds Assistant**, hỗ trợ trả lời những câu hỏi cơ bản liên quan đến việc đổi trả và hoàn tiền.

Ví dụ, người dùng có thể nhập yêu cầu mô tả một trường hợp cần hoàn trả sản phẩm. Agent tiếp nhận nội dung, phân tích prompt và tạo câu trả lời phù hợp dựa trên hướng dẫn được cấu hình.

### Vai trò của AI trong quá trình lập trình

Qua phần vibe coding, em nhận thấy AI có thể hỗ trợ đáng kể trong quá trình phát triển phần mềm, đặc biệt ở các công việc như:

- Tạo mã nguồn mẫu.
- Giải thích đoạn mã chưa quen thuộc.
- Đề xuất thư viện hoặc cấu trúc dự án.
- Hỗ trợ tìm nguyên nhân lỗi.
- Tạo tài liệu và hướng dẫn cài đặt.
- Chuyển yêu cầu bằng ngôn ngữ tự nhiên thành các bước kỹ thuật.
- Rút ngắn thời gian xây dựng phiên bản thử nghiệm.

Tuy nhiên, AI chỉ đóng vai trò là công cụ hỗ trợ. Người lập trình vẫn cần hiểu yêu cầu, kiểm tra tính chính xác của mã nguồn, quản lý quyền truy cập và bảo đảm không đưa thông tin nhạy cảm vào prompt hoặc repository.

## Kết quả đạt được trong Day 1

Sau khi tham gia Day 1 của AgentForge Deep Dive, em đã:

- Hiểu khái niệm cơ bản về Agentic AI.
- Phân biệt được chatbot, trợ lý AI và AI Agent.
- Hiểu các thành phần cơ bản của một Agent như mục tiêu, mô hình, công cụ, bộ nhớ, dữ liệu và hành động.
- Biết được các mức độ tự chủ khác nhau của hệ thống AI.
- Hiểu vòng lặp xử lý giữa Agent, mô hình và công cụ.
- Tìm hiểu vai trò của Amazon Bedrock AgentCore trong việc đưa Agent lên môi trường thực tế.
- Nắm được chức năng cơ bản của AgentCore Runtime, Identity và Gateway.
- Biết cách Agent kết nối an toàn với API và công cụ bên ngoài.
- Làm quen với Strands Agents và AgentCore CLI.
- Chuẩn bị các công cụ cần thiết cho môi trường phát triển.
- Sử dụng Kiro để hỗ trợ tạo và giải thích mã nguồn.
- Thực hành xây dựng một AI Assistant có khả năng tiếp nhận prompt và tạo câu trả lời.
- Hiểu thêm lợi ích và giới hạn của AI trong quá trình lập trình.

## Kinh nghiệm rút ra

Thông qua sự kiện, em nhận thấy quá trình xây dựng AI Agent cần kết hợp nhiều kiến thức khác nhau, bao gồm lập trình, thiết kế hệ thống, quản lý dữ liệu, xác thực, bảo mật và vận hành trên cloud.

Một Agent có thể tạo phản hồi tốt trong môi trường thử nghiệm nhưng để triển khai thực tế còn cần Runtime ổn định, cơ chế quản lý danh tính, quyền truy cập, khả năng quan sát và các giới hạn an toàn.

Phần Hands-on Lab cũng giúp em nhận ra rằng vibe coding có thể hỗ trợ người học tiếp cận công nghệ mới nhanh hơn. Tuy nhiên, người sử dụng cần chủ động đọc, kiểm tra và hiểu mã nguồn thay vì hoàn toàn phụ thuộc vào kết quả do AI tạo ra.

## Kết luận

Day 1 của **AgentForge Deep Dive** giúp em xây dựng nền tảng kiến thức về Agentic AI và Amazon Bedrock AgentCore. Sự kết hợp giữa lý thuyết và Hands-on Lab giúp em vừa hiểu cách một AI Agent hoạt động, vừa trực tiếp tiếp cận quy trình chuẩn bị môi trường và xây dựng một Agent đơn giản.

Sự kiện mang lại cho em thêm kiến thức về AWS, AI Agent, Strands Agents và phương pháp phát triển phần mềm có sự hỗ trợ của AI. Đây cũng là nền tảng hữu ích để em tiếp tục tìm hiểu các nội dung nâng cao hơn về bộ nhớ, khả năng quan sát, đánh giá, bảo mật và triển khai Agent trong tương lai.