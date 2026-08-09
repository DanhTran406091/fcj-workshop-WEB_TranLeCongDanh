---
title: "Event 2 - FCAJ x Agentic AI Build Week"
date: 2026-06-25
weight: 2
chapter: false
pre: " <b> 4.2. </b> "
---

# FCAJ x Agentic AI Build Week

## Tổng quan sự kiện

**FCAJ x Agentic AI Build Week** là buổi chia sẻ về quá trình tham gia hackathon, trong đó các nhóm trình bày ý tưởng, giải pháp, kiến trúc hệ thống, sản phẩm demo và những kinh nghiệm đã tích lũy trong quá trình phát triển ứng dụng Agentic AI.

Mở đầu chương trình, các khách mời giới thiệu về **Agentic AI Build Week**, mục tiêu của cuộc thi và cơ hội để người tham gia vận dụng kiến thức về AWS, trí tuệ nhân tạo và phát triển phần mềm vào việc giải quyết các bài toán thực tế.

![Khách mời giới thiệu chương trình Agentic AI Build Week](/fcj-workshop-WEB_TranLeCongDanh/images/4-EventParticipated/4.2-Event2/01-agentic-ai-build-week-introduction.png)

*Khách mời giới thiệu tổng quan về Agentic AI Build Week và chương trình hackathon.*

## Các giải pháp tiêu biểu được trình bày

Trong phần chính của sự kiện, các nhóm lần lượt giới thiệu quá trình tham gia hackathon, từ việc lựa chọn bài toán, hình thành ý tưởng, phân chia công việc, thiết kế kiến trúc đến xây dựng sản phẩm demo. Mỗi nhóm lựa chọn một lĩnh vực khác nhau nhưng đều sử dụng AI Agent và các dịch vụ AWS để giải quyết những vấn đề thực tế.

### 1. AI-Powered Conversation Ordering

Nhóm đầu tiên trình bày giải pháp **AI-Powered Conversation Ordering – From Idea to a Multi-Channel AI Agent**. Đây là một AI Agent hỗ trợ người dùng đặt món thông qua hội thoại trên nhiều kênh khác nhau.

![Nhóm trình bày giải pháp AI-Powered Conversation Ordering](/fcj-workshop-WEB_TranLeCongDanh/images/4-EventParticipated/4.2-Event2/02-conversation-ordering-presentation.png)

*Nhóm giới thiệu ý tưởng xây dựng AI Agent hỗ trợ đặt món qua hội thoại.*

#### Bài toán được đặt ra

Nhóm nhận thấy việc đặt món thông qua ứng dụng truyền thống vẫn còn một số bất tiện. Khi đang trò chuyện và có nhu cầu đặt món, người dùng thường phải:

1. Rời khỏi ứng dụng nhắn tin.
2. Mở ứng dụng đặt đồ ăn.
3. Đăng nhập hoặc tìm kiếm cửa hàng.
4. Xem thực đơn và lựa chọn sản phẩm.
5. Kiểm tra giỏ hàng và hoàn thành đơn hàng.

Quá trình chuyển đổi giữa nhiều ứng dụng tạo ra sự gián đoạn và có thể khiến người dùng từ bỏ ý định đặt món. Bên cạnh đó, xử lý yêu cầu bằng ngôn ngữ tự nhiên cũng không đơn giản vì hệ thống cần hiểu đúng tên món, số lượng, kích thước, lựa chọn đi kèm và những thay đổi trong hội thoại.

#### Giải pháp của nhóm

Nhóm đề xuất xây dựng một AI Agent có khả năng tiếp nhận yêu cầu đặt món trực tiếp từ cuộc trò chuyện. Người dùng có thể diễn đạt nhu cầu bằng ngôn ngữ tự nhiên, trong khi Agent chịu trách nhiệm phân tích và thực hiện các bước cần thiết.

Quy trình xử lý của Agent gồm:

1. **Goal:** Xác định mục tiêu và ý định đặt món của người dùng.
2. **Plan:** Lập kế hoạch xử lý yêu cầu.
3. **Tools:** Sử dụng công cụ để truy xuất thực đơn và dữ liệu liên quan.
4. **Act:** Thực hiện hành động như chọn món, cập nhật giỏ hàng hoặc tạo đơn.
5. **Verify:** Kiểm tra kết quả trước khi xác nhận lại với người dùng.

