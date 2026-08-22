# DevSecOps Interview Questions & Answers

An interview-ready reference for **DevSecOps** — the culture and principles of building
security into the delivery pipeline, the tooling (SAST/DAST/SCA/IaC scanning), secrets
management, container and cloud security, supply-chain integrity, and compliance as code.
Grouped by theme, with answers concise enough to say aloud.

---

## Fundamentals & culture

**1. What is DevSecOps?**

DevSecOps integrates **security** into every stage of the DevOps lifecycle — making it a
shared responsibility across development, security, and operations, automated into the
pipeline — rather than a gate bolted on at the end.

**2. How does DevSecOps differ from traditional security?**

Traditional security is a **late, manual gate** (a security team reviews before release).
DevSecOps **shifts security left**, automating checks continuously throughout development
and delivery, so issues are found early and fixed cheaply.

**3. What does "shift left on security" mean?**

Moving security activities (threat modeling, scanning, reviews) earlier in the SDLC — into
design, coding, and CI — so vulnerabilities are caught when they're cheapest and fastest to
remediate, not in production.

**4. Why is DevSecOps important?**

Software ships faster and more frequently, attack surfaces (cloud, containers, OSS
dependencies) grow, and late-stage security can't keep up. Building security in reduces
risk, cost, and release friction while meeting compliance.

**5. What are the core principles of DevSecOps?**

Security as everyone's responsibility, automation of security testing, shift-left,
continuous monitoring, least privilege, immutable/repeatable infrastructure, fast
feedback to developers, and a blameless, collaborative culture.

**6. What is the "security as code" concept?**

Expressing security policies, controls, and tests as version-controlled code
(policy-as-code, IaC scanning, pipeline security gates) so they're automated, repeatable,
reviewable, and consistently enforced.

**7. What is a "paved road" / "golden path" in DevSecOps?**

A pre-approved, secure-by-default set of tools, templates, and pipelines that make the
secure way the easy way — developers who follow it inherit good security without deep
expertise.

**8. What is the shared responsibility model?**

In cloud, the provider secures the infrastructure ("of the cloud") and the customer
secures what they put in it ("in the cloud") — data, IAM, config, OS/app patches. Knowing
the boundary is essential to avoid gaps.

---

## Application security testing

**9. What is SAST?**

**Static Application Security Testing** analyzes source code, bytecode, or binaries
**without executing** it, to find vulnerabilities (injection, insecure patterns) early —
"white-box." Runs in the IDE/CI. Tools: SonarQube, Semgrep, Checkmarx, CodeQL.

**10. What is DAST?**

**Dynamic Application Security Testing** tests a **running** application from the outside
(black-box), simulating attacks against endpoints to find runtime issues (XSS,
auth flaws). Tools: OWASP ZAP, Burp Suite. Runs later against a deployed/staging app.

**11. SAST vs. DAST — key differences?**

SAST is white-box, early, finds code-level issues, but can produce false positives and
misses runtime/config issues. DAST is black-box, later, finds runtime issues and
environment misconfig, but no code visibility and needs a running app. Use both — they're
complementary.

**12. What is IAST?**

**Interactive Application Security Testing** instruments a running app (agents) to analyze
code behavior during tests, combining SAST's code insight with DAST's runtime context —
fewer false positives, but requires instrumentation.

**13. What is SCA?**

**Software Composition Analysis** identifies **open-source/third-party dependencies** and
their known vulnerabilities (CVEs) and license risks. Critical because most code is now
dependencies. Tools: Snyk, Dependabot, OWASP Dependency-Check, Trivy.

**14. What is RASP?**

**Runtime Application Self-Protection** — security built into the running application/runtime
that detects and blocks attacks in real time from inside the app, complementing perimeter
defenses like WAFs.

**15. Where do these fit in the pipeline?**

Pre-commit/IDE and CI → **SAST**, **SCA**, secret scanning, IaC scanning. Build → image
scanning. Staging/pre-prod → **DAST**, IAST. Production → RASP, runtime/behavioral
monitoring, continuous SCA for new CVEs.

