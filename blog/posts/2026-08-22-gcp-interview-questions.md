# Google Cloud Platform (GCP) Interview Questions & Answers

A broad, interview-ready reference for **Google Cloud Platform** — covering the
resource hierarchy, IAM, compute, storage, networking, databases, data & ML,
DevOps, and security. Questions are grouped by domain so you can jump to whatever
you're brushing up on. Answers are short enough to say aloud but complete enough to
defend a follow-up.

---

## Fundamentals & resource hierarchy

**1. What is Google Cloud Platform?**

GCP is Google's suite of cloud computing services running on the same infrastructure
Google uses internally. It offers IaaS, PaaS, and serverless across compute, storage,
networking, databases, big data, and AI/ML.

**2. What is the GCP resource hierarchy?**

From top to bottom: **Organization → Folders → Projects → Resources**. IAM policies
and org policies set at a higher level are inherited downward. The **Project** is the
core unit that groups resources, billing, and APIs.

**3. What is a Project and why does it matter?**

A project is the fundamental organizing entity: it has a unique **Project ID**
(immutable), a project number, and a name. All resources belong to a project, and
billing, quotas, permissions, and enabled APIs are tracked per project.

**4. What are regions and zones?**

A **region** is a specific geographic location (e.g. `us-central1`); a **zone** is an
isolated deployment area within a region (e.g. `us-central1-a`). Zones are the unit of
failure isolation — spread resources across zones (and regions) for high availability.

**5. Global, regional, and zonal resources — give examples.**

- **Global**: VPC networks, images, snapshots, HTTP(S) load balancers.
- **Regional**: regional managed instance groups, regional Persistent Disks, subnets.
- **Zonal**: VM instances, zonal Persistent Disks.

**6. How do you interact with GCP?**

Google Cloud Console (web UI), the `gcloud` CLI (part of the Cloud SDK), client
libraries (REST/gRPC), Cloud Shell (browser-based shell), and Terraform / Deployment
Manager for IaC.

**7. What is Cloud Shell?**

A free, browser-based shell with a temporary Compute Engine VM, the Cloud SDK
pre-installed, 5 GB of persistent home storage, and built-in editor — handy for quick
admin without local setup.

---

## IAM & security

**8. What is Cloud IAM?**

Identity and Access Management controls **who** (identity) can do **what** (role) on
**which** resource. It answers authorization by binding members to roles on a
resource, inherited down the hierarchy.

**9. What are the types of IAM roles?**

- **Primitive/Basic roles**: Owner, Editor, Viewer — broad, project-wide, discouraged for production.
- **Predefined roles**: curated, service-specific roles following least privilege (e.g. `roles/storage.objectViewer`).
- **Custom roles**: you assemble exactly the permissions needed.

**10. What is a service account?**

A special non-human identity used by applications/VMs to authenticate and call GCP
APIs. It has its own email, can be granted roles, and authenticates via keys or
(preferably) short-lived tokens / Workload Identity.

**11. Service account keys — best practice?**

Avoid downloading long-lived JSON keys where possible. Prefer attached service
accounts on GCP resources, **Workload Identity Federation** for external/CI workloads,
and short-lived credentials. Rotate and restrict keys if you must use them.

**12. What is Workload Identity Federation?**

It lets external identities (from AWS, Azure, GitHub Actions OIDC, etc.) impersonate a
GCP service account without a downloaded key — exchanging an external token for
short-lived GCP credentials. Big security win for CI/CD.

**13. What is the principle of least privilege in IAM?**

Grant the minimum roles needed. Favor predefined/custom roles over basic ones, scope
bindings to the narrowest resource, and use conditions (IAM Conditions) to constrain
access by attributes like time or resource name.

**14. What is an IAM policy binding?**

A binding maps `members` (users, groups, service accounts, domains) to a single
`role`, optionally with a `condition`. A policy is the collection of bindings on a
resource.

**15. What are Organization Policies?**

