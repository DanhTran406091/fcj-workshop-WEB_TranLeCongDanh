---
title: "Event 2 - FCAJ x Agentic AI Build Week"
date: 2026-06-25
weight: 2
chapter: false
pre: " <b> 4.2. </b> "
---

# FCAJ x Agentic AI Build Week

## Event Overview

**FCAJ x Agentic AI Build Week** was a sharing session about the participants’ hackathon journey. During the event, teams presented their ideas, solutions, system architectures, product demonstrations, and experiences gained while developing Agentic AI applications.

At the beginning of the event, the guest speakers introduced **Agentic AI Build Week**, the objectives of the hackathon, and the opportunity for participants to apply their knowledge of AWS, artificial intelligence, and software development to real-world problems.

![Guest speakers introducing Agentic AI Build Week](/fcj-workshop-WEB_TranLeCongDanh/images/4-EventParticipated/4.2-Event2/01-agentic-ai-build-week-introduction.png)

*The guest speakers introduced Agentic AI Build Week and provided an overview of the hackathon.*

## Notable Solutions Presented

During the main session, the teams presented their hackathon journeys, from selecting a problem and developing an idea to assigning responsibilities, designing the system architecture, and building a working demonstration.

Each team selected a different field of application, but they all used AI Agents and AWS services to solve practical problems.

### 1. AI-Powered Conversation Ordering

The first team presented **AI-Powered Conversation Ordering – From Idea to a Multi-Channel AI Agent**. This solution uses an AI Agent to help users place food orders through conversations across multiple communication channels.

![The team presenting AI-Powered Conversation Ordering](/fcj-workshop-WEB_TranLeCongDanh/images/4-EventParticipated/4.2-Event2/02-conversation-ordering-presentation.png)

*The team introduced its idea of building an AI Agent for conversational food ordering.*

#### Problem Statement

The team identified several inconveniences in the traditional application-based food-ordering process. When users want to order food while having a conversation, they usually need to:

1. Leave the messaging application.
2. Open a food-ordering application.
3. Sign in or search for a restaurant.
4. Browse the menu and select products.
5. Review the shopping cart and complete the order.

Switching between multiple applications interrupts the user experience and may cause users to abandon their orders.

Processing natural-language requests is also challenging because the system must correctly understand product names, quantities, sizes, additional options, and changes made during the conversation.

#### The Team’s Solution

The team proposed an AI Agent capable of receiving food-ordering requests directly from a conversation. Users can describe their requirements using natural language, while the Agent analyzes the request and performs the necessary actions.

The Agent follows a five-stage process:

1. **Goal:** Identify the user’s objective and ordering intent.
2. **Plan:** Determine the steps required to process the request.
3. **Tools:** Use available tools to retrieve the menu and related information.
4. **Act:** Perform actions such as selecting products, updating the shopping cart, or creating an order.
5. **Verify:** Check the result before confirming it with the user.

An important distinction highlighted by the team was: **“A chatbot replies. An agent acts.”**

A conventional chatbot mainly generates responses. In contrast, an AI Agent can use tools and perform actions to complete the user’s intended task.

#### Architecture and Integration

The system can receive requests from communication channels such as Zalo, WhatsApp, or a dedicated application. These requests are then sent to the backend, where the system processes the conversation, retrieves product data, manages session states, and performs ordering operations.

![Overall architecture of the AI Agent ordering system](/fcj-workshop-WEB_TranLeCongDanh/images/4-EventParticipated/4.2-Event2/03-conversation-ordering-architecture.png)

*The overall architecture supports requests from multiple communication channels and processes them through an AI Agent.*

The team also presented a live demonstration of how users could interact with the Agent through natural-language conversations. The solution demonstrated the potential to reduce manual steps and provide a more convenient ordering experience.

#### Knowledge Gained

From this presentation, I learned that building an AI Agent involves more than simply connecting an application to a language model. The system must also manage conversation states, validate data, connect to business tools, and verify results before performing an action.

I also gained a clearer understanding of the difference between a chatbot and an AI Agent, particularly the Agent’s ability to **plan, use tools, perform actions, and verify results**.

---

### 2. SignalScout – Collecting and Analyzing Business Signals

The next team introduced **SignalScout**, a system designed to collect, verify, and analyze signals related to businesses and market activities.

#### Problem Statement

Corporate strategy teams often need to monitor large amounts of information from multiple sources. Manually reading, consolidating, and validating this information requires considerable time, while important changes may not be identified early enough.

The information may include:

- Business activities and corporate directions.
- Changes in business strategies.
- Information about competitors.
- Signs of restructuring or organizational changes.
- Potential risks that may affect business operations.

#### The Team’s Solution

SignalScout combines data-collection tools with AI to search for, verify, and summarize information.

