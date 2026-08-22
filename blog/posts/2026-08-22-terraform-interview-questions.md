# Terraform Interview Questions & Answers

An interview-ready reference for **Terraform** — the core language and workflow, state
management, modules, provisioners, and the operational practices that come up in real
infrastructure work. Questions are grouped by theme; answers are concise but complete.

---

## Fundamentals

**1. What is Terraform?**

Terraform is an open-source **Infrastructure as Code (IaC)** tool by HashiCorp that lets
you define, provision, and manage infrastructure declaratively using a configuration
language (HCL). It works across many providers (AWS, Azure, GCP, Kubernetes, etc.).

**2. What is Infrastructure as Code?**

Managing and provisioning infrastructure through machine-readable definition files
rather than manual processes — giving you version control, repeatability, review, and
automation for infrastructure just like application code.

**3. What language does Terraform use?**

**HCL (HashiCorp Configuration Language)** — a declarative, human-readable language.
Terraform can also read JSON. Declarative means you describe the *desired end state*,
not the steps to get there.

**4. Declarative vs. imperative — where does Terraform sit?**

Terraform is **declarative**: you specify what you want and Terraform figures out how to
achieve it. Tools like Ansible procedural playbooks lean more imperative (though Ansible
is also largely declarative).

**5. How does Terraform differ from Ansible/Chef/Puppet?**

Terraform focuses on **provisioning** infrastructure (creating VMs, networks, managed
services) and tracks state. Ansible/Chef/Puppet focus on **configuration management**
(installing software, configuring existing servers). They're complementary — often
Terraform provisions and Ansible configures.

**6. What are Terraform providers?**

Plugins that let Terraform interact with an API (AWS, Azure, GCP, GitHub, Kubernetes,
etc.). Each provider exposes resources and data sources. You declare providers in the
configuration and Terraform downloads them during `init`.

**7. What is a resource?**

The most important element — a piece of infrastructure (a VM, a bucket, a DNS record):

```hcl
resource "aws_instance" "web" {
  ami           = "ami-123456"
  instance_type = "t3.micro"
}
```

`aws_instance` is the type, `web` is the local name; together `aws_instance.web` is the
resource address.

**8. What is a data source?**

A read-only lookup of existing information you didn't create in this config:

```hcl
data "aws_ami" "ubuntu" {
  most_recent = true
  owners      = ["099720109477"]
}
```

Referenced as `data.aws_ami.ubuntu.id`.

---

## Core workflow

**9. What is the core Terraform workflow?**

**Write → Plan → Apply.** Concretely: `terraform init` (initialize), `terraform plan`
(preview changes), `terraform apply` (make changes), and `terraform destroy` (tear down).

**10. What does `terraform init` do?**

Initializes the working directory: downloads provider plugins and modules, configures
the backend, and creates the `.terraform` directory and dependency lock file. It's the
first command you run.

**11. What does `terraform plan` do?**

Creates an execution plan by comparing the desired configuration against current state
and real infrastructure, showing what will be **created, updated, or destroyed** —
without making any changes. You can save it with `-out`.

**12. What does `terraform apply` do?**

Executes the actions proposed in a plan to reach the desired state. Without a saved
plan it generates one and prompts for confirmation. `-auto-approve` skips the prompt.

**13. What does `terraform destroy` do?**

Removes all resources managed by the configuration/state. `terraform plan -destroy`
previews it. You can target specific resources with `-target`.

**14. What is `terraform validate`?**

Checks configuration for syntactic validity and internal consistency (correct argument
names, types) without contacting providers or touching state.

**15. What is `terraform fmt`?**

Rewrites configuration files to the canonical style/formatting. Commonly enforced in CI.

**16. What is `terraform refresh` (and its modern equivalent)?**

It updates the state file to match real infrastructure. `terraform refresh` is
deprecated in favor of `terraform apply -refresh-only`, which reconciles drift
explicitly and safely.

---

## State

**17. What is Terraform state?**

A file (`terraform.tfstate`) that maps your configuration to real-world resources,
stores metadata and resource attributes, and tracks dependencies. It's how Terraform
knows what it manages and detects drift.

