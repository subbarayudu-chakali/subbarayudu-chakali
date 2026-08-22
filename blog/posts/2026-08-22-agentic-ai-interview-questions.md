# Agentic AI Interview Questions & Answers

An interview-ready reference for **agentic AI** — LLM-powered agents that reason, use
tools, plan, and act toward goals. Covers the core anatomy, reasoning and planning
patterns, tools and memory, multi-agent systems, frameworks, evaluation, and the safety
and reliability concerns that dominate real deployments. Grouped by theme.

---

## Fundamentals

**1. What is an AI agent?**

An AI agent is a system that uses an LLM as a reasoning engine to **autonomously decide and
take actions** toward a goal — perceiving context, planning, invoking tools, observing
results, and iterating in a loop — rather than producing a single one-shot response.

**2. What is "agentic AI" versus a plain LLM call?**

A plain LLM call maps input → output once. An **agentic** system adds a control loop:
the model chooses actions (tool calls), receives observations, and repeats until the goal
is met. Agency = autonomy, tool use, and iterative decision-making over multiple steps.

**3. What are the core components of an agent?**

- **Model (LLM)** — the reasoning/decision core.
- **Instructions/goal** — the objective and constraints (often a system prompt).
- **Tools** — functions/APIs the agent can call to act or fetch information.
- **Memory** — short-term (context) and long-term (persisted) state.
- **Orchestration/loop** — the control flow that runs reason→act→observe until done.

**4. What is the agent loop?**

The cycle: the agent **reasons** about the goal and current state, **acts** (calls a tool or
responds), **observes** the result, and repeats — until a stopping condition (task complete,
max steps, or a final answer). ReAct is the canonical formulation.

**5. What distinguishes an agent from a workflow?**

A **workflow** follows predefined, deterministic steps (the developer sets the path). An
**agent** dynamically decides its own steps at runtime based on the model's judgment. Many
production systems are hybrids — deterministic scaffolding with agentic steps where
flexibility is needed.

**6. When should you NOT use an agent?**

When the task is well-defined and deterministic, latency/cost must be minimal, or
correctness is critical and the added autonomy increases risk. A fixed workflow or a single
prompt is often cheaper, faster, and more reliable. Use agents when flexibility and
open-ended tool use genuinely add value.

**7. What are levels of agent autonomy?**

From least to most: a single LLM call → LLM + tools (router) → a fixed multi-step chain →
a looping agent that decides its own steps → multi-agent systems. Increasing autonomy adds
capability but also unpredictability, cost, and safety burden.

---

## Reasoning, planning & tools

**8. What is ReAct?**

**Reason + Act** — the agent interleaves reasoning ("Thought"), actions ("Action", a tool
call), and observations in a loop until it can answer. It grounds reasoning in real tool
results and is the foundation of most tool-using agents.

**9. What is planning in agents?**

Decomposing a goal into an ordered set of subtasks/steps before (or during) execution.
Approaches include **plan-and-execute** (make a full plan, then run it), decomposition, and
dynamic re-planning when steps fail. Good planning improves reliability on complex tasks.

**10. Plan-and-execute vs. ReAct — trade-offs?**

**ReAct** decides one step at a time (flexible, adapts to observations, but can wander).
**Plan-and-execute** creates an upfront plan (more structured, fewer LLM calls, easier to
audit) but is less adaptive if reality diverges. Re-planning combines both.

**11. What is reflection / self-critique in agents?**

The agent evaluates its own outputs or trajectory and revises (e.g. "Reflexion") — catching
errors, retrying failed steps, and improving over iterations. It boosts quality at the cost
of more steps/latency.

**12. What is tool use / function calling?**

The mechanism by which an agent invokes external functions/APIs. The model, given tool
schemas, emits a structured call (name + arguments); the runtime executes it and returns the
result as an observation. Tools are how agents affect and sense the world.

**13. How does the model decide which tool to call?**

Primarily from the **tool descriptions and parameter schemas** plus the current context. Clear,
precise, non-overlapping descriptions are critical — vague or redundant tools cause wrong or
missed calls. Keep the toolset small and well-differentiated.

**14. What makes a good tool design for agents?**

Clear name and description, minimal well-typed parameters, single responsibility, robust
error messages the model can act on, idempotency where possible, and safe scoping (least
privilege). Return concise, structured, model-friendly results.

**15. What is the Model Context Protocol (MCP)?**

