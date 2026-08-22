# MLOps Interview Questions & Answers

An interview-ready reference for **MLOps** — the practices and tooling for taking machine
learning models from notebook to reliable production and keeping them healthy. Covers the ML
lifecycle, data and feature management, experiment tracking, CI/CD for ML, deployment,
monitoring and drift, and governance. Grouped by theme, with answers concise enough to say
aloud.

---

## Fundamentals

**1. What is MLOps?**

MLOps (Machine Learning Operations) is the set of practices that apply DevOps principles to
the ML lifecycle — automating and standardizing the development, deployment, monitoring, and
governance of ML models in production so they're reliable, reproducible, and maintainable.

**2. How does MLOps differ from DevOps?**

DevOps manages **code**; MLOps manages **code + data + models**. ML adds challenges DevOps
doesn't: data/model versioning, experiment tracking, training pipelines, model decay over
time (drift), continuous retraining, and testing on statistical performance, not just pass/fail.

**3. Why do ML projects need MLOps?**

Because most models never make it to production or silently degrade there. MLOps addresses
reproducibility, deployment friction, monitoring for drift/decay, scaling, collaboration
between data scientists and engineers, and governance/compliance — turning experiments into
dependable services.

**4. What are the three axes of change/versioning in ML?**

**Code**, **data**, and **model** (and often config/hyperparameters). Reproducing a result
requires versioning all of them together — unlike traditional software, where code is usually
enough.

**5. What is the ML lifecycle?**

Problem framing → data collection/labeling → data preparation/feature engineering → model
training/experimentation → evaluation/validation → deployment → monitoring → (retraining) —
an iterative loop, not a one-way pipeline.

**6. What are the levels of MLOps maturity?**

Roughly: **Level 0** — manual, notebook-driven, manual deploys. **Level 1** — automated
training pipeline with continuous training. **Level 2** — full CI/CD automation of pipelines,
testing, and deployment. Maturity increases automation and reduces manual handoffs.

**7. What is reproducibility and why is it hard in ML?**

The ability to recreate the same model/results from the same inputs. It's hard because of
data changes, randomness (seeds), non-deterministic hardware/libraries, environment drift,
and untracked hyperparameters. MLOps enforces it via versioning, seeds, pinned environments,
and tracked pipelines.

---

## Data & features

**8. What is data versioning and what tools do it?**

Tracking versions of datasets so training runs are reproducible and changes are auditable.
Tools: **DVC**, LakeFS, Delta Lake, Git LFS, and dataset registries. It ties a specific data
snapshot to a specific model.

**9. Why can't you just use Git for data?**

Git is built for text/code, not large binary datasets — it bloats, is slow, and doesn't
handle multi-GB files well. Tools like DVC store data in object storage and keep lightweight
pointers in Git, giving Git-like versioning for data.

**10. What is a feature store?**

A centralized system to define, compute, store, and serve **features** for both training and
inference — ensuring consistency, reuse across teams, and low-latency online serving.
Examples: Feast, Tecton, Vertex AI Feature Store.

**11. What is training/serving skew?**

When the features (or their computation) at **training** time differ from those at **serving**
time — different code paths, data sources, or transformations — causing production performance
to diverge from offline evaluation. A feature store and shared transformation code prevent it.

**12. What is data validation and why is it part of the pipeline?**

Automated checks on incoming data for schema, ranges, distributions, missing values, and
anomalies (e.g. TFDV, Great Expectations, Pandera) — catching bad data **before** it corrupts
training or predictions. "Garbage in, garbage out" is the top cause of model failure.

**13. What is data labeling and its MLOps concerns?**

Annotating data for supervised learning. MLOps concerns: label quality/consistency,
inter-annotator agreement, labeling pipelines, active learning to prioritize samples, and
versioning labels alongside data.

**14. What is a data pipeline vs. an ML pipeline?**

A **data pipeline** ingests, cleans, and transforms data (ETL/ELT). An **ML pipeline** adds
training-specific steps: feature engineering, training, evaluation, and registration. ML
pipelines often consume the outputs of data pipelines.

---

## Experimentation & training

**15. What is experiment tracking?**

Recording each training run's parameters, code version, data version, metrics, and artifacts
so you can compare, reproduce, and choose the best model. Tools: **MLflow**, Weights & Biases,
Neptune, Comet. Without it, results are unreproducible and comparisons are guesswork.

**16. What do you log in an experiment?**

Hyperparameters, dataset/version, code/git commit, environment, metrics (per epoch and final),
evaluation results, artifacts (model files, plots), and metadata (who/when/hardware) — enough
to reproduce and audit the run.

**17. What is a model registry?**

A central store for trained models with versioning, metadata, lineage, stage transitions
(staging → production → archived), and approvals. It's the handoff point between training and
deployment. MLflow Model Registry, SageMaker/Vertex registries are examples.

**18. What is hyperparameter tuning and how is it automated?**

