# Kubernetes Interview Questions & Answers

A thorough, interview-ready reference for **Kubernetes** — architecture, core objects,
workloads, networking, storage, configuration, scheduling, security, and operations.
Grouped by theme, with answers concise enough to say aloud but complete enough to defend.

---

## Fundamentals & architecture

**1. What is Kubernetes?**

Kubernetes (K8s) is an open-source container **orchestration** platform that automates
deploying, scaling, and managing containerized applications across a cluster of
machines. It provides self-healing, service discovery, load balancing, rollouts, and
declarative configuration.

**2. Why use Kubernetes over plain Docker?**

Docker runs containers on one host; Kubernetes orchestrates them across many hosts with
self-healing, auto-scaling, rolling updates/rollbacks, service discovery, secret
management, and declarative desired-state reconciliation.

**3. What is a cluster?**

A set of machines (nodes) running containerized apps, managed by Kubernetes. It has a
**control plane** (the brain) and **worker nodes** (where workloads run).

**4. What are the control plane components?**

- **kube-apiserver** — the front door; all interactions go through its REST API.
- **etcd** — consistent key-value store holding all cluster state.
- **kube-scheduler** — assigns Pods to nodes based on resources/constraints.
- **kube-controller-manager** — runs controllers that reconcile desired vs. actual state.
- **cloud-controller-manager** — integrates with the cloud provider (load balancers, volumes, nodes).

**5. What are the node (worker) components?**

- **kubelet** — the node agent that ensures containers in Pods are running/healthy.
- **kube-proxy** — maintains network rules for Service networking.
- **container runtime** — runs containers (containerd, CRI-O).

**6. What is etcd and why is it critical?**

etcd is the distributed, consistent key-value store that holds the **entire cluster
state** (objects, config, secrets). Losing/corrupting it means losing the cluster's
desired state — back it up. It's the single source of truth.

**7. What is the declarative model / reconciliation loop?**

You declare the **desired state** (YAML manifests); controllers continuously compare it
to the **actual state** and take action to converge them. This control loop is the heart
of Kubernetes.

**8. What is `kubectl`?**

The CLI that talks to the API server to create, inspect, update, and delete resources
(`kubectl get/describe/apply/logs/exec`, etc.).

---

## Pods & workloads

**9. What is a Pod?**

The smallest deployable unit — one or more containers that share a network namespace
(same IP/port space), storage volumes, and lifecycle. Containers in a Pod are always
co-scheduled on the same node.

**10. Why can a Pod have multiple containers?**

For tightly coupled helpers using the **sidecar** pattern (logging, proxy, init/config
sync). They share the Pod's network and volumes, enabling local communication over
`localhost`.

**11. What is an init container?**

A container that runs to completion **before** the app containers start, used for setup
tasks (waiting for a dependency, running migrations, fetching config). Multiple init
containers run sequentially.

**12. What is a sidecar container?**

A helper container running alongside the main app in the same Pod (e.g. a log shipper or
service-mesh proxy), sharing resources and lifecycle.

**13. What is a ReplicaSet?**

A controller that ensures a specified number of identical Pod replicas are running at
all times, replacing failed ones. You rarely create it directly — a Deployment manages
it for you.

**14. What is a Deployment?**

A higher-level controller for stateless apps that manages ReplicaSets to provide
declarative updates, **rolling updates**, **rollbacks**, and scaling. The standard way
to run stateless workloads.

**15. What is a StatefulSet and when do you use it?**

A controller for **stateful** apps needing stable network identities and persistent
storage per replica (e.g. databases, Kafka). It gives ordered, predictable Pod names
(`app-0`, `app-1`), stable per-Pod storage, and ordered deployment/scaling.

**16. What is a DaemonSet?**

Ensures a copy of a Pod runs on **every** node (or a subset) — used for node-level agents
like log collectors, monitoring agents, or CNI plugins.

**17. What is a Job and a CronJob?**

A **Job** runs Pods to **completion** (batch/one-off tasks) and tracks success. A
**CronJob** creates Jobs on a **schedule** (cron syntax) for recurring tasks.

**18. Deployment vs. StatefulSet vs. DaemonSet — one line each?**

Deployment = stateless, interchangeable replicas. StatefulSet = stateful, stable
identity/storage per Pod. DaemonSet = one Pod per node.