**18. Why is state important?**

It's the source of truth linking config to real resources, enables performance (caching
attributes), tracks metadata/dependencies, and allows collaboration. Without it,
Terraform couldn't map `aws_instance.web` to a specific instance ID.

**19. What is remote state and why use it?**

Storing state in a shared backend (S3, GCS, Azure Blob, Terraform Cloud) instead of
locally. Benefits: team collaboration, state **locking**, encryption, versioning, and
keeping secrets out of local disks and Git.

**20. What is state locking?**

A mechanism that prevents concurrent operations from corrupting state by locking it
during writes (e.g. S3 + DynamoDB, or native locking in GCS/Terraform Cloud). If a lock
is held, others must wait.

**21. What is a backend?**

The configuration of *where* and *how* state is stored and operations run. Examples:
`local`, `s3`, `gcs`, `azurerm`, `remote`/`cloud` (Terraform Cloud).

```hcl
terraform {
  backend "s3" {
    bucket = "my-tfstate"
    key    = "prod/network.tfstate"
    region = "us-east-1"
  }
}
```

**22. Should you commit state to Git?**

No. State can contain **secrets** in plain text and causes conflicts. Use a remote
backend and add `*.tfstate*` to `.gitignore`.

**23. What is `terraform state` used for?**

Advanced state management: `state list`, `state show`, `state mv` (rename/move a
resource), `state rm` (stop managing without destroying), and `state pull/push`.

**24. What is drift and how do you detect it?**

Drift is when real infrastructure diverges from state (someone changed it manually).
Detect with `terraform plan` or `apply -refresh-only`, which shows the differences so
you can reconcile.

**25. How do you import existing infrastructure?**

`terraform import <resource_address> <id>` brings an existing resource under Terraform
management by writing it into state (you still write the matching config). Newer
Terraform supports the declarative `import` block and `-generate-config-out`.

**26. How do you move a resource without recreating it?**

`terraform state mv`, or the declarative `moved` block in config — useful when
refactoring modules/renaming resources so Terraform doesn't destroy and recreate.

---

## Variables, outputs & data

**27. What are input variables?**

Parameters that make configurations reusable:

```hcl
variable "instance_type" {
  type    = string
  default = "t3.micro"
}
```

Set via defaults, `-var`, `-var-file`, `terraform.tfvars`, `*.auto.tfvars`, or
`TF_VAR_` environment variables.

**28. What is the variable precedence order?**

From lowest to highest: environment variables (`TF_VAR_`) → `terraform.tfvars` →
`*.auto.tfvars` (alphabetical) → `-var` / `-var-file` on the command line (last wins).

**29. What are output values?**

Values a configuration exposes after apply (e.g. an IP address), shown on the CLI and
consumable by other configs via remote state or by parent modules:

```hcl
output "public_ip" {
  value = aws_instance.web.public_ip
}
```

**30. What are local values (`locals`)?**

Named expressions computed once and reused, to avoid repetition:

```hcl
locals {
  name_prefix = "${var.env}-app"
}
```

**31. What variable types does Terraform support?**

Primitives (`string`, `number`, `bool`) and complex types (`list`, `set`, `map`,
`object`, `tuple`). You can also mark variables `sensitive = true` to hide values in output.

**32. How do you handle sensitive variables?**

Mark them `sensitive = true` (redacts from plan/apply output), supply them via env vars
or a secrets manager, and never commit them. Note: sensitive values are still stored in
state, so protect the backend.

---

## Modules

**33. What is a module?**

A container for multiple resources used together — any directory with `.tf` files is a
module. Modules enable reuse, encapsulation, and organization. The top-level directory
is the **root module**.

**34. How do you call a module?**

```hcl
module "network" {
  source  = "./modules/network"
  cidr    = "10.0.0.0/16"
}
```

You pass input variables via arguments and read the module's outputs as
`module.network.<output>`.

**35. Where can module sources come from?**

Local paths, the Terraform Registry, Git repositories, HTTP URLs, and S3/GCS buckets.
You can pin versions with the `version` argument for registry modules.

**36. Why version-pin modules and providers?**

