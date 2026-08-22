# DevOps Interview Questions & Answers

A broad, interview-ready reference for **DevOps** — culture and principles, CI/CD,
Infrastructure as Code, containers and orchestration, configuration management,
monitoring and observability, SRE concepts, and cloud practices. Grouped by theme, with
answers concise enough to say aloud.

---

## Culture & principles

**1. What is DevOps?**

DevOps is a culture, set of practices, and toolchain that unites software **development**
and IT **operations** to shorten the delivery lifecycle and deliver software
continuously and reliably. It emphasizes collaboration, automation, and feedback.

**2. What are the key goals/benefits of DevOps?**

Faster time to market, more frequent and reliable releases, improved collaboration
between teams, higher quality and stability, faster recovery from failures, and better
scalability through automation.

**3. What is CALMS?**

A framework for assessing DevOps maturity: **Culture** (collaboration), **Automation**,
**Lean** (eliminate waste, small batches), **Measurement** (metrics/feedback), and
**Sharing** (knowledge and responsibility).

**4. What are the DORA metrics?**

Four key metrics measuring delivery performance: **Deployment Frequency**, **Lead Time
for Changes**, **Change Failure Rate**, and **Mean Time to Restore (MTTR)**. They
correlate throughput and stability.

**5. What is "shift left"?**

Moving activities — testing, security, quality checks — **earlier** in the development
lifecycle so issues are caught sooner (cheaper to fix) rather than late in production.

**6. What is the difference between DevOps and Agile?**

**Agile** focuses on iterative software *development* and responding to change. **DevOps**
extends collaboration to *operations and delivery*, automating the path from code to
production. They're complementary — Agile builds it, DevOps ships and runs it.

**7. What is a blameless post-mortem?**

A retrospective after an incident that focuses on **systemic causes and improvements**
rather than blaming individuals, encouraging honesty and learning so the same failure
doesn't recur.

**8. What is "infrastructure as cattle, not pets"?**

Treat servers as disposable, interchangeable units (cattle) you recreate from code rather
than uniquely hand-maintained machines (pets). It enables automation, scaling, and
resilience.

---

## Version control

**9. Why is version control central to DevOps?**

It provides a single source of truth, enables collaboration, tracks history, supports
branching/merging, and is the trigger for CI/CD automation. Everything as code —
including infra and config — lives in version control.

**10. What are common Git branching strategies?**

- **Git Flow** — long-lived `develop`/`main` + feature/release/hotfix branches (heavier).
- **GitHub Flow** — short-lived feature branches off `main`, PR, merge, deploy (simple).
- **Trunk-Based Development** — everyone commits to `main` (short branches), behind feature flags; favored for high-frequency CI/CD.

**11. What is trunk-based development and why do CI/CD teams favor it?**

Developers integrate small changes into a single trunk (`main`) frequently, avoiding
long-lived branches and painful merges. It maximizes continuous integration and pairs
with feature flags for safe incomplete work.

**12. Merge vs. rebase?**