Constraints set at org/folder/project level to enforce guardrails (e.g. restrict
which regions can be used, disable service account key creation, require OS Login).
Distinct from IAM — they limit *what configurations are allowed*, not *who can act*.

**16. How does GCP encrypt data?**

Data is encrypted at rest by default (Google-managed keys) and in transit. You can
use **CMEK** (customer-managed keys via Cloud KMS) or **CSEK** (customer-supplied
keys) for more control.

**17. What is Cloud KMS?**

Key Management Service — creates, rotates, and manages cryptographic keys for
encryption, backed by software or Cloud HSM. Used for CMEK across storage, databases,
and more.

**18. What is Secret Manager?**

A managed service to store, version, and access secrets (API keys, passwords) with
IAM-controlled access and audit logging, instead of hardcoding them.

---

## Compute

**19. What compute options does GCP offer?**

- **Compute Engine** — IaaS virtual machines.
- **Google Kubernetes Engine (GKE)** — managed Kubernetes.
- **Cloud Run** — serverless containers.
- **App Engine** — PaaS for apps.
- **Cloud Functions** — event-driven functions (FaaS).

**20. When would you choose each compute option?**

VMs (Compute Engine) for full OS control / lift-and-shift; GKE for container
orchestration at scale with fine control; Cloud Run for stateless containerized
services that scale to zero; App Engine for managed web apps; Cloud Functions for
small event-driven glue code.

**21. What are machine types in Compute Engine?**

Predefined families (e.g. `e2`, `n2`, `c3` for general/compute, `m` for memory,
`a`/`g` for GPU/accelerators) plus **custom machine types** where you pick vCPU and
memory. Choose based on the workload's CPU/memory profile.

**22. What are preemptible / Spot VMs?**

Deeply discounted VMs that GCP can reclaim with short notice. **Spot VMs** are the
newer model (no 24-hour cap). Ideal for fault-tolerant, batch, or stateless work;
not for anything that can't tolerate interruption.

**23. What is a Managed Instance Group (MIG)?**

A group of identical VMs created from an instance template, providing autoscaling,
autohealing (recreate unhealthy VMs), rolling updates, and load-balancer integration.
Regional MIGs span zones for HA.

**24. What is an instance template?**

An immutable definition of a VM's configuration (machine type, image, disks, network,
metadata) used by MIGs to create consistent instances.

**25. What is autoscaling and how is it configured?**

MIGs scale the number of VMs based on metrics — CPU utilization, load-balancing
capacity, Cloud Monitoring metrics, or schedules — between a min and max count.

**26. Compute Engine vs. GKE vs. Cloud Run in one line each?**

Compute Engine = you manage VMs; GKE = you manage containers on managed Kubernetes;
Cloud Run = you just ship a container and Google runs it serverlessly.

**27. What is App Engine Standard vs. Flexible?**

**Standard** runs in a sandbox with fast scaling (including to zero) and specific
language runtimes. **Flexible** runs containers on managed Compute Engine VMs — more
flexibility, slower scaling, no scale-to-zero.

**28. How does Cloud Run scale to zero?**

Cloud Run creates container instances on request and removes them when idle, so you
pay only while serving (in the request-based model). Cold starts are the trade-off;
min-instances can mitigate them.

---

## Storage

**29. What are the main storage services?**

- **Cloud Storage** — object storage (buckets).
- **Persistent Disk / Hyperdisk** — block storage for VMs.
- **Filestore** — managed NFS file storage.
- **Local SSD** — ephemeral high-performance block storage attached to a VM.

**30. What are Cloud Storage storage classes?**

- **Standard** — hot, frequently accessed data.
- **Nearline** — accessed < once/month.
- **Coldline** — accessed < once/quarter.
- **Archive** — accessed < once/year, lowest cost, highest retrieval cost.

All share the same API and millisecond access; classes differ in storage vs.
retrieval pricing and minimum storage duration.

**31. How do you optimize Cloud Storage costs?**