To ensure reproducible builds and avoid unexpected breaking changes. Providers are
pinned via `required_providers` and the **dependency lock file** (`.terraform.lock.hcl`);
modules via the `version` constraint.

**37. What makes a good module?**

Clear inputs/outputs, sensible defaults, single responsibility, no hardcoded values,
documentation, and no embedded provider/backend config (let the caller supply those).

---

## Meta-arguments & expressions

**38. What is `count`?**

A meta-argument to create multiple instances of a resource by index:

```hcl
resource "aws_instance" "web" {
  count = 3
}
```

Referenced as `aws_instance.web[0]`. Downside: reindexing on removal can force
recreation of others.

**39. What is `for_each` and when is it better than `count`?**

`for_each` creates instances keyed by a map/set instead of a numeric index:

```hcl
resource "aws_instance" "web" {
  for_each      = toset(["a", "b", "c"])
  instance_type = "t3.micro"
}
```

Prefer it when items have stable identities — removing one doesn't reindex the others.

**40. What is `depends_on`?**

Explicitly declares a dependency Terraform can't infer, forcing ordering when there's no
direct reference between resources.

**41. What is the `lifecycle` block?**

Controls resource lifecycle behavior:

- `create_before_destroy` — create the replacement before destroying the old one (zero downtime).
- `prevent_destroy` — block accidental destruction.
- `ignore_changes` — ignore drift on specified attributes.
- `replace_triggered_by` — force replacement when a referenced value changes.

**42. What are dynamic blocks?**

Programmatically generate repeatable nested blocks (like multiple `ingress` rules) from
a collection:

```hcl
dynamic "ingress" {
  for_each = var.ports
  content {
    from_port = ingress.value
    to_port   = ingress.value
    protocol  = "tcp"
  }
}
```

**43. What are `for` expressions?**

Transform collections inline:

```hcl
[for s in var.names : upper(s)]
{ for k, v in var.map : k => v * 2 }
```

**44. What is the ternary/conditional expression?**

`condition ? true_val : false_val`, e.g. `var.env == "prod" ? "m5.large" : "t3.micro"`.

**45. Name some commonly used built-in functions.**

`lookup`, `merge`, `concat`, `join`, `split`, `element`, `length`, `file`, `templatefile`,
`jsonencode`/`jsondecode`, `cidrsubnet`, `coalesce`, `try`, `flatten`, `toset`, `format`.

---

## Provisioners & workspaces

**46. What are provisioners and when should you use them?**

Provisioners (`local-exec`, `remote-exec`, `file`) run scripts on a local or remote
machine as part of create/destroy. HashiCorp recommends them as a **last resort** —
prefer cloud-init, images, or configuration management tools instead, because
provisioners aren't tracked in state.

**47. What are `local-exec` and `remote-exec`?**

`local-exec` runs a command on the machine running Terraform; `remote-exec` runs
commands on the created remote resource (via SSH/WinRM).

**48. What is a `null_resource`?**

A resource with no real infrastructure, used to run provisioners or group logic,
triggered by the `triggers` argument. (Often replaced now by `terraform_data`.)

**49. What are Terraform workspaces?**

Named instances of state within one backend/config, letting you manage multiple
environments (e.g. `dev`, `prod`) with the same code. `terraform workspace new/select`.
Caveat: for strong isolation, many teams prefer separate directories/backends over CLI
workspaces.

**50. Workspaces vs. separate state files/directories?**

Workspaces are lightweight and share the same backend and config — good for small
variations. Separate directories/backends give stronger isolation (different
permissions, blast radius) — preferred for prod vs. non-prod.

---

## Real-world & operations

**51. How do you structure a large Terraform codebase?**

Split by layer/component (network, data, app), use modules for reuse, separate state per
environment, keep a consistent directory layout, use remote state with locking, and pull
shared values via `terraform_remote_state` or a data source.

**52. How do you manage multiple environments?**

Common patterns: directory-per-environment with shared modules, `*.tfvars` per env,
workspaces, or Terraform Cloud workspaces. Keep environment differences in variables,
not duplicated code.

**53. How do you handle secrets in Terraform?**