According to the Value Creation & Delivery Canvas presented by the team, the system’s main activities include:

- Collecting and validating evidence.
- Detecting signals of organizational change or restructuring.
- Analyzing metrics and generating insights.
- Presenting results through reports or dashboards.
- Helping users monitor important signals.

![The team presenting SignalScout's value creation and delivery model](/fcj-workshop-WEB_TranLeCongDanh/images/4-EventParticipated/4.2-Event2/04-signalscout-value-canvas.png)

*The Value Creation & Delivery Canvas of the SignalScout solution.*

The solution is intended for corporate strategy teams, enterprise risk departments, management teams, and competitive intelligence analysts.

Its primary value is helping users identify important changes at an early stage based on collected and verified signals.

#### Architecture and Demonstration

The system architecture represents a multi-stage workflow consisting of data collection, content processing, AI-based analysis, result storage, and information presentation.

The team used tools such as **AWS, Langfuse, TinyFish, and Apify** while developing the solution. Each tool played a different role in data collection, system monitoring, and information analysis.

During the demonstration, the team presented an interface that consolidated and evaluated signals according to multiple criteria. Instead of manually reading a large number of documents, users could view the processed information through a dashboard.

![SignalScout business signal analysis demonstration](/fcj-workshop-WEB_TranLeCongDanh/images/4-EventParticipated/4.2-Event2/05-signalscout-demo.png)

*The demonstration interface consolidated and analyzed signals related to businesses.*

#### Knowledge Gained

Through the SignalScout solution, I learned how to design a multi-stage data-processing workflow, from collecting and validating information to analyzing and presenting the final results.

I also realized that AI only delivers reliable value when the input data comes from identifiable and verified sources. If the input data is inaccurate, the analytical results may also be unreliable.

Therefore, in addition to the capabilities of the AI model, the system must also consider data quality and the traceability of information sources.

---

### 3. Solution Architect Professional Native App

The third team presented the **Solution Architect Professional Native App**. Their presentation was organized into several sections, including the problem statement, proposed solution, workflow, system architecture, impact, and product demonstration.

![The team introducing the Solution Architect Professional Native App](/fcj-workshop-WEB_TranLeCongDanh/images/4-EventParticipated/4.2-Event2/06-solution-architect-app-introduction.png)

*Plan V introduced the Solution Architect Professional Native App.*

#### Product Development Journey

In addition to presenting the technical product, the team shared its hackathon journey through four main stages:

1. Registering for the hackathon and selecting a track.
2. Developing the product within a limited period.
3. Presenting and demonstrating the solution to the judges.
4. Reviewing the lessons learned and reflecting on the experience.

This sharing session helped me understand the pressure involved in developing a product within a short time. The team had to agree on an idea, assign responsibilities, select suitable technologies, and complete a functional version for demonstration.

#### System Architecture

The solution used a cloud-native architecture that combined multiple AWS services:

- **Amazon S3:** Stores the frontend, uploaded files, and generated results.
- **Amazon CloudFront:** Distributes frontend content to users.
- **Amazon Cognito:** Provides user authentication and management.
- **Application Load Balancer:** Receives and distributes requests to the backend.
- **Amazon ECS and AWS Fargate:** Run the backend and AI Agent as containers.
- **Amazon ECR:** Stores and manages container images.
- **Amazon RDS for PostgreSQL:** Stores application data.
- **Amazon EFS:** Provides a shared file system when required.
- **Amazon Bedrock:** Provides generative AI capabilities for the Agent.
- **Amazon CloudWatch:** Collects logs and monitors system operations.
- **Terraform:** Defines and deploys the infrastructure as code.

![AWS architecture of the Solution Architect Professional Native App](/fcj-workshop-WEB_TranLeCongDanh/images/4-EventParticipated/4.2-Event2/07-solution-architect-app-architecture.png)

*The architecture combines AWS services to operate the frontend, backend, AI Agent, and database.*

The frontend is stored on Amazon S3 and distributed through Amazon CloudFront. Users are authenticated through Amazon Cognito. Business requests are routed through the Application Load Balancer to the backend and AI Agent running on Amazon ECS with AWS Fargate.

Application data is stored in PostgreSQL, while Amazon Bedrock provides AI-processing capabilities. Amazon CloudWatch supports system monitoring, and Terraform helps the team deploy its infrastructure consistently.

#### Knowledge Gained

This presentation helped me understand how AWS services perform different roles within a complete system. A production application requires more than a frontend and a backend; it may also require authentication, load balancing, storage, databases, monitoring, and infrastructure management.

I particularly learned how the team separated its architecture into independent components. This approach makes the system easier to manage, scale, and modify than deploying all functionality within a single component.

---

### 4. SHEPHERD – Venue Operations Agent

The next team presented **SHEPHERD Venue Operations**, a solution that uses computer vision and Agentic AI to monitor crowded areas, detect congestion, and support staff coordination at events.

#### Problem Statement

At crowded event venues, operations staff must monitor multiple areas simultaneously. When the number of people in an area increases rapidly, delayed detection may lead to congestion and negatively affect the attendee experience.

Fully manual monitoring also presents several limitations:

- Staff cannot continuously observe every area.
- It is difficult to count people accurately.
- Responses may be delayed when congestion begins to develop.
- Staff allocation depends heavily on the experience of the operator.

#### The Team’s Solution

SHEPHERD receives live video streams from cameras and uses computer vision to detect and track people in different areas. The processed data is sent to a monitoring system that displays the number of people, area status, and related alerts.

The AI Agent then analyzes the operational data and produces recommendations. For example, it can recommend assigning additional staff to a crowded area or directing new attendees toward a less congested area.

![SHEPHERD Venue Operations system architecture](/fcj-workshop-WEB_TranLeCongDanh/images/4-EventParticipated/4.2-Event2/08-shepherd-architecture.png)

*The architecture processes video streams, analyzes data, and supports venue operations using Agentic AI.*

#### Processing Workflow on AWS

Based on the architecture presented by the team, the system follows this workflow:

1. Cameras transmit live video to **Amazon Kinesis Video Streams**.
2. A container-based Stream Processor processes the video stream.
3. An **Amazon SageMaker Endpoint** performs computer vision model inference.
4. Operational data and incident evidence are stored using Amazon DynamoDB and related storage components.
5. The frontend is stored on Amazon S3 and distributed through Amazon CloudFront.
6. Amazon API Gateway and AWS Lambda receive requests from the user interface.
7. The Agent runs on **Amazon Bedrock AgentCore** to analyze data and generate recommendations.
8. Amazon Cognito provides user authentication.
9. IAM, Secrets Manager, CloudTrail, and CloudWatch support access control, secret management, activity auditing, and system monitoring.

#### Product Demonstration

During the demonstration, the system displayed a live video stream with bounding boxes around detected individuals and statistics for different areas.

The dashboard allowed operators to monitor the number of people and identify whether an area was stable or at risk of becoming overcrowded.

The Operations Agent could analyze the data and provide specific recommendations. For example, when the number of people in an area increased rapidly, the Agent could recommend assigning additional staff and redirecting new arrivals to a less crowded area.

![Operations Agent analyzing conditions and recommending staff allocation](/fcj-workshop-WEB_TranLeCongDanh/images/4-EventParticipated/4.2-Event2/09-shepherd-operations-agent.png)

*The Operations Agent evaluated each area and provided staff-allocation recommendations.*

#### Challenges Shared by the Team

During the development process, the team encountered several challenges:

- Maintaining a reliable live video stream.
- Reducing inference latency.
- Tracking the same object between video frames.
- Selecting effective camera positions.
- Completing the system within the hackathon timeline.
- Making the AI Agent’s recommendations understandable and actionable.
- Controlling cloud resource costs.

#### Knowledge Gained

Through the SHEPHERD solution, I learned how **real-time data processing, computer vision, cloud computing, and Agentic AI** can be integrated into a single system.

I also realized that a practical AI product requires more than an accurate model. The system must also provide suitable processing speed, stability, result explainability, security, and operational cost control.

## Other Presentations

In addition to the four notable solutions described above, several other teams presented practical ideas, system architectures, and product demonstrations.

Although each team selected a different problem, they all demonstrated the ability to apply AWS services and Agentic AI to build practical solutions.

By observing these presentations, I learned that a successful hackathon product requires more than a creative idea. The team must clearly define the problem, select an appropriate architecture, build a demonstrable product, and effectively communicate the value of the solution.

## Knowledge and Experience Gained

After participating in the event, I was able to:

- Develop a clearer understanding of **Agentic AI** and the differences between an AI Agent and a conventional chatbot.
- Learn how teams analyze problems and transform ideas into demonstrable products.
- Understand how multiple AWS services can be combined within a complete system architecture.
- Explore applications of AI in commerce, business data analysis, and venue operations.
- Learn how to prepare presentation slides, explain system architectures, and demonstrate technical products.
- Recognize challenges in building AI systems, including latency, data quality, scalability, and operational costs.
- Gain experience that can be applied to the development and presentation of my group project during the internship program.

## Conclusion

The **FCAJ x Agentic AI Build Week** event provided me with an opportunity to observe how teams developed products in a hackathon environment.

Through the presentations and demonstrations, I not only expanded my knowledge of AWS and Agentic AI but also learned how to analyze requirements, design system architectures, collaborate within a team, and clearly communicate a technical solution.