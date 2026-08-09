---
title: "Event 3 - AgentForge Deep Dive"
date: 2026-07-21
weight: 3
chapter: false
pre: " <b> 4.3. </b> "
---

# AgentForge Deep Dive – Day 1

## Event Overview

**AgentForge Deep Dive** is an in-depth workshop series that introduces participants to building production-ready Agentic AI systems using **Amazon Bedrock AgentCore**.

Unlike a chatbot that mainly receives questions and generates responses, an Agentic AI system can analyze goals, create plans, use tools, retrieve data, and perform actions on behalf of users within an authorized scope.

The event combined theoretical presentations with guided hands-on exercises. Participants not only learned how AI Agents work but were also guided through preparing a development environment, building a basic Agent, and approaching the process of deploying an Agent to AWS.

![Introduction to AgentForge Deep Dive](/images/4-EventParticipated/4.3-Event3/01-agentforge-introduction.png)

*The speakers introduced AgentForge and its objective of building production-ready Agentic AI systems with Amazon Bedrock AgentCore.*

## Three-Day AgentForge Roadmap

AgentForge was designed as a three-day program. Each day combined theoretical knowledge with Hands-on Labs of increasing complexity.

![Three-day AgentForge workshop agenda](/images/4-EventParticipated/4.3-Event3/02-three-day-agenda.png)

*An overview of the three-day AgentForge workshop agenda.*

### Day 1 – Agentic AI Fundamentals

The first day focused on:

- An introduction to Agentic AI.
- Fundamental concepts of Amazon Bedrock AgentCore.
- An introduction to AgentCore Runtime, Gateway, and Identity.
- Building and deploying a basic AI Agent.
- Connecting an Agent to external tools and knowledge sources.
- Exploring web interface integration and user authentication.

### Day 2 – Expanding Agent Capabilities

According to the workshop agenda, Day 2 focused on:

- AgentCore Memory.
- AgentCore Evaluations.
- AgentCore Observability.
- AgentCore Registry.
- AgentCore Tools and optimization mechanisms.
- Monitoring Agent performance.
- Adding memory to support personalized behavior.

### Day 3 – DevOps and Security

Day 3 was planned to cover:

- DevOps use cases with Amazon Bedrock AgentCore.
- Best practices for building Agentic Systems.
- Using AgentCore Policy to secure tool calls.
- Open-ended experiments and advanced hands-on exercises.

For this event, I participated only in **Day 1**. Day 2 and Day 3 were introduced as part of the workshop roadmap but had not yet been included in my hands-on practice.

## Part 1 – Agentic AI Fundamentals

### What Is Agentic AI?

Agentic AI was introduced as a partially or fully autonomous software system that uses AI to **reason, plan, and complete tasks** on behalf of people or other systems.

![Definition of Agentic AI](/images/4-EventParticipated/4.3-Event3/03-what-is-agentic-ai.png)

*Agentic AI uses artificial intelligence to reason, plan, and complete tasks with different levels of autonomy.*

An Agent can receive a goal from a user, analyze the request, select appropriate tools, and perform multiple steps to produce the final result.

However, the Agent’s level of autonomy must be designed according to its use case. Not every system requires or should be allowed to operate fully autonomously.

### Basic Components of an AI Agent

A basic AI Agent can include the following components:

- **Goal:** The objective that the Agent must complete.
- **LLM:** The central component that understands requests and performs reasoning.
- **Tools:** APIs or tools that the Agent can use.
- **Memory:** A component that stores information and conversational context.
- **Data:** Information required during processing.
- **Actions:** Operations that the Agent can perform.
- **Observability:** The ability to monitor the Agent’s behavior and results.
- **Guardrails:** Rules that limit and control the Agent’s behavior.
- **User Interaction:** The communication channel between the user and the Agent.

![Basic components of an AI Agent](/images/4-EventParticipated/4.3-Event3/04-basic-agent-components.png)

*An Agent combines goals, memory, data, tools, and a language model to perform actions.*

From this diagram, I learned that a language model is only one component of an Agentic AI system. To create an Agent that can operate in a practical environment, the system also requires tools, data, memory, monitoring mechanisms, and safety controls.

### Levels of Autonomy

The speaker introduced different levels of autonomy in AI systems:

1. **Simple Assistants:** Respond to basic queries and perform single-step tasks.
2. **Deterministic Agents:** Follow strict procedures and predefined plans.
3. **Autonomous Agents:** Create plans at runtime and complete multi-step tasks.
4. **Agentic Virtual Workers:** Coordinate multiple Agents, operate over extended periods, and imitate the collaboration of human teams.

![Different levels of autonomy in AI systems](/images/4-EventParticipated/4.3-Event3/05-autonomy-gradient.png)

*The level of autonomy increases from simple assistants to Agentic Virtual Workers.*

