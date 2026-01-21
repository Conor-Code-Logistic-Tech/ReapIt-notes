**Core Compute & Hosting**

EC2

I’ve commonly used for various customers and projects for small websites and long-running services, especially free-tier EC2 instances for personal APIs and early-stage services before moving to managed/serverless options.

Elastic Beanstalk

I used a lot with Verisk \~2yrs ago to rapidly deploy and manage web applications during early AWS migrations, abstracting EC2, scaling, and load balancing while removing operational overhead.

Lambda

I use a lot in various serverless microservice architectures for event-driven processing, background jobs, and API handlers, particularly in Search-as-a-Service and device automation workflows.

ECS (Fargate)

Used to run containerised .NET services without managing EC2 hosts, mainly for APIs and background workers during monolith decomposition and modernisation efforts.

\


**API & Networking**

API Gateway

I’ve used a lot to expose REST APIs for .NET services, including a recent C# file server API, handling routing, authentication, throttling, and integration with Lambda. Used anytime I would’ve setup API’s on AWS

Load Balancers (NLB,ALB,ELB,etc.)

Used to route HTTP traffic to containerised services, particularly with ECS-hosted APIs requiring path- and host-based routing.

VPC

Used to isolate application infrastructure, secure databases, and control traffic flow between services across both serverless and container-based architectures. I set these up a lot with clients on my website, it’s all automated within my infra through PHP.

Route 53

Self explanatory use for DNS management; most recently in ServAce to migrate from 123-reg, use to manage DNS via CloudFormation for traceability and repeatable infrastructure changes.

WAF\
Used a few times on various websites and projects for Firewalling & blocking country’s & IP’s based on IP lists\
\
CloudFront CDN\
Used on website to serve Public images and videos for faster page loading

**Storage & Data**

S3

Self explanatory I use buckets often for object storage such as images, private app files, public files & uploads, including replicable infra & seed data for multiple environments.

Aurora MySQL

Used as the primary relational database for distributed .NET systems, focusing on schema design, query optimisation, and performance during AWS migrations. Current C# MAUI app I’m creating uses this for the DB’s

Storage Gateway\
I’ve used personally to link Home Servers to Buckets and ECR for container images

**Messaging & Eventing**

SQS

Used to queue image uploads from the MAUI App’s C# file server API, decoupling ingestion from processing and improving reliability and throughput.

SNS

Used for notifications and pub/sub messaging, including event notifications triggered by file uploads and processing completion.

Step Functions

Used to orchestrate multi-step, retryable workflows across Lambda-based services in serverless microservice architectures.

\


EventBridge

Used for event-driven integrations between services, enabling loosely coupled communication across AWS workloads.

\


**Infrastructure, Security & Observability**

CloudFormation

Used for \~70% of my AWS projects for Infrastructure as Code to provision and version AWS resources, ensuring auditability, repeatability, and controlled change management. Mostly I use YAML for these, rarely the JSON (not that there's particularly any difference ahah!)

IAM

Used every AWS project to define service roles/accounts and permissions following PoLP (Principle of least-privilege)  principles across all services.

CloudWatch

Used for centralized logging, metrics, and alarms across APIs, Lambdas, and container workloads to support troubleshooting and performance tuning.

\


**CI/CD & Containers**

ECR

Used to store and version Docker images deployed to ECS and other container-based environments.

CodePipeline / GitHub Actions

Used to automate build, test, and deployment pipelines for .NET services and infrastructure changes, supporting frequent and reliable releases.