**Merge** preserves history and creates a merge commit (non-destructive). **Rebase**
rewrites commits onto a new base for a linear history (cleaner but rewrites history —
don't rebase shared branches).

**13. What is a monorepo vs. polyrepo?**

A **monorepo** holds many projects in one repository (shared tooling, atomic
cross-project changes, but needs scaling tooling). A **polyrepo** uses separate repos per
project (independence, clear ownership, but harder cross-cutting changes).

---

## CI/CD

**14. What is Continuous Integration (CI)?**

The practice of frequently merging code into a shared branch, with each change
automatically **built and tested** to catch integration problems early. Small, frequent
commits keep the codebase always integrable.

**15. Continuous Delivery vs. Continuous Deployment?**

**Continuous Delivery** — every change that passes automated checks is **ready** to
deploy, with a manual approval before production. **Continuous Deployment** — every
passing change is automatically **released to production**, no manual gate.

**16. What is a CI/CD pipeline?**

An automated sequence of stages that takes code from commit to production: typically
build → test → security scan → package → deploy (to staging/prod), with gates and
notifications. Defined as code (pipeline-as-code).

**17. What are the typical stages of a pipeline?**

Source (trigger) → build/compile → unit tests → static analysis/security scans →
package/artifact → deploy to staging → integration/e2e tests → approval → deploy to
production → post-deploy verification/monitoring.

**18. Name common CI/CD tools.**

Jenkins, GitLab CI/CD, GitHub Actions, CircleCI, Argo CD, Azure DevOps, Travis CI,
TeamCity, Spinnaker. For CD to Kubernetes: Argo CD and Flux (GitOps).

**19. What is an artifact and an artifact repository?**

An **artifact** is a build output (binary, container image, package). An **artifact
repository** (Artifactory, Nexus, GHCR, ECR) stores versioned artifacts for
promotion/reuse across environments, ensuring you deploy the same build you tested.

**20. What is "build once, deploy everywhere"?**

Build a single immutable artifact and promote that exact artifact through environments
(dev → staging → prod), configuring per-environment via external config — avoiding
rebuild drift between environments.

**21. What deployment strategies do you know?**

- **Rolling** — replace instances gradually.
- **Blue-Green** — two environments; switch traffic instantly, easy rollback.
- **Canary** — release to a small percentage first, then ramp up.
- **A/B testing** — route by criteria to compare variants.
- **Recreate** — stop old, start new (downtime).

**22. Blue-green vs. canary — trade-offs?**

**Blue-green** gives instant cutover and rollback but needs double the resources.
**Canary** limits blast radius by exposing a small user subset first, catching issues
with minimal impact, but is more complex to route and monitor.

**23. What is a feature flag?**

A runtime toggle that enables/disables functionality without deploying new code —
decoupling deploy from release, enabling trunk-based dev, canary/gradual rollouts, and
quick kill-switches.

**24. How do you achieve zero-downtime deployments?**

Rolling/blue-green/canary strategies, health/readiness checks, graceful shutdown
(draining connections), backward-compatible database migrations, and a load balancer that
only routes to healthy instances.

**25. How do you handle database schema changes safely?**

Use **backward-compatible, expand-then-contract migrations**: add new columns/tables
first, deploy code that works with both, backfill, then remove old schema later. Version
migrations and run them as a pipeline step; avoid destructive changes tied to a single deploy.

**26. What is a rollback strategy?**

A plan to revert to the last known-good version quickly — redeploying the previous
artifact, switching blue-green traffic back, or using automated rollback on failed health
checks. Immutable artifacts and versioning make this reliable.

---

## Infrastructure as Code & configuration management

**27. What is Infrastructure as Code (IaC)?**

Managing infrastructure through machine-readable definition files under version control,
enabling repeatable, automated, reviewable provisioning instead of manual clicks.

**28. Declarative vs. imperative IaC?**

**Declarative** (Terraform, CloudFormation) — describe the desired end state; the tool
reconciles. **Imperative** — specify the exact steps/commands to reach a state.
Declarative is generally preferred for idempotency and drift management.

**29. What is idempotency and why does it matter?**

An operation is **idempotent** if applying it multiple times yields the same result as
once. IaC and config management rely on it so re-running safely converges to the desired
state without unintended side effects.

**30. IaC provisioning vs. configuration management?**

**Provisioning** (Terraform) creates infrastructure (VMs, networks, managed services).
**Configuration management** (Ansible, Chef, Puppet) configures software on existing
machines. They're often used together.

**31. Compare Ansible, Chef, and Puppet.**

**Ansible** — agentless (SSH), push-based, YAML playbooks, easy to start. **Puppet** —
agent-based, pull model, declarative DSL, strong for large fleets. **Chef** — agent-based,
Ruby DSL, procedural "recipes." Ansible is the most common today for its simplicity.

**32. Push vs. pull configuration model?**

**Push** (Ansible) — a control node pushes config to targets on demand. **Pull** (Puppet/Chef) — agents periodically fetch and apply config from a server. Pull scales well
and self-heals drift; push is simpler and on-demand.

**33. What is drift and how do you manage it?**

Drift is when real infrastructure diverges from the code (manual changes). Detect via
`terraform plan`/agent runs; remediate by re-applying IaC. Prevent by disallowing manual
changes and enforcing changes only through pipelines.

---

## Containers & orchestration

**34. Why are containers important to DevOps?**

They package apps with dependencies for consistent behavior across environments
("works on my machine" solved), are lightweight/fast, enable microservices, and are the
unit of deployment for orchestration and CI/CD.

**35. What is container orchestration?**

Automating deployment, scaling, networking, and lifecycle of containers across a cluster
— provided by Kubernetes (dominant), Docker Swarm, or Nomad — adding self-healing, service
discovery, and rolling updates.

**36. What is GitOps?**

An operating model where **Git is the single source of truth** for declarative infra and
app config. An agent (Argo CD, Flux) continuously reconciles the cluster to match Git.
Changes happen via pull requests; the cluster auto-syncs.

**37. Benefits of GitOps?**

Auditable change history, easy rollback (revert a commit), consistency between declared
and actual state, PR-based review/approval, and self-healing reconciliation. Deployments
become Git operations.

**38. What is a service mesh?**

An infrastructure layer (Istio, Linkerd) that manages service-to-service communication —
mTLS, traffic routing, retries, observability, and policy — via sidecar proxies, offloading
these concerns from application code.

---

## Monitoring, logging & observability

**39. What is observability and the three pillars?**

The ability to understand a system's internal state from its outputs. The three pillars:
**metrics** (numeric time series), **logs** (discrete events), and **traces**
(request flow across services). Together they answer "what/why is it broken?"

**40. Monitoring vs. observability?**

**Monitoring** tracks known/predefined conditions ("is CPU > 80%?"). **Observability**
lets you explore unknown/unforeseen problems by querying rich telemetry. Monitoring
answers known questions; observability helps ask new ones.

**41. What tools do you use for monitoring/logging?**

Metrics: **Prometheus + Grafana**, Datadog, CloudWatch. Logging: **ELK/Elastic Stack**
(Elasticsearch, Logstash, Kibana), Loki, Splunk. Tracing: Jaeger, Zipkin, OpenTelemetry.
APM: Datadog, New Relic, Dynatrace.

**42. What is Prometheus?**

An open-source metrics monitoring system that **pulls** (scrapes) time-series metrics
from instrumented targets, stores them, and supports alerting (Alertmanager) and querying
(PromQL). Grafana visualizes it.

**43. What is the ELK stack?**

Elasticsearch (search/store), Logstash (ingest/transform), Kibana (visualize) — a common
centralized logging platform. Beats/Fluentd/Fluent Bit ship logs in.

**44. What is OpenTelemetry?**

A vendor-neutral, open standard and toolset for generating and collecting telemetry
(traces, metrics, logs), so you can instrument once and export to any backend.

**45. What are the golden signals?**

Google SRE's four key service signals: **Latency**, **Traffic**, **Errors**, and
**Saturation** — a focused set to monitor user-facing systems.

**46. What is alert fatigue and how do you reduce it?**

Too many (often noisy/non-actionable) alerts, causing responders to ignore them. Reduce
by alerting on **symptoms/SLOs** not every cause, tuning thresholds, deduplicating,
setting severities, and ensuring every alert is actionable.

---

## SRE & reliability

**47. What is SRE and how does it relate to DevOps?**

Site Reliability Engineering is Google's discipline applying software engineering to
operations. It's often described as a concrete implementation of DevOps principles, using
error budgets, SLOs, and automation to balance reliability with feature velocity.

**48. What are SLI, SLO, and SLA?**

**SLI** — a measured indicator (e.g. request success rate). **SLO** — an internal target
for an SLI (e.g. 99.9%). **SLA** — a contractual agreement with consequences if missed.
SLA ⊇ SLO ⊇ SLI in stringency of commitment.

**49. What is an error budget?**

The allowed amount of unreliability (1 − SLO). It quantifies acceptable risk: if the
budget is spent, teams slow feature releases to focus on reliability; if healthy, they
can move faster. It aligns dev and ops incentives.

**50. What is MTTR, MTBF, MTTD?**

**MTTR** (Mean Time To Restore/Repair) — average time to recover from failure. **MTBF**
(Mean Time Between Failures) — reliability/uptime. **MTTD** (Mean Time To Detect) — how
fast you notice. DevOps aims to lower MTTR/MTTD.

**51. How do you improve system resilience?**

Redundancy/HA across zones, health checks + auto-healing, graceful degradation, circuit
breakers, retries with backoff, rate limiting, autoscaling, chaos testing, and
well-practiced runbooks/incident response.

**52. What is chaos engineering?**

Deliberately injecting failures (killing instances, adding latency, network partitions)
in a controlled way to validate resilience and surface weaknesses before real outages
(e.g. Chaos Monkey/Gremlin).

**53. What is a runbook?**

A documented procedure for operating a system or responding to a specific incident/alert
— steps to diagnose and remediate — reducing MTTR and reliance on tribal knowledge.

---

## Cloud & scaling

**54. Horizontal vs. vertical scaling?**

**Horizontal** (scale out) — add more instances; better for resilience and near-unlimited
scale, needs stateless design/load balancing. **Vertical** (scale up) — bigger instance;
simpler but limited and often requires downtime.

**55. What is auto-scaling?**

Automatically adjusting capacity based on demand (metrics, schedules, or queue depth) —
adding instances under load and removing them when idle — for performance and cost efficiency.

**56. Stateless vs. stateful applications — why does it matter?**

**Stateless** apps hold no session/local state, so any instance can serve any request —
ideal for horizontal scaling and resilience. **Stateful** apps need externalized state
(databases, caches, object storage) to scale safely.

**57. What is a load balancer's role in DevOps?**

Distributes traffic across healthy instances, enables horizontal scaling and zero-downtime
deploys (drain/add instances), performs health checks, and can terminate TLS — foundational
for reliable, scalable services.

**58. How do you manage secrets in a DevOps pipeline?**

Store them in a secrets manager (Vault, AWS/GCP/Azure secret stores) or CI secret store,
inject at runtime, never commit to Git, rotate regularly, scope with least privilege, and
prefer short-lived credentials/OIDC over static keys.

**59. What is immutable infrastructure?**

Servers/images are never modified after deployment; to change something you build a new
image and replace instances. Eliminates config drift and snowflake servers, and makes
rollbacks trivial.

---

## Troubleshooting & scenarios

**60. A deployment succeeded but the app is down — how do you investigate?**

Check health/readiness, application logs and error rates, recent config/secret changes,
resource limits (OOM/CPU), dependency availability (DB, cache), and rollback if needed.
Correlate the deploy time with metrics/traces to pinpoint the cause.

**61. Your pipeline is slow — how do you speed it up?**

Parallelize independent stages, cache dependencies/layers, run only affected
tests/builds, use faster runners, split monolithic pipelines, fail fast, and move slow
non-blocking checks off the critical path.

**62. How do you decide what to automate first?**

Automate the highest-frequency, most error-prone, and most time-consuming manual tasks
first (builds, tests, deployments, provisioning) — where automation yields the biggest
reliability and time payoff.

**63. How do you handle a production incident?**

Detect/acknowledge → assess impact/severity → mitigate (rollback, scale, failover) →
communicate status → resolve → conduct a blameless post-mortem with action items.
Prioritize restoring service over root-causing in the moment.

**64. How do you reduce cloud costs without hurting reliability?**

Right-size resources, use auto-scaling and spot/preemptible instances for tolerant
workloads, apply committed-use discounts, clean up idle resources, tier storage, set
budgets/alerts, and monitor cost per service with tagging.

---

## Quick-fire round

- **Merge frequently + auto-test?** Continuous Integration.
- **Auto-release to prod?** Continuous Deployment.
- **Git as source of truth for infra?** GitOps.
- **Metric monitoring stack?** Prometheus + Grafana.
- **Centralized logging?** ELK / Loki.
- **Reliability discipline?** SRE (with SLOs + error budgets).
- **Release to a small % first?** Canary.
- **Decouple deploy from release?** Feature flags.
- **Servers never modified in place?** Immutable infrastructure.
- **Delivery performance metrics?** DORA.

---

These questions span most DevOps interviews — from culture and DORA metrics through
CI/CD strategies, IaC, observability, and SRE. The strongest prep is to have built and
run something end to end: a pipeline that builds an immutable artifact, tests and scans
it, and deploys it with a canary + rollback, wired to metrics and alerts. Once you've run
a real deploy and recovered from a real incident, these answers stop being definitions
and become stories you can tell.