An important lesson was that developers do not always need to build fully autonomous Agents. The appropriate level of autonomy should be selected based on business requirements, risk levels, and the system’s ability to control Agent behavior.

### The Agentic Loop

The next topic introduced the basic Agent loop implemented with **Strands Agents**.

![Basic Strands Agent loop](/images/4-EventParticipated/4.3-Event3/06-strands-agent-loop.png)

*The processing loop between the user, Agent, model, and tools.*

The workflow can be described as follows:

1. The user sends a prompt to the Agent.
2. The Agent sends the required information to the model.
3. The model analyzes the request and selects a tool when necessary.
4. The Agent executes the selected tool.
5. The tool result is returned to the model.
6. The process continues until the Agent has sufficient information.
7. The Agent returns the final response to the user.

This session helped me understand that an AI Agent does not simply send one prompt to a model and immediately return its response. The Agent may perform several reasoning and tool-execution cycles before producing its final result.

## Part 2 – Overview of Amazon Bedrock AgentCore

### Why Does an Agent Need an Operating Environment?

When an Agent is moved into a production environment, it requires a secure platform to:

- Run the Agent’s source code.
- Connect to AI models.
- Access tools and APIs.
- Use structured data and Knowledge Bases.
- Manage memory and context.
- Control access permissions.
- Monitor activities and errors.
- Scale resources when the number of requests increases.

Amazon Bedrock AgentCore provides components that help move an Agent from the development environment into production.

![Components of Amazon Bedrock AgentCore](/images/4-EventParticipated/4.3-Event3/07-agentcore-components.png)

*Amazon Bedrock AgentCore provides the environment, context, tools, optimization capabilities, and platform controls required to operate Agents.*

The introduced components included:

- **AgentCore Runtime:** Provides an environment for running Agents.
- **AgentCore Gateway:** Connects Agents to APIs, tools, and resources.
- **AgentCore Identity:** Manages identities and authentication.
- **AgentCore Memory:** Provides memory capabilities for Agents.
- **AgentCore Observability:** Monitors Agent activities and performance.
- **AgentCore Evaluations:** Evaluates Agent quality and behavior.
- **AgentCore Registry:** Manages Agents and related resources.
- **AgentCore Policy:** Controls Agent permissions and actions.
- **Browser and Code Interpreter:** Allow Agents to interact with browsers or execute code.

### AgentCore Runtime

AgentCore Runtime provides an environment for deploying and operating Agent source code. An Agent can be packaged as a container or deployment archive and then deployed to the Runtime.

AgentCore Runtime supports several communication methods:

- **HTTP:** Standard request and response communication.
- **MCP:** Connects an Agent to tools through the Model Context Protocol.
- **A2A:** Supports communication between Agents.
- **AG-UI:** Supports communication between an Agent and a user interface.

![Protocols supported by AgentCore Runtime](/images/4-EventParticipated/4.3-Event3/08-agentcore-runtime-protocols.png)

*AgentCore Runtime supports several communication methods between applications, tools, Agents, and users.*

Support for multiple protocols allows AgentCore Runtime to be used in different application architectures, ranging from a basic Agent to a system containing multiple Agents and tools.

### AgentCore Identity

AgentCore Identity supports identity and authentication management for Agents. When an Agent needs to access an external API or resource, the system must determine which identity the Agent is using and which resources it is allowed to access.

The authentication patterns introduced during the session included:

- AWS credentials with SigV4 request signing.
- OAuth 2.0 for service-to-service communication.
- OAuth 2.0 for an Agent accessing resources on behalf of a user.
- Integration with Amazon Cognito for user authentication.

These mechanisms allow an Agent to access resources with appropriate permissions instead of storing credentials directly in the source code or using overly broad permissions.

### AgentCore Gateway

AgentCore Gateway serves as a secure connection layer between an Agent and external APIs, tools, and resources.

![AgentCore Gateway providing secure access](/images/4-EventParticipated/4.3-Event3/09-agentcore-gateway-secure-access.png)

*AgentCore Gateway manages connections and authentication when an Agent accesses external tools.*

The Gateway can connect an Agent to:

- REST APIs described using an OpenAPI schema.
- AWS Lambda functions.
- MCP servers.
- Internal tools and resources.
- External APIs that require API keys or OAuth tokens.

AgentCore Gateway works with AgentCore Identity to manage credentials, tokens, and access permissions. This approach reduces the need to place sensitive credentials directly in the Agent’s source code.

## Part 3 – Day 1 Hands-on Lab

After the theoretical session, the participants proceeded to a **Hands-on Lab**. The speakers provided step-by-step guidance, beginning with environment preparation and continuing with the development of a basic AI Agent.

The lab used **Kiro**, an AI-assisted development environment. Through an approach commonly known as **vibe coding**, users can describe their requirements using natural language, while AI assists with generating source code, explaining the project structure, and suggesting implementation steps.