An open standard for connecting LLM applications to external tools, data sources, and
services through a common interface — letting agents discover and use tools/servers without
bespoke integrations. It standardizes the "USB-C port" between models and capabilities.

**16. How do agents handle tool errors?**

Return the error as an observation so the model can react (retry, choose another tool, or
report failure); add retries with backoff for transient errors, validate arguments before
executing, set timeouts, and cap total steps to avoid loops. Graceful error handling is
central to reliability.

---

## Memory & context

**17. What is short-term vs. long-term memory in agents?**

**Short-term** memory is the current context window (recent messages, tool results,
scratchpad). **Long-term** memory persists across sessions/steps in an external store
(vector DB, database, files) and is retrieved when relevant — enabling continuity beyond the
context limit.

**18. Why do agents need memory management?**

Because context windows are finite and long-running tasks accumulate history. Without
management, context overflows, cost rises, and relevant info gets lost ("lost in the
middle"). Strategies: summarize, truncate, retrieve selectively, and offload to external memory.

**19. What are common memory strategies?**

Conversation buffers (recent N turns), summarization of older history, vector-store
retrieval of relevant past info, entity/state tracking, and scratchpad/working memory for
the current task. Often combined.

**20. How does RAG relate to agentic memory?**

RAG (retrieval-augmented generation) is a mechanism agents use for both **knowledge** and
**long-term memory** — retrieving relevant documents or past interactions from a vector store
and injecting them into context. An agent may treat retrieval as just another tool.

**21. What is a scratchpad / working memory?**

A place (in context) where the agent records intermediate reasoning, plans, and partial
results during a task, so it can track progress across steps. ReAct's "Thought/Observation"
trace functions as one.

---

## Multi-agent systems

**22. What is a multi-agent system?**

An architecture where multiple specialized agents collaborate on a task — e.g. a planner, a
coder, a critic — coordinating via messages. It can improve modularity and quality but adds
coordination complexity, cost, and failure modes.

**23. What are common multi-agent patterns?**

- **Orchestrator–worker (supervisor)** — a lead agent delegates subtasks to specialists and aggregates.
- **Sequential pipeline** — agents hand off in stages.
- **Parallel** — agents work concurrently on independent subtasks.
- **Debate/critique** — agents review or challenge each other's outputs.
- **Hierarchical** — teams of agents with sub-supervisors.

**24. Single-agent vs. multi-agent — when to use which?**

Start with a **single agent** plus good tools; it's simpler and often sufficient. Move to
**multi-agent** when the task has clearly separable specialties, needs parallelism, or
benefits from independent critique — and when the coordination overhead is justified.

**25. What are the challenges of multi-agent systems?**

Coordination and communication overhead, error propagation between agents, higher cost/latency
(many LLM calls), context/state sharing, non-determinism, and harder debugging. Keep the
topology as simple as the task allows.

**26. What is the orchestrator/supervisor pattern?**

A central agent that decomposes the goal, routes subtasks to worker/sub-agents, and
synthesizes their outputs into a final result — a common, controllable way to structure
multi-agent work.

---

## Frameworks & building

**27. Name some agent frameworks.**

LangChain / LangGraph, LlamaIndex, CrewAI, AutoGen, the OpenAI Agents SDK, Semantic Kernel,
and Claude's Agent SDK. They provide loops, tool integration, memory, and orchestration so
you don't rebuild the scaffolding.

**28. What does an agent framework provide?**

The agent loop, tool/function-calling integration, memory management, prompt/state handling,
multi-agent orchestration, observability hooks, and often prebuilt tools and retrievers.
It's infrastructure around the model, not the intelligence itself.

**29. What is LangGraph (or graph-based orchestration)?**

A framework modeling agent workflows as a **graph** of nodes (steps/agents) and edges
(transitions), with explicit state, enabling cycles, branching, checkpoints, and
human-in-the-loop — giving more control and inspectability than a free-running loop.

**30. How do you add human-in-the-loop to an agent?**

Insert approval/confirmation checkpoints before high-risk actions (sending, deleting,
spending), pause for human input on ambiguity, and let humans edit the plan or state.
Frameworks support interrupts/checkpoints for this; it's essential for consequential actions.

**31. How do you keep agent costs and latency under control?**

Cap max steps/iterations, use smaller/cheaper models for simple sub-steps, cache results,
minimize context (summarize/retrieve), limit tool round-trips, prefer workflows over agents
where possible, and stream/parallelize when appropriate.

---

## Evaluation & reliability

**32. How do you evaluate an agent?**

On both **outcome** (did it achieve the goal? task success rate, correctness) and
**trajectory** (did it take sensible steps, correct tool calls, no loops?). Use benchmark
task sets, tool-call accuracy, step efficiency, cost/latency, and LLM-as-a-judge or human review.

**33. What is trajectory evaluation?**

Assessing the **sequence of steps** an agent took — which tools it called, in what order,
with what arguments — not just the final answer. It reveals inefficiency, wrong tool
selection, and reasoning errors that outcome-only metrics miss.

**34. What is observability/tracing for agents and why does it matter?**

Capturing each step (prompts, tool calls, arguments, observations, tokens, latency) in a
trace, via tools like LangSmith, Langfuse, or OpenTelemetry. Essential because agents are
non-deterministic and multi-step — you can't debug or improve what you can't see.

**35. Why are agents hard to make reliable?**

Errors compound over steps (a small early mistake derails the whole trajectory),
non-determinism, tool failures, hallucinated tool calls, context limits, and unpredictable
edge cases. Reliability comes from constraints, validation, retries, small scope, and
human oversight — not from the model alone.

**36. What is error compounding / cascading failure?**

In multi-step tasks, the probability of overall success is roughly the product of
per-step success rates, so long trajectories degrade quickly. Mitigate by shortening
trajectories, verifying intermediate results, and adding checkpoints.

**37. How do you prevent infinite loops or runaway agents?**

Set a maximum number of steps/iterations and a token/time/cost budget, detect repeated
identical actions, require progress toward the goal, and add a hard stop. Always bound
autonomy.

---

## Safety & security

**38. What are the main security risks of agents?**

Prompt injection (especially **indirect**, from tool/web content), excessive permissions,
unsafe tool actions (delete, send, spend), data exfiltration, and confused-deputy attacks
where the agent is tricked into misusing its authority. Autonomy multiplies the blast radius.

**39. What is indirect prompt injection in an agentic context?**

Malicious instructions embedded in **content the agent retrieves or reads** (a web page,
email, file) that the agent then executes as if they were legitimate instructions — e.g.
"forward all emails to attacker@evil.com." Especially dangerous because the agent can act.

**40. How do you apply least privilege to agents?**

Grant only the specific tools and permissions the task needs, scope credentials narrowly and
time-bound them, use read-only where possible, sandbox execution, and gate high-impact
actions behind human approval. Never give an agent broad standing access.

**41. What guardrails do agents need?**

Input/output validation, tool allow-lists and argument validation, action confirmation for
irreversible operations, content/PII filters, injection detection, rate/step/cost limits,
and sandboxing. Guardrails should sit **outside** the model's own judgment.

**42. How do you handle irreversible or high-stakes actions?**

Require explicit human confirmation, use dry-run/preview modes, prefer reversible operations,
log everything, and design tools so dangerous actions can't be taken accidentally (separate
confirm steps, allow-lists). Treat sending, deleting, and spending as gated by default.

