# Prompt Engineering Interview Questions & Answers

An interview-ready reference for **prompt engineering** — how large language models
consume prompts, the core techniques (few-shot, chain-of-thought, structured output),
context and parameter control, evaluation, and the failure modes and safety concerns that
come up in real work. Grouped by theme, with answers concise enough to say aloud.

---

## Fundamentals

**1. What is prompt engineering?**

The practice of designing, structuring, and refining the input (prompt) given to a large
language model to reliably elicit accurate, useful, and well-formatted outputs. It's how
you steer a general model toward a specific task without changing its weights.

**2. What is a prompt made of?**

Typically an **instruction** (what to do), **context** (background/reference data),
**input data** (the specific thing to act on), and an **output indicator** (desired format).
In chat models these map onto **system**, **user**, and **assistant** messages.

**3. What is the difference between system, user, and assistant messages?**

The **system** message sets persistent behavior, role, and rules. **User** messages are
the human's requests. **Assistant** messages are the model's responses (and, in few-shot
prompting, example responses you supply). The system message has the strongest steering
effect on tone and constraints.

**4. How does an LLM actually process a prompt?**

It tokenizes the text, then predicts the next token repeatedly (autoregressively) based on
all preceding tokens, sampling from a probability distribution. The prompt conditions that
distribution — it doesn't "understand" instructions so much as complete the most probable
continuation given them.

**5. What is a token?**

A token is the unit an LLM reads/writes — roughly a word piece (about 3–4 characters or
~0.75 words in English). Prompt length, cost, and context limits are all measured in tokens,
not characters or words.

**6. What is zero-shot prompting?**

Asking the model to perform a task with only an instruction and no examples, e.g.
"Classify this review as positive or negative: …". It relies entirely on the model's
pretrained knowledge.

**7. What is few-shot prompting?**

Providing a handful of input→output **examples** in the prompt before the actual query, so
the model infers the pattern and format. It usually improves accuracy and consistency for
tasks where the desired behavior is easier to show than describe.

**8. What is in-context learning?**

The model's ability to learn a task from examples **within the prompt** at inference time,
without any weight updates. Few-shot prompting is a form of it. The "learning" is temporary,
confined to that context window.

---

## Core techniques

**9. What is chain-of-thought (CoT) prompting?**