Searching for the best hyperparameters via grid search, random search, or Bayesian
optimization (Optuna, Ray Tune, Katib). MLOps integrates it into pipelines and tracks all
trials as experiments.

**19. What is model lineage/provenance?**

The recorded chain linking a model to the exact data, code, features, and parameters that
produced it. Essential for reproducibility, debugging, auditing, and compliance ("why did the
model make this decision, and what produced it?").

**20. How do you ensure training pipelines are reproducible?**

Version code, data, and config; fix random seeds; pin dependencies/containers; log everything
via experiment tracking; and run in orchestrated, parameterized pipelines rather than ad-hoc
notebooks.

---

## Pipelines, CI/CD & orchestration

**21. What is a training/ML pipeline?**

An automated, orchestrated sequence of steps (data ingestion → validation → preprocessing →
training → evaluation → registration) that's parameterized, versioned, and repeatable —
replacing manual notebook runs. Tools: Kubeflow Pipelines, Vertex AI Pipelines, TFX, Airflow,
Metaflow, ZenML.

**22. What is CI/CD in the ML context?**

**CI** tests code *and* validates data/models (unit tests, data checks, model quality gates).
**CD** deploys the pipeline and/or the model. ML adds **CT (Continuous Training)** —
automatically retraining and redeploying models as new data arrives or performance drops.

**23. What is Continuous Training (CT)?**

The MLOps-specific practice of automatically retraining models on fresh data — triggered by a
schedule, new data, or detected drift/decay — then validating and (if better) promoting the
new model. It keeps models current without manual intervention.

**24. What triggers a retraining pipeline?**

Scheduled cadence, arrival of significant new data, detected **data/concept drift**, a drop in
monitored performance below a threshold, or manual/business events. Guard promotions with
evaluation gates so a worse model never replaces a good one.

**25. How is testing different for ML systems?**

Beyond code unit/integration tests, you test **data** (schema, distributions), **model
quality** (metrics thresholds, on slices/subgroups), **behavioral** tests (invariance,
directional expectations), and non-regression against a baseline. Passing "runs without error"
is not enough — statistical quality must be validated.

**26. What is orchestration and why is it needed?**

Coordinating the ordered, dependent steps of pipelines (with retries, scheduling, parallelism,
and monitoring) via tools like Airflow, Kubeflow, Prefect, or Dagster. Manual step-running
doesn't scale or reproduce reliably.

**27. Why package models/pipelines in containers?**

Containers pin the exact environment (libraries, versions, system deps), eliminating
"works on my machine" and ensuring training and serving run identically across dev, CI, and
prod. Docker + orchestration (Kubernetes) is the common stack.

---

## Deployment & serving

**28. What are the main model deployment patterns?**

- **Batch/offline** — predictions computed on a schedule and stored (e.g. nightly scoring).
- **Online/real-time** — a low-latency API serving predictions per request.
- **Streaming** — predictions on event streams.
- **Edge/embedded** — model runs on-device.

Choose by latency needs, throughput, and freshness requirements.

**29. How do you serve a model as an API?**

Wrap it behind a REST/gRPC endpoint using a serving framework (TensorFlow Serving,
TorchServe, KServe, BentoML, Seldon, NVIDIA Triton, or FastAPI), containerize it, and deploy
with autoscaling and load balancing.

**30. What deployment/release strategies apply to models?**

- **Shadow** — the new model runs alongside prod, receiving traffic but not serving responses, to compare safely.
- **Canary** — route a small % of traffic to the new model first.
- **Blue-green** — instant switch with easy rollback.
- **A/B testing** — compare models on business metrics with live traffic.

**31. What is shadow deployment?**

Running a candidate model in parallel with the production model on real traffic, logging its
predictions **without** using them for decisions — validating real-world behavior and
performance before promoting it. Zero user risk.

**32. What is a champion/challenger setup?**

The current production model (**champion**) runs against one or more candidate models
(**challengers**) on live or shadow traffic; a challenger that outperforms is promoted.
Formalizes continuous model improvement.

**33. How do you scale model inference?**

Horizontal autoscaling of stateless serving replicas behind a load balancer, batching
requests, GPU/accelerator use for heavy models, model **optimization** (quantization,
distillation, pruning, ONNX/TensorRT), and caching. Match infra to latency/throughput SLOs.

**34. What are model optimization techniques for serving?**

**Quantization** (lower precision), **pruning** (remove weights), **distillation** (train a
smaller student model), **compilation** (ONNX Runtime, TensorRT), and batching — reducing
latency, memory, and cost, ideally with minimal accuracy loss.

**35. How do you roll back a bad model?**

Keep the previous version in the registry and serving infra; switch traffic back (blue-green)
or redeploy the prior artifact. Immutable, versioned models plus automated promotion/rollback
make this fast and safe — which is why evaluation gates and canaries matter.

---

## Monitoring & drift

**36. What do you monitor for a production ML model?**

**Operational** metrics (latency, throughput, errors, resource use — like any service) **and**
**ML-specific** ones: input data quality/distribution, prediction distribution, model
performance (when labels arrive), and **drift**. ML systems fail silently, so monitoring is
critical.

**37. What is data drift?**

A change in the **distribution of input features** over time versus training data (e.g. user
behavior shifts). The model still runs but sees inputs it wasn't trained for, degrading
accuracy. Detected via distribution tests (PSI, KL divergence, KS test).

**38. What is concept drift?**

A change in the **relationship between inputs and the target** — the underlying pattern the
model learned no longer holds (e.g. fraud tactics evolve). The mapping X→y shifts, so even
unchanged inputs need different predictions. Usually requires retraining.

**39. Data drift vs. concept drift — the key distinction?**

**Data drift** = inputs change distribution (P(X) shifts). **Concept drift** = the input→output
relationship changes (P(y|X) shifts). Data drift may or may not hurt accuracy; concept drift
directly degrades it and demands retraining.

**40. How do you detect drift?**

Compare production feature/prediction distributions to a training/reference baseline using
statistical tests (**PSI**, KS test, KL/JS divergence, Chi-square) and monitor performance
metrics where ground truth is available. Tools: Evidently, WhyLabs, NannyML, Arize,
Fiddler, cloud model monitors.

**41. How do you monitor model performance when labels are delayed?**

Use **proxy metrics** (prediction distribution shifts, confidence, drift signals) as early
warnings, and compute true performance retrospectively once labels arrive. NannyML-style
methods estimate performance without immediate labels.

**42. What is model decay/staleness?**

The gradual degradation of a model's real-world performance over time due to data/concept
drift and a changing world. It's why monitoring and continuous/triggered retraining are
core MLOps practices — a deployed model is not "done."

**43. What alerts should an ML monitoring system raise?**

Significant drift beyond thresholds, performance dropping below SLO, data quality failures
(schema, nulls, out-of-range), prediction distribution anomalies, and operational issues
(latency/errors). Alerts should be actionable and tied to a runbook (investigate → retrain →
roll back).

---

## Governance, reliability & cost

**44. What is model governance?**

The policies and controls ensuring models are documented, approved, compliant, auditable, and
fair — covering lineage, access control, approval workflows, model cards, and audit trails.
Increasingly required by regulation (e.g. financial, healthcare, EU AI Act).

**45. What is a model card?**

A standardized document describing a model's intended use, training data, performance
(including across subgroups), limitations, and ethical considerations — supporting
transparency, governance, and responsible use.

**46. How do you address bias/fairness in MLOps?**

Evaluate performance across demographic **slices**, monitor for disparate impact in production,
use fairness metrics and mitigation techniques, document limitations (model card), and include
fairness checks as pipeline gates. It's an ongoing monitoring concern, not one-time.

**47. What is explainability and where does it fit?**

Making model decisions interpretable (SHAP, LIME, feature importance) — for debugging,
trust, compliance, and stakeholder communication. Often surfaced in monitoring dashboards and
required for regulated decisions.

**48. How do you manage costs in MLOps?**

Right-size training/serving compute, use spot/preemptible instances for training, autoscale
serving (scale to zero for low traffic), optimize models (quantization/distillation), batch
where possible, cache, and monitor cost per model/endpoint. Training and GPU serving are the
big line items.

**49. What are common reasons ML projects fail in production?**

Training/serving skew, unmonitored drift/decay, poor data quality, non-reproducible
experiments, manual/fragile deployment, no rollback path, and misalignment with business
metrics. MLOps exists to address each of these systematically.

**50. What is the difference between offline and online evaluation?**

**Offline** evaluation measures model quality on held-out/historical data before deployment
(accuracy, F1, AUC). **Online** evaluation measures real-world impact on live traffic
(A/B tests, business KPIs). A model can look great offline and underperform online — you need
both.

---

## Quick-fire round

- **DevOps for ML?** MLOps (code + data + model).
- **Version datasets?** DVC / LakeFS / Delta.
- **Track runs, params, metrics?** Experiment tracking (MLflow, W&B).
- **Store/promote trained models?** Model registry.
- **Consistent features train vs. serve?** Feature store (prevents training/serving skew).
- **Auto-retrain on new data?** Continuous Training (CT).
- **Inputs change distribution?** Data drift; input→output changes → concept drift.
- **New model on live traffic, no user impact?** Shadow deployment.
- **Small % traffic first?** Canary.
- **Doc of a model's use/limits?** Model card.
- **Detect drift?** PSI / KS test / KL divergence.

---

These questions cover the breadth of most MLOps interviews — from reproducibility and
feature stores through CI/CD/CT, deployment strategies, and drift monitoring. The strongest
prep is to ship a model end to end yourself: version the data with DVC, track runs in MLflow,
build a training pipeline, register and serve the model behind an API, and wire up drift
monitoring with Evidently. Once you've watched a model decay in production and triggered a
retrain, these answers stop being definitions and become experience.