Điểm khác biệt được nhóm nhấn mạnh là: **“A chatbot replies. An agent acts.”** Chatbot thông thường chủ yếu tạo phản hồi, trong khi AI Agent có thể sử dụng công cụ và thực hiện hành động để hoàn thành mục tiêu của người dùng.

#### Kiến trúc và khả năng tích hợp

Hệ thống có thể tiếp nhận yêu cầu từ những kênh như Zalo, WhatsApp hoặc ứng dụng riêng. Yêu cầu sau đó được chuyển đến backend để xử lý hội thoại, truy xuất dữ liệu sản phẩm, quản lý trạng thái phiên làm việc và thực hiện nghiệp vụ đặt hàng.

![Kiến trúc tổng quan của hệ thống đặt món bằng AI Agent](/fcj-workshop-WEB_TranLeCongDanh/images/4-EventParticipated/4.2-Event2/03-conversation-ordering-architecture.png)

*Kiến trúc tổng quan cho phép tiếp nhận yêu cầu từ nhiều kênh và xử lý bằng AI Agent.*

Nhóm cũng thực hiện phần demo để minh họa cách người dùng tương tác với Agent thông qua hội thoại. Qua đó, giải pháp cho thấy khả năng giảm bớt các thao tác thủ công và mang lại trải nghiệm đặt món thuận tiện hơn.

#### Kiến thức em tiếp thu được

Qua phần trình bày này, em hiểu rằng việc xây dựng AI Agent không chỉ dừng lại ở kết nối với mô hình ngôn ngữ. Hệ thống còn cần quản lý trạng thái hội thoại, kiểm tra dữ liệu, kết nối với các công cụ nghiệp vụ và xác minh kết quả trước khi thực hiện hành động.

Em cũng hiểu rõ hơn sự khác nhau giữa chatbot và AI Agent, đặc biệt là khả năng **lập kế hoạch, sử dụng công cụ, thực hiện hành động và kiểm tra kết quả** của Agent.

---

### 2. SignalScout – thu thập và phân tích tín hiệu doanh nghiệp

Nhóm tiếp theo giới thiệu giải pháp **SignalScout**, một hệ thống hỗ trợ thu thập, kiểm chứng và phân tích các tín hiệu liên quan đến doanh nghiệp và thị trường.

#### Bài toán được đặt ra

Các nhóm chiến lược doanh nghiệp thường phải theo dõi một lượng lớn thông tin từ nhiều nguồn khác nhau. Việc đọc, tổng hợp và kiểm tra thông tin thủ công mất nhiều thời gian, trong khi những thay đổi quan trọng có thể không được phát hiện kịp thời.

Những thông tin này có thể liên quan đến:

- Hoạt động và định hướng của doanh nghiệp.
- Thay đổi trong chiến lược kinh doanh.
- Thông tin về đối thủ cạnh tranh.
- Dấu hiệu tái cấu trúc hoặc thay đổi tổ chức.
- Những rủi ro có khả năng ảnh hưởng đến hoạt động doanh nghiệp.

#### Giải pháp của nhóm

SignalScout sử dụng các công cụ thu thập dữ liệu kết hợp với AI để tìm kiếm, xác minh và tổng hợp thông tin. Theo Value Creation & Delivery Canvas được trình bày, các hoạt động chính của hệ thống gồm:

- Thu thập và kiểm chứng bằng chứng.
- Phát hiện những tín hiệu thay đổi hoặc tái cấu trúc.
- Phân tích số liệu và xây dựng nhận định.
- Trình bày kết quả dưới dạng báo cáo hoặc dashboard.
- Hỗ trợ người dùng theo dõi các tín hiệu quan trọng.

![Nhóm trình bày mô hình tạo và cung cấp giá trị của SignalScout](/fcj-workshop-WEB_TranLeCongDanh/images/4-EventParticipated/4.2-Event2/04-signalscout-value-canvas.png)

*Mô hình Value Creation & Delivery Canvas của giải pháp SignalScout.*

Giải pháp hướng đến các đối tượng như nhóm chiến lược doanh nghiệp, bộ phận quản trị rủi ro, nhóm quản lý và bộ phận phân tích thông tin cạnh tranh. Giá trị chính của hệ thống là giúp người dùng phát hiện sớm những thay đổi quan trọng dựa trên các tín hiệu đã được tổng hợp và kiểm chứng.

#### Kiến trúc và phần demo