Use **Object Lifecycle Management** to auto-transition objects to colder classes or
delete them by age; pick the right class per access pattern; use Autoclass to let GCP
move objects automatically.

**32. What is a signed URL?**

A time-limited URL that grants temporary access to a Cloud Storage object without
requiring the requester to have a Google identity — useful for controlled downloads/uploads.

**33. Cloud Storage consistency model?**

Cloud Storage provides **strong consistency** for reads-after-writes and listing —
once an upload succeeds, subsequent reads and bucket listings reflect it.

**34. Persistent Disk types?**

Standard (HDD-backed), Balanced, SSD, and Extreme, plus the newer **Hyperdisk**
families that decouple capacity from IOPS/throughput. Choose by IOPS/throughput needs.

**35. Regional vs. zonal Persistent Disk?**

A regional PD synchronously replicates across two zones for HA (survives a zone
failure); a zonal PD lives in one zone. Regional PD trades some latency/cost for durability.

**36. What is a snapshot?**

An incremental, differential backup of a Persistent Disk stored globally, used for
backup/restore and to create new disks or images.

---

## Networking

**37. What is a VPC in GCP?**

A Virtual Private Cloud is a **global**, software-defined private network. Unlike some
clouds, a GCP VPC spans all regions; **subnets** are regional within it.

**38. Auto mode vs. custom mode VPC?**

**Auto mode** creates one subnet per region automatically with predefined ranges.
**Custom mode** creates no subnets by default — you define subnets and IP ranges
explicitly (recommended for production).

**39. What are firewall rules?**

Stateful rules on a VPC that allow/deny traffic based on direction, source/destination
ranges, protocols/ports, and target tags/service accounts. They control instance-level
connectivity.

**40. What is Cloud Load Balancing? Types?**

Managed, scalable load balancing. Types include **Global external HTTP(S)** (Layer 7,
anycast IP), **external TCP/UDP (Network)**, **internal** load balancers (L4/L7), and
SSL proxy / TCP proxy. Global LBs give a single anycast IP with worldwide backends.

**41. What is Cloud CDN?**

A content delivery network that caches content at Google's edge locations, integrated
with the HTTP(S) load balancer to reduce latency and origin load.

**42. What is Cloud DNS?**

A scalable, managed authoritative DNS service with 100% SLA, supporting public and
private zones.

**43. What is Cloud NAT?**

Managed Network Address Translation that lets instances **without** external IPs reach
the internet for outbound traffic, without exposing them to inbound connections.

**44. VPC Peering vs. Shared VPC vs. VPN/Interconnect?**

- **VPC Peering** — connects two VPCs privately (non-transitive).
- **Shared VPC** — one host project shares subnets with service projects (central network admin).
- **Cloud VPN / Interconnect** — connect on-prem to GCP over IPsec VPN or dedicated/partner physical links.

**45. What is Private Google Access?**

Lets VMs with only internal IPs reach Google APIs and services (e.g. Cloud Storage)
without external IPs, keeping traffic on Google's network.

**46. What is VPC Service Controls?**

A security perimeter around managed services (like Cloud Storage, BigQuery) to
mitigate data exfiltration — even valid credentials can't move data outside the perimeter.

---

## Databases

**47. Overview of GCP database options?**

- **Cloud SQL** — managed MySQL, PostgreSQL, SQL Server (relational, regional).
- **Cloud Spanner** — globally distributed, strongly consistent relational DB with horizontal scale.
- **Firestore** — serverless NoSQL document database.
- **Bigtable** — wide-column NoSQL for huge analytical/time-series workloads.
- **Memorystore** — managed Redis/Memcached (in-memory cache).
- **BigQuery** — serverless analytics data warehouse.

**48. Cloud SQL vs. Cloud Spanner — when to use which?**

Use **Cloud SQL** for traditional relational workloads that fit on a single regional
instance. Use **Cloud Spanner** when you need horizontal scalability, global
distribution, and strong consistency (high-throughput, mission-critical) — at higher cost.

**49. When would you use Bigtable?**

