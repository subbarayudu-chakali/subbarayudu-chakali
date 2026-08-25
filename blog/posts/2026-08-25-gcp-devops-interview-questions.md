# GCP DevOps Interview Questions & Answers

This reference collects 100 interview questions and answers covering DevOps on Google Cloud Platform — the culture, tooling, and automation practices that ship software reliably on GCP. It spans SRE fundamentals (a discipline Google itself created), CI/CD with Cloud Build and Cloud Deploy, Infrastructure as Code with Terraform and Config Connector, GKE and Cloud Run, secrets and configuration, GitOps, the Cloud Operations observability suite, and security in delivery pipelines. Questions are grouped into themed sections but numbered continuously from 1 to 100, so you can work through them end to end or jump to the area you are weakest in. A quick-fire round and some closing advice round things out.

---

## DevOps fundamentals and SRE on GCP

**1. What is DevOps and how does Google's approach differ from the traditional definition?**

DevOps is a set of cultural practices and tools that shorten the software delivery lifecycle by breaking down silos between development and operations, emphasizing automation, continuous delivery, and shared ownership. Google's distinctive contribution is Site Reliability Engineering (SRE), which it describes as a concrete implementation of DevOps principles. Where DevOps says "reduce silos" and "measure everything," SRE gives prescriptive answers: use error budgets, define SLOs, cap toil, and staff reliability with engineers who write software to operate systems.

The mental model to state in an interview is "class SRE implements interface DevOps." DevOps is the philosophy; SRE is one opinionated, engineering-driven implementation of it.

**2. What is Site Reliability Engineering (SRE)?**

SRE is a discipline that applies software engineering to operations problems. Instead of hiring operators to run systems by hand, you hire engineers who automate operational work, build tooling, and treat reliability as a feature with an explicit target. Google originated SRE around 2003 and later published the practices in the SRE books.

Core tenets include: measuring reliability with SLIs and SLOs, using error budgets to balance velocity against stability, capping the amount of manual "toil" an SRE does (commonly at 50%), blameless postmortems, and gradual rollouts with fast rollback.

**3. What are SLIs, SLOs, and SLAs?**

- **SLI (Service Level Indicator):** a quantitative measure of some aspect of service quality, e.g. the proportion of successful HTTP requests, or request latency under 300 ms. Usually expressed as a ratio of good events to total events.
- **SLO (Service Level Objective):** the target value or range for an SLI, e.g. "99.9% of requests succeed over a 28-day window." It is an internal reliability goal.
- **SLA (Service Level Agreement):** a contract with customers that includes consequences (credits, penalties) if objectives are missed. SLAs are typically looser than SLOs so that you breach the internal SLO before the external SLA.

**4. What is an error budget and how is it used?**

An error budget is the allowable amount of unreliability, derived directly from the SLO: if your SLO is 99.9% success, your error budget is 0.1% of requests (or time) that may fail over the window. It reframes reliability as a shared, spendable resource.

The budget governs release decisions. If the budget has room, teams ship features aggressively. If the budget is exhausted (too many recent failures), non-critical launches freeze and effort shifts to reliability work until the budget recovers. This turns "dev wants speed, ops wants stability" into a data-driven, self-regulating policy instead of an argument.

**5. What is toil and why does SRE try to eliminate it?**

Toil is manual, repetitive, automatable, tactical operational work that scales linearly with service size and has no enduring value — things like manually restarting services, applying the same config by hand, or running the same remediation each week. It is distinct from overhead (meetings, admin).