**19. What is a rolling update and how do you roll back?**

A Deployment gradually replaces old Pods with new ones (controlled by `maxSurge`/`maxUnavailable`) for zero-downtime updates. Roll back with
`kubectl rollout undo deployment/<name>`; inspect with `kubectl rollout status/history`.

---

## Services & networking

**20. What is a Service?**

A stable abstraction that exposes a set of Pods (selected by labels) under a single,
durable virtual IP and DNS name, load-balancing traffic to healthy Pods — decoupling
clients from ephemeral Pod IPs.

**21. What are the Service types?**

- **ClusterIP** (default) — internal-only virtual IP within the cluster.
- **NodePort** — exposes the Service on a static port on every node.
- **LoadBalancer** — provisions an external cloud load balancer.
- **ExternalName** — maps the Service to an external DNS name (CNAME).

**22. What is a headless Service?**

A Service with `clusterIP: None` — no virtual IP or proxying; DNS returns the Pod IPs
directly. Used with StatefulSets for stable per-Pod DNS.

**23. What is an Ingress?**

An API object that manages **external HTTP/HTTPS** access, providing host/path-based
routing, TLS termination, and virtual hosting to Services. It requires an **Ingress
controller** (nginx, Traefik, cloud) to actually fulfill the rules.

**24. Ingress vs. LoadBalancer Service?**

A LoadBalancer Service typically exposes one Service per external IP (L4). An Ingress
uses a single entry point to route many hostnames/paths (L7) to multiple Services — more
cost-effective and feature-rich for HTTP.

**25. What is the Gateway API?**

A newer, more expressive successor to Ingress for managing traffic (L4/L7) with
role-oriented resources (GatewayClass, Gateway, HTTPRoute), addressing Ingress's
limitations.

**26. How does DNS work in Kubernetes?**

CoreDNS provides in-cluster DNS. Services get a name like
`<service>.<namespace>.svc.cluster.local`, so Pods resolve Services by name across
namespaces.

**27. What is the Kubernetes networking model?**

Every Pod gets its own IP; all Pods can communicate with all other Pods **without NAT**;
nodes can reach all Pods. A **CNI** plugin (Calico, Cilium, Flannel) implements this.

**28. What is a NetworkPolicy?**

A spec that controls allowed ingress/egress traffic between Pods/namespaces (a Pod-level
firewall) by labels. Requires a CNI that enforces policies (e.g. Calico, Cilium). Default
is allow-all until a policy selects a Pod.

**29. What does kube-proxy do?**

Programs the node's networking (iptables/IPVS/eBPF) so traffic to a Service's ClusterIP
is load-balanced to backend Pods.

---

## Configuration & storage

**30. What is a ConfigMap?**

An object storing **non-sensitive** configuration as key-value pairs, injected into Pods
as env vars, command-line args, or mounted files — decoupling config from images.

**31. What is a Secret?**

Like a ConfigMap but for **sensitive** data (passwords, tokens, keys). Values are
base64-encoded (not encrypted by default) — enable **encryption at rest** in etcd and
restrict RBAC. Injected as env vars or mounted files.

**32. Are Secrets encrypted by default?**

No — they're only base64-encoded in etcd unless you enable encryption at rest. Treat
them as sensitive: limit RBAC, enable etcd encryption, and consider external secret
managers (Vault, external-secrets).

**33. What is a Volume?**

Storage attached to a Pod that outlives individual container restarts. Types include
`emptyDir` (ephemeral, Pod-lifetime), `hostPath`, `configMap`/`secret`, and persistent
volumes via PVCs.

**34. What is a PersistentVolume (PV) and PersistentVolumeClaim (PVC)?**

A **PV** is a cluster storage resource (provisioned by an admin or dynamically). A
**PVC** is a user's request for storage (size, access mode). Kubernetes binds a PVC to a
suitable PV, decoupling apps from storage details.

**35. What is a StorageClass?**

Defines a "class" of storage and enables **dynamic provisioning** — when a PVC references
it, a PV is created automatically by the provisioner (e.g. cloud disks) with the
specified parameters.

**36. What are access modes for volumes?**

`ReadWriteOnce` (one node RW), `ReadOnlyMany` (many nodes RO), `ReadWriteMany` (many
nodes RW), and `ReadWriteOncePod` (a single Pod RW).