For very large-scale (TB–PB), high-throughput, low-latency workloads with a known
key-based access pattern — time-series, IoT, fintech tick data, analytics. Not for
ad-hoc SQL or transactions.

**50. Firestore vs. Bigtable?**

Firestore is a document DB with rich queries, transactions, real-time listeners, and
strong consistency — good for app/mobile backends. Bigtable is a key/wide-column store
for massive scale with simple lookups, no complex queries.

**51. What is BigQuery?**

A serverless, highly scalable data warehouse for SQL analytics over huge datasets. It
separates storage and compute, charges by data scanned (or slots), and supports
streaming inserts, partitioning, clustering, and BigQuery ML.

**52. How do you control BigQuery cost?**

Query only needed columns (avoid `SELECT *`), use **partitioned** and **clustered**
tables to prune scanned data, preview with the dry-run byte estimate, set custom
quotas, and consider flat-rate/slot pricing for heavy predictable use.

**53. What is Memorystore?**

Fully managed in-memory data store for **Redis** and **Memcached**, used for caching
and low-latency data access to offload databases.

**54. How does Cloud SQL provide high availability?**

With an HA configuration, Cloud SQL runs a primary and a synchronous **standby** in
another zone; on failure it fails over automatically. Read replicas add read scaling
(and can be promoted).

---

## Data engineering, big data & ML

**55. What is Pub/Sub?**

A globally scalable, asynchronous messaging service for event ingestion and
decoupling. Publishers send messages to topics; subscribers receive via push or pull.
Foundation for streaming pipelines.

**56. What is Dataflow?**

A managed service for stream and batch data processing based on **Apache Beam**. It
autoscales, handles windowing/late data, and is commonly paired with Pub/Sub and BigQuery.

**57. What is Dataproc?**

Managed **Hadoop and Spark** clusters that spin up in seconds, ideal for lifting
existing Spark/Hadoop jobs to the cloud with per-second billing and ephemeral clusters.

**58. Dataflow vs. Dataproc — which to pick?**

Choose **Dataflow** for new, serverless, autoscaling Beam pipelines (no cluster to
manage). Choose **Dataproc** when you already have Spark/Hadoop code or need that
ecosystem and cluster control.

**59. What is Cloud Composer?**

Managed **Apache Airflow** for authoring, scheduling, and monitoring workflow DAGs
(orchestration across services).

**60. What is Vertex AI?**

GCP's unified ML platform for building, training, tuning, deploying, and monitoring
models — including AutoML, custom training, pipelines, a model registry, feature
store, and access to foundation models (Gemini) via the API.

**61. What is BigQuery ML?**

The ability to create and run ML models directly in BigQuery using SQL (`CREATE MODEL`),
so analysts can do regression, classification, forecasting, etc. without moving data.

**62. What is a data lakehouse pattern on GCP?**

Land raw data in **Cloud Storage** (lake), process with **Dataflow/Dataproc**, and
query/analyze in **BigQuery** (warehouse), with **Dataplex** for governance — combining
lake flexibility with warehouse performance.

---

## DevOps, IaC & operations

**63. How do you do Infrastructure as Code on GCP?**

**Terraform** (most common, with the Google provider), **Cloud Deployment Manager**
(native, being de-emphasized), or **Config Connector** (manage GCP via Kubernetes
resources). IaC gives repeatable, version-controlled infrastructure.

**64. What is Cloud Build?**

A serverless CI/CD service that runs build steps in containers, defined in
`cloudbuild.yaml`, to build/test/deploy — integrates with source repos, Artifact
Registry, and GKE/Cloud Run.

**65. What is Artifact Registry?**

The successor to Container Registry: a managed store for container images **and**
language packages (Maven, npm, Python, etc.) with fine-grained IAM and regional repos.

**66. What is Cloud Operations (formerly Stackdriver)?**

