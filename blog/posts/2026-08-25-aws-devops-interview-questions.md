# AWS DevOps Interview Questions & Answers

An interview-ready reference for **AWS DevOps** — the CI/CD, automation, and operational
side of AWS. It covers DevOps culture on AWS, the Code* pipeline services, Infrastructure
as Code with CloudFormation and CDK, configuration management with Systems Manager,
containers and serverless delivery, observability, security in pipelines, autoscaling,
disaster recovery, and real-world troubleshooting. Questions are grouped by theme and
numbered continuously; answers are concise but complete.

---

## DevOps fundamentals on AWS

**1. What is DevOps and how does AWS support it?**

DevOps is a set of cultural practices and tooling that shortens the software delivery
lifecycle by uniting development and operations — emphasizing automation, continuous
delivery, and shared ownership. AWS supports it with managed services for source control,
build, deploy, and pipeline orchestration (the Code* suite), IaC (CloudFormation, CDK),
configuration management (Systems Manager), and observability (CloudWatch, X-Ray), so
teams automate the full lifecycle without running the tooling themselves.

**2. What is the CALMS framework?**

CALMS is a model for assessing DevOps maturity:

- **Culture** — collaboration and shared responsibility.
- **Automation** — automate builds, tests, deployments, and infrastructure.
- **Lean** — reduce waste, small batch sizes, fast feedback.
- **Measurement** — collect metrics on everything to improve.
- **Sharing** — open communication and knowledge across teams.

On AWS, Automation maps to the Code* services and IaC; Measurement maps to CloudWatch and
DevOps Guru.

**3. What are the DORA metrics and how do you measure them on AWS?**

The four DORA (DevOps Research and Assessment) metrics are **deployment frequency**,
**lead time for changes**, **change failure rate**, and **mean time to recovery (MTTR)**.
On AWS you derive them from CodePipeline execution history and CodeDeploy deployment data
(frequency, lead time, failure rate) and from CloudWatch alarms plus incident timelines
(MTTR). Many teams push these events to EventBridge and aggregate in a dashboard.

**4. What is continuous integration vs. continuous delivery vs. continuous deployment?**

**Continuous integration (CI)** merges code frequently and runs automated builds/tests on
each change. **Continuous delivery (CD)** keeps every change in a releasable state with an
automated pipeline up to a manual approval before production. **Continuous deployment**
removes that manual gate — every passing change goes to production automatically.

**5. What is a "pipeline as code" and why does it matter?**

Defining the CI/CD pipeline itself in version-controlled files (e.g. a CloudFormation/CDK
definition of a CodePipeline, or `buildspec.yml`) rather than clicking in a console. It
gives you review, repeatability, auditability, and the ability to recreate the pipeline in
another account or region.

**6. What is the shared responsibility model and how does it affect DevOps?**

AWS secures the infrastructure *of* the cloud (hardware, managed service internals); the
customer secures what they run *in* the cloud (IAM, data, patching of EC2, application
config). For DevOps this means pipelines must bake in the customer-side controls —
least-privilege IAM, patch automation, encryption, and compliance scanning.

**7. What is a Well-Architected pipeline?**

A delivery pipeline aligned with the AWS Well-Architected Framework's operational
excellence pillar: small, frequent, reversible changes; everything as code; automated
testing and rollback; observability at each stage; and least-privilege permissions. AWS
publishes a "Deployment Pipeline Reference Architecture" describing these stages.

**8. What is AWS DevOps Guru?**

An ML-powered service that analyzes operational data (CloudWatch metrics, CloudTrail
events, config changes) to detect anomalous behavior, surface likely root causes, and give
remediation recommendations — reducing MTTR without hand-tuned alarms.

---

## CI/CD on AWS — the Code* suite

**9. What are the main AWS "Code" services?**

- **CodeCommit** — managed Git repositories (now closed to new customers; still used by
  existing accounts).
- **CodeBuild** — managed build/test service.
- **CodeDeploy** — automated application deployment to EC2, on-prem, Lambda, and ECS.
- **CodePipeline** — orchestrates the end-to-end release workflow.
- **CodeArtifact** — managed artifact/package repository.
- **CodeCatalyst** — unified DevOps service tying source, CI/CD, and project tooling.

**10. What is AWS CodeCommit?**

A fully managed source-control service hosting private Git repositories, integrated with
IAM for access control and KMS for encryption. Note that AWS stopped onboarding new
CodeCommit customers in 2024; many teams now use GitHub/GitLab with CodePipeline instead.

**11. What is AWS CodeBuild?**

