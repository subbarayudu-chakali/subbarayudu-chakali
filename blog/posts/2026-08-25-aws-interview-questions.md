# Amazon Web Services (AWS) Interview Questions & Answers

This reference collects 100 commonly asked AWS interview questions covering general cloud architecture and core services — not DevOps tooling. Questions progress from global infrastructure fundamentals through IAM, compute, storage, databases, networking, messaging, monitoring, cost optimization, and real-world reliability scenarios. Each answer is written to be short enough to say aloud in an interview yet complete enough to survive a follow-up. Questions are numbered continuously from 1 to 100 and grouped by topic, with a quick-fire round and closing advice at the end.

---

## Fundamentals & Global Infrastructure

**1. What is a Region in AWS and why does it matter?**

A Region is a physically isolated geographic area (for example `us-east-1`, `eu-west-1`) containing multiple data centers. Regions are independent of one another for fault isolation, data sovereignty, and latency. You choose a Region based on proximity to users, compliance requirements (data residency), service availability (not every service is in every Region), and price, which varies by Region.

**2. What is an Availability Zone (AZ)?**

An AZ is one or more discrete data centers within a Region, each with independent power, cooling, and networking. AZs in a Region are interconnected by low-latency, high-bandwidth links but are far enough apart to avoid a single flood, fire, or power event affecting more than one. Spreading resources across multiple AZs is the foundation of high availability on AWS.

**3. What is the difference between a Region, an AZ, and an edge location?**

- **Region** — a geographic collection of AZs.
- **Availability Zone** — one or more data centers inside a Region.
- **Edge location** — a Point of Presence (PoP) used by CloudFront, Route 53, and AWS Global Accelerator to cache content and terminate connections close to users. There are far more edge locations than Regions.

**4. What is the AWS shared responsibility model?**

AWS is responsible for security **of** the cloud — the hardware, physical facilities, and the managed service software. The customer is responsible for security **in** the cloud — data, IAM configuration, OS patching (for EC2), network configuration, and encryption choices. The split shifts with the service: for S3 you manage data and access policy; for Lambda AWS also manages the OS and runtime.

**5. What are the main ways to interact with AWS?**

The Management Console (web UI), the AWS CLI, the SDKs (Python/boto3, Java, JavaScript, etc.), and infrastructure-as-code tools like CloudFormation, CDK, or Terraform. All of them ultimately call the same public AWS service APIs over HTTPS, authenticated with SigV4 signatures.

**6. What is an AWS service endpoint?**

An endpoint is the URL a client sends API requests to, typically regional (for example `ec2.us-east-1.amazonaws.com`). Some services also offer global endpoints (IAM, Route 53). VPC endpoints let you reach services privately without traversing the public internet.

## IAM & Security

**7. What is IAM and what are its core entities?**

IAM (Identity and Access Management) controls authentication and authorization for AWS. Core entities are **users** (long-lived identities for people or apps), **groups** (collections of users for shared permissions), **roles** (temporary identities assumed by trusted principals), and **policies** (JSON documents defining permissions).

**8. What is the difference between an IAM user and an IAM role?**

A user has permanent credentials (password and/or access keys) and is tied to one identity. A role has no long-term credentials; it is assumed temporarily, yielding short-lived credentials via STS. Roles are the preferred way to grant access to EC2 instances, Lambda functions, cross-account access, and federated users.

**9. What are the types of IAM policies?**

- **Identity-based policies** — attached to users, groups, or roles.
- **Resource-based policies** — attached to a resource (S3 bucket policy, SQS queue policy) and specify who may access it.
- **Permission boundaries** — set the maximum permissions an identity can have.
- **Service control policies (SCPs)** — org-wide guardrails in AWS Organizations.
- **Session policies** — passed at assume-role time to further limit a session.

**10. How is an effective permission decided when multiple policies apply?**

Evaluation logic: an explicit **Deny** always wins. Otherwise, access requires an explicit **Allow** and no Deny, and the request must pass every applicable boundary (SCP, permission boundary, session policy). By default everything is implicitly denied until allowed.