The practical exercise demonstrated the development of a **Returns & Refunds Assistant** using **Strands Agents** and **Amazon Bedrock AgentCore**.

### Preparing the Development Environment

Before beginning the practical exercise, participants were instructed to prepare the necessary tools:

| Tool | Minimum version | Purpose |
|---|---:|---|
| Node.js | 20 or later | Runs the AgentCore CLI and tools distributed through npm |
| Python | 3.12 or later | Runs the Strands Agent code and Lambda handlers |
| AgentCore CLI | Latest version | Scaffolds, tests, and deploys Agents |
| AWS CDK | Version 2 | Defines and provisions AWS infrastructure |
| uv | Latest version | Manages Python packages and project environments |
| AWS CLI | Version 2 | Configures and interacts with AWS resources |
| Kiro | Suitable version | Supports AI-assisted software development |

Participants were also instructed to verify the version of each tool to ensure that the development environment was ready.

AWS credentials were configured in the working environment so that the Agent could access Amazon Bedrock and AgentCore APIs. Sensitive information such as Access Keys, Secret Access Keys, and Session Tokens was used only for the workshop session and should never be written directly into source code or committed to GitHub.

### Using Kiro for AI-Assisted Development

During the Hands-on Lab, Kiro was used to assist with project development. Participants could enter requirements using natural language, after which the AI could help with:

- Creating the project directory structure.
- Generating initial source files.
- Installing or recommending the required packages.
- Explaining the purpose of each component.
- Identifying errors and proposing corrections.
- Assisting with prompt development.
- Providing instructions for running the application.

This approach reduces the time required to manually write repetitive source code. However, developers must still review the generated code, understand the processing flow, and verify that the result meets the project requirements.

### Building a Basic AI Assistant

After preparing the environment, participants were guided through building a basic AI Assistant with Strands Agents.

The basic interaction flow consisted of the following steps:

1. The user entered a prompt or question.
2. The application sent the prompt to the Agent.
3. The Agent used a model provided through Amazon Bedrock.
4. The model analyzed the content and generated a response.
5. The Agent returned the result to the interface or terminal.
6. The user could continue entering prompts to maintain the interaction.

In the practical exercise, the Agent was designed as a **Returns & Refunds Assistant**, which could provide basic answers related to product returns and refunds.

For example, a user could enter a prompt describing a product-return situation. The Agent received the request, analyzed the prompt, and generated an appropriate response based on its configured instructions.

### The Role of AI in Software Development

Through the vibe coding exercise, I realized that AI can provide significant assistance during software development, particularly in tasks such as:

- Generating source code templates.
- Explaining unfamiliar code.
- Recommending libraries and project structures.
- Helping identify the causes of errors.
- Creating documentation and setup instructions.
- Converting natural-language requirements into technical steps.
- Reducing the time required to create an initial prototype.

However, AI remains a supporting tool. Developers still need to understand the requirements, review the accuracy of the generated source code, manage access permissions, and ensure that sensitive information is not included in prompts or repositories.

## Day 1 Outcomes

After participating in Day 1 of AgentForge Deep Dive, I was able to:

- Understand the basic concept of Agentic AI.
- Distinguish between chatbots, AI assistants, and AI Agents.
- Understand the basic components of an Agent, including goals, models, tools, memory, data, and actions.
- Identify the different autonomy levels of AI systems.
- Understand the processing loop between an Agent, model, and tools.
- Learn the role of Amazon Bedrock AgentCore in moving Agents into production.
- Understand the basic functions of AgentCore Runtime, Identity, and Gateway.
- Learn how an Agent can securely connect to external APIs and tools.
- Become familiar with Strands Agents and the AgentCore CLI.
- Prepare the necessary tools for an Agent development environment.
- Use Kiro to assist with source-code generation and explanation.
- Practice building an AI Assistant that accepts prompts and generates responses.
- Understand the benefits and limitations of using AI during software development.

## Lessons Learned

Through this event, I realized that building an AI Agent requires knowledge from several areas, including programming, system design, data management, authentication, security, and cloud operations.

An Agent may generate suitable responses in a test environment, but deploying it to production requires a stable Runtime, identity management, appropriate access permissions, observability, and safety controls.

The Hands-on Lab also demonstrated that vibe coding can help learners approach new technologies more quickly. However, users must actively read, review, and understand the generated source code instead of depending entirely on AI-generated results.

## Conclusion

Day 1 of **AgentForge Deep Dive** provided me with foundational knowledge of Agentic AI and Amazon Bedrock AgentCore.

The combination of theoretical presentations and a Hands-on Lab helped me understand how an AI Agent works while also providing practical exposure to preparing a development environment and building a basic Agent.

The event expanded my knowledge of AWS, AI Agents, Strands Agents, and AI-assisted software development. It also provided a useful foundation for further study of advanced topics such as memory, observability, evaluation, security, and production Agent deployment.