A fully managed build service that compiles source, runs tests, and produces artifacts. It
scales automatically (no build servers to manage), bills per build minute, and runs each
build in an isolated container using a build spec.

**12. What is a `buildspec.yml`?**

A YAML file (by default at the repo root) that tells CodeBuild what to do in each phase:

```yaml
version: 0.2
env:
  variables:
    ENV: "prod"
  parameter-store:
    DB_PASS: /myapp/db/password
phases:
  install:
    runtime-versions:
      nodejs: 20
    commands:
      - npm ci
  pre_build:
    commands:
      - npm run lint
  build:
    commands:
      - npm run build
      - npm test
  post_build:
    commands:
      - echo Build completed on `date`
artifacts:
  files:
    - '**/*'
  base-directory: dist
cache:
  paths:
    - 'node_modules/**/*'
```

The phases run in order: `install`, `pre_build`, `build`, `post_build`.

**13. What are the phases of a CodeBuild build?**

`SUBMITTED` → `QUEUED` → `PROVISIONING` → `DOWNLOAD_SOURCE` → `INSTALL` → `PRE_BUILD` →
`BUILD` → `POST_BUILD` → `UPLOAD_ARTIFACTS` → `FINALIZING` → `COMPLETED`. The user-defined
`install`/`pre_build`/`build`/`post_build` commands come from the buildspec.

**14. How do you speed up CodeBuild builds?**

- Enable **caching** (local Docker layer cache or S3 cache for dependencies).
- Use a larger **compute type** for CPU/memory-bound builds.
- Pull base images from **ECR** (with a pull-through cache) instead of Docker Hub.
- Split independent stages to run in parallel via batch builds.
- Keep the source and artifacts small; use `.gitignore`-style excludes.

**15. What is AWS CodeDeploy?**

A deployment service that automates releasing applications to EC2/on-prem instances,
Lambda functions, and ECS services. It manages the rollout strategy, health checks, and
automatic rollback on failure.

**16. What is an `appspec.yml` (appspec file)?**

The CodeDeploy configuration that defines what to deploy and which lifecycle hooks to run.
Format differs by platform. For EC2/on-prem (YAML):

```yaml
version: 0.0
os: linux
files:
  - source: /
    destination: /var/www/html
hooks:
  BeforeInstall:
    - location: scripts/stop_server.sh
      timeout: 300
  ApplicationStart:
    - location: scripts/start_server.sh
      timeout: 300
  ValidateService:
    - location: scripts/validate.sh
```

For Lambda/ECS the appspec is often JSON/YAML referencing the function version or task
definition and a traffic-shifting hook.

**17. What are CodeDeploy lifecycle event hooks?**

Named points in a deployment where you run scripts. For EC2/on-prem, common ones are
`ApplicationStop`, `BeforeInstall`, `AfterInstall`, `ApplicationStart`, and
`ValidateService`. For Lambda/ECS there are `BeforeAllowTraffic` and `AfterAllowTraffic`
hooks for validation before/after traffic shifts.

**18. What deployment strategies does CodeDeploy support?**

- **In-place (rolling)** — update instances in the existing fleet, in batches.
- **Blue/green** — provision a parallel (green) environment, shift traffic, then retire
  blue.
- **Canary** — shift a small percentage of traffic first, wait, then the rest.
- **Linear** — shift equal increments at fixed intervals.
- **All-at-once** — deploy to everything simultaneously (fastest, riskiest).

**19. Explain blue/green deployment on AWS.**

Two identical environments run in parallel: **blue** (current) and **green** (new). You
deploy and test on green, then switch traffic (via an ALB target group, Route 53, or
Lambda alias) to green. If something breaks, you flip back to blue instantly. It gives
near-zero-downtime releases and fast rollback at the cost of running two environments.

**20. Explain canary vs. linear deployment.**

**Canary** shifts a small slice of traffic (e.g. 10%) to the new version, pauses to bake
and observe metrics, then shifts the remaining 90% at once. **Linear** shifts traffic in
equal steps at fixed intervals (e.g. 10% every minute). Both limit blast radius; canary
front-loads the risk check, linear spreads it evenly.

**21. What is AWS CodePipeline?**

A fully managed continuous delivery service that models your release process as a series
of **stages** (source, build, test, deploy) containing **actions**. It listens for source
changes, moves artifacts between stages, and integrates with CodeBuild, CodeDeploy,
CloudFormation, third-party tools, and Lambda.

**22. What are stages, actions, and transitions in CodePipeline?**

A **stage** is a logical grouping (e.g. "Build"). An **action** is a task within a stage
(e.g. run CodeBuild). **Transitions** are the links between stages that can be disabled to
pause flow. Actions in a stage can run in parallel via `runOrder`, and a stage must
succeed before the transition to the next.

