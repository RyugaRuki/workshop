---
title: "Proposal"
date: 2024-01-01
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

# AI-Powered Data Extraction & Summarization Platform
## Group Project: Automated Data Collection and Summarization Solution using AWS & Amazon Bedrock

### 1. Executive Summary  
The "AI-Powered Data Extraction & Summarization Platform" project is designed and developed by our team to solve the problem of automatically collecting, extracting, and summarizing large volumes of data from external APIs. Instead of time-consuming manual processing, the platform leverages the power of Amazon Bedrock (Generative AI) combined with a highly scalable AWS architecture (EC2 Auto Scaling, SQS, EventBridge) to process data periodically in a fully automated, secure, and cost-optimized manner.

### 2. Problem Statement  
*Current Problem*  
Currently, collecting information from external data sources (External APIs) and summarizing content is done manually or through disjointed scripts. This leads to instability, difficulty in scaling when data volume increases, and consumes a lot of personnel time to read and analyze data.

*Our Team's Solution*  
Our team proposes a fully automated system on AWS. The user interface is distributed via CloudFront. The data processing backend runs on EC2 instances (located in a Private Subnet for security) managed by an Auto Scaling Group. The data retrieval and processing workflow is automatically scheduled every 8 hours by EventBridge and SQS. The core strength of the solution is the integration of Amazon Bedrock so that AI can automatically read, clean, and summarize data. The results are durably stored in S3 and DynamoDB.

*Benefits and Return on Investment (ROI)*  
The solution helps 100% automate the periodic data collection and reporting process, saving dozens of working hours per week for the analysis team. By using Auto Scaling and resource limits, the system optimizes operating costs as it only allocates resources when there are actual data processing tasks.

### 3. Solution Architecture and Execution Flow
The project's architecture is designed by the team to comply with security standards and high availability on AWS, ensuring the data flow is processed in a closed and secure manner.

![Architecture Diagram](/images/2-Proposal/edge_architecture.jpeg)

*Detailed Execution Flow:*
1. **User Communication (Frontend)**: Users access the application via **CloudFront** (protected by **AWS WAF**). CloudFront securely distributes static content from the **S3 bucket (frontend, private, versioned)** via OAC (Origin Access Control) and configured behaviors.
2. **Automation Trigger**: An **EventBridge Trigger** is configured to trigger every 8 hours, sending a message to an **SQS** queue to initiate the data extraction process.
3. **Compute Network & Load Balancing**: The system utilizes an **Application Load Balancer (ALB)** to balance API requests across the EC2 server cluster. These servers are managed by an **Auto Scaling Group** and are safely allocated within **Private Subnets (A and B)** (without Public IPs) to ensure security. Simultaneously, the EC2 Workers continuously pull jobs from the queue via the **SQS VPC Endpoint**.
4. **Data Collection (Outbound Traffic)**: To retrieve data from **External APIs**, the EC2 Workers connect to the internet via a **NAT Gateway** (located in a Public Subnet) and route through an **Internet Gateway**.
5. **Generative AI Integration**: After retrieving the data, the EC2 Worker pushes the content to **Amazon Bedrock** via the **Bedrock VPC Endpoint** for the language model to extract information and summarize.
6. **Secure Result Storage**: 
   - Cleaned text data (Cleaned content) is pushed directly by the Worker into an **S3 bucket** via the **S3 Gateway Endpoint**.
   - Metadata and structured information are saved to **DynamoDB** via the **DynamoDB VPC Endpoint**. This database is configured for automatic Backup.
   - Advanced summary jobs or status messages can also be pushed to the **SQS Summarize** queue via the SQS Endpoint.
7. **Monitoring & Alerting**:
   - The EC2 Workers are managed via **Systems Manager (Session Manager - SSM)**, eliminating the need to open SSH ports.
   - All logs and metrics are collected by **CloudWatch**.
   - If the system detects abnormalities exceeding thresholds (CloudWatch Alarm), notifications are sent via an **SNS Topic** to the **Admin's Email**.
8. **Security and Cost Management**:
   - Use **Security Groups** and **IAM Roles** for the ASG following the principle of least privilege (least scoped).
   - Set up **AWS Budgets** to automatically send email alerts when system costs exceed control levels.

### 4. Technical Implementation  
*Team Assignment and Implementation Phases:*
- **Phase 1 (Design & Initialization)**: The team builds the foundational VPC network with full Public/Private Subnets, NAT Gateway. Establishes the secure S3 Frontend via CloudFront and WAF.
- **Phase 2 (AI Integration & Scheduling)**: Installs and configures the Auto Scaling Group for EC2 Workers. Builds the EventBridge schedule and SQS. Develops source code for the EC2 Worker to call External APIs and Amazon Bedrock.
- **Phase 3 (Internal Storage)**: Sets up VPC Endpoints (S3, DynamoDB, SQS, Bedrock) to ensure traffic does not go over the public internet. Creates DynamoDB tables and corresponding S3 buckets.
- **Phase 4 (Monitoring & Finalization)**: Integrates CloudWatch, SNS, Systems Manager. Reviews and tests IAM Roles and Security Groups to accept the project.