Prompting the model to **reason step by step** before giving a final answer (e.g. "Let's
think step by step"). It improves performance on multi-step reasoning, math, and logic by
letting the model use intermediate tokens as scratch space.

**10. Zero-shot CoT vs. few-shot CoT?**

**Zero-shot CoT** just adds a trigger phrase like "think step by step." **Few-shot CoT**
provides worked examples that themselves show step-by-step reasoning, teaching both the
reasoning style and the answer format. Few-shot CoT is usually stronger but costs more tokens.

**11. What is self-consistency?**

Sampling multiple reasoning paths (with temperature > 0) for the same prompt and taking the
**majority-vote** answer, instead of a single greedy chain. It improves reliability on
reasoning tasks by marginalizing over noisy individual chains.

**12. What is few-shot vs. fine-tuning — when do you choose which?**

Use **few-shot/prompting** when you need flexibility, quick iteration, small data, or can't
train. Use **fine-tuning** when you have many examples, need consistent behavior/format at
scale, want lower per-request token cost, or must encode domain style the prompt can't
reliably capture.

**13. What is role prompting / persona prompting?**

Assigning the model a role ("You are a senior security engineer…") to shape tone, depth,
vocabulary, and priorities. It's a cheap way to bias the response toward a domain and
expertise level.

**14. What is structured output prompting?**

Instructing the model to respond in a machine-parseable format (JSON, XML, a fixed schema).
Best practices: give an explicit schema/example, use delimiters, and where the API supports
it, use a **JSON mode / tool-calling / structured-output** feature that constrains decoding
to valid JSON.

**15. What are delimiters and why use them?**

Explicit separators (triple backticks, XML tags, `###`) that mark boundaries between
instructions, context, and user input. They reduce ambiguity, improve parsing, and are a
key defense against prompt injection by clearly demarcating untrusted content.

**16. What is prompt chaining?**

Breaking a complex task into a sequence of smaller prompts where each step's output feeds
the next (e.g. extract → summarize → format). It improves reliability and debuggability
versus one giant prompt.

**17. What is ReAct prompting?**

**Reason + Act** — a pattern where the model interleaves reasoning ("Thought") with actions
("Action", e.g. a tool/search call) and observations, looping until it answers. It's the
foundation of many tool-using agents.

**18. What is least-to-most prompting?**

Decomposing a hard problem into progressively simpler subproblems, solving them in order,
and using earlier answers to solve later ones — helpful for complex reasoning that a single
CoT chain handles poorly.

**19. What is "generated knowledge" prompting?**

Asking the model to first generate relevant facts/knowledge about the question, then use
that generated context to answer — improving accuracy on knowledge-intensive tasks.

---

## Context, decoding & parameters

**20. What is the context window?**

The maximum number of tokens (prompt + response) a model can consider at once. Everything
outside it is invisible. Managing it — trimming, summarizing, retrieving only relevant
chunks — is central to prompt and RAG design.

**21. What is the "lost in the middle" problem?**

LLMs tend to attend best to information at the **beginning and end** of a long context and
can overlook content buried in the middle. Mitigate by placing the most important
instructions/context at the edges and keeping prompts focused.

**22. What is temperature?**

A decoding parameter controlling randomness by scaling the token probability distribution.
**Low** (≈0) → deterministic, focused, best for factual/extraction tasks. **High** (≈0.8–1)
→ diverse, creative, best for brainstorming. It trades consistency for variety.

**23. What are top-p and top-k?**

Sampling truncation methods. **Top-k** samples from the k most likely tokens. **Top-p
(nucleus)** samples from the smallest set of tokens whose cumulative probability ≥ p. Both
limit unlikely tokens; usually you tune temperature *or* top-p, not both aggressively.

**24. What are max tokens and stop sequences?**

**Max tokens** caps the response length (and cost). **Stop sequences** are strings that
halt generation when produced (e.g. a closing tag), useful for controlling output boundaries
in structured or chained prompts.

**25. What is a frequency/presence penalty?**

Parameters that discourage repetition: **frequency penalty** reduces the probability of
tokens proportional to how often they've appeared; **presence penalty** discourages any
already-used token, nudging the model toward new topics.

**26. How do you handle a task that exceeds the context window?**

Chunk the input and process pieces (map-reduce/refine summarization), use retrieval (RAG) to
pull only relevant sections, summarize prior context, or use a model with a larger window.
The goal is to fit only what's necessary and relevant.

---

## Grounding, evaluation & iteration

**27. What is hallucination and how do you reduce it via prompting?**

Hallucination is the model producing confident but false/unsupported content. Reduce it by
**grounding** in provided context (RAG), instructing "answer only from the context; say 'I
don't know' if it's not there," asking for citations, lowering temperature, and
discouraging speculation.

**28. How do you make a model say "I don't know"?**

Explicitly permit and instruct it: state that uncertainty is acceptable, that it must not
fabricate, and give it an escape hatch ("If the answer isn't in the context, respond 'I
don't know'"). Without permission, models tend to guess.

**29. How do you evaluate prompt quality?**

Build a **test set** of representative inputs with expected outputs/criteria, then measure
accuracy, format compliance, and consistency. Methods: exact/rule-based checks, human
review, and **LLM-as-a-judge**. Track results as you iterate to avoid regressions.

**30. What is LLM-as-a-judge?**

Using a (often stronger) LLM to score or compare model outputs against a rubric or a
reference — a scalable evaluation method. Caveats: judges have biases (position, verbosity,
self-preference), so calibrate with human spot-checks and randomize order.

**31. How do you iterate on a prompt systematically?**

Change one variable at a time, test against a fixed eval set, keep versioned prompts,
compare metrics, and analyze failures to form the next hypothesis. Treat prompts like code:
versioned, tested, and reviewed.

**32. What are common causes of inconsistent outputs?**

High temperature, ambiguous instructions, missing format constraints, conflicting
instructions, over-long context ("lost in the middle"), and model nondeterminism. Fix with
clearer instructions, examples, lower temperature, and structured output constraints.

**33. What is prompt sensitivity / brittleness?**

Small wording, ordering, or formatting changes can noticeably shift outputs. It's why you
evaluate empirically rather than assume, and why robust prompts use clear structure,
examples, and explicit constraints.

---

## Advanced patterns

**34. What is meta-prompting?**

Using an LLM to generate, critique, or improve prompts — e.g. asking the model to rewrite a
prompt for clarity, or to produce few-shot examples. It bootstraps better prompts and can be
automated.

**35. What is a rubric / grading prompt?**

A prompt that gives the model explicit criteria to evaluate or produce content against
(scoring dimensions and definitions), improving consistency for evaluation, feedback, or
quality-controlled generation.

**36. What is self-critique / reflection prompting?**

Having the model review and revise its own output ("Critique your answer for errors, then
produce an improved version"). It can catch mistakes, though it's not guaranteed and adds
cost/latency.

**37. How do you design few-shot examples well?**

Make them representative and diverse, cover edge cases, keep the format identical to what
you want, order them thoughtfully (harder/recent last can help), and ensure they're correct
— errors in examples propagate. Don't overload; a few good examples beat many mediocre ones.

**38. How do you prompt for tool/function calling?**

Provide clear tool schemas (name, description, parameters), instruct when to use each, and
let the model emit a structured call the app executes. Descriptions matter: the model
chooses tools largely from their descriptions, so write them precisely.

**39. How do you control output length and style precisely?**

Specify constraints explicitly ("in 3 bullet points," "under 100 words," "formal tone"),
give an example of the target length/style, use max tokens as a hard cap, and consider
stop sequences. Combine instruction + example for best adherence.

**40. What is the difference between prompting and RAG?**

Prompting alone relies on the model's parametric knowledge and whatever you paste in. **RAG**
augments the prompt by **retrieving** relevant external documents at query time and inserting
them as context — grounding answers in up-to-date, source-specific data.

---

## Safety & failure modes

**41. What is prompt injection?**

An attack where malicious instructions embedded in **user input or retrieved content**
override or subvert the developer's intended prompt (e.g. "ignore previous instructions…").
It's a top LLM security risk.

**42. Direct vs. indirect prompt injection?**

**Direct** — the attacker types malicious instructions straight into the input. **Indirect**
— malicious instructions hide in **external content** the model ingests (a web page,
document, email) and act when processed. Indirect is especially dangerous for agents/RAG.

**43. How do you defend against prompt injection?**

Treat all external/user content as **untrusted data, not instructions**; separate it with
clear delimiters; keep authoritative instructions in the system prompt; constrain tools and
apply least privilege; validate/sanitize outputs; and add guardrails/allowlists. No single
fix is complete — layer defenses.

**44. What is jailbreaking and how is it different from injection?**

**Jailbreaking** tricks the model into bypassing its safety/policy constraints (via
role-play, obfuscation, hypotheticals). **Injection** hijacks the *application's* prompt
logic. They overlap but target different layers — model policy vs. app instructions.

**45. What is prompt leaking?**

An attack (or accident) where the model reveals its confidential system prompt or embedded
secrets. Mitigate by not putting secrets in prompts, instructing the model not to disclose
system content, and validating outputs — though instructions alone aren't a hard guarantee.

**46. Why shouldn't secrets or PII go in prompts?**

Prompts may be logged, cached, leaked via prompt-leaking, or exposed to third-party model
providers. Keep secrets in a vault and out of context; minimize and redact PII to reduce
privacy and compliance risk.

**47. What are guardrails?**

Input/output validation layers around the model — content filters, schema/format
validators, allow/deny lists, toxicity/PII detectors, and injection detectors — that
enforce safety and correctness independent of the model's own behavior.

---

## Quick-fire round

- **No examples?** Zero-shot; with examples → few-shot.
- **Reason step by step?** Chain-of-thought.
- **Majority vote over samples?** Self-consistency.
- **Reason + act with tools?** ReAct.
- **Randomness knob?** Temperature.
- **Max tokens the model sees?** Context window.
- **Ground answers in retrieved docs?** RAG.
- **Malicious instructions in input?** Prompt injection.
- **Model reveals its system prompt?** Prompt leaking.
- **Evaluate outputs with another model?** LLM-as-a-judge.
- **Force valid JSON?** Structured output / JSON mode.

---

These questions cover most prompt-engineering interviews — from tokens and few-shot through
chain-of-thought, decoding parameters, evaluation, and injection defense. To make them
stick, build a small eval harness: write a prompt, a test set, and an LLM-judge, then
iterate temperature, examples, and structure while watching the metrics move. Once you've
seen a prompt injection break your app and fixed it with delimiters and least-privilege
tools, these answers become instinct rather than theory. See also my companion posts on
[Agentic AI](post.html?p=2026-08-22-agentic-ai-interview-questions) and
[RAG](post.html?p=2026-08-22-rag-interview-questions).