**23. How do artifacts flow between CodePipeline stages?**

Each action can declare input and output artifacts, stored in an S3 **artifact bucket**.
The source action outputs source code; build outputs a build artifact; deploy consumes it.
Encryption uses KMS, and artifact names must match between producing and consuming actions.

**24. What are approval actions in CodePipeline?**

A **manual approval** action pauses the pipeline until a human with the right IAM
permission approves or rejects — typically before a production deploy. It can post to an
SNS topic and include a review URL and comments.

**25. How do you trigger a CodePipeline?**

- **Source changes** via EventBridge (CodeCommit) or webhooks (GitHub/Bitbucket).
- A **schedule** via EventBridge Scheduler.
- **Manually** via console/CLI (`start-pipeline-execution`).
- **Programmatically** from another pipeline or Lambda.

Modern pipelines use CodeStar Connections for GitHub/Bitbucket/GitLab integration.

**26. What is a CodeStar Connection?**

A managed OAuth-style connection (now under "Developer Tools connections") that lets
CodePipeline, CodeBuild, and other services access third-party source providers like
GitHub, GitHub Enterprise, Bitbucket, and GitLab without storing personal tokens.

**27. What is AWS CodeArtifact?**

A fully managed artifact repository for software packages — npm, PyPI, Maven/Gradle,
NuGet, and generic. It can proxy public registries (upstream repositories) and cache them,
giving you a private, controlled, versioned package source integrated with IAM.

**28. What is AWS CodeCatalyst?**

A unified DevOps service that combines source repos, CI/CD workflows, issue tracking,
dev environments (Dev Environments/Cloud IDEs), and project blueprints in one place, with
built-in integrations to AWS accounts — aimed at spinning up an end-to-end project quickly.

**29. How would you build a full CI/CD pipeline for a containerized app on AWS?**

1. **Source** — GitHub via CodeStar Connection.
2. **Build** — CodeBuild runs tests, builds a Docker image, pushes to **ECR**.
3. **Scan** — ECR image scanning / Inspector gates on vulnerabilities.
4. **Deploy** — CodeDeploy (ECS blue/green) or CodePipeline ECS deploy action updates the
   service using a new task definition.
5. **Verify** — CloudWatch alarms + smoke tests; auto-rollback on failure.

Everything defined as code (CDK/CloudFormation) and permissioned with least-privilege IAM
roles.

**30. How do you implement automatic rollback in a pipeline?**

- **CodeDeploy** — configure a deployment group to roll back on alarm or on a failed
  deployment/health check.
- **CloudFormation** — stack rollback triggers on CloudWatch alarms; failed updates roll
  back automatically.
- **CodePipeline** — stage-level automatic rollback to the last successful execution.
- Attach **CloudWatch alarms** (error rate, latency) as the rollback signal.

---

## Infrastructure as Code — CloudFormation & CDK

**31. What is AWS CloudFormation?**

A service that provisions and manages AWS resources declaratively using **templates**
(JSON/YAML). You describe the desired resources and CloudFormation figures out the order,
handles dependencies, and manages the lifecycle as a **stack**.

**32. What are the main sections of a CloudFormation template?**

```yaml
AWSTemplateFormatVersion: "2010-09-09"
Description: Example stack
Parameters:      # inputs at deploy time
Mappings:        # static lookup tables (e.g. region -> AMI)
Conditions:      # create resources conditionally
Resources:       # the only REQUIRED section
Outputs:         # values to export/return
Transform:       # macros, e.g. AWS::Serverless (SAM)
```

Only `Resources` is mandatory.

**33. What is a CloudFormation stack?**

A collection of AWS resources managed together as a single unit, created from one
template. You create, update, and delete the whole set atomically; deleting the stack
deletes its resources (unless retained).

**34. What are change sets?**

A preview of the changes CloudFormation will make before you apply an update. It shows
which resources will be added, modified, or **replaced** — letting you catch destructive
changes (e.g. a rename that forces resource replacement) before executing.

**35. What is stack drift and how do you handle it?**

**Drift** is when the actual resource configuration differs from the template (someone
changed it manually in the console). Use **drift detection** to find it, then either
update the template to match, or re-apply the stack to bring resources back in line.
Prevent it by restricting console/CLI changes via IAM.

**36. What are nested stacks?**

Stacks created as resources within a parent stack (`AWS::CloudFormation::Stack`). They let
you break a large template into reusable components (e.g. a VPC stack, a security-group
stack) referenced by the parent — improving reuse and staying within template size limits.