**37. How do you mount a ConfigMap/Secret as a file?**

Reference it as a volume of type `configMap`/`secret` and mount it at a path; each key
becomes a file. Alternatively inject keys as env vars via `valueFrom`.

---

## Scheduling & resource management

**38. How does the scheduler place Pods?**

kube-scheduler filters nodes that meet a Pod's requirements (resources, taints,
affinities) then scores the feasible ones, assigning the Pod to the best-scoring node.

**39. What are requests and limits?**

**Requests** are guaranteed resources used for scheduling; **limits** cap what a
container may use. Exceeding a memory limit gets the container OOM-killed; exceeding a
CPU limit throttles it.

**40. What are QoS classes?**

Based on requests/limits: **Guaranteed** (requests == limits for all resources),
**Burstable** (requests < limits), **BestEffort** (none set). They influence eviction
order under node pressure (BestEffort evicted first).

**41. What are node selectors, affinity, and anti-affinity?**

`nodeSelector` is simple label-based node targeting. **Node affinity** adds expressive
required/preferred rules. **Pod affinity/anti-affinity** schedules Pods relative to
other Pods (co-locate or spread apart).

**42. What are taints and tolerations?**

A **taint** on a node repels Pods that don't **tolerate** it. Together they reserve nodes
for specific workloads (e.g. GPU nodes) or keep general Pods off control-plane nodes.

**43. What is a topology spread constraint?**

Rules to evenly distribute Pods across failure domains (zones, nodes) for resilience,
using `topologySpreadConstraints`.

**44. What is Horizontal Pod Autoscaler (HPA)?**

Automatically scales the **number of Pod replicas** based on observed metrics (CPU,
memory, or custom/external metrics) between a min and max.

**45. What is Vertical Pod Autoscaler (VPA)?**

Adjusts the **CPU/memory requests/limits** of Pods (scaling up/down resources per Pod)
rather than the replica count.

**46. What is Cluster Autoscaler?**

Adds/removes **nodes** in the cluster based on pending Pods that can't be scheduled and
underutilized nodes — scaling the infrastructure itself.

**47. What is a PodDisruptionBudget (PDB)?**

Limits how many Pods of an app can be voluntarily disrupted (during drains/upgrades) at
once, ensuring a minimum availability during maintenance.

---

## Health, probes & lifecycle

**48. What probes does Kubernetes support?**

- **Liveness** — is the container alive? Failing restarts it.
- **Readiness** — is it ready to serve? Failing removes it from Service endpoints.
- **Startup** — has a slow-starting app finished booting? Delays the other probes until it passes.

**49. Liveness vs. readiness — why both?**

Liveness recovers a hung process by restarting it; readiness prevents routing traffic to
a Pod that's up but not ready (still warming up or temporarily overloaded). Confusing
them causes needless restarts or dropped traffic.

**50. What are the Pod phases/statuses?**

`Pending`, `Running`, `Succeeded`, `Failed`, `Unknown`. Container states include
`Waiting`, `Running`, `Terminated`. `CrashLoopBackOff` indicates repeated crashes with
increasing backoff.

**51. What is `CrashLoopBackOff` and how do you debug it?**

A container keeps crashing and Kubernetes backs off restarts. Debug with
`kubectl logs <pod> --previous`, `kubectl describe pod`, check the command/args,
env/config, resource limits (OOM), and dependency readiness.

**52. What happens during Pod termination?**

The Pod is marked Terminating, removed from Service endpoints, `preStop` hook runs, then
containers get SIGTERM; after `terminationGracePeriodSeconds` they get SIGKILL. Handle
SIGTERM for graceful shutdown.

---

## Organization, security & RBAC

**53. What is a Namespace?**

A virtual cluster within a cluster to isolate and organize resources (per team/env),
scope names, and apply quotas/policies/RBAC. Some resources are cluster-scoped (nodes,
PVs), others namespaced.

**54. What are labels and selectors?**

**Labels** are key-value tags on objects; **selectors** query objects by labels. They're
how Services find Pods, how Deployments manage ReplicaSets, and how you group/filter
resources.

**55. Labels vs. annotations?**

Labels are for **identifying/selecting** objects (queryable). **Annotations** hold
arbitrary non-identifying metadata (build info, tooling config) and aren't used for
selection.

**56. What is RBAC?**