**11. What does a basic IAM policy look like?**

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": ["s3:GetObject", "s3:PutObject"],
      "Resource": "arn:aws:s3:::my-bucket/*",
      "Condition": { "StringEquals": { "aws:RequestedRegion": "us-east-1" } }
    }
  ]
}
```

The key elements are `Effect`, `Action`, `Resource`, and optional `Condition` and `Principal` (for resource-based policies).

**12. What is AWS STS?**

The Security Token Service issues temporary, limited-privilege credentials. It backs `AssumeRole`, `AssumeRoleWithWebIdentity` (OIDC federation), and `AssumeRoleWithSAML` (enterprise SSO). Temporary credentials automatically expire, which is more secure than distributing long-lived keys.

**13. What is the principle of least privilege and how do you apply it on AWS?**

Grant only the permissions needed to perform a task, nothing more. Apply it by starting from a deny-all baseline, using IAM Access Analyzer to right-size policies from CloudTrail activity, scoping resources with ARNs and conditions, and preferring roles over long-lived keys.

**14. What is AWS KMS?**

Key Management Service is a managed service for creating and controlling encryption keys. It provides customer-managed keys (CMKs), AWS-managed keys, and integrates with most services for envelope encryption. KMS keys never leave the service unencrypted, and every use is logged in CloudTrail. Key policies plus IAM control who can use or manage a key.

**15. What is envelope encryption?**

Data is encrypted with a data key, and that data key is itself encrypted with a KMS key (the key-encrypting key). You store the encrypted data key alongside the ciphertext. This avoids sending large payloads to KMS — you only call KMS to decrypt the small data key, then decrypt the bulk data locally.

**16. When would you use AWS Secrets Manager vs. Systems Manager Parameter Store?**

Use **Secrets Manager** for secrets that need automatic rotation (database credentials, API keys) and fine-grained lifecycle management; it charges per secret. Use **Parameter Store** for general configuration and secrets that do not need built-in rotation; the standard tier is free and SecureString values are KMS-encrypted.

**17. What is MFA and where should it be enforced?**

Multi-factor authentication requires a second factor (hardware/virtual token) beyond the password. Enforce it on the root account, all privileged IAM users, and via IAM conditions (`aws:MultiFactorAuthPresent`) for sensitive actions. The root account should be locked down with MFA and used almost never.

**18. What are some IAM security best practices?**

- Lock away root credentials; enable MFA on root.
- Use roles and temporary credentials instead of long-lived access keys.
- Apply least privilege and use permission boundaries/SCPs as guardrails.
- Rotate credentials regularly and remove unused users/keys.
- Enable CloudTrail and IAM Access Analyzer across the organization.

**19. What is AWS Organizations and what are SCPs?**

Organizations lets you centrally manage multiple accounts under a management account, group them into organizational units (OUs), and consolidate billing. Service Control Policies are org-level guardrails that cap what identities in member accounts can do — they never grant permissions, only restrict the maximum.

**20. How does cross-account access work?**

Create a role in account B with a trust policy naming account A as a trusted principal. A principal in account A calls `sts:AssumeRole` on that role and receives temporary credentials scoped to account B. This avoids sharing credentials and centralizes auditing.

## Compute

**21. What is Amazon EC2?**

Elastic Compute Cloud provides resizable virtual servers (instances). You choose an AMI (machine image), an instance type (CPU/memory/network profile), storage, networking, and security groups. EC2 gives you full OS control, making it suited to lift-and-shift and workloads needing custom runtimes.

**22. What are the EC2 instance family categories?**

- **General purpose** (T, M) — balanced compute/memory, web servers.
- **Compute optimized** (C) — CPU-bound workloads, batch, gaming.
- **Memory optimized** (R, X) — in-memory databases, caches.
- **Storage optimized** (I, D) — high local IOPS/throughput.
- **Accelerated computing** (P, G, Inf) — GPUs/ML accelerators.

**23. What are the EC2 pricing models?**

- **On-Demand** — pay per second/hour, no commitment; good for spiky/unknown workloads.
- **Reserved Instances / Savings Plans** — 1- or 3-year commitment for up to ~72% savings on steady workloads.
- **Spot Instances** — spare capacity at up to ~90% off, but can be reclaimed with two minutes notice; good for fault-tolerant, stateless work.
- **Dedicated Hosts/Instances** — physical isolation for licensing or compliance.

**24. What is an AMI?**

An Amazon Machine Image is a template containing the OS, application server, and applications used to launch an instance. You can use AWS-provided, Marketplace, or custom AMIs. Baking dependencies into a custom AMI (a "golden image") speeds boot time and ensures consistency.

**25. What is EC2 Auto Scaling?**

An Auto Scaling group maintains a target number of instances, replacing unhealthy ones and scaling in/out based on policies. Scaling can be target-tracking (keep CPU at 50%), step, scheduled, or predictive. It works with a launch template and typically spans multiple AZs for resilience.

**26. What is the difference between vertical and horizontal scaling?**

Vertical scaling (scaling up) means moving to a larger instance — more CPU/RAM on one machine, but with a ceiling and usually downtime. Horizontal scaling (scaling out) means adding more instances behind a load balancer — near-limitless and resilient, but the app must be stateless or externalize state.

**27. What are the Elastic Load Balancer types?**

- **Application Load Balancer (ALB)** — Layer 7, HTTP/HTTPS, path/host routing, WebSockets, target groups.
- **Network Load Balancer (NLB)** — Layer 4, TCP/UDP, ultra-low latency, static IPs, millions of requests/sec.
- **Gateway Load Balancer (GWLB)** — deploys and scales third-party virtual appliances (firewalls, IDS).

(The older Classic Load Balancer is legacy.)

**28. What is AWS Lambda?**

Lambda runs code without provisioning servers; you upload a function and it executes in response to events (API Gateway, S3, SQS, EventBridge). You pay per request and per GB-second of execution. AWS manages scaling, patching, and availability. Constraints include a 15-minute max runtime and ephemeral `/tmp` storage.

**29. What is a cold start in Lambda and how do you reduce it?**

A cold start is the added latency when Lambda initializes a new execution environment (download code, start runtime, run init). Mitigations: use Provisioned Concurrency, keep packages small, choose lighter runtimes, minimize init-time work, and reuse connections outside the handler.

**30. When would you choose Lambda over EC2 or containers?**

Choose Lambda for event-driven, spiky, or short-lived workloads where you want zero server management and pay-per-use. Choose EC2/containers for long-running processes, predictable high utilization, workloads exceeding Lambda's limits, or when you need fine control over the runtime.

**31. What is the difference between ECS, EKS, and Fargate?**

- **ECS** — AWS's native container orchestrator, simpler and tightly integrated.
- **EKS** — managed Kubernetes, portable and standards-based but more complex.
- **Fargate** — a serverless *launch type* for both ECS and EKS that runs containers without you managing EC2 hosts. ECS/EKS are the orchestrators; Fargate is the compute model.

**32. What is AWS Elastic Beanstalk?**

A PaaS that deploys and manages web applications by provisioning the underlying EC2, ELB, Auto Scaling, and monitoring for you from your uploaded code. You keep control of the resources but skip most of the setup. It suits teams wanting quick deploys without deep infrastructure work.

**33. What are placement groups?**

They control how instances are placed on hardware: **cluster** (packed close for low latency/high throughput, HPC), **spread** (each on distinct hardware for max fault isolation), and **partition** (grouped into partitions on separate racks, for large distributed systems like Hadoop/Kafka).

**34. What is the EC2 instance metadata service (IMDS)?**

A link-local endpoint (`169.254.169.254`) that instances query for metadata and temporary role credentials. Always use IMDSv2, which requires a session token to defend against SSRF attacks that could otherwise steal credentials through a vulnerable app.

## Storage

**35. What is Amazon S3?**

Simple Storage Service is object storage offering 11 nines of durability. Objects live in buckets, are accessed via keys, and can be up to 5 TB. It is used for backups, data lakes, static website hosting, and media. It is not a filesystem — objects are immutable and replaced whole, not edited in place.

**36. Describe the main S3 storage classes.**

- **S3 Standard** — frequent access, low latency.
- **S3 Intelligent-Tiering** — auto-moves objects between tiers based on access; good for unknown patterns.
- **Standard-IA / One Zone-IA** — infrequent access, lower storage cost, retrieval fee; One Zone stores in a single AZ.
- **Glacier Instant / Flexible / Deep Archive** — archival, from milliseconds to hours of retrieval, cheapest storage.

**37. How do you secure data in S3?**

Block Public Access at the account/bucket level, use bucket policies and IAM for access, enable default encryption (SSE-S3, SSE-KMS, or SSE-C), enforce TLS with a policy condition, enable versioning and MFA delete for critical data, and use Access Points and VPC endpoints to constrain network paths.

**38. What is S3 versioning and why use it?**

Versioning keeps multiple versions of an object in a bucket so overwrites and deletes are recoverable — a delete just adds a delete marker. It protects against accidental loss and, combined with MFA delete, guards against malicious deletion. It also underpins cross-region replication.

**39. What is S3 lifecycle management?**

Lifecycle rules automatically transition objects to cheaper classes (for example Standard to Glacier after 90 days) and expire them after a retention period. This is the primary lever for controlling S3 cost as data ages, and it can also clean up incomplete multipart uploads and old versions.

**40. What is the difference between EBS, instance store, EFS, and S3?**

- **EBS** — network block storage attached to one instance (or multi-attach for io2), persists independently, AZ-scoped.
- **Instance store** — physically attached ephemeral storage, very fast, lost on stop/terminate.
- **EFS** — managed NFS filesystem, shared by many instances across AZs, elastic.
- **S3** — object storage over HTTPS, effectively unlimited, not a mountable filesystem.

**41. What are the EBS volume types?**

- **gp3/gp2** — general-purpose SSD; gp3 lets you provision IOPS/throughput independently of size.
- **io2/io1** — provisioned IOPS SSD for latency-sensitive databases; io2 Block Express supports very high IOPS and Multi-Attach.
- **st1** — throughput-optimized HDD for big, sequential workloads.
- **sc1** — cold HDD for infrequently accessed data.

**42. What is an EBS snapshot?**

A point-in-time, incremental backup of a volume stored in S3 (managed by AWS). Only changed blocks since the last snapshot are stored, saving cost. Snapshots can be copied across Regions and used to create new volumes or AMIs, forming the basis of backup and DR.

**43. When would you use Amazon EFS vs. FSx?**

Use **EFS** for a shared, elastic POSIX/NFS filesystem for Linux workloads. Use **FSx** when you need a specific filesystem: FSx for Windows File Server (SMB/Active Directory), FSx for Lustre (HPC/ML high throughput), or FSx for NetApp ONTAP/OpenZFS for enterprise features.

**44. What is AWS Storage Gateway?**

A hybrid service that gives on-premises applications access to cloud storage. Modes include File Gateway (NFS/SMB backed by S3), Volume Gateway (iSCSI block volumes backed by S3/EBS snapshots), and Tape Gateway (virtual tape library backed by Glacier). It is common for backup and gradual cloud migration.

**45. How would you serve large files to users worldwide at low latency and cost?**

Store the files in S3 and put CloudFront in front as a CDN. CloudFront caches content at edge locations, reduces origin load and egress cost, and can use signed URLs/cookies and Origin Access Control to keep the bucket private. Add S3 Transfer Acceleration for fast uploads if needed.

## Databases

**46. What is Amazon RDS?**

Relational Database Service is managed relational databases (MySQL, PostgreSQL, MariaDB, Oracle, SQL Server). AWS handles provisioning, patching, backups, and failover, while you keep schema and query control. It supports Multi-AZ for HA and read replicas for read scaling.

**47. What is the difference between RDS Multi-AZ and read replicas?**

Multi-AZ maintains a synchronous standby in another AZ purely for **availability** — automatic failover, no read scaling from the standby. Read replicas are **asynchronous** copies used to **scale reads** and can be promoted to standalone databases; they can also live in other Regions.

**48. What is Amazon Aurora and why is it different?**

Aurora is a MySQL- and PostgreSQL-compatible database built for the cloud, with a distributed storage layer that replicates six copies across three AZs and auto-heals. It delivers higher throughput than standard RDS engines, up to 15 low-latency read replicas, fast failover, and storage that auto-scales. Aurora Serverless v2 scales capacity on demand.

**49. What is Amazon DynamoDB?**

A fully managed, serverless NoSQL key-value and document database with single-digit-millisecond latency at any scale. You define a partition key (and optional sort key); it auto-partitions and replicates across three AZs. It offers on-demand or provisioned capacity, Global Tables for multi-Region, DynamoDB Streams, and TTL.

**50. How do you design a good DynamoDB partition key?**

Choose a high-cardinality attribute so traffic spreads evenly and you avoid "hot partitions." Model access patterns first, then design keys and indexes to serve them (single-table design is common). Use composite sort keys and Global/Local Secondary Indexes for additional query patterns rather than scanning.

**51. When would you choose DynamoDB over RDS?**

Choose DynamoDB for massive scale, predictable low latency, flexible schema, and serverless operations where access patterns are known and mostly key-based. Choose RDS/Aurora when you need complex joins, ad-hoc queries, strong relational integrity, and SQL. It is a trade-off between scale/simplicity and query flexibility.

**52. What is Amazon ElastiCache?**

A managed in-memory cache supporting **Redis** and **Memcached**. It reduces database load and latency for read-heavy or compute-heavy workloads. Redis adds persistence, replication, pub/sub, sorted sets, and cluster mode; Memcached is simpler and multi-threaded for pure caching. Common patterns are cache-aside and session storage.

**53. What is Amazon Redshift?**

A petabyte-scale, columnar data warehouse for OLAP analytics. It uses massively parallel processing across nodes, columnar storage, and compression for fast aggregate queries. Redshift Spectrum queries data directly in S3, and it integrates with BI tools. Use it for analytics, not transactional (OLTP) workloads.

**54. What is the difference between OLTP and OLAP, and which AWS services fit each?**

OLTP (transactional) handles many small reads/writes with low latency — fit RDS, Aurora, DynamoDB. OLAP (analytical) runs large aggregate queries over historical data — fit Redshift, Athena (serverless SQL on S3), and EMR. Data typically flows from OLTP stores into an OLAP warehouse or lake.

**55. How do RDS backups work?**

RDS supports automated backups with point-in-time recovery within a retention window (up to 35 days) using daily snapshots plus transaction logs, and manual snapshots that persist until deleted. Snapshots can be copied cross-Region and shared, forming the backup and DR strategy.

## Networking

**56. What is a VPC?**

A Virtual Private Cloud is a logically isolated virtual network in AWS where you define IP ranges (CIDR), subnets, route tables, and gateways. It gives you control over your network topology and connectivity, and is the foundation for deploying most resources securely.

**57. What is the difference between a public and a private subnet?**

The distinction is routing: a **public subnet** has a route to an Internet Gateway, so resources with public IPs can reach and be reached from the internet. A **private subnet** has no such route; it reaches the internet outbound only via a NAT Gateway, keeping instances unreachable from the outside.

**58. What is an Internet Gateway vs. a NAT Gateway?**

An **Internet Gateway (IGW)** allows bidirectional internet traffic for resources with public IPs and is attached to the VPC. A **NAT Gateway** lets private-subnet resources initiate **outbound** connections (updates, API calls) while blocking inbound, translating their private IPs to a public one. It lives in a public subnet.

**59. What is the difference between security groups and network ACLs?**

- **Security groups** — stateful, operate at the instance/ENI level, support only allow rules, evaluate all rules; return traffic is automatically allowed.
- **Network ACLs** — stateless, operate at the subnet level, support allow and deny rules, evaluate in numbered order, and require explicit rules for return traffic.

Security groups are the primary firewall; NACLs are a coarse secondary layer.

**60. What is VPC peering and its limitations?**

VPC peering connects two VPCs privately so resources communicate as if on one network. It is non-transitive (A-B and B-C does not give A-C), cannot have overlapping CIDRs, and does not scale well to many VPCs — that is where Transit Gateway comes in.

**61. What is AWS Transit Gateway?**

A cloud router that connects many VPCs and on-premises networks through a single hub, replacing a mesh of peering connections. It supports transitive routing, route tables for segmentation, and cross-Region peering, greatly simplifying large multi-VPC/multi-account architectures.

**62. What is AWS PrivateLink / a VPC endpoint?**

VPC endpoints let you reach AWS or third-party services privately without an internet gateway. **Gateway endpoints** (S3, DynamoDB) add a route-table entry at no cost. **Interface endpoints** (PrivateLink) create an ENI with a private IP in your subnet to reach a service, keeping traffic on the AWS network.

**63. What is AWS Direct Connect?**

A dedicated physical network connection between your data center and AWS, offering consistent low latency and higher throughput than the internet. It is used for large data transfer, hybrid architectures, and compliance. For resilience it is often paired with a VPN backup or a second Direct Connect link.

**64. What is Amazon Route 53?**

A scalable DNS and domain-registration service. Beyond resolving names, it offers health checks and routing policies: simple, weighted, latency-based, failover, geolocation, geoproximity, and multivalue. It is central to global HA and DR designs, directing users to healthy endpoints in the best Region.

**65. Explain Route 53 routing policies with an example use.**

- **Simple** — one record, no logic.
- **Weighted** — split traffic by percentage (canary/A-B testing).
- **Latency-based** — send users to the lowest-latency Region.
- **Failover** — active-passive DR using health checks.
- **Geolocation/Geoproximity** — route by user location or shift traffic bias.
- **Multivalue** — return multiple healthy IPs for basic load spreading.

**66. What is Amazon CloudFront?**

A content delivery network that caches content at edge locations to reduce latency and origin load. Origins can be S3, ALB, EC2, or any HTTP server. It supports TLS, signed URLs/cookies, field-level encryption, Lambda@Edge / CloudFront Functions for edge logic, and integrates with AWS WAF and Shield.

**67. How do security groups behave across AZs and instances?**

Security groups are regional constructs referenced by ENIs; they apply regardless of AZ. You can reference another security group as a source, which is a clean way to allow, say, app-tier instances to reach the database tier without hardcoding IPs, and it survives instance replacement.

**68. What is a CIDR block and how do you plan VPC addressing?**

CIDR (Classless Inter-Domain Routing) notation like `10.0.0.0/16` defines an IP range. Plan a VPC range large enough for growth, carve non-overlapping subnets per AZ and tier, avoid overlaps with on-prem/other VPCs (important for peering/VPN), and reserve space for future subnets. AWS reserves five IPs per subnet.

**69. How would you connect an on-premises network to AWS securely?**

Options: a **Site-to-Site VPN** over the internet (quick, encrypted, lower cost) or **Direct Connect** for dedicated bandwidth and consistent latency, often with a VPN as encrypted backup. For many networks, terminate them on a **Transit Gateway** hub for scalable, segmented connectivity.

## Messaging & Integration

**70. What is Amazon SQS and what are its queue types?**

Simple Queue Service is a fully managed message queue for decoupling components. **Standard** queues offer near-unlimited throughput with at-least-once delivery and best-effort ordering; **FIFO** queues guarantee exactly-once processing and strict ordering at lower throughput. Consumers poll the queue and delete messages after processing.

**71. What is the SQS visibility timeout and dead-letter queue?**

The **visibility timeout** hides a message from other consumers after one receives it, giving time to process before it becomes visible again; if not deleted in time, it reappears. A **dead-letter queue** captures messages that fail processing after a set number of receives, isolating "poison" messages for inspection.

**72. What is Amazon SNS and how does it differ from SQS?**

Simple Notification Service is pub/sub: publishers send to a topic that pushes to many subscribers (SQS, Lambda, HTTP, email, SMS). SQS is pull-based point-to-point queuing. A common pattern is **fan-out**: publish to an SNS topic that delivers to multiple SQS queues so several services process the same event independently.

**73. What is Amazon EventBridge?**

A serverless event bus that routes events from AWS services, custom apps, and SaaS partners to targets using rules and content-based filtering. It adds a schema registry and scheduler. Compared with SNS it excels at routing and filtering structured events across many sources and targets in event-driven architectures.

**74. What is Amazon Kinesis and its main services?**

Kinesis handles real-time streaming data. **Data Streams** ingest high-throughput records with shards and ordered, replayable reads (retention up to 365 days). **Data Firehose** delivers streams to S3/Redshift/OpenSearch with optional transformation. **Managed Service for Apache Flink** does stream analytics. Use it for clickstreams, logs, IoT, and telemetry.

**75. How do SQS and Kinesis differ?**

SQS is a queue: each message is consumed and deleted by one consumer, with no replay. Kinesis is a stream: records are ordered per shard, retained for a window, and can be read by multiple independent consumers and replayed. Choose Kinesis for ordered, multi-consumer, real-time analytics; SQS for simple decoupling and work distribution.

**76. What is AWS Step Functions?**

A serverless workflow orchestrator that coordinates services as a state machine defined in Amazon States Language. It handles sequencing, branching, parallelism, retries, error handling, and human approval steps. Standard workflows suit long-running, auditable processes; Express workflows suit high-volume, short-duration event processing.

**77. How would you design a decoupled order-processing pipeline?**

Accept orders via API Gateway to a Lambda that publishes to an **SNS** topic or **EventBridge**; fan out to **SQS** queues for inventory, payment, and shipping so each service scales independently. Use dead-letter queues for failures and **Step Functions** to orchestrate multi-step flows with retries — no component blocks another.

## Monitoring & Management

**78. What is Amazon CloudWatch?**

CloudWatch collects metrics, logs, and events for AWS resources and applications. It supports dashboards, alarms (which can trigger Auto Scaling or SNS notifications), Logs Insights for querying logs, and custom metrics. It is the primary observability service for performance and operational health.

**79. What is the difference between CloudWatch and CloudTrail?**

**CloudWatch** is about performance and operational monitoring — metrics, logs, alarms ("how is it running"). **CloudTrail** is about governance and audit — it records API calls (who did what, when, from where) across the account. You use CloudWatch to watch health and CloudTrail to investigate and audit activity.

**80. What is AWS Config?**

Config tracks the configuration state of resources over time and evaluates them against rules for compliance (for example "all EBS volumes must be encrypted"). It provides configuration history, change timelines, and can trigger remediation. It answers "what did this resource look like at time T and is it compliant."

**81. What is AWS Trusted Advisor?**

An advisory service that inspects your account and recommends improvements across cost optimization, performance, security, fault tolerance, and service limits. Full checks require Business/Enterprise Support. It surfaces things like idle resources, open security groups, and low utilization.

**82. How do CloudWatch alarms drive automated responses?**

An alarm watches a metric against a threshold and, on breach, sends to an SNS topic, triggers an Auto Scaling policy, or runs an EC2/Systems Manager action. For example, high average CPU can add instances, and a failed health check can notify on-call and initiate failover.

**83. What is Amazon CloudWatch Logs vs. metrics vs. events?**

**Logs** are text records from applications/services you can query and set retention on. **Metrics** are time-series numeric data (CPU, latency) alarms watch. **Events** (now largely EventBridge) are near-real-time notifications of state changes used to trigger automation. Metric filters can extract metrics from log data.

## Cost Optimization & Well-Architected

**84. What are the pillars of the AWS Well-Architected Framework?**

Operational Excellence, Security, Reliability, Performance Efficiency, Cost Optimization, and Sustainability. Each provides design principles and questions to review a workload. It is a structured way to evaluate trade-offs rather than a checklist of specific services.

**85. What levers exist for cost optimization on AWS?**

- Right-size instances and use auto scaling to match demand.
- Commit with Savings Plans/Reserved Instances for steady workloads; use Spot for fault-tolerant ones.
- Move data to cheaper S3 classes with lifecycle rules.
- Delete idle/unattached resources (EBS, EIP, old snapshots).
- Use serverless/managed services to cut operational overhead.
- Monitor with Cost Explorer, Budgets, and the Cost and Usage Report.

**86. What is the difference between Savings Plans and Reserved Instances?**

Reserved Instances commit to specific instance attributes (family, Region) for a discount. Savings Plans commit to a dollars-per-hour spend and are more flexible: Compute Savings Plans apply across EC2, Fargate, and Lambda regardless of family or Region. Both offer 1- or 3-year terms and save up to ~72%.

**87. How do you monitor and control spending?**

Use **AWS Budgets** to set thresholds and alerts, **Cost Explorer** to analyze trends and forecasts, the **Cost and Usage Report** for granular data, cost allocation **tags** to attribute spend, and consolidated billing in Organizations. Set up anomaly detection to catch unexpected spikes early.

**88. What does "design for cost" look like in architecture?**

Prefer managed/serverless services so you pay for use, not idle capacity; cache to reduce database and egress costs; keep traffic within a Region/AZ to avoid transfer charges; choose the cheapest storage class that meets the access pattern; and automate shutdown of non-production environments outside business hours.

## Reliability, HA, DR & Architecture Scenarios

**89. How do you design a highly available web application on AWS?**

Deploy stateless app instances across multiple AZs in an Auto Scaling group behind an ALB; use a Multi-AZ database (RDS/Aurora); store static assets in S3 fronted by CloudFront; keep session state in ElastiCache or DynamoDB; and use Route 53 health checks. No single AZ or instance failure should take the app down.

**90. What is the difference between high availability and fault tolerance?**

High availability minimizes downtime and recovers quickly from failure (brief disruption acceptable). Fault tolerance means the system keeps running with no interruption despite component failures, usually via redundancy. Fault tolerance is stronger and costlier; most designs target HA with fast recovery.

**91. What are RTO and RPO?**

**RTO (Recovery Time Objective)** is how long you can be down — the target time to restore service. **RPO (Recovery Point Objective)** is how much data loss is acceptable — the target maximum age of data you recover to. They drive the DR strategy and its cost.

**92. What are the main AWS disaster-recovery strategies?**

- **Backup & Restore** — cheapest, highest RTO/RPO; restore from backups after an event.
- **Pilot Light** — core services (like a replicated database) always on, scale up on failover.
- **Warm Standby** — a scaled-down full copy running, scaled up on failover.
- **Multi-Site Active/Active** — full capacity in multiple Regions serving traffic, near-zero RTO/RPO, most expensive.

**93. How would you achieve multi-Region resilience for a database?**

For DynamoDB use Global Tables (active-active multi-Region replication). For Aurora use Global Database (one primary Region, low-latency read replicas in others, fast promotion on failover). For RDS use cross-Region read replicas that can be promoted. Combine with Route 53 failover routing to shift traffic.

**94. How do you make an application stateless, and why does it matter?**

Externalize state: store sessions in ElastiCache/DynamoDB, files in S3/EFS, and data in a database rather than on local disk. Statelessness lets any instance serve any request, so you can scale horizontally, replace unhealthy instances freely, and survive AZ failures — the basis of elastic, resilient design.

**95. A workload has unpredictable spiky traffic and must minimize ops. What do you propose?**

A serverless architecture: API Gateway plus Lambda for compute, DynamoDB on-demand for storage, S3/CloudFront for static content, and SQS/EventBridge for decoupling. It scales automatically to zero and to peaks, you pay only for use, and there are no servers to patch — ideal for spiky, low-ops workloads.

**96. How would you migrate a large on-premises database to AWS with minimal downtime?**

Use **AWS DMS** (Database Migration Service) with **SCT** for heterogeneous engine conversion. DMS does an initial full load then applies ongoing changes via change data capture, keeping source and target in sync until a short cutover. For huge datasets, seed with Snowball to reduce transfer time.

**97. How do you protect a public web application from attacks?**

Layer defenses: AWS **Shield** (DDoS protection, Standard is automatic), **WAF** for L7 rules (SQLi, XSS, rate limiting) on CloudFront/ALB/API Gateway, security groups and NACLs for network filtering, private subnets for backends, TLS everywhere, and GuardDuty for threat detection. Keep least-privilege IAM throughout.

**98. What is Amazon GuardDuty?**

A managed threat-detection service that continuously analyzes CloudTrail, VPC Flow Logs, and DNS logs using ML and threat intelligence to flag anomalies like compromised credentials, crypto-mining, or reconnaissance. It requires no agents and generates findings you can route to EventBridge for automated response.

**99. How do you handle secrets and configuration for a fleet of applications?**

Store secrets in Secrets Manager (with rotation) or Parameter Store SecureString, encrypt with KMS, and grant apps access via IAM roles rather than embedding credentials. Applications fetch secrets at runtime; never bake them into AMIs, code, or environment variables committed to source control.

**100. Walk through a resilient, cost-aware three-tier architecture on AWS.**

Route 53 directs users to CloudFront serving static content from S3. Dynamic traffic hits an ALB spanning multiple AZs, forwarding to an Auto Scaling group of stateless app instances (or Fargate/Lambda). The data tier uses Multi-AZ Aurora with read replicas plus ElastiCache for caching. Everything runs in a VPC with public subnets for the ALB/NAT and private subnets for app and data tiers, secured by security groups and least-privilege IAM roles. Add CloudWatch alarms, WAF/Shield, automated backups, and cross-Region replication sized to the required RTO/RPO — using Savings Plans and Spot where appropriate to control cost.

## Quick-fire round

- **Max S3 object size?** 5 TB (single PUT up to 5 GB; use multipart above).
- **Default VPCs per Region?** One default VPC, created automatically.
- **Reserved IPs per subnet?** 5 (first four and the last).
- **Lambda max timeout?** 15 minutes.
- **S3 durability?** 11 nines (99.999999999%).
- **Is a security group stateful?** Yes; NACLs are stateless.
- **Default SQS message retention?** 4 days (configurable 1 minute to 14 days).
- **Which S3 class for unknown access patterns?** Intelligent-Tiering.
- **DynamoDB consistency default?** Eventually consistent (strongly consistent optional).
- **What issues temporary credentials?** AWS STS.
- **Global service, not regional?** IAM, Route 53, CloudFront, WAF (for CloudFront).
- **Cross-Region, active-active DynamoDB?** Global Tables.
- **Cheapest S3 archive tier?** Glacier Deep Archive.
- **Layer 7 load balancer?** Application Load Balancer.
- **Service to audit API calls?** CloudTrail.
- **Fastest way to give an EC2 app AWS access?** Attach an IAM role.

Closing advice: in an AWS interview, do not just name services — explain **why** you would choose one over another and name the trade-off (cost, latency, durability, operational effort). Anchor answers in the Well-Architected pillars, always mention security and high availability by default, and when given a scenario, state your assumptions and reason out loud from requirements (RTO/RPO, scale, budget) to a design. Practice sketching the three-tier and serverless reference architectures until you can draw them from memory.