Kiến trúc của hệ thống thể hiện một quy trình gồm nhiều bước: thu thập dữ liệu, xử lý nội dung, phân tích bằng AI, lưu trữ kết quả và hiển thị thông tin cho người dùng.

Nhóm sử dụng một số công cụ như **AWS, Langfuse, TinyFish và Apify** trong quá trình xây dựng giải pháp. Mỗi công cụ đảm nhận một vai trò khác nhau trong việc thu thập dữ liệu, theo dõi hoạt động của hệ thống và hỗ trợ phân tích thông tin.

Trong phần demo, nhóm trình bày giao diện tổng hợp các tín hiệu và đánh giá chúng theo nhiều tiêu chí. Thay vì tự đọc một lượng lớn tài liệu, người dùng có thể xem thông tin đã được hệ thống xử lý trên dashboard.

![Giao diện demo phân tích tín hiệu doanh nghiệp](/fcj-workshop-WEB_TranLeCongDanh/images/4-EventParticipated/4.2-Event2/05-signalscout-demo.png)

*Giao diện demo tổng hợp và phân tích các tín hiệu liên quan đến doanh nghiệp.*

#### Kiến thức em tiếp thu được

Qua giải pháp SignalScout, em hiểu thêm cách xây dựng một quy trình xử lý dữ liệu gồm nhiều giai đoạn, từ thu thập, kiểm chứng, phân tích đến trình bày kết quả.

Em cũng nhận thấy rằng AI chỉ tạo ra giá trị tốt khi dữ liệu đầu vào có nguồn rõ ràng và được kiểm tra. Nếu dữ liệu không chính xác, kết quả phân tích cũng có thể không đáng tin cậy. Vì vậy, ngoài khả năng của mô hình AI, hệ thống cần quan tâm đến chất lượng dữ liệu và khả năng truy xuất nguồn thông tin.

---

### 3. Solution Architect Professional Native App

Nhóm thứ ba trình bày giải pháp **Solution Architect Professional Native App**. Nội dung được tổ chức theo các phần: bài toán, giải pháp, quy trình hoạt động, kiến trúc, tác động và demo sản phẩm.

![Nhóm giới thiệu Solution Architect Professional Native App](/fcj-workshop-WEB_TranLeCongDanh/images/4-EventParticipated/4.2-Event2/06-solution-architect-app-introduction.png)

*Nhóm Plan V giới thiệu Solution Architect Professional Native App.*

#### Quá trình xây dựng sản phẩm

Bên cạnh sản phẩm kỹ thuật, nhóm còn chia sẻ hành trình tham gia hackathon qua bốn giai đoạn:

1. Đăng ký và lựa chọn chủ đề.
2. Phát triển sản phẩm trong thời gian giới hạn.
3. Trình bày và demo trước ban giám khảo.
4. Tổng kết bài học và kinh nghiệm.

Phần chia sẻ này giúp em thấy rõ áp lực của việc phát triển một sản phẩm trong thời gian ngắn. Nhóm phải nhanh chóng thống nhất ý tưởng, phân chia công việc, lựa chọn công nghệ và hoàn thiện một phiên bản có thể demo.

#### Kiến trúc hệ thống

Giải pháp sử dụng kiến trúc cloud-native và kết hợp nhiều dịch vụ AWS:

- **Amazon S3:** lưu trữ frontend, tệp tải lên và các tệp kết quả.
- **Amazon CloudFront:** phân phối nội dung frontend đến người dùng.
- **Amazon Cognito:** xác thực và quản lý người dùng.
- **Application Load Balancer:** tiếp nhận và phân phối yêu cầu đến backend.
- **Amazon ECS và AWS Fargate:** vận hành backend và AI Agent dưới dạng container.
- **Amazon ECR:** lưu trữ và quản lý container image.
- **Amazon RDS for PostgreSQL:** lưu trữ dữ liệu của ứng dụng.
- **Amazon EFS:** cung cấp hệ thống tệp dùng chung khi cần thiết.
- **Amazon Bedrock:** cung cấp khả năng AI tạo sinh cho Agent.
- **Amazon CloudWatch:** thu thập log và theo dõi hoạt động hệ thống.
- **Terraform:** mô tả và triển khai hạ tầng bằng mã nguồn.