### 5. Timeline & Milestones  
- **Week 10**:
  - *Architecture & Foundation Design*: Finalize the overall architecture diagram, set up the core VPC network (Public/Private Subnets, NAT Gateway, Internet Gateway), and configure basic security infrastructure (Security Groups, IAM Roles).
  - *Core Logic Development*: Write source code (Node.js/Python) for the Collector Worker to gather data and design the DynamoDB table schema to store metadata. Successfully conduct trial integrations with Amazon Bedrock via SDKs.
- **Week 11**: 
  - *Automation & Integration*: Configure the EC2 Auto Scaling Group combined with Launch Templates. Build the EventBridge schedule to trigger periodic cron jobs and utilize SQS as a fault-tolerant message queue between worker processes.
  - *Internal Network Security*: Set up and test traffic routing through VPC Endpoints (S3, DynamoDB, SQS, Bedrock) to ensure no sensitive data traverses the public internet.
- **Week 12**:
  - *Frontend & Content Delivery*: Deploy the static user interface to Amazon S3 and configure CloudFront (with OAC) alongside AWS WAF to protect and accelerate page loading worldwide.
  - *Monitoring & Finalization*: Configure CloudWatch Dashboards to monitor system performance and set up SNS to send email alerts during incidents. Conduct internal penetration/security testing and prepare presentation slides for the final project showcase.

### 6. Budget Estimation
You can find the detailed budget estimation on the [AWS Pricing Calculator](#).  

*Infrastructure Costs (AWS Services)*
- **Amazon EC2 (t4g.small)**: ~$6.00/month (Auto Scaling, assuming partial usage).
- **NAT Gateway & Data Transfer**: ~$15.00/month (for outbound API calls to collect data).
- **Amazon Bedrock (Nova Lite)**: ~$2.50/month (Estimated token usage for summarization).
- **Amazon S3 Standard**: ~$0.15/month (Storage for frontend and text contents).
- **Amazon DynamoDB**: $0.00/month (Within Free Tier limits).
- **Amazon SQS & EventBridge**: $0.00/month (Within Free Tier limits).
- **Amazon CloudFront & WAF**: ~$5.00/month (WAF WebACL base fee, CloudFront within Free Tier).

**Total Estimated Cost**: ~$28.65/month

### 7. Risk Assessment and Contingency Plan 
- **Connection Errors or External API Format Changes**: External APIs might impose rate limits or unexpectedly change their JSON structure.
  - *Contingency*: Implement an *Exponential Backoff & Retry* mechanism within the Worker code. Utilize a *Dead-Letter Queue (DLQ)* in SQS to store failed requests, allowing for reprocessing once engineers fix the API parsing logic.
- **Unexpected Costs from Amazon Bedrock & NAT Gateway**: Since Bedrock charges per token and NAT Gateway charges per hour/data processing, an infinite loop in the Worker could cause severe financial impact.
  - *Contingency*: Set strict, multi-tiered limits in *AWS Budgets* to immediately alert the team via email/SMS if costs reach 50%, 80%, and 100% of the allocated threshold (e.g., $30/month).
- **Server Security Risks & Data Leaks**: Hackers often scan and attack EC2 instances if SSH ports are exposed to the public internet.
  - *Contingency*: Strictly do not assign Public IPs to EC2 instances (keeping them in Private Subnets). Access and administer servers exclusively via *AWS Systems Manager (Session Manager)*. Enforce the principle of Least Privilege for all IAM Roles.
- **Data Loss During Processing**: Hardware faults or software bugs might crash an EC2 Worker while it is analyzing an article.
  - *Contingency*: SQS utilizes a Visibility Timeout mechanism. If an EC2 instance crashes before deleting the message, the message reappears in the queue for another EC2 Worker (spawned by Auto Scaling) to pick up and process, ensuring Zero Data Loss.

### 8. Expected Outcomes  
Our team's AWS architecture project is expected to achieve comprehensive technical and operational milestones:
- **Fully Automated Workflow**: Transform the manual data collection and summarization process into a 100% automated system. This saves up to 80% of daily data processing time for business analysis teams.
- **High Scalability & Cost Efficiency**: By integrating EC2 Auto Scaling with SQS, the system autonomously spins up more workers during traffic spikes and scales down to zero when idle, drastically reducing operational costs.
- **Practical Application of Generative AI**: Successfully harness the power of Amazon Bedrock (Nova Lite/Claude models) to comprehend and condense thousands of document pages into concise, highly accurate summaries with low latency.
- **Enterprise-Grade Security Standards**: The VPC network is meticulously designed with Private Subnets, VPC Endpoints, data encryption at rest (S3/DynamoDB), and Frontend protection via CloudFront + WAF. This solution goes beyond a simple academic group project—it is built with the reliability and security required for a real-world, commercial Software-as-a-Service (SaaS) product.