**16. How do you deal with false positives from scanners?**

Tune rules/baselines, triage and suppress verified false positives (with justification),
prioritize by exploitability/severity/reachability, and only **fail the build** on
high-confidence, high-severity findings to avoid alert fatigue.

**17. Should security scans break the build?**

Yes for **high-severity, high-confidence** issues (a quality gate), but graduate the
policy: warn on low severity, block on criticals/secrets. Overly strict gates cause
bypasses; too lax ones let risk through. Balance with risk-based thresholds.

---

## Secrets & credentials

**18. How do you manage secrets in DevSecOps?**

Never hardcode or commit secrets. Use a **secrets manager** (HashiCorp Vault, AWS/GCP/Azure
secret stores), inject at runtime, encrypt at rest and in transit, scope with least
privilege, rotate regularly, and audit access.

**19. What is secret scanning?**

Automated scanning of code, commits, and history for exposed credentials (API keys,
tokens, private keys). Tools: git-secrets, TruffleHog, Gitleaks, GitHub secret scanning.
Best paired with **pre-commit hooks** to stop leaks before they land.

**20. A secret was committed to Git — what do you do?**

**Rotate/revoke it immediately** (assume it's compromised — it's in history). Then purge it
from history (BFG/filter-repo) if needed, add scanning + pre-commit hooks to prevent
recurrence, and investigate for misuse.

**21. What is HashiCorp Vault?**

A tool for secrets management that centrally stores, controls access to, and audits
secrets, and can issue **dynamic, short-lived** credentials (e.g. per-request database
creds), encryption-as-a-service, and automatic rotation.

**22. Static vs. dynamic secrets?**

**Static** secrets are long-lived and stored (must be rotated). **Dynamic** secrets are
generated on demand with a short TTL and automatically revoked — reducing exposure window
and blast radius. Prefer dynamic/short-lived where possible.

**23. What is secret rotation and why does it matter?**

Regularly changing credentials so that a leaked secret is only valid briefly. Automating
rotation limits the damage of exposure and is often a compliance requirement.

---

## Container & Kubernetes security

**24. How do you secure container images?**

Use minimal/trusted base images (distroless/alpine), run as **non-root**, scan for
vulnerabilities in CI, pin versions/digests, don't embed secrets, apply multi-stage builds
to strip build tools, and sign images.

**25. What is container image scanning?**

Analyzing image layers/packages for known CVEs and misconfigurations before deploy. Tools:
Trivy, Grype, Docker Scout, Clair. Integrate into CI and registry, and re-scan for newly
disclosed CVEs.

**26. What is image signing / why?**

Cryptographically signing images (Cosign/Sigstore, Docker Content Trust) so the runtime
verifies authenticity and integrity, ensuring only trusted, untampered images run.

**27. What are Kubernetes security best practices?**

RBAC least privilege, namespaces + NetworkPolicies for isolation, Pod Security Standards
(non-root, no privileged), scan images, manage secrets properly (encrypt etcd at rest),
resource limits, admission controllers/policy engines, and audit logging.

**28. What is a Pod Security admission / standard?**

Kubernetes controls (Privileged/Baseline/Restricted profiles) enforced at admission to
prevent risky Pod settings (privileged containers, host mounts, running as root),
replacing the deprecated PodSecurityPolicy.

**29. What is an admission controller / policy engine?**