![Kiến trúc AWS của Solution Architect Professional Native App](/fcj-workshop-WEB_TranLeCongDanh/images/4-EventParticipated/4.2-Event2/07-solution-architect-app-architecture.png)

*Kiến trúc kết hợp các dịch vụ AWS để vận hành frontend, backend, AI Agent và cơ sở dữ liệu.*

Frontend được lưu trữ trên Amazon S3 và phân phối qua CloudFront. Người dùng được xác thực thông qua Amazon Cognito. Các yêu cầu nghiệp vụ được chuyển qua Application Load Balancer đến backend và AI Agent chạy trên ECS Fargate.

Dữ liệu được lưu trong PostgreSQL, trong khi Amazon Bedrock cung cấp khả năng xử lý AI. CloudWatch hỗ trợ theo dõi hệ thống và Terraform giúp nhóm triển khai hạ tầng một cách nhất quán.

#### Kiến thức em tiếp thu được

Phần trình bày giúp em hiểu rõ hơn cách các dịch vụ AWS đảm nhận những vai trò khác nhau trong một hệ thống hoàn chỉnh. Một ứng dụng thực tế không chỉ có frontend và backend mà còn cần xác thực, cân bằng tải, lưu trữ, cơ sở dữ liệu, giám sát và quản lý hạ tầng.

Em đặc biệt học được cách nhóm phân chia kiến trúc thành các thành phần độc lập. Phương pháp này giúp hệ thống dễ quản lý, mở rộng và thay đổi hơn so với việc triển khai toàn bộ chức năng trong một thành phần duy nhất.

---

### 4. SHEPHERD – Venue Operations Agent

Nhóm tiếp theo trình bày giải pháp **SHEPHERD Venue Operations**, sử dụng computer vision và Agentic AI để hỗ trợ giám sát, phát hiện tình trạng đông người và điều phối nhân sự tại sự kiện.

#### Bài toán được đặt ra

Ở những địa điểm tổ chức sự kiện đông người, nhân viên vận hành phải theo dõi nhiều khu vực cùng lúc. Khi lượng người tại một khu vực tăng nhanh, việc phát hiện chậm có thể gây ùn tắc hoặc ảnh hưởng đến trải nghiệm của người tham dự.

Theo dõi hoàn toàn bằng con người cũng gặp một số hạn chế:

- Không thể quan sát liên tục tất cả khu vực.
- Khó thống kê chính xác số lượng người.
- Phản ứng có thể chậm khi tình trạng ùn tắc bắt đầu xuất hiện.
- Việc điều phối nhân viên phụ thuộc nhiều vào kinh nghiệm của người vận hành.

#### Giải pháp của nhóm

SHEPHERD tiếp nhận video trực tiếp từ camera, sử dụng computer vision để phát hiện và theo dõi người trong từng khu vực. Dữ liệu được chuyển đến hệ thống giám sát để hiển thị số lượng người, trạng thái khu vực và những cảnh báo liên quan.

AI Agent tiếp tục phân tích dữ liệu vận hành và đưa ra đề xuất, chẳng hạn điều phối thêm nhân viên đến khu vực đang đông hoặc hướng người tham dự sang khu vực ít người hơn.

![Kiến trúc hệ thống SHEPHERD Venue Operations](/fcj-workshop-WEB_TranLeCongDanh/images/4-EventParticipated/4.2-Event2/08-shepherd-architecture.png)

*Kiến trúc xử lý video, phân tích dữ liệu và hỗ trợ vận hành bằng Agentic AI.*

#### Quy trình xử lý trên AWS

Dựa trên sơ đồ kiến trúc được trình bày, hệ thống hoạt động theo quy trình:

1. Camera truyền hình ảnh trực tiếp vào **Amazon Kinesis Video Streams**.
2. Stream Processor chạy trong container xử lý luồng video.
3. **Amazon SageMaker Endpoint** hỗ trợ suy luận mô hình computer vision.
4. Dữ liệu và bằng chứng sự cố được lưu trữ bằng Amazon DynamoDB và hệ thống lưu trữ liên quan.
5. Frontend được lưu trên Amazon S3 và phân phối qua Amazon CloudFront.
6. Amazon API Gateway và AWS Lambda tiếp nhận các yêu cầu từ giao diện.
7. Agent chạy trên **Amazon Bedrock AgentCore** để phân tích dữ liệu và đưa ra khuyến nghị.
8. Amazon Cognito hỗ trợ xác thực người dùng.
9. IAM, Secrets Manager, CloudTrail và CloudWatch hỗ trợ bảo mật, quản lý thông tin bí mật, kiểm tra hoạt động và giám sát hệ thống.