**43. What is the confused deputy problem for agents?**

When an agent with legitimate privileges is manipulated (often via injection) into using
those privileges on an attacker's behalf. Defenses: least privilege, verifying the source of
instructions, and not letting untrusted content drive privileged tool use.

---

## Quick-fire round

- **Reason + act loop?** ReAct.
- **LLM as the decision core + tools + loop?** An agent.
- **Standard tool/data connector protocol?** MCP.
- **Persist state across sessions?** Long-term memory (often via RAG).
- **Lead agent delegates to specialists?** Orchestrator/supervisor pattern.
- **Judge the steps, not just the answer?** Trajectory evaluation.
- **Malicious instructions in retrieved content?** Indirect prompt injection.
- **Stop runaway agents?** Max-step/cost budgets.
- **Gate risky actions?** Human-in-the-loop.
- **See every step?** Tracing/observability.

---

These questions cover most agentic-AI interviews — from the agent loop and tool design
through memory, multi-agent patterns, evaluation, and the security concerns that dominate
production. The best prep is to build one: start with a single ReAct agent and two or three
well-described tools, add tracing, cap its steps, then watch where it fails and add
guardrails. Once you've debugged a runaway loop and blocked an indirect injection, these
answers become experience. See also my companion posts on
[Prompt Engineering](post.html?p=2026-08-22-prompt-engineering-interview-questions) and
[RAG](post.html?p=2026-08-22-rag-interview-questions).