A component that validates or mutates resources at creation. Policy engines like **OPA/Gatekeeper** and **Kyverno** enforce custom security policies (e.g. "no `latest`
tags," "must have limits," "no privileged Pods") as code.

**30. How do you secure the container runtime?**

Drop unnecessary Linux capabilities, use read-only root filesystems, seccomp/AppArmor/SELinux
profiles, avoid `--privileged`, isolate with user namespaces, and monitor runtime behavior
(Falco) for anomalies.

**31. What is Falco?**

A runtime security tool that detects anomalous/suspicious behavior in containers and hosts
(unexpected shells, file access, network connections) using kernel event monitoring, and
alerts in real time.

---

## Infrastructure & cloud security

**32. What is IaC security scanning?**

Scanning Terraform/CloudFormation/Kubernetes manifests for misconfigurations (open
security groups, unencrypted storage, public buckets) **before** they're deployed. Tools:
Checkov, tfsec, Terrascan, KICS. Shift-left for infrastructure.

**33. What are common cloud misconfigurations?**

Publicly exposed storage buckets/databases, over-permissive IAM/security groups, unencrypted
data, disabled logging, default credentials, and open management ports (SSH/RDP). These are
a leading cause of breaches.

**34. What is CSPM?**

**Cloud Security Posture Management** continuously monitors cloud environments for
misconfigurations and compliance drift against benchmarks (CIS), alerting and often
auto-remediating. Tools: Prisma Cloud, Wiz, AWS Security Hub.

**35. What is CWPP and CNAPP?**

**CWPP** (Cloud Workload Protection Platform) protects workloads (VMs, containers,
serverless) at runtime. **CNAPP** (Cloud-Native Application Protection Platform) is an
integrated suite combining CSPM, CWPP, SCA, and more across the app lifecycle.

**36. What is the principle of least privilege?**

Granting each identity/service the **minimum** permissions needed for its function, scoped
narrowly and time-bound where possible — limiting the blast radius of a compromise. A core
DevSecOps and IAM principle.

**37. How do you secure IAM in the cloud?**

Least-privilege roles, no long-lived keys (use roles/OIDC/short-lived creds), MFA, no use
of root/owner for daily work, regular access reviews, permission boundaries/SCPs, and
audit logging of all API activity.

**38. What is network segmentation / micro-segmentation?**

Dividing the network into isolated zones (and, at fine grain, per-workload) so a breach in
one area can't move laterally. Implemented with security groups, NetworkPolicies, and
service meshes.

**39. What is a WAF?**

A **Web Application Firewall** filters and blocks malicious HTTP traffic (SQLi, XSS, bots)
at L7, protecting web apps/APIs. It's a layer of defense, not a substitute for secure code.

**40. What is encryption at rest vs. in transit?**

**At rest** — data encrypted where stored (disks, DBs, buckets) via KMS-managed keys. **In
transit** — data encrypted while moving (TLS/mTLS). Both are baseline controls, often
compliance-mandated.

---

## Supply chain security

**41. What is software supply chain security?**

Securing everything that goes into building/delivering software — dependencies, build
systems, artifacts, and pipelines — against tampering and compromise (e.g. SolarWinds,
dependency confusion, malicious packages).

**42. What is an SBOM?**

A **Software Bill of Materials** — a complete inventory of all components/dependencies
(and versions) in an application. It enables rapid impact assessment when a new CVE drops
(e.g. Log4Shell). Formats: SPDX, CycloneDX.

**43. What is SLSA?**

**Supply-chain Levels for Software Artifacts** — a security framework with graduated levels
of build/provenance integrity guarantees, helping ensure artifacts are built from trusted
sources through tamper-resistant pipelines.

**44. What is Sigstore / Cosign?**

Open-source tooling for **signing and verifying** software artifacts and container images
(keyless signing with short-lived certs), providing provenance and integrity across the
supply chain.

**45. What is a dependency confusion attack?**

An attacker publishes a malicious public package with the same name as an internal
private one; misconfigured resolvers pull the public (malicious) version. Mitigate with
scoped/namespaced packages, private registries, and pinned sources.

**46. How do you secure the CI/CD pipeline itself?**

Least-privilege pipeline credentials (OIDC/short-lived), isolated/ephemeral runners, pin
and verify actions/dependencies (by SHA), protect secrets, sign artifacts, restrict who can
modify pipelines, and audit pipeline activity. The pipeline is a high-value target.

**47. Why pin dependencies and actions to hashes?**

Tags/versions can be moved to point at malicious code. Pinning to a **cryptographic hash/digest**
ensures you always run the exact reviewed artifact, defeating tampering and typo/hijack attacks.

---

## Compliance, threat modeling & operations

**48. What is compliance as code?**

Encoding compliance/security requirements as automated, testable policies (OPA, InSpec,
Checkov) that run in the pipeline, giving continuous, auditable enforcement instead of
periodic manual audits.

**49. What is policy as code?**

Defining and enforcing rules (who/what is allowed) as version-controlled code evaluated
automatically — e.g. OPA/Rego, Kyverno — for consistent, reviewable governance across infra
and pipelines.

**50. What is threat modeling?**

A structured process to identify, enumerate, and prioritize potential threats to a system
early in design, so you can design mitigations. **STRIDE** is a common framework.

**51. What is STRIDE?**

A threat classification: **Spoofing, Tampering, Repudiation, Information disclosure,
Denial of service, Elevation of privilege** — used to reason systematically about how a
system can be attacked.

**52. What is the OWASP Top 10?**

A widely referenced list of the most critical web application security risks (e.g. Broken
Access Control, Injection, Cryptographic Failures, SSRF, Security Misconfiguration) — a
baseline checklist for app security.

**53. What is CVSS?**

**Common Vulnerability Scoring System** — a standardized 0–10 severity score for
vulnerabilities based on exploitability and impact, used to prioritize remediation
(often combined with reachability/exploit context).

**54. What is vulnerability management?**

The continuous process of identifying, classifying, prioritizing, remediating, and
verifying vulnerabilities across code, dependencies, images, and infrastructure — with
SLAs by severity and tracking to closure.

**55. How do you prioritize which vulnerabilities to fix first?**

By risk, not just CVSS: consider severity, **exploitability** (is there a known exploit?),
**reachability** (is the code path used/exposed?), asset sensitivity, and business impact.
Fix internet-facing, exploitable, high-impact issues first.

**56. What is continuous security monitoring?**

Ongoing collection and analysis of security telemetry (logs, events, runtime behavior,
config drift) to detect and respond to threats in real time — feeding a SIEM/SOAR and
tying into incident response.

**57. What is a SIEM and SOAR?**

**SIEM** (Security Information and Event Management) aggregates and correlates security
logs/events for detection and analysis. **SOAR** (Security Orchestration, Automation, and
Response) automates response playbooks to speed and standardize remediation.

**58. What is zero trust?**

A model of "never trust, always verify" — no implicit trust based on network location;
every request is authenticated, authorized, and encrypted, with least privilege and
continuous verification. Assumes breach.

**59. What is DevSecOps' role in incident response?**

Automate detection and response, maintain runbooks, ensure logging/traceability, enable
fast rollback/patching through the pipeline, and feed lessons from blameless post-mortems
back into controls and tests.

**60. How do you measure a DevSecOps program's success?**

Metrics like mean time to remediate vulnerabilities (MTTR), vulnerability escape rate to
production, percentage of pipelines with security gates, secret-leak incidents, scan
coverage, and reduction in high-severity findings over time.

---

## Quick-fire round

- **Analyze code without running it?** SAST.
- **Attack a running app?** DAST.
- **Find vulnerable dependencies?** SCA.
- **Inventory of all components?** SBOM.
- **Detect leaked credentials?** Secret scanning.
- **Scan Terraform for misconfig?** IaC scanning (Checkov/tfsec).
- **Never trust, always verify?** Zero trust.
- **Minimum permissions?** Least privilege.
- **Threat framework?** STRIDE (+ OWASP Top 10).
- **Sign container images?** Cosign/Sigstore.
- **Policy enforced as code in K8s?** OPA/Gatekeeper or Kyverno.
- **Runtime container threat detection?** Falco.

---

These questions cover the arc most DevSecOps interviews follow — from shift-left culture
through SAST/DAST/SCA, secrets management, container and cloud security, and supply-chain
integrity. To make them concrete, wire security into a real pipeline yourself: add secret
scanning and a pre-commit hook, run SCA and IaC scanning as build gates, scan and sign a
container image, and enforce a Kyverno/OPA policy on a cluster. Once you've triaged a real
CVE and rotated a leaked secret, these answers become practice rather than theory.