**37. What are StackSets?**

A feature to deploy a single CloudFormation template across **multiple accounts and
regions** from a central administrator account. Ideal for org-wide baselines (guardrails,
IAM roles, logging) managed with AWS Organizations.

**38. What is a CloudFormation Custom Resource?**

A resource backed by a Lambda function (or SNS topic) that runs custom logic during stack
create/update/delete — used to provision things CloudFormation doesn't natively support or
to fetch/compute values at deploy time.

**39. What are `DependsOn`, `Ref`, and `Fn::GetAtt`?**

- **`DependsOn`** — explicitly orders resource creation.
- **`Ref`** — returns a resource's default identifier or a parameter's value.
- **`Fn::GetAtt`** — returns a specific attribute of a resource (e.g. an instance's
  private IP, a bucket's ARN).

**40. What are CloudFormation deletion and update policies?**

- **`DeletionPolicy`** — `Retain`, `Delete`, or `Snapshot` a resource when the stack or
  resource is deleted.
- **`UpdateReplacePolicy`** — controls what happens to the old resource when an update
  forces replacement.
- **`UpdatePolicy`** — governs how updates roll out for ASGs, ElastiCache, etc.

**41. What is a CloudFormation stack policy?**

A JSON document attached to a stack that protects specific resources from being updated or
replaced during a stack update — a guardrail against accidental changes to critical
resources like databases.

**42. What is the AWS CDK?**

The **Cloud Development Kit** lets you define infrastructure using real programming
languages (TypeScript, Python, Java, C#, Go). It **synthesizes** to CloudFormation
templates, so you get loops, conditionals, and abstractions ("constructs") while still
deploying through CloudFormation.

**43. What are CDK constructs (L1/L2/L3)?**

- **L1 (Cfn* )** — raw one-to-one mappings of CloudFormation resources.
- **L2** — higher-level constructs with sensible defaults and helper methods (e.g.
  `s3.Bucket`).
- **L3 (patterns)** — opinionated multi-resource patterns (e.g.
  `ApplicationLoadBalancedFargateService`).

**44. What are the key CDK CLI commands?**

```bash
cdk init app --language typescript
cdk synth        # produce the CloudFormation template
cdk diff         # compare deployed stack vs local
cdk deploy       # deploy the stack
cdk destroy      # tear it down
cdk bootstrap    # set up the CDKToolkit resources in an account/region
```

**45. What is `cdk bootstrap`?**

A one-time (per account/region) setup that provisions the resources CDK needs to deploy —
an S3 bucket and ECR repo for assets, and IAM roles. Deployments that use assets or
cross-account roles require the environment to be bootstrapped first.

**46. CloudFormation/CDK vs. Terraform — how do they compare?**

CloudFormation is AWS-only, uses JSON/YAML, and AWS manages state for you. CDK adds
real languages on top of CloudFormation. **Terraform** is cloud-agnostic (multi-provider),
uses HCL, and manages state externally (e.g. in S3 with DynamoDB locking). Teams pick
Terraform for multi-cloud and its module ecosystem; CDK for deep AWS integration and
familiar programming languages.

**47. How do you manage secrets in IaC?**

Never hardcode them. Reference **Secrets Manager** or **SSM Parameter Store** dynamically
(e.g. CloudFormation dynamic references `{{resolve:secretsmanager:...}}` or
`{{resolve:ssm-secure:...}}`). In CDK, look up secrets at deploy time rather than
embedding values, and keep them out of synthesized templates and version control.

**48. What is `cfn-lint` and why use it?**

An open-source linter that validates CloudFormation templates against the resource
specification — catching invalid properties, bad references, and region issues before
deployment. It's a common pipeline gate alongside `cfn-nag`/`cfn-guard` for policy checks.

---

## Configuration management — Systems Manager & more

**49. What is AWS Systems Manager (SSM)?**

A suite of tools to view and control your AWS and on-prem infrastructure at scale —
including Parameter Store, Session Manager, Run Command, Patch Manager, State Manager,
Automation, and Inventory. It relies on the **SSM Agent** running on managed instances.

**50. What is SSM Parameter Store?**

A hierarchical store for configuration data and secrets (plaintext `String`, `StringList`,
or KMS-encrypted `SecureString`). It's free for standard parameters, versioned, and
integrates with CodeBuild, CloudFormation, Lambda, and ECS for pulling config at runtime.

**51. Parameter Store vs. Secrets Manager — when to use which?**

Both store secrets. **Secrets Manager** adds automatic **rotation** (built-in for RDS,
Redshift, etc.), cross-region replication, and generation of random secrets — but costs per
secret. **Parameter Store** is cheaper/free and good for general config and simple secrets
without rotation. Use Secrets Manager when you need managed rotation.

**52. What is SSM Session Manager?**

A way to get an interactive shell on EC2/on-prem instances **without SSH, bastion hosts,
or open inbound ports**. Access is IAM-controlled, sessions can be logged to S3/CloudWatch,
and it works over the SSM Agent — a big security win for operations.

**53. What is SSM Run Command?**

Lets you run scripts or predefined **documents (SSM documents)** across many instances at
once — patching, restarting services, collecting logs — without logging in. It targets
instances by tags or resource groups and records the results.

**54. What is SSM Patch Manager?**

Automates patching of OS and applications across instances using **patch baselines**
(which patches are approved) and **maintenance windows** (when to apply). It reports
compliance so you can see which instances are missing patches.

**55. What is an SSM document?**

A JSON/YAML definition of actions Systems Manager performs — `Command` documents (Run
Command scripts), `Automation` documents (multi-step runbooks), and `Session` documents.
AWS provides many managed documents (e.g. `AWS-RunShellScript`), and you can author your
own.

**56. What is SSM State Manager?**

A configuration-management feature that keeps instances in a defined **desired state** —
e.g. ensuring the agent is installed, a config file is present, or antivirus is running —
by applying associations (a document + targets + schedule) on a cadence.

**57. What is SSM Automation?**

A runbook engine for multi-step operational tasks — patching an AMI, creating a golden
image, remediating a finding — with steps, branching, approvals, and integration with
EventBridge for event-driven remediation.

**58. What is AWS OpsWorks?**

A managed configuration-management service based on **Chef** and **Puppet** (OpsWorks for
Chef Automate, Puppet Enterprise, and OpsWorks Stacks). Note AWS has announced end-of-life
for OpsWorks; new workloads should use Systems Manager or run Chef/Puppet themselves.

**59. How do you bake configuration into images vs. apply at runtime?**

**Baking** (immutable infrastructure) uses tools like **EC2 Image Builder** or Packer to
create a golden AMI/container image with everything installed — faster boot, consistent,
easy rollback. **Runtime configuration** (mutable) uses cloud-init/user-data or SSM to
configure on launch — more flexible but slower and riskier for drift. Modern DevOps favors
baking + immutable deploys.

**60. What is EC2 Image Builder?**

A managed service that automates building, testing, and distributing hardened, up-to-date
machine images (AMIs and container images) on a pipeline schedule — replacing hand-rolled
Packer/scripts for golden-image creation.

---

## Containers & orchestration

**61. What is Amazon ECR?**

The **Elastic Container Registry** — a managed Docker/OCI image registry with IAM-based
access, encryption, lifecycle policies to expire old images, image scanning (basic and
enhanced via Inspector), and pull-through cache for upstream registries.

**62. What is Amazon ECS?**

The **Elastic Container Service** — AWS's native container orchestrator. You define
**task definitions** (containers, CPU/memory, IAM roles) and run them as **tasks** or
long-running **services** behind a load balancer, on either EC2 or Fargate capacity.

**63. What is AWS Fargate?**

A serverless compute engine for containers (ECS and EKS). You specify CPU/memory per task
and AWS runs the containers without you managing EC2 instances — no cluster capacity,
patching, or scaling of the underlying hosts to worry about.

**64. What is Amazon EKS?**

The **Elastic Kubernetes Service** — managed Kubernetes control plane. AWS runs and scales
the highly available control plane; you run workloads on managed node groups, self-managed
nodes, or Fargate. It's the choice when you need Kubernetes portability and ecosystem.

**65. ECS vs. EKS — how do you choose?**

**ECS** is simpler, tightly integrated with AWS, and has less operational overhead — good
when you're all-in on AWS and want minimal Kubernetes complexity. **EKS** gives you
standard Kubernetes (portability, Helm, operators, a huge ecosystem) at the cost of more
complexity. Choose ECS for simplicity, EKS for portability/Kubernetes-native tooling.

**66. What is an ECS task definition?**

A JSON blueprint describing how to run containers: the images, CPU/memory, port mappings,
environment variables, secrets (from SSM/Secrets Manager), logging config, and the
**task role** (app permissions) and **execution role** (pulling images/secrets).

**67. What is AWS Copilot?**

A CLI that makes deploying containerized apps to ECS/Fargate (and App Runner) easy — it
generates the infrastructure (VPC, load balancer, service) and CI/CD pipelines from a few
commands and a simple manifest, hiding the CloudFormation underneath.

**68. How do you deploy a new container version to ECS with zero downtime?**

Register a new task definition revision and update the service; ECS performs a **rolling
update** governed by `minimumHealthyPercent`/`maximumPercent`, draining old tasks only as
new ones pass health checks. For safer releases, use **CodeDeploy ECS blue/green** to spin
up a new task set, shift ALB traffic, and roll back on alarm.

**69. How do you scale ECS services automatically?**

Use **Service Auto Scaling** with target-tracking (e.g. keep average CPU at 60%),
step scaling, or scheduled scaling. For the EC2 launch type, also use **Cluster Capacity
Providers** / ASG scaling to add instances. Fargate removes the instance-scaling concern.

**70. How do you handle secrets and config for containers?**

Inject them at runtime via the task definition's `secrets` (pulled from SSM Parameter
Store or Secrets Manager by the execution role) and `environment` for non-sensitive values
— never bake secrets into the image. For EKS, use IRSA (IAM Roles for Service Accounts)
plus the Secrets Store CSI driver.

---

## Serverless deployment

**71. What is AWS SAM?**

The **Serverless Application Model** — an open-source framework and a CloudFormation
transform (`AWS::Serverless-2016-10-31`) that provides shorthand for Lambda, API Gateway,
DynamoDB, and event sources. `sam build`, `sam local`, and `sam deploy` streamline building
and testing serverless apps locally and in the cloud.

**72. What are Lambda versions and aliases?**

A **version** is an immutable snapshot of function code+config (`$LATEST` is mutable;
publishing creates numbered versions). An **alias** is a named pointer to a version (e.g.
`prod`) that you can move between versions — and it can split traffic between two versions
for weighted/canary deploys.

**73. How do you do canary deployments for Lambda?**

Use **CodeDeploy** with a Lambda deployment (SAM's `AutoPublishAlias` +
`DeploymentPreference`). CodeDeploy shifts traffic from the old version to the new one on a
canary or linear schedule using the alias, runs pre/post-traffic validation hooks, and
rolls back automatically if a CloudWatch alarm fires.

```yaml
Resources:
  MyFunction:
    Type: AWS::Serverless::Function
    Properties:
      AutoPublishAlias: live
      DeploymentPreference:
        Type: Canary10Percent5Minutes
        Alarms:
          - !Ref ErrorAlarm
```

**74. What is Lambda provisioned concurrency and when do you use it?**

Pre-initialized execution environments kept warm so invocations avoid **cold starts**. Use
it for latency-sensitive functions with predictable demand (e.g. behind a synchronous API).
You can even shift provisioned concurrency to a new version during a canary deploy.

**75. How do you package and deploy Lambda functions?**

Either a **.zip** archive uploaded to S3/directly, or a **container image** (up to 10 GB)
in ECR. Deploy via SAM/CDK/CloudFormation or the CLI. Share code with **Lambda layers**,
and pin dependencies to keep builds reproducible in the pipeline.

**76. What is the difference between SAM and the Serverless Framework?**

Both simplify serverless deployment. **SAM** is AWS-native, deploys via CloudFormation, and
integrates with AWS tooling (SAM CLI, CodeDeploy). The **Serverless Framework** is
third-party, multi-cloud, plugin-rich, and also compiles to CloudFormation on AWS. Choose
SAM for AWS-only simplicity, Serverless Framework for multi-provider or its plugin
ecosystem.

---

## Monitoring, logging & observability

**77. What is Amazon CloudWatch?**

AWS's central monitoring service for metrics, logs, alarms, dashboards, and events. It
collects metrics from AWS services (and custom metrics you publish), aggregates logs, and
triggers alarms/automation — the backbone of AWS observability.

**78. What are CloudWatch metrics, alarms, and dashboards?**

- **Metrics** — time-series data (CPU, latency, custom values).
- **Alarms** — thresholds on a metric that change state (OK/ALARM/INSUFFICIENT_DATA) and
  trigger SNS, Auto Scaling, or automation.
- **Dashboards** — customizable visualizations combining metrics and logs across services.

**79. What is CloudWatch Logs and the CloudWatch agent?**

**CloudWatch Logs** centralizes log data into log groups/streams, with retention,
metric filters, and Logs Insights for querying. The **unified CloudWatch agent** runs on
EC2/on-prem to ship system logs and custom metrics (memory, disk) that aren't available by
default.

**80. What is a CloudWatch metric filter and a composite alarm?**

A **metric filter** turns matching log patterns (e.g. "ERROR") into a metric you can alarm
on. A **composite alarm** combines multiple alarms with AND/OR logic so you alert on a
meaningful condition (e.g. high errors AND high latency) and reduce noise.

**81. What is AWS X-Ray?**

A distributed tracing service that follows requests across services (API Gateway → Lambda →
DynamoDB), producing a **service map** and latency breakdowns. It helps pinpoint
bottlenecks and errors in microservices and serverless apps.

**82. What is AWS CloudTrail and how is it different from CloudWatch?**

**CloudTrail** records **API activity** — who did what, when, from where — for auditing,
security, and compliance. **CloudWatch** monitors **performance and operational health**
(metrics/logs). Rule of thumb: CloudTrail = audit/"who did it"; CloudWatch = "how is it
performing".

**83. What is Amazon EventBridge and how is it used in DevOps automation?**

An event bus that routes events from AWS services, SaaS, and custom apps to targets (Lambda,
Step Functions, SNS, pipelines). In DevOps it powers **event-driven automation** — e.g.
trigger a pipeline on a code push, auto-remediate a Config finding, or run an SSM Automation
when GuardDuty raises an alert. EventBridge Scheduler also replaces cron-style triggers.

**84. What is the "three pillars of observability" and how does AWS cover them?**

**Metrics, logs, and traces.** AWS covers metrics with CloudWatch Metrics, logs with
CloudWatch Logs, and traces with X-Ray — increasingly unified under **CloudWatch
Application Signals** and OpenTelemetry (ADOT, the AWS Distro for OpenTelemetry).

**85. How do you set up alerting and on-call notification?**

Create CloudWatch alarms on the right SLI metrics, route them to an **SNS topic**, and
subscribe email/SMS/Chatbot (Slack/Teams) or a paging tool like PagerJob/Opsgenie/PagerDuty.
Use composite alarms to avoid noise and **AWS Chatbot** (Amazon Q Developer in chat) for
ChatOps.

---

## Security & compliance in pipelines

**86. How do you apply least-privilege IAM to CI/CD?**

Give each pipeline stage its own **service role** scoped to exactly what it needs (build
role can push to one ECR repo; deploy role can update one service). Avoid broad `*`
permissions, use resource-level ARNs and conditions, separate roles per environment, and
audit with IAM Access Analyzer.

**87. What is OIDC federation and why use it for GitHub Actions?**

OpenID Connect lets an external CI system (like **GitHub Actions**) assume an AWS IAM role
using a short-lived token **without storing long-lived AWS access keys** as secrets. You
add GitHub's OIDC provider as an IAM identity provider and trust specific repos/branches:

```yaml
# GitHub Actions
permissions:
  id-token: write
  contents: read
steps:
  - uses: aws-actions/configure-aws-credentials@v4
    with:
      role-to-assume: arn:aws:iam::123456789012:role/gh-deploy
      aws-region: us-east-1
```

The trust policy restricts `token.actions.githubusercontent.com:sub` to your repo/branch.

**88. How do you manage secrets in pipelines?**

Store them in **Secrets Manager** or **SSM Parameter Store (SecureString)** and fetch at
runtime via IAM — never commit them. In CodeBuild, reference `secrets-manager`/
`parameter-store` in the buildspec `env`. Prefer OIDC/role assumption over static keys, and
enable secret scanning to catch leaks in commits.

**89. What is Amazon Inspector?**

An automated **vulnerability management** service that continuously scans EC2 instances,
container images in ECR, and Lambda functions for known CVEs and network exposure,
prioritizing findings by risk. It's a natural gate in a pipeline to block vulnerable images.

**90. What is AWS Security Hub?**

A central service that aggregates and normalizes security findings from GuardDuty,
Inspector, Macie, Config, and partners, runs automated **security standards** checks
(CIS, PCI, AWS Foundational Best Practices), and gives a single prioritized view of your
security posture.

**91. What is Amazon GuardDuty?**

A **threat-detection** service that continuously analyzes CloudTrail, VPC Flow Logs, and DNS
logs (plus EKS/S3/RDS/Lambda protection) using ML and threat intel to detect compromised
credentials, crypto-mining, reconnaissance, and other malicious activity — findings can
trigger automated remediation via EventBridge.

**92. What is "shift left" security and DevSecOps on AWS?**

Moving security checks earlier ("left") in the lifecycle instead of at the end. On AWS:
static analysis and secret scanning in the build (Amazon Q/CodeGuru, `cfn-guard`), image
scanning (Inspector/ECR), policy-as-code gates, and runtime protection (GuardDuty, Security
Hub) — so vulnerabilities are caught before production.

**93. What is AWS Config and how does it help compliance?**

A service that records resource configurations over time and evaluates them against
**Config rules** (managed or custom). It detects non-compliant resources (e.g. a public S3
bucket), can auto-remediate via SSM Automation, and provides a compliance timeline for
audits.

**94. What is policy-as-code on AWS?**

Encoding governance rules as machine-checkable policy so they run in pipelines and at
runtime — e.g. **`cfn-guard`** or **OPA/Conftest** validating IaC, **SCPs** (Service
Control Policies) in Organizations setting account-wide guardrails, and **IAM permission
boundaries** capping what roles can grant.

---

## Autoscaling, self-healing, DR & reliability

**95. How does EC2 Auto Scaling provide self-healing?**

An **Auto Scaling Group (ASG)** maintains a desired count of instances across AZs. If an
instance fails its **health check** (EC2 or ELB), the ASG terminates and replaces it
automatically. Combined with a launch template and target-tracking scaling policies, it
keeps capacity healthy and matched to load without manual intervention.

**96. What are the main autoscaling options on AWS?**

- **EC2 Auto Scaling** — scale a fleet of instances.
- **Application Auto Scaling** — ECS services, DynamoDB, Aurora replicas, Lambda
  provisioned concurrency, etc.
- **Predictive scaling** — ML forecasts demand and scales ahead of it.
- **Scheduled scaling** — scale on known time patterns.

**97. What are the AWS disaster-recovery strategies?**

Ordered by cost and RTO/RPO:

- **Backup & Restore** — cheapest, slowest recovery (restore from backups/AMIs).
- **Pilot Light** — core services always running minimally, scale up on failover.
- **Warm Standby** — a scaled-down but fully functional copy always running.
- **Multi-site active/active** — full capacity in multiple regions, near-zero RTO/RPO.

**98. What are RTO and RPO?**

**RTO (Recovery Time Objective)** — how quickly you must restore service after an outage.
**RPO (Recovery Point Objective)** — how much data loss (measured in time) is acceptable.
They drive which DR strategy and backup frequency you choose.

**99. How do you automate DR and reliability on AWS?**

Automate backups with **AWS Backup**, replicate cross-region (S3 CRR, RDS/DynamoDB global,
AMI copy), model infrastructure as code so you can redeploy anywhere, use **Route 53
health checks + failover routing** for regional failover, and validate with **AWS Fault
Injection Service** (chaos engineering) and game days.

**100. Describe a real troubleshooting scenario: a CodePipeline deploy keeps failing at the deploy stage. How do you debug it?**

1. **Read the action failure** in the CodePipeline console — note which action and error.
2. If **CodeBuild/CodeDeploy**, open its logs in CloudWatch Logs; for CodeDeploy check the
   instance's deployment logs and the failing lifecycle hook.
3. Check **IAM** — the pipeline/deploy service role may lack a permission (very common) or
   the artifact bucket KMS key policy blocks access.
4. Verify the **artifact** exists and input/output names match between stages.
5. For ECS/EC2, confirm **health checks** pass and the new task/instance actually starts
   (image pull errors, port mismatches, failing readiness).
6. Reproduce locally where possible, fix, and let the pipeline auto-retry; add a CloudWatch
   alarm so the failure surfaces faster next time.

---

## Quick-fire round

- **Managed build service?** CodeBuild (`buildspec.yml`).
- **CodeDeploy config file?** `appspec.yml`.
- **Orchestrates the release workflow?** CodePipeline.
- **Preview CloudFormation changes?** A change set.
- **Deploy one template across accounts/regions?** StackSets.
- **CDK: template without deploying?** `cdk synth`.
- **Interactive shell, no SSH?** SSM Session Manager.
- **Free-tier config/secret store?** SSM Parameter Store.
- **Managed secret rotation?** Secrets Manager.
- **Serverless container compute?** Fargate.
- **Lambda traffic pointer you can shift?** An alias.
- **CI without static AWS keys?** OIDC role assumption.
- **Audit "who did what"?** CloudTrail.
- **Distributed tracing?** X-Ray.
- **Event-driven automation bus?** EventBridge.
- **Threat detection?** GuardDuty.
- **Self-healing EC2 fleet?** Auto Scaling Group health checks.

---

These questions track the arc most AWS DevOps interviews follow — from CALMS/DORA and the
Code* pipeline, through CloudFormation/CDK and Systems Manager, into containers,
observability, and pipeline security. The fastest way to make them stick: build one small
end-to-end pipeline for real — source in GitHub via OIDC, CodeBuild to build and scan a
container, CodeDeploy blue/green to ECS, CloudWatch alarms wired to automatic rollback, all
defined in CDK. Once you've watched a canary roll back on an alarm and debugged an IAM
permission on a deploy role, the answers stop being trivia.