SRE caps toil (Google targets under 50% of an SRE's time) because unbounded toil crowds out engineering, causes burnout, and does not improve the system. The remedy is to automate it away, so that operating the service scales sub-linearly with its growth.

**6. What is a blameless postmortem?**

A blameless postmortem is a written retrospective after an incident that focuses on the systemic and process causes of a failure rather than assigning individual fault. The premise is that people act rationally given the information and tools they had, so failures reveal weaknesses in systems, safeguards, and documentation.

Blamelessness encourages honest disclosure, which surfaces the real root causes and produces concrete, tracked action items. If people fear punishment, they hide details and the organization stops learning.

**7. How do the DORA metrics relate to GCP DevOps?**

DORA (DevOps Research and Assessment, now part of Google Cloud) identifies four key metrics that correlate with software delivery performance:

- **Deployment frequency** — how often you deploy to production.
- **Lead time for changes** — commit to running in production.
- **Change failure rate** — percentage of deployments causing a failure.
- **Time to restore service (MTTR)** — how quickly you recover from failures.

Elite performers deploy on demand, have lead times under an hour, low change-failure rates, and restore in under an hour. On GCP you instrument these using Cloud Build history, Cloud Deploy rollout data, and Cloud Monitoring incident timelines.

**8. What does "shift left" mean in a GCP pipeline?**

Shift left means moving quality and security checks earlier in the lifecycle — into the developer's inner loop and the CI stage — rather than discovering issues in production. On GCP this looks like running unit tests, `terraform validate`/`plan`, linting, and vulnerability scanning inside Cloud Build before anything is deployed, plus Artifact Analysis scanning images as they are pushed to Artifact Registry.

The payoff is cheaper, faster feedback: a failing test in CI costs minutes, while the same defect in production can cost an incident.

**9. What is the difference between reliability and availability?**

Availability is the fraction of time a service is up and responding, often expressed in "nines" (99.9%, 99.99%). Reliability is broader: it is whether the service does what users expect, correctly, within acceptable latency and error rates, over time. A service can be "available" (returning responses) while being unreliable (returning wrong or slow answers). SRE measures reliability through multiple SLIs — availability, latency, correctness, freshness, durability — not availability alone.

---

## CI/CD with Cloud Build

**10. What is Cloud Build?**

Cloud Build is Google Cloud's fully managed, serverless CI/CD platform. It executes a series of build steps, each running as a container, on ephemeral worker VMs. You define the pipeline in a `cloudbuild.yaml` (or JSON), and Cloud Build handles provisioning, isolation, and teardown. It integrates natively with Cloud Source Repositories, GitHub, GitLab, Artifact Registry, GKE, Cloud Run, and Cloud Deploy.

Because every step is just a container image, you can run essentially any tool by pointing to the right image.

**11. Show the basic structure of a cloudbuild.yaml.**

```yaml
steps:
  - name: 'gcr.io/cloud-builders/docker'
    args: ['build', '-t', 'us-docker.pkg.dev/$PROJECT_ID/app/api:$SHORT_SHA', '.']
  - name: 'gcr.io/cloud-builders/docker'
    args: ['push', 'us-docker.pkg.dev/$PROJECT_ID/app/api:$SHORT_SHA']
  - name: 'gcr.io/google.com/cloudsdktool/cloud-sdk'
    entrypoint: gcloud
    args: ['run', 'deploy', 'api', '--image', 'us-docker.pkg.dev/$PROJECT_ID/app/api:$SHORT_SHA', '--region', 'us-central1']
images:
  - 'us-docker.pkg.dev/$PROJECT_ID/app/api:$SHORT_SHA'
options:
  logging: CLOUD_LOGGING_ONLY
timeout: 1200s
```

Each `name` is the container image for that step; steps run sequentially by default and share the `/workspace` volume.

**12. What are Cloud Build substitutions?**

Substitutions are variables interpolated into the build config at runtime. Cloud Build provides built-in ones such as `$PROJECT_ID`, `$BUILD_ID`, `$SHORT_SHA`, `$COMMIT_SHA`, `$BRANCH_NAME`, `$TAG_NAME`, `$REPO_NAME`, and `$_SERVICE_ACCOUNT`. You can also define user substitutions, which by convention start with an underscore:

```yaml
substitutions:
  _REGION: us-central1
  _SERVICE: api
steps:
  - name: 'gcr.io/google.com/cloudsdktool/cloud-sdk'
    entrypoint: gcloud
    args: ['run', 'deploy', '${_SERVICE}', '--region', '${_REGION}', '--image', '$_IMAGE']
```

Use `options: substitution_option: 'ALLOW_LOOSE'` to tolerate missing substitutions, or `dynamic_substitutions` for bash-style parameter expansion.

**13. How do Cloud Build triggers work?**

A trigger connects a source event to a build. You configure it to fire on a push to a branch, a new tag, or a pull request, optionally filtered by a regex on the branch/tag name and by included/ignored file paths. The trigger references either an inline build config or a `cloudbuild.yaml` path in the repo, and can set substitution values.

```bash
gcloud builds triggers create github \
  --name=deploy-main \
  --repo-name=my-app --repo-owner=my-org \
  --branch-pattern='^main$' \
  --build-config=cloudbuild.yaml
```

Path filters (`--included-files`, `--ignored-files`) are how you build only the affected service in a monorepo.

**14. How do you run build steps in parallel or control ordering in Cloud Build?**

By default steps run sequentially. To parallelize, give steps a unique `id` and use `waitFor` to declare dependencies. A step with `waitFor: ['-']` starts immediately at the beginning of the build; steps that list several ids wait for all of them.

```yaml
steps:
  - id: test
    name: 'node'
    args: ['npm', 'test']
    waitFor: ['-']
  - id: lint
    name: 'node'
    args: ['npm', 'run', 'lint']
    waitFor: ['-']
  - id: build
    name: 'gcr.io/cloud-builders/docker'
    args: ['build', '-t', 'img', '.']
    waitFor: ['test', 'lint']
```

**15. How do you pass data between Cloud Build steps?**

Every step shares the `/workspace` directory, which persists for the whole build, so writing files there is the simplest hand-off. For environment-style values, write them to a file in `/workspace` and read them in a later step, or use `--substitutions`. For secrets, use `availableSecrets` backed by Secret Manager. Anything outside `/workspace` (like installed binaries or `/root`) is not guaranteed to persist between steps unless you add a `volumes` mount.

**16. How do you use secrets securely in Cloud Build?**

Reference Secret Manager secrets via `availableSecrets` and expose them to a step with `secretEnv`:

```yaml
availableSecrets:
  secretManager:
    - versionName: projects/$PROJECT_ID/secrets/api-key/versions/latest
      env: 'API_KEY'
steps:
  - name: 'gcr.io/cloud-builders/curl'
    entrypoint: bash
    secretEnv: ['API_KEY']
    args: ['-c', 'curl -H "Authorization: Bearer $$API_KEY" https://example.com']
```

Note the `$$` to escape the shell variable so Cloud Build does not treat it as a substitution. The build service account needs `roles/secretmanager.secretAccessor`. Never echo secrets to logs.

**17. What machine and pool options does Cloud Build offer, and what are private pools?**

By default Cloud Build runs on a shared, Google-managed pool with public egress. You can request larger machine types (`options.machineType`, e.g. `E2_HIGHCPU_8`) for faster builds. Private pools (worker pools) are dedicated build workers that run inside your VPC, letting builds reach private resources (a private GKE control plane, Cloud SQL over private IP, internal artifact servers) and giving you a stable egress range for firewalling. Configure with `options.pool.name` pointing at the worker pool resource.

**18. What service account does Cloud Build use, and why does it matter?**

Historically Cloud Build ran as the legacy Cloud Build service account (`PROJECT_NUMBER@cloudbuild.gserviceaccount.com`). Current best practice — and the default for newer projects — is to specify a user-managed service account per trigger with least-privilege roles (e.g. only `run.developer` and `artifactregistry.writer` for a Cloud Run deploy). This limits blast radius if the pipeline is compromised and makes permissions auditable. You set it with `serviceAccount` in the trigger or build config.

**19. How would you build a Docker image efficiently in Cloud Build?**

Use layer caching by pulling the previous image and passing `--cache-from`, use multi-stage Dockerfiles to keep final images small, and consider Kaniko or Cloud Build's built-in caching. Example with cache-from:

```yaml
steps:
  - name: 'gcr.io/cloud-builders/docker'
    entrypoint: bash
    args: ['-c', 'docker pull us-docker.pkg.dev/$PROJECT_ID/app/api:latest || exit 0']
  - name: 'gcr.io/cloud-builders/docker'
    args: ['build', '--cache-from', 'us-docker.pkg.dev/$PROJECT_ID/app/api:latest',
           '-t', 'us-docker.pkg.dev/$PROJECT_ID/app/api:$SHORT_SHA', '.']
```

Also pick a bigger `machineType` for CPU-heavy builds and keep the build context small with a `.dockerignore`.

**20. How do you integrate testing and quality gates into Cloud Build?**

Add explicit steps for unit tests, integration tests, linting, and scanning; if any step exits non-zero the build fails and nothing downstream runs. You can gate deployment on a passing test suite by ordering the deploy step after the test step. For coverage or reports, write artifacts to `/workspace` and upload them to Cloud Storage with the `artifacts` field. Security gates like `terraform plan` review, container scanning, and Binary Authorization attestation are also implemented as build steps.

**21. What is the difference between Cloud Build and a self-hosted CI like Jenkins on GCP?**

Cloud Build is serverless: no servers to patch, per-build isolation, pay-per-use, and native IAM/Artifact Registry/Cloud Deploy integration. Jenkins gives you a mature plugin ecosystem and full control but requires you to run and maintain the controller and agents (often on GKE or a Compute Engine VM), handle scaling, security patching, and high availability yourself. Many teams run Jenkins for legacy pipelines and use Cloud Build (or GitHub Actions with Workload Identity Federation) for cloud-native workloads. Cloud Build can also be triggered from, or trigger, Jenkins.

---

## Cloud Deploy and deployment strategies

**22. What is Cloud Deploy?**

Cloud Deploy is a managed, opinionated continuous delivery service that automates promotion of releases through a sequence of environments — for example dev to staging to prod. You define a `DeliveryPipeline` and a set of `Target`s, create a `Release` from a rendered manifest, and Cloud Deploy manages rollouts, promotions, approvals, and one-click rollback. It supports GKE, Cloud Run, and GKE Enterprise targets and renders manifests with Skaffold under the hood.

**23. What are the main resources in a Cloud Deploy configuration?**

- **Delivery pipeline:** the ordered progression of stages (targets) a release moves through.
- **Target:** a deployment destination (a GKE cluster, a Cloud Run service/region), possibly requiring approval.
- **Release:** an immutable, versioned artifact rendered from your source and manifests.
- **Rollout:** the act of deploying a release to a specific target.

```yaml
apiVersion: deploy.cloud.google.com/v1
kind: DeliveryPipeline
metadata:
  name: web-pipeline
serialPipeline:
  stages:
    - targetId: dev
    - targetId: staging
    - targetId: prod
      strategy:
        canary:
          runtimeConfig:
            cloudRun: {}
          canaryDeployment:
            percentages: [25, 50]
```

**24. How does promotion and approval work in Cloud Deploy?**

After a release is deployed to the first target, you "promote" it to the next stage. Promotion creates a new rollout on the subsequent target. Any target can be marked as requiring approval (`requireApproval: true`), which pauses the rollout until an authorized user approves it — useful as a manual gate before production. Approvals and promotions are all auditable and can be driven via `gcloud deploy` or the console, and triggered automatically from Cloud Build.

**25. How does Cloud Deploy relate to Cloud Build?**

They are complementary. Cloud Build handles CI — building, testing, and pushing artifacts — and then typically calls `gcloud deploy releases create` to hand a versioned release to Cloud Deploy, which owns CD: promotion through environments, approvals, and rollback. Cloud Build is the "build the artifact" half; Cloud Deploy is the "safely progress it to production" half. Cloud Deploy itself uses Cloud Build worker pools to render and apply manifests.

**26. What is a rolling deployment?**

A rolling deployment gradually replaces instances of the old version with the new one, a few at a time, until all instances run the new version. In GKE this is the default Deployment strategy (`RollingUpdate`) controlled by `maxSurge` and `maxUnavailable`. It needs no extra capacity beyond the surge and keeps the service available throughout, but during the rollout both versions serve traffic simultaneously and rollback means rolling back the same way.

**27. What is a blue/green deployment?**

Blue/green runs two full environments: "blue" (current) and "green" (new). You deploy the new version to green while blue serves all traffic, test green, then switch all traffic to green at once (e.g. by repointing a load balancer or a Kubernetes Service selector). Rollback is instant — flip traffic back to blue. The cost is running double capacity during the switch. Cloud Run supports this naturally via revisions and traffic tags; on GKE you can use two Deployments and a Service, or Gateway/Ingress traffic switching.

**28. What is a canary deployment?**

A canary release routes a small percentage of traffic (say 5–10%) to the new version while the rest stays on the old version. You monitor error rates, latency, and business metrics; if the canary is healthy you progressively increase its traffic share until it is 100%, otherwise you roll back with minimal user impact. Cloud Deploy has first-class canary support with configurable percentages, and Cloud Run supports it by splitting traffic across revisions.

**29. When would you choose canary over blue/green?**

Choose canary when you want to limit blast radius and validate against real production traffic incrementally — ideal for high-traffic services where a bad release should only affect a small fraction of users. Choose blue/green when you need instant, all-or-nothing cutover and instant rollback, and can afford double capacity, or when the change is hard to run side by side at partial traffic (e.g. a schema-coupled change). Canary needs good observability to make the promote/rollback decision; blue/green needs capacity headroom.

**30. How do you do a canary or blue/green deployment on Cloud Run?**

Cloud Run keeps immutable revisions and lets you split traffic across them. Deploy the new revision with `--no-traffic`, then shift traffic gradually:

```bash
gcloud run deploy api --image IMAGE --no-traffic --tag canary
# send 10% to the new revision
gcloud run services update-traffic api --to-tags canary=10
# promote to 100% when healthy
gcloud run services update-traffic api --to-latest
```

For blue/green, deploy the new revision with no traffic, validate via its tagged URL, then flip 100% to it in one command.

**31. How would you roll back a bad deployment on GKE and on Cloud Run?**

On GKE, `kubectl rollout undo deployment/api` reverts to the previous ReplicaSet, or you redeploy a known-good image tag; `kubectl rollout status` and `rollout history` help. On Cloud Run, because revisions are immutable, you just shift traffic back to the previous revision with `gcloud run services update-traffic api --to-revisions PREV=100`. In Cloud Deploy, use the built-in rollback action on the target, which redeploys the prior successful release.

---

## Artifact management

**32. What is Artifact Registry?**

Artifact Registry is Google Cloud's unified, managed repository for build artifacts. It stores container images and language packages (Maven, npm, Python, Go, Debian/apt, RPM/yum) and also OS packages. It supports regional and multi-regional repositories, fine-grained IAM per repository, CMEK encryption, remote repositories (proxy/cache of upstream public registries), and virtual repositories (a single endpoint aggregating multiple repos). It integrates with Artifact Analysis for vulnerability scanning.

**33. What happened to Container Registry (gcr.io)?**

Container Registry (`gcr.io`) is deprecated and has been superseded by Artifact Registry. Google has shut down Container Registry, with automatic redirection of `gcr.io` requests to Artifact Registry in many projects. New work should use Artifact Registry (`*-docker.pkg.dev` hostnames). Container Registry stored images in a Cloud Storage bucket with coarse permissions, whereas Artifact Registry offers per-repo IAM, multiple formats, and regional control. The migration path is the `gcr.io` domain support in Artifact Registry plus tooling to copy existing images.

**34. How do you authenticate Docker to Artifact Registry?**

Configure the Docker credential helper for the registry's region:

```bash
gcloud auth configure-docker us-docker.pkg.dev
docker tag api:latest us-docker.pkg.dev/my-project/app/api:1.0.0
docker push us-docker.pkg.dev/my-project/app/api:1.0.0
```

In CI you rely on the build service account's identity rather than a stored password. For language repos, tools like Maven/npm use dedicated Artifact Registry credential helpers or settings.

**35. How do you manage artifact retention and cleanup?**

Artifact Registry supports cleanup policies per repository that delete or keep versions based on age, tag state (tagged vs untagged), and most-recent-N counts. This prevents unbounded storage growth from CI pushing an image per commit. You can also apply immutable tags to protect release artifacts, use version labels, and export/scan for compliance. Define keep and delete rules so, for example, untagged images older than 30 days are removed while the last 10 release tags are always retained.

---

## Infrastructure as Code

**36. What is Infrastructure as Code and why does it matter on GCP?**

Infrastructure as Code (IaC) means defining and provisioning infrastructure through machine-readable definition files rather than manual console clicks. It gives you version control, code review, repeatability, drift detection, and the ability to recreate environments deterministically. On GCP the dominant tool is Terraform via the Google provider; Google also offers Config Connector (Kubernetes-native) and Infrastructure Manager (managed Terraform). IaC is foundational to DevOps because it makes environments reproducible and auditable and lets infrastructure changes flow through the same CI/CD pipeline as application code.

**37. How is Terraform used with GCP?**

You use the `google` and `google-beta` providers to declare GCP resources, then run `terraform init`, `plan`, and `apply`. State is stored remotely, typically in a GCS backend with state locking. Authentication in CI usually uses Workload Identity Federation or a service account.

```hcl
terraform {
  backend "gcs" {
    bucket = "my-tfstate"
    prefix = "prod"
  }
  required_providers {
    google = { source = "hashicorp/google", version = "~> 5.0" }
  }
}
provider "google" {
  project = "my-project"
  region  = "us-central1"
}
resource "google_storage_bucket" "assets" {
  name     = "my-project-assets"
  location = "US"
}
```

**38. How do you manage Terraform state safely on GCP?**

Use a GCS backend with a dedicated bucket that has versioning enabled and appropriate IAM; GCS backends provide state locking automatically (via object generation), so you do not need a separate lock table. Never commit state to git — it can contain secrets. Separate state per environment (via `prefix` or separate buckets/workspaces), restrict who can read the bucket, and enable object versioning so you can recover from a corrupted state. Run `terraform plan` in CI and require review before `apply`.

**39. How do you structure Terraform for multiple environments?**

Common patterns: use reusable modules for shared resource definitions, then thin per-environment root configurations (dev/staging/prod) that pass different variables and use separate state prefixes. Alternatively use Terraform workspaces, though many teams prefer directory-per-environment because it is more explicit and avoids accidental cross-environment applies. Keep environment-specific values in `*.tfvars`, keep modules versioned, and gate prod applies behind manual approval in the pipeline.

**40. What is Config Connector?**

Config Connector is a GKE add-on (a set of Kubernetes controllers/operators) that lets you manage GCP resources as Kubernetes custom resources. You declare, say, a `StorageBucket` or `PubSubTopic` as a Kubernetes object, and Config Connector reconciles it into actual GCP infrastructure and keeps it in sync. It brings GCP resource management into the Kubernetes control plane and GitOps workflows, so the same tooling (kubectl, Config Sync) that manages workloads can manage cloud infrastructure.

**41. What happened to Deployment Manager?**

Deployment Manager, Google's original native IaC service using YAML/Jinja/Python templates, is deprecated. Google recommends migrating to Terraform (with the Google provider) or to Infrastructure Manager for managed Terraform, and to Config Connector for Kubernetes-native infrastructure. New projects should not adopt Deployment Manager; it will be shut down, and the ecosystem/tooling momentum is firmly behind Terraform.

**42. What is Infrastructure Manager?**

Infrastructure Manager (Infra Manager) is a managed GCP service that runs Terraform for you. You point it at Terraform configuration (e.g. in a Cloud Storage bucket or a Git repo), and it executes plans/applies using a service account, stores state, and integrates with Cloud Build worker pools. It gives you managed state, IAM-scoped execution, and a GCP-native API for Terraform without you operating your own Terraform automation, positioning it as the modern replacement for Deployment Manager.

**43. What is drift and how do you detect it on GCP?**

Drift is when the real state of infrastructure diverges from what your IaC declares — usually because someone made a manual change in the console. Undetected drift breaks reproducibility and can cause surprising `terraform apply` diffs. You detect it by running `terraform plan` on a schedule (in CI) and alerting on non-empty plans, or with `terraform plan -detailed-exitcode`. Config Connector continuously reconciles, so it actively corrects drift for the resources it manages. You can also lock down console edits via IAM and Org Policy to prevent drift in the first place.

**44. How do you validate and test Terraform in a pipeline?**

Run `terraform fmt -check`, `terraform validate`, and a security/policy scan (e.g. `tflint`, `checkov`, or `terraform-compliance`) in CI, then `terraform plan` and post the plan for review. Use policy-as-code (OPA/Conftest or Org Policy) to enforce guardrails, and require manual approval before `apply` on production. For higher assurance, use Terratest or `terraform test` to spin up throwaway resources and assert behavior, tearing them down afterward.

---

## GKE and containers

**45. What is the difference between GKE Autopilot and Standard?**

GKE Standard gives you full control over the node infrastructure: you manage node pools, machine types, autoscaling, and pay per node. GKE Autopilot is a hands-off mode where Google manages nodes entirely — you just deploy pods and pay per pod resource request (vCPU/memory/storage) actually used. Autopilot enforces security and best-practice defaults, reduces operational toil, and is great when you do not want to tune node infrastructure; Standard is better when you need custom node configurations, GPUs/TPUs with special settings, DaemonSets requiring node access, or specific machine types Autopilot restricts.

**46. What are node pools in GKE?**

A node pool is a group of nodes within a GKE cluster that share the same configuration — machine type, disk, labels, taints, and autoscaling settings. You use multiple node pools to run heterogeneous workloads on one cluster: for example a general-purpose pool, a memory-optimized pool, a GPU pool, and a Spot/preemptible pool for fault-tolerant batch work. Taints and tolerations plus node selectors/affinity steer pods to the right pool. Only Standard clusters expose node pools directly; Autopilot manages them for you.

**47. What is Workload Identity on GKE?**

Workload Identity is the recommended way for GKE pods to authenticate to Google Cloud APIs without service account keys. It binds a Kubernetes service account (KSA) to a Google service account (GSA) via IAM, so pods using that KSA automatically get short-lived credentials for the GSA. This eliminates long-lived key files (a major security risk), scopes permissions per workload, and integrates with IAM auditing.

```bash
gcloud iam service-accounts add-iam-policy-binding GSA@PROJECT.iam.gserviceaccount.com \
  --role roles/iam.workloadIdentityUser \
  --member "serviceAccount:PROJECT.svc.id.goog[NAMESPACE/KSA]"
kubectl annotate serviceaccount KSA \
  iam.gke.io/gcp-service-account=GSA@PROJECT.iam.gserviceaccount.com
```

**48. How do you use GKE in a CI/CD pipeline?**

You can run the pipeline itself on GKE (e.g. self-hosted runners, Tekton, or Argo Workflows) and/or deploy to GKE from Cloud Build/Cloud Deploy. A typical flow: Cloud Build builds and pushes an image to Artifact Registry, then either `kubectl apply`/`kustomize`/`helm` deploys it, or Cloud Deploy renders manifests with Skaffold and rolls them out with a canary strategy. For a GitOps model you push manifests to a repo and Config Sync/Argo CD reconciles them onto the cluster instead of pushing from the pipeline.

**49. What is Cloud Run and when do you use it over GKE?**

Cloud Run is a serverless container platform that runs stateless containers and scales them automatically, including to zero, in response to requests or events. Use Cloud Run when you want minimal operational overhead, per-request billing, fast scaling, and you do not need cluster-level control — ideal for web services, APIs, and event-driven workers. Use GKE when you need fine-grained control over networking, custom controllers/operators, stateful workloads, DaemonSets, service meshes, or when you are running many services that benefit from a shared cluster and advanced scheduling.

**50. What is GKE Enterprise (formerly Anthos)?**

GKE Enterprise (rebranded from Anthos) is Google's platform for managing Kubernetes across multiple clusters and environments — GCP, on-premises, and other clouds — from a single control plane. It bundles fleet management, multi-cluster services and ingress, Config Sync/Config Management for GitOps, Policy Controller, service mesh (managed Istio), and centralized security and observability. It targets hybrid and multi-cloud DevOps where you need consistent policy, config, and delivery across a fleet of clusters.

**51. What is a fleet in GKE Enterprise?**

A fleet is a logical grouping of clusters (and other resources) that you manage together as a unit, with a shared identity and namespace sameness across members. Fleets enable fleet-wide features: consistent GitOps config rollout with Config Sync, uniform policy with Policy Controller, multi-cluster service discovery, and centralized management. It is the organizing concept that lets you treat many clusters — across regions and even clouds — as one manageable surface.

**52. How does Horizontal Pod Autoscaling work in GKE?**

The Horizontal Pod Autoscaler (HPA) adjusts the number of pod replicas in a Deployment based on observed metrics — CPU, memory, or custom/external metrics (e.g. Pub/Sub queue depth or requests per second via the Custom Metrics adapter). It periodically compares current metric values to a target and scales replicas up or down to meet it. HPA (pod count) is complementary to the Vertical Pod Autoscaler (per-pod resource sizing) and to Cluster Autoscaler / node auto-provisioning (node count), which adds nodes when pods cannot be scheduled.

**53. What is the difference between HPA, VPA, and Cluster Autoscaler?**

- **HPA (Horizontal Pod Autoscaler):** changes the number of pod replicas based on load.
- **VPA (Vertical Pod Autoscaler):** adjusts CPU/memory requests and limits of pods to right-size them.
- **Cluster Autoscaler:** adds or removes nodes (in Standard node pools) when pods cannot be scheduled or nodes are underutilized.

They operate at different layers; HPA and VPA together can conflict on the same resource metric, so use VPA in recommendation mode or on different metrics. Autopilot handles node scaling automatically.

---

## Configuration and secrets management

**54. What is Secret Manager?**

Secret Manager is a managed service for storing, versioning, and accessing secrets — API keys, passwords, certificates, connection strings. Secrets are encrypted at rest, access is controlled with IAM (`roles/secretmanager.secretAccessor`), every access is audit-logged, and each secret supports multiple immutable versions so you can rotate and roll back. Applications and pipelines fetch secrets at runtime rather than baking them into images or config, and Secret Manager integrates with Cloud Build, Cloud Run, GKE, and Compute Engine.

**55. How do you inject secrets into Cloud Run and GKE?**

On Cloud Run you can mount a Secret Manager secret as an environment variable or as a file, referencing a specific version:

```bash
gcloud run deploy api --set-secrets=API_KEY=api-key:latest
```

On GKE you can use the Secret Manager CSI driver to mount secrets as files, or sync them into Kubernetes Secrets. Both rely on the workload's service account (via Workload Identity) having `secretAccessor`. Avoid hardcoding secrets in manifests or images.

**56. What is OS Config and what is it used for?**

OS Config (the VM Manager suite) manages fleets of Compute Engine VMs. It provides OS patch management (scheduling and reporting on OS updates), OS inventory (what packages and versions are installed), and OS configuration/policies (declaratively ensuring packages, files, and services are present). It is Google's answer to configuration management on VMs, reducing the need for separate agents like a standalone Ansible/Chef setup for basic patching and package state, and it reports compliance centrally.

**57. How does gcloud fit into DevOps automation?**

`gcloud` is the primary CLI for GCP and the workhorse of scripted automation and CI steps — creating resources, deploying to Cloud Run/GKE, managing IAM, and driving Cloud Deploy. In pipelines you run it from the `cloud-sdk` container image. For repeatable infrastructure prefer Terraform over imperative `gcloud`, but `gcloud` is ideal for deploys, one-off operations, and glue logic. Related tools: `gsutil`/`gcloud storage` for Cloud Storage, `bq` for BigQuery, and `kubectl` (installed via gcloud components) for GKE.

**58. How would you manage application configuration across environments?**

Keep environment-specific configuration out of the image. Options: environment variables set at deploy time (Cloud Run `--set-env-vars`, Kubernetes ConfigMaps), Secret Manager for sensitive values, and a rendering tool (Kustomize overlays or Helm values per environment) so the same base manifests are parameterized per env. Store non-secret config in version control alongside code and secrets in Secret Manager referenced by name. This keeps a single immutable artifact promoted across environments with config injected at runtime.

---

## GitOps

**59. What is GitOps?**

GitOps is an operational model where the desired state of your system — infrastructure and application config — is declared in a Git repository, and an automated agent continuously reconciles the running system to match Git. Git becomes the single source of truth, all changes go through pull requests (giving review, audit, and easy rollback via revert), and the reconciler pulls changes rather than a pipeline pushing them. It applies DevOps principles of version control and automation to operations.

**60. What is Config Sync?**

Config Sync is Google's GitOps engine for GKE and GKE Enterprise fleets. It watches one or more Git repositories (or OCI artifacts) containing Kubernetes manifests and continuously applies and reconciles them onto clusters, correcting drift. It supports hierarchical, multi-team repo structures, multi-cluster/fleet rollout, and works with Config Connector so you can GitOps your GCP infrastructure too. It is part of Anthos/GKE Enterprise Config Management.

**61. What is Anthos Config Management / Config Management?**

Anthos Config Management (now GKE Enterprise Config Management) is a suite for policy and configuration across a fleet. It includes Config Sync (GitOps reconciliation), Policy Controller (a managed Gatekeeper/OPA that enforces constraints like "no public load balancers" or "images must come from an approved registry"), and Config Connector integration. Together they let a platform team define desired config and guardrails centrally in Git and have them consistently enforced across every cluster in the fleet.

**62. How does GitOps compare to push-based CI/CD for Kubernetes?**

In push-based CD, the pipeline has cluster credentials and runs `kubectl apply` to push changes in. In GitOps (pull-based), an in-cluster agent (Config Sync, Argo CD, Flux) pulls the desired state from Git and applies it, so the cluster credentials never leave the cluster and drift is auto-corrected. GitOps gives stronger audit (everything is a Git commit), easier rollback (git revert), and better security posture (no external system holds cluster admin). Many teams still use a pipeline for CI (build/test/push image + update manifest) and GitOps for CD.

**63. What is Policy Controller?**

Policy Controller is GKE Enterprise's managed implementation of Open Policy Agent Gatekeeper. It enforces programmable policies (constraints) on Kubernetes resources at admission time and audits existing resources for violations. Platform teams use it to enforce guardrails — required labels, disallowed privileged containers, approved image registries, resource limits — consistently across the fleet, delivered via Config Sync as part of the GitOps flow. It turns compliance requirements into version-controlled, automatically enforced code.

---

## Monitoring and observability

**64. What is the Cloud Operations suite?**

Cloud Operations (formerly Stackdriver) is GCP's integrated observability suite. Its main components are Cloud Monitoring (metrics, dashboards, alerting, uptime checks, SLOs), Cloud Logging (log collection, storage, routing, and analysis), Cloud Trace (distributed tracing/latency analysis), Cloud Profiler (continuous CPU/memory profiling of production code), and Error Reporting (aggregation and alerting on application errors). Together they cover the metrics, logs, and traces "three pillars" plus profiling and error tracking, with tight integration into GKE, Cloud Run, and Compute Engine.

**65. What is Cloud Monitoring and what can it do?**

Cloud Monitoring collects metrics from GCP services, the Ops Agent on VMs, GKE, and custom/OpenTelemetry sources. It provides dashboards, uptime checks (synthetic probes of endpoints), alerting policies, and native SLO monitoring with error-budget tracking. You use it to build golden-signal dashboards (latency, traffic, errors, saturation), watch resource utilization, and drive alerts to on-call. Metrics can be from Google services automatically or user-defined custom metrics.

**66. What is Cloud Logging and what is a log sink?**

Cloud Logging ingests, stores, searches, and routes logs from GCP resources and applications. It offers a powerful query language, log-based metrics, and configurable retention via log buckets. A **log sink** routes matching log entries (selected by a filter) to a destination: another log bucket, Cloud Storage (cheap long-term archival), BigQuery (analytics/SQL over logs), or Pub/Sub (streaming to external systems/SIEM). Sinks are how you implement retention tiers, compliance archival, and export to third-party tools.

```bash
gcloud logging sinks create audit-archive \
  storage.googleapis.com/my-audit-logs-bucket \
  --log-filter='logName:"cloudaudit.googleapis.com"'
```

**67. How do alerting policies work in Cloud Monitoring?**

An alerting policy defines conditions on metrics (or log-based metrics or uptime checks) — for example "5xx error ratio > 1% for 5 minutes" — and fires an incident when the condition is met, notifying via configured channels (email, PagerDuty, Slack, Pub/Sub, SMS, webhook). Policies support multiple conditions, aggregation windows, and auto-close. Best practice is to alert on symptoms that affect users (SLO burn, high error rate/latency) rather than every low-level cause, to avoid alert fatigue.

**68. What is SLO monitoring in Cloud Monitoring?**

Cloud Monitoring lets you define SLOs on a service (availability, latency, etc.) directly, based on SLIs computed from metrics. It then tracks compliance against the objective and computes the remaining error budget over the rolling window, and you can create burn-rate alerts that fire when you are consuming the error budget too fast (e.g. a fast-burn alert for a 2% budget spend in an hour and a slow-burn alert for sustained smaller spend). This operationalizes the SRE error-budget model with tooling.

**69. What is a burn-rate alert?**

A burn-rate alert triggers based on how quickly you are consuming your error budget rather than on a raw threshold. Burn rate is the ratio of the current error rate to the rate that would exactly exhaust the budget over the SLO window. Multi-window, multi-burn-rate alerting combines a fast-burn condition (catches sudden severe outages quickly) with a slow-burn condition (catches gradual degradation) to balance fast detection against false positives. It focuses paging on situations that genuinely threaten the SLO.

**70. What is Cloud Trace and when do you use it?**

Cloud Trace is a distributed tracing system that captures how requests propagate across services and how long each span takes. In a microservices architecture you use it to find latency bottlenecks — which downstream call is slow, where time is spent — and to understand request flow. It integrates with OpenTelemetry instrumentation and correlates with logs. It is essential when a single user request fans out across many services and you need to know which hop is the problem.

**71. What are Cloud Profiler and Error Reporting?**

Cloud Profiler is a continuous, low-overhead statistical profiler that runs in production, collecting CPU and heap profiles so you can find the functions consuming the most resources over time and optimize cost/performance without a synthetic test. Error Reporting automatically aggregates and de-duplicates exceptions/errors from your logs into groups, shows frequency and first/last seen, and can alert when a new error type appears — so you notice and triage regressions quickly instead of grepping logs.

**72. What are the "golden signals" and how do you monitor them on GCP?**

The four golden signals from the SRE book are latency, traffic, errors, and saturation. On GCP you monitor them with Cloud Monitoring dashboards and metrics: latency (request latency distributions from the load balancer or app metrics), traffic (request rate/QPS), errors (5xx ratio, log-based error metrics), and saturation (CPU, memory, connection pool utilization). Alerting on these user-facing symptoms — especially via SLO burn-rate alerts — is more effective than alerting on every internal metric.

---

## Security in pipelines

**73. How do you handle IAM and service accounts for CI/CD?**

Give each pipeline a dedicated, user-managed service account with least-privilege roles scoped to exactly what it needs (e.g. `artifactregistry.writer` and `run.developer`, not `editor`). Avoid downloading service account keys — prefer keyless auth via Workload Identity Federation (for external CI like GitHub Actions) or Workload Identity (for GKE-hosted runners). Separate identities per environment so a dev pipeline cannot touch prod. Audit usage with Cloud Audit Logs and use IAM Recommender to trim unused permissions.

**74. What is Workload Identity Federation and why does it matter for CI/CD?**

Workload Identity Federation lets external workloads (GitHub Actions, GitLab CI, AWS, on-prem, any OIDC/SAML provider) impersonate a GCP service account using their own short-lived identity tokens — no long-lived service account key files. You configure a workload identity pool and a provider that trusts the external issuer, then map claims to a service account. This removes the biggest CI security risk — exported JSON keys that can leak and never expire — replacing them with short-lived, attributable credentials.

**75. Show how GitHub Actions authenticates to GCP with OIDC.**

You create a workload identity pool + provider trusting GitHub's OIDC issuer, restrict it to your repo, and grant a service account. In the workflow:

```yaml
permissions:
  id-token: write
  contents: read
steps:
  - uses: google-github-actions/auth@v2
    with:
      workload_identity_provider: projects/123/locations/global/workloadIdentityPools/gh-pool/providers/gh-provider
      service_account: deployer@my-project.iam.gserviceaccount.com
  - uses: google-github-actions/setup-gcloud@v2
  - run: gcloud run deploy api --image ... --region us-central1
```

GitHub mints an OIDC token, GCP exchanges it for short-lived credentials for the service account — no stored keys.

**76. What is Binary Authorization?**

Binary Authorization is a deploy-time security control for GKE and Cloud Run that enforces that only trusted container images run. You define a policy requiring images to be signed by attestors (attestations produced at build/scan time proving they passed CI, were scanned, came from an approved registry). At admission, images without required attestations are blocked (or logged in dry-run mode). It gives you supply-chain enforcement — preventing untested or tampered images from reaching production.

**77. How does container/artifact vulnerability scanning work on GCP?**

Artifact Analysis (formerly Container Analysis) scans images in Artifact Registry for known vulnerabilities (CVEs) in OS packages and language dependencies, on push and continuously as new CVEs are disclosed. Results are exposed as metadata/occurrences you can query and gate on. You integrate it into the pipeline by failing a build or blocking deployment (via Binary Authorization attestation) when high-severity vulnerabilities are found, shifting security left. It also supports SBOM generation for supply-chain transparency.

**78. What is software supply chain security and how does GCP address it (SLSA)?**

Supply chain security protects the integrity of everything that goes into your software — source, dependencies, build process, and artifacts — against tampering. SLSA (Supply-chain Levels for Software Artifacts) is a framework of increasing assurance levels. GCP addresses it with Cloud Build generating verifiable build provenance, Artifact Analysis scanning and SBOMs, Artifact Registry access controls, Binary Authorization enforcing signed/attested images, and Software Delivery Shield as the overarching branded solution combining these to raise SLSA levels.

**79. How do you prevent secrets from leaking in pipelines?**

Never hardcode secrets in code, images, or manifests. Store them in Secret Manager and inject at runtime; use `secretEnv` in Cloud Build so values are not printed. Add secret-scanning to CI (and enable it on the repo) to catch accidental commits. Use keyless auth (Workload Identity / WIF) to avoid key files entirely. Restrict who can read logs (secrets can end up in logs), rotate any exposed secret immediately, and scope service account permissions so a leaked credential has limited blast radius.

**80. What is least privilege and how do you apply it on GCP?**

Least privilege means granting each identity only the permissions it needs, nothing more. On GCP: use predefined or custom roles instead of broad primitive roles (`owner`/`editor`), grant at the narrowest resource scope (folder/project/resource, not org), give pipelines and workloads dedicated service accounts, use Workload Identity to avoid keys, and apply IAM Conditions for time/context-bound access. Use IAM Recommender and Policy Analyzer to find and remove excess permissions, and audit with Cloud Audit Logs.

---

## Reliability, autoscaling, and disaster recovery

**81. How does autoscaling work for Compute Engine?**

Compute Engine uses Managed Instance Groups (MIGs) with autoscalers. A MIG maintains a set of identical VMs from an instance template; the autoscaler adds or removes instances based on target CPU utilization, load-balancing serving capacity, Cloud Monitoring metrics, or a schedule. MIGs also provide auto-healing (recreating VMs that fail health checks), rolling updates for new templates (canary/rolling), and regional distribution across zones for resilience. This is the VM-layer analog to Kubernetes autoscaling.

**82. What is a Managed Instance Group and why use it?**

A MIG is a group of identical VM instances created from a common instance template, managed as a single entity. Benefits: autoscaling to match load, auto-healing via health checks, automated rolling and canary updates when you change the template, and regional (multi-zone) deployment for high availability. It is the standard building block for scalable, self-healing VM-based services on GCP and integrates with load balancers as a backend.

**83. How do you design for high availability on GCP?**

Distribute across multiple zones (and regions for the highest tiers): regional MIGs or regional GKE clusters, multi-zone databases (Cloud SQL HA, Spanner), and a global or regional load balancer in front. Remove single points of failure, use health checks and auto-healing, replicate data, and design stateless services that can be recreated. Set capacity headroom and autoscaling. For region-level resilience, deploy to multiple regions with global load balancing and cross-region replication.

**84. What is the difference between RTO and RPO?**

- **RTO (Recovery Time Objective):** the maximum acceptable time to restore service after a disaster — how long you can be down.
- **RPO (Recovery Point Objective):** the maximum acceptable amount of data loss measured in time — how far back your last usable backup/replica can be.

They drive your DR architecture: a tight RPO demands continuous replication (e.g. Spanner, synchronous replicas), while a tight RTO demands fast automated failover (warm/hot standby) rather than restoring from cold backups.

**85. How do you automate disaster recovery on GCP?**

Codify everything as IaC (Terraform) so you can recreate infrastructure in another region on demand; store state and artifacts redundantly. Automate data protection with scheduled backups and cross-region replication (Cloud SQL automated backups + read replicas, GCS dual/multi-region buckets, Spanner multi-region). Use a warm/pilot-light standby or a hot multi-region active-active depending on RTO/RPO and cost. Automate failover with health checks and DNS/load-balancer repointing, and regularly test the DR runbook (game days) so recovery actually works when needed.

**86. What DR patterns exist and how do they trade off?**

- **Backup and restore (cold):** cheapest, highest RTO/RPO — restore from backups after disaster.
- **Pilot light:** core minimal services always running in the DR region, scaled up on failover.
- **Warm standby:** a scaled-down full copy running, promoted and scaled on failover — moderate cost, faster recovery.
- **Hot / multi-region active-active:** full capacity live in multiple regions with global load balancing — near-zero RTO/RPO, highest cost.

You choose based on how much downtime and data loss the business can tolerate versus budget.

**87. How do you make deployments themselves reliable (safe rollouts)?**

Use progressive delivery: canary or gradual rollouts with automated health/metric checks and automatic rollback on regression (Cloud Deploy supports this). Deploy behind feature flags to decouple release from deploy. Ensure backward-compatible changes (especially database schema — expand/contract migrations) so old and new versions can coexist during rollout. Keep deploys small and frequent to shrink blast radius, and always have a fast, tested rollback path. Track the change failure rate to know if your process is working.

---

## Real-world troubleshooting

**88. A Cloud Build build suddenly fails with a permission error pushing to Artifact Registry. How do you debug it?**

Check the build service account and its roles first. Confirm which service account the trigger uses and whether it has `roles/artifactregistry.writer` on the target repository (or project). A recent change from the legacy Cloud Build SA to a user-managed SA, or a new repo without the binding, is the usual cause. Verify the repository exists in the correct region and the image path host matches (`us-docker.pkg.dev/...`). Inspect the build logs for the exact denied permission, then grant the minimal role needed and re-run.

**89. A GKE pod is stuck in Pending. What do you check?**

`kubectl describe pod` and look at the events. Common causes: insufficient cluster resources (no node has enough CPU/memory — Cluster Autoscaler should add a node; if not, check node pool autoscaling limits or quotas), unsatisfiable node affinity/taints/tolerations, missing PersistentVolume, or image pull issues. Check `kubectl get nodes` for capacity, node pool max size, and any scheduling constraints on the pod. On Autopilot, Pending usually resolves as capacity is provisioned unless a constraint is unsatisfiable.

**90. A pod is in CrashLoopBackOff. How do you investigate?**

Look at the container logs with `kubectl logs POD --previous` to see why the last run exited, and `kubectl describe pod` for exit codes and events. Common causes: an application error/exception on startup, a failing liveness probe restarting a healthy-but-slow app (tune `initialDelaySeconds`), a missing config/secret or environment variable, an OOMKill (exit 137 — raise memory limits), or a bad image/command. Reproduce locally if possible, fix config or resource limits, and redeploy.

**91. Your Cloud Run service returns 503s under load. What is happening and how do you fix it?**

503s under load usually mean the service cannot keep up: requests are queuing beyond the max instances, container concurrency is set too low or too high, cold starts are adding latency, or the container is being killed (memory). Check Cloud Monitoring for instance count vs. max instances, request latency, and memory. Fixes: raise max instances, tune concurrency, set minimum instances to avoid cold starts, increase CPU/memory, and ensure the container binds to `$PORT` promptly. Also check downstream dependencies (a slow database) causing timeouts.

**92. Deployments are slow and CPU-heavy in Cloud Build. How do you speed them up?**

Enable image layer caching (`--cache-from` or Kaniko cache), use multi-stage builds and a `.dockerignore` to shrink context, and choose a larger `machineType`. Parallelize independent steps with `waitFor: ['-']`. Cache dependencies (e.g. node_modules, Go/Maven caches) in Cloud Storage between builds. Split monorepo builds with path filters so only changed services build. For very frequent builds consider a private pool sized appropriately. Measure with build timing to find the actual bottleneck before optimizing.

**93. Terraform apply fails with a state lock error. What do you do?**

This means another apply holds the lock, or a previous run crashed without releasing it. First confirm no other pipeline/person is actually running `apply`. If it is a stale lock, use `terraform force-unlock LOCK_ID` (get the ID from the error) — but only after verifying no active operation, since force-unlocking during a real apply can corrupt state. Prevent recurrence by ensuring only one pipeline runs apply per state (serialize CD jobs) and by using proper timeouts.

**94. Users report intermittent errors but dashboards look green. How do you find the problem?**

Green averages can hide tail problems. Look at percentile latency (p95/p99), not means; break metrics down by version, region, and endpoint to find a bad canary or a single failing zone. Use Cloud Trace to inspect slow/failed requests end to end and Error Reporting for spikes in a specific exception. Check logs for correlated errors and recent deploys/config changes. The issue is often a small subset — a specific pod, a dependency timeout, or a newly deployed revision — masked by aggregate metrics.

**95. A deployment succeeded but the new version is not receiving traffic on Cloud Run. Why?**

Most likely the new revision was deployed with `--no-traffic` (or tagged) and traffic is still pinned to the previous revision. Check `gcloud run services describe` traffic allocations. If you intended it live, run `gcloud run services update-traffic api --to-latest`. Other causes: traffic is split for a canary and the new revision only gets a small percentage, or the revision failed its readiness/health check and Cloud Run kept the old one serving. Verify the revision is healthy and traffic is routed as intended.

**96. How do you troubleshoot Workload Identity permission failures on GKE?**

Verify the full binding chain: the KSA is annotated with the correct GSA (`iam.gke.io/gcp-service-account`), the GSA has an IAM policy binding granting `roles/iam.workloadIdentityUser` to `serviceAccount:PROJECT.svc.id.goog[NAMESPACE/KSA]`, and the GSA itself has the target API roles. Confirm the pod actually uses that KSA and runs in the right namespace, and that Workload Identity is enabled on the cluster and node pool. Test from inside the pod by calling the metadata server or `gcloud auth list`. A namespace/KSA typo in the member string is a very common cause.

**97. A log sink stopped exporting logs to BigQuery. How do you diagnose it?**

Check the sink's writer identity permissions first — sinks use a dedicated service account that must have write access to the destination (e.g. `bigquery.dataEditor` on the dataset). A rotated/removed permission or a deleted dataset breaks export. Verify the sink filter still matches the logs you expect (a filter or logName change can silently exclude everything), check for BigQuery quota/schema errors, and confirm the destination dataset/table exists in the right project. Cloud Logging surfaces export errors you can inspect.

**98. After a region outage, how would you fail over an application automatically?**

If you designed for it: a global external load balancer with backends in multiple regions automatically stops sending traffic to the unhealthy region based on health checks, so surviving regions absorb the load — no manual action if capacity exists. For data, promote a cross-region replica (or rely on a multi-region database like Spanner). If you have only a warm standby, automation (triggered by monitoring/alerting via Cloud Functions or a runbook) scales it up and repoints DNS/LB. The key is that this must be built and tested beforehand; you cannot invent DR during the outage.

**99. Your change failure rate is climbing. How do you bring it down?**

Treat it as an SRE signal that the error budget is being spent on releases. Investigate postmortems to find patterns (insufficient testing, risky manual steps, big-bang deploys). Remedies: strengthen CI gates (more tests, integration/canary analysis), adopt progressive delivery with automated rollback, shrink deploy size and increase frequency to reduce blast radius, add feature flags, and enforce backward-compatible migrations. If the error budget is exhausted, freeze feature launches and prioritize reliability work until the trend reverses.

**100. How do you approach an incident where a recent deploy is suspected but not confirmed as the cause?**

Follow incident response fundamentals: first mitigate to restore users (often rolling back the suspect deploy is the fastest safe action — it is cheap and reversible), then investigate. Correlate the incident start time with the deploy timeline in Cloud Deploy/Cloud Build, compare metrics before/after per revision, and check Error Reporting for new error signatures appearing at deploy time. Communicate status, assign an incident commander for larger events, and once resolved run a blameless postmortem with tracked action items. Mitigate first, root-cause second.

---

## Quick-fire round

- **What replaced Container Registry?** Artifact Registry.
- **Default GKE deployment strategy?** RollingUpdate.
- **Serverless container platform on GCP?** Cloud Run.
- **Terraform state backend on GCP?** GCS bucket (with built-in locking).
- **Keyless CI auth to GCP?** Workload Identity Federation.
- **GKE pod-to-GCP-API auth without keys?** Workload Identity.
- **Managed CD service on GCP?** Cloud Deploy.
- **Managed CI service on GCP?** Cloud Build.
- **Error budget formula?** 100% minus the SLO target.
- **Tool to enforce only trusted images run?** Binary Authorization.
- **GitOps engine for GKE fleets?** Config Sync.
- **Manage GCP resources as Kubernetes objects?** Config Connector.
- **Old GCP native IaC being retired?** Deployment Manager.
- **Managed Terraform on GCP?** Infrastructure Manager.
- **Suite for metrics, logs, traces?** Cloud Operations (formerly Stackdriver).
- **Route logs to BigQuery/GCS/Pub-Sub?** Log sinks.
- **Continuous production profiler?** Cloud Profiler.
- **Cap on an SRE's toil?** About 50%.
- **Instant-rollback deploy strategy?** Blue/green.
- **Limited-blast-radius deploy strategy?** Canary.
- **VM autoscaling building block?** Managed Instance Group.
- **Max acceptable data loss?** RPO. **Max acceptable downtime?** RTO.

## Closing advice

For a GCP DevOps interview, be ready to move fluidly between the culture (SRE, SLOs, error budgets, toil, blameless postmortems) and the concrete tooling (Cloud Build, Cloud Deploy, Artifact Registry, Terraform, GKE, Cloud Run, the Operations suite). Interviewers value candidates who can name the current, recommended service and explain why a legacy one was deprecated — say Artifact Registry over Container Registry, Terraform/Infrastructure Manager over Deployment Manager, and Workload Identity Federation over service account keys. Ground your answers in trade-offs: when canary versus blue/green, when Autopilot versus Standard, when Cloud Run versus GKE. Above all, practice narrating a realistic pipeline end to end — commit, build and scan, push to Artifact Registry, release through Cloud Deploy with a canary and automated rollback, observed via SLO burn-rate alerts — because the ability to tell that story coherently is what separates a memorized-facts answer from a genuinely operational one. Set up a free-tier GCP project and actually run a small pipeline; hands-on muscle memory shows in the details.