Role-Based Access Control governs who can do what. **Role**/**ClusterRole** define
permissions; **RoleBinding**/**ClusterRoleBinding** grant them to users, groups, or
**ServiceAccounts** (namespaced vs. cluster-wide).

**57. What is a ServiceAccount?**

An identity for **processes in Pods** to authenticate to the API server. Pods get a
default ServiceAccount; you assign specific ones and bind RBAC to control their API access.

**58. What is a ResourceQuota and a LimitRange?**

A **ResourceQuota** caps aggregate resource usage (CPU, memory, object counts) per
namespace. A **LimitRange** sets default/min/max requests and limits per Pod/container in
a namespace.

**59. What are Pod Security Standards / admission?**

Predefined security profiles (Privileged, Baseline, Restricted) enforced by the **Pod
Security Admission** controller to restrict risky Pod settings (privileged, hostPath,
running as root), replacing the deprecated PodSecurityPolicy.

**60. What is a SecurityContext?**

Pod/container-level settings for security: `runAsNonRoot`, `runAsUser`,
`readOnlyRootFilesystem`, dropped capabilities, `allowPrivilegeEscalation: false`, and
seccomp profiles.

---

## Extensibility & operations

**61. What are Custom Resource Definitions (CRDs)?**

They let you extend the Kubernetes API with your own resource types, managed the same
declarative way as built-ins — the foundation of many add-ons.

**62. What is an Operator?**

A pattern combining CRDs with a **custom controller** that encodes operational knowledge
to manage complex/stateful apps (databases, message queues) — automating provisioning,
backups, upgrades, and healing.

**63. What is Helm?**

A package manager for Kubernetes. A **chart** bundles templated manifests with
configurable **values**, enabling versioned, repeatable installs/upgrades/rollbacks of
applications.

**64. What is Kustomize?**

A built-in (`kubectl -k`) tool for template-free customization of manifests via
overlays/patches — a base config with environment-specific overlays, no templating language.

**65. Helm vs. Kustomize?**

Helm uses templating + packaging + release management (good for distributing apps).
Kustomize uses declarative overlays without templating (good for managing environment
variants of your own manifests). They can be combined.

**66. How do you troubleshoot a Pod that won't schedule (`Pending`)?**

`kubectl describe pod` shows scheduling events. Common causes: insufficient
resources/requests, no node matches affinity/selectors/taints, unbound PVC, or image
pull issues.

**67. What common `kubectl` commands do you use to debug?**

`kubectl get pods -o wide`, `describe`, `logs [-f] [--previous]`, `exec -it`,
`get events --sort-by=.lastTimestamp`, `top pod/node` (metrics), and `port-forward`.

**68. How do you perform a zero-downtime deployment?**

Use a Deployment with a rolling update strategy, readiness probes (so traffic only hits
ready Pods), a PDB, and proper `terminationGracePeriodSeconds` + `preStop` for graceful
shutdown.

**69. What is a node drain and cordon?**

`kubectl cordon` marks a node unschedulable; `kubectl drain` safely evicts Pods
(respecting PDBs) to prepare a node for maintenance/upgrade.

**70. How do you back up a cluster?**

Back up **etcd** (snapshots) for cluster state, and use tools like **Velero** to back up
resources and persistent volumes. Store backups off-cluster and test restores.

---

## Quick-fire round

- **Smallest deployable unit?** Pod.
- **Stateless workload controller?** Deployment.
- **One Pod per node?** DaemonSet.
- **Stable identity + storage?** StatefulSet.
- **Cluster state store?** etcd.
- **Internal-only Service?** ClusterIP.
- **HTTP routing at L7?** Ingress.
- **Non-sensitive config?** ConfigMap; sensitive → Secret.
- **Scale replicas by metrics?** HPA.
- **Pod-level firewall?** NetworkPolicy.
- **Package manager?** Helm.
- **Extend the API?** CRD (+ Operator).

---

These questions span the breadth of most Kubernetes interviews — from architecture and
Pods through scheduling, networking, RBAC, and day-2 operations. The fastest way to
internalize them is hands-on: spin up a cluster (kind/minikube), deploy an app with a
Deployment + Service + Ingress, add ConfigMaps/Secrets and probes, then break something
and debug the `CrashLoopBackOff`. Once you've drained a node and rolled back a bad
deploy for real, these answers become second nature.