Never hardcode. Use a secrets manager (Vault, AWS Secrets Manager) via data sources,
env vars, or CI secret injection. Mark variables sensitive, and secure the state backend
because state may hold secrets.

**54. How do you run Terraform in CI/CD?**

`init` → `fmt -check` → `validate` → `plan` (post as artifact/comment) → manual approval
→ `apply`. Use remote state with locking, a machine identity (OIDC/service account), and
store no credentials in code. Tools: Atlantis, Terraform Cloud, GitHub Actions.

**55. What is `terraform_remote_state`?**

A data source that reads outputs from another configuration's remote state, letting one
stack consume another's values (e.g. app stack reading the network stack's VPC ID).

**56. How do you do a zero-downtime resource replacement?**

Use `lifecycle { create_before_destroy = true }` so the new resource is provisioned and
wired in before the old one is destroyed.

**57. How do you target a specific resource?**

`terraform apply -target=aws_instance.web` limits operations to that resource and its
dependencies. Use sparingly — it can mask drift and is meant for exceptional recovery.

**58. What is `-replace` (taint)?**

`terraform apply -replace=<address>` forces recreation of a specific resource. It
replaces the older `terraform taint` command.

**59. How do you upgrade a provider version safely?**

Update the constraint in `required_providers`, run `terraform init -upgrade`, review the
provider changelog, run `plan` to see impacts, and commit the updated
`.terraform.lock.hcl`.

**60. What is the dependency lock file?**

`.terraform.lock.hcl` records the exact provider versions and hashes selected, ensuring
everyone and CI use identical providers. Commit it to version control.

---

## Advanced & gotchas

**61. How does Terraform build its dependency graph?**

It parses references between resources to build a **directed acyclic graph (DAG)**, then
creates/updates resources in dependency order and in parallel where independent. View it
with `terraform graph`.

**62. What is the difference between `plan` and `apply` refresh behavior?**

By default both refresh state against real infrastructure before acting. You can disable
with `-refresh=false`, or refresh only with `apply -refresh-only`.

**63. Why might `terraform plan` show changes on every run?**

Common causes: provider normalizes a value differently, a computed/default attribute,
non-deterministic input, or an attribute that should be in `ignore_changes`. Diagnose by
inspecting the specific attribute in the plan diff.

**64. What happens if the state file is lost?**

Terraform loses track of managed resources and would try to recreate them. Recovery
options: restore from backend versioning/backup, or `terraform import` each resource
back. This is why versioned remote backends matter.

**65. How do you refactor without destroying resources?**

Use `moved` blocks (declarative) or `terraform state mv` to tell Terraform a resource's
address changed, so it updates state instead of destroy/recreate.

**66. What is `terraform console`?**

An interactive REPL to evaluate expressions and inspect state/values — great for testing
functions and debugging interpolations.

**67. What are the benefits and risks of `-auto-approve`?**

It skips the interactive confirmation (needed in automation) but removes the human review
gate — safe only when a reviewed, saved plan is applied or in tightly controlled pipelines.

**68. Terraform vs. CloudFormation?**

Terraform is cloud-agnostic (multi-provider), uses HCL, and manages state externally.
CloudFormation is AWS-only, uses JSON/YAML, and AWS manages state for you. Terraform's
provider ecosystem and modules are a common reason teams choose it.

---

## Quick-fire round

- **Config file extension?** `.tf` (and `.tf.json`).
- **Preview changes?** `terraform plan`.
- **Where are providers downloaded?** `.terraform/` after `init`.
- **Read-only lookup?** A `data` source.
- **Stop managing without destroying?** `terraform state rm`.
- **Prevent accidental deletion?** `lifecycle { prevent_destroy = true }`.
- **Multiple instances by key?** `for_each`.
- **Sensitive output?** Mark `sensitive = true`.
- **Registry for modules/providers?** The Terraform Registry.

---

These questions cover the arc most Terraform interviews follow — from "what is state"
through modules, `for_each` vs `count`, and safe refactoring. The fastest way to make
them stick: build a small two-stack project (a network module and an app module) with
remote state and a CI plan/apply pipeline. Once you've hit a state lock and resolved a
drift diff for real, the answers stop being trivia.