#### Phần demo

Trong phần demo, hệ thống hiển thị video có các khung nhận diện người và số liệu theo từng khu vực. Dashboard cho phép người vận hành theo dõi số người, nhận biết khu vực đang ổn định hoặc có nguy cơ đông.

Operations Agent có thể phân tích dữ liệu và đưa ra đề xuất cụ thể. Ví dụ, khi một khu vực có số lượng người tăng nhanh, Agent có thể đề nghị điều động thêm nhân viên và hướng người mới đến khu vực ít đông hơn.

![Giao diện Operations Agent phân tích và đề xuất điều phối](/fcj-workshop-WEB_TranLeCongDanh/images/4-EventParticipated/4.2-Event2/09-shepherd-operations-agent.png)

*Operations Agent đánh giá tình trạng từng khu vực và đưa ra đề xuất điều phối.*

#### Những thách thức được nhóm chia sẻ

Trong quá trình thực hiện, nhóm gặp một số khó khăn như:

- Duy trì luồng video trực tiếp ổn định.
- Giảm độ trễ của quá trình suy luận.
- Theo dõi cùng một đối tượng giữa nhiều khung hình.
- Lựa chọn vị trí camera phù hợp.
- Hoàn thành hệ thống trong thời gian giới hạn.
- Làm cho khuyến nghị của AI Agent dễ hiểu và có thể thực hiện.
- Kiểm soát chi phí sử dụng tài nguyên cloud.

#### Kiến thức em tiếp thu được

Qua giải pháp SHEPHERD, em hiểu thêm cách kết hợp **xử lý dữ liệu thời gian thực, computer vision, cloud computing và Agentic AI** trong cùng một hệ thống.

Em cũng nhận ra rằng một sản phẩm AI thực tế không chỉ cần mô hình có độ chính xác tốt. Hệ thống còn phải bảo đảm tốc độ xử lý, tính ổn định, khả năng giải thích kết quả, bảo mật và chi phí vận hành phù hợp.

## Các phần trình bày khác

Bên cạnh bốn giải pháp tiêu biểu trên, một số nhóm khác cũng chia sẻ những ý tưởng, kiến trúc và sản phẩm demo có tính ứng dụng cao. Mỗi nhóm lựa chọn một bài toán khác nhau nhưng đều thể hiện khả năng vận dụng AWS và Agentic AI để xây dựng giải pháp thực tế.

Quá trình theo dõi các phần trình bày giúp em nhận thấy rằng một sản phẩm hackathon không chỉ cần có ý tưởng tốt mà còn phải xác định rõ bài toán, lựa chọn kiến trúc phù hợp, xây dựng được sản phẩm demo và trình bày rõ giá trị của giải pháp.

## Kiến thức và kinh nghiệm đạt được

Sau khi tham gia sự kiện, em đã:

- Hiểu rõ hơn về khái niệm **Agentic AI** và sự khác nhau giữa AI Agent với chatbot thông thường.
- Biết cách các nhóm phân tích bài toán và chuyển ý tưởng thành một sản phẩm có thể demo.
- Hiểu thêm cách kết hợp nhiều dịch vụ AWS trong một kiến trúc hoàn chỉnh.
- Tiếp cận các ứng dụng của AI trong thương mại, phân tích dữ liệu và vận hành địa điểm.
- Học hỏi cách xây dựng slide, trình bày kiến trúc và thực hiện demo sản phẩm.
- Nhận biết một số khó khăn khi xây dựng hệ thống AI như độ trễ, chất lượng dữ liệu, khả năng mở rộng và chi phí vận hành.
- Có thêm kinh nghiệm để định hướng việc xây dựng và trình bày đồ án nhóm trong chương trình thực tập.

## Kết luận

Sự kiện **FCAJ x Agentic AI Build Week** mang đến cho em cơ hội quan sát trực tiếp cách các nhóm phát triển sản phẩm trong môi trường hackathon. Thông qua các phần trình bày và demo, em không chỉ mở rộng kiến thức về AWS và Agentic AI mà còn học hỏi được cách phân tích yêu cầu, thiết kế kiến trúc, làm việc nhóm và truyền đạt một giải pháp kỹ thuật rõ ràng.