The observability suite: **Cloud Monitoring** (metrics, dashboards, alerts), **Cloud
Logging** (centralized logs), **Cloud Trace** (latency/distributed tracing), **Error
Reporting**, and **Cloud Profiler**.

**67. What is Cloud Logging and what is a log sink?**

Cloud Logging aggregates logs from GCP services and apps. A **sink** routes/exports
logs to destinations (Cloud Storage, BigQuery, Pub/Sub) based on filters — for
retention, analysis, or streaming to external SIEMs.

**68. How do you monitor and alert?**

Cloud Monitoring collects metrics; you define **alerting policies** with conditions
(thresholds, absence, rate) and **notification channels** (email, PagerDuty, Slack).
Uptime checks probe endpoints externally.

**69. What is an SLI, SLO, and error budget?**

An **SLI** is a measured indicator (e.g. request success rate); an **SLO** is the
target (e.g. 99.9%); the **error budget** is the allowed shortfall (0.1%) that governs
how much risk/change you can take. Cloud Monitoring supports SLO tracking.

---

## Billing, quotas & cost

**70. How does GCP billing work?**

Billing is per **billing account** linked to projects. You're charged per resource
usage (often per-second). Tools: budgets & alerts, billing reports, and export to
BigQuery for detailed analysis.

**71. What are committed use and sustained use discounts?**

**Sustained use discounts** apply automatically for running certain resources a large
fraction of the month. **Committed use discounts (CUDs)** give bigger savings for
committing to 1- or 3-year usage of vCPU/memory or specific resources.

**72. How do you control cloud spend?**

Set **budgets and alerts**, use quotas, right-size resources, use Spot/preemptible
VMs, apply CUDs/SUDs, lifecycle-manage storage, and analyze the billing BigQuery
export / Recommender for idle-resource cleanup.

**73. What is the Recommender?**

A service that surfaces recommendations (right-sizing VMs, idle resource removal, IAM
over-permission, CUD purchases) to optimize cost, security, and performance.

---

## Reliability & architecture

**74. How do you design for high availability on GCP?**

Deploy across multiple zones (and regions for DR), use regional MIGs + load balancers,
managed HA databases (Cloud SQL HA / Spanner), health checks + autohealing, and
stateless services so instances are replaceable.

**75. Disaster recovery strategies?**

Choose per RTO/RPO: **backup & restore** (cheapest, slow), **pilot light**,
**warm standby**, and **multi-region active-active** (fastest, priciest). Use
snapshots, cross-region replication, and multi-region storage as building blocks.

**76. How do you achieve global low latency?**

Global HTTP(S) load balancer with an anycast IP, Cloud CDN caching, multi-region
backends, and globally distributed data (Spanner, multi-region Cloud Storage/BigQuery).

**77. What is anthos / GKE Enterprise?**

A platform for running and managing Kubernetes and services consistently across GCP,
on-prem, and other clouds (hybrid/multi-cloud), with centralized config and service mesh.

---

## Quick-fire round

- **Immutable project identifier?** The Project ID.
- **Global network construct?** VPC (subnets are regional).
- **Serverless containers?** Cloud Run.
- **Object storage service?** Cloud Storage.
- **Serverless data warehouse?** BigQuery.
- **Managed Kubernetes?** GKE.
- **Messaging/eventing?** Pub/Sub.
- **Managed Airflow?** Cloud Composer.
- **Non-human identity for apps?** Service account.
- **Key management?** Cloud KMS; secrets → Secret Manager.
- **CLI tool?** `gcloud` (plus `gsutil`/`bq`, now largely folded into `gcloud storage`/`bq`).

---

These questions span the breadth most GCP interviews probe — associate-level concept
checks up through architecture and cost trade-offs. If you're prepping for a certification
(Associate Cloud Engineer or Professional Cloud Architect), the highest-leverage move is
to actually deploy a small stack: a VPC with a custom subnet, a MIG behind a global load
balancer, a Cloud SQL HA instance, and a BigQuery dataset fed by Pub/Sub → Dataflow. Wiring
it once makes every answer above concrete.
