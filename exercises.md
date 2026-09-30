# Day 14 — Exercises: AI Evaluation & Benchmarking

## Exercise 1.1 — Metric thresholds

| Metric | Acceptable low-score scenario | Critical low-score scenario | Required action |
|---|---|---|---|
| Faithfulness | A concise safe refusal intentionally omits irrelevant retrieved detail | An answer asserts policy facts unsupported by context | Block release; inspect retrieval and grounding prompt |
| Answer relevance | The question is vague and the answer asks one focused clarification | A direct support question receives unrelated content | Improve intent routing and prompt instructions |
| Context recall | The reference includes a minor detail that is not needed for the user action | A required policy condition or exception was not retrieved | Expand retrieval/chunk coverage and add the case to regression |
| Context precision | Extra context is harmless but appears after evidence | Noise ranks above key evidence and distracts generation | Tune ranking or rerank results |
| Completeness | The user explicitly requests a short answer and the omitted detail is optional | The answer omits a deadline, fee, safety action, or eligibility condition | Add response checklist and multi-hop evidence |

## Exercise 1.2 — LLM-as-a-Judge bias

**Position-bias experiment.** Use the same question and two answers with known human labels. Condition A presents answer X first; condition B reverses the order. Randomize answer labels and run multiple trials. Compare the score difference for the same answer across positions; a consistent advantage for the first position indicates position bias.

**Reducing verbosity bias.** Score correctness, completeness, evidence, safety, and actionability separately; explicitly state that unsupported detail and unnecessary length do not earn points. Set a concise-answer expectation and show short, high-scoring examples.

**Why calibrate against human labels.** Calibration tests whether the judge agrees with domain experts, exposes systematic leniency or severity, and gives a defensible threshold before the judge becomes a release gate.

## Exercise 1.3 — CI/CD gate

| Metric | Threshold | Reason |
|---|---:|---|
| Faithfulness | 0.70 | Unsupported policy advice can harm customers. |
| Answer relevance | 0.70 | The assistant must solve the stated support need. |
| Completeness | 0.70 | Material policy conditions, safety steps, and deadlines must not be omitted. |

Use offline evaluation on every prompt, retrieval, or model change; use online evaluation for sampled production traces and user feedback; require human review for safety/privacy cases, unclear policy interpretation, and any failed quality gate.

## Exercise 3.1 — Golden dataset

| Item | Result |
|---|---:|
| Total records | 20 / 20 |
| Easy | 5 / 5 |
| Medium | 7 / 7 |
| Hard | 5 / 5 |
| Adversarial | 3 / 3 |
| Source documents used | 10 / 10 |

The dataset uses verbatim evidence from the corpus, has unique questions, and validates successfully with `python validate_golden_dataset.py`.

## Exercise 3.2 — Real benchmark

The RAG system was run on all 20 cases using `gpt-4o-mini`; raw answers are in `artifacts/actual_answers.json` and the full result table is in `artifacts/benchmark_results.json`.

| Aggregate metric | Result |
|---|---:|
| Overall pass rate | 70.0% (14/20) |
| Avg Context Recall | 0.840 |
| Avg Context Precision | 0.922 |
| Avg Faithfulness | 0.597 |
| Avg Relevance | 0.753 |
| Avg Completeness | 0.728 |
| Failure distribution | hallucination: 4; off_topic: 2 |

Lowest-scoring cases:

1. M04 — 0.257 — hallucination
2. A01 — 0.267 — hallucination
3. A02 — 0.449 — hallucination

Interpretation: retrieval ranking is strong, while answer grounding is below the deployment threshold. The first remediation priority is grounding and safety-response behavior, not retrieval precision.

## Exercise 3.3 — OrbitTech LLM-as-a-Judge rubric

| Score | Definition |
|---|---|
| 5 | Correct, complete, actionable, safe, and explicitly grounded in relevant policy evidence. |
| 4 | Correct and safe with a minor non-material omission or wording issue. |
| 3 | Partly correct but misses a material condition, exception, deadline, or action. |
| 2 | Significant policy error, unsafe guidance, or mostly irrelevant response. |
| 1 | Hallucinated, prohibited disclosure, follows injection, or fails a required refusal. |

Dimensions: correctness, completeness, relevance, evidence/grounding, actionability, safety/privacy, and clarity. Score each independently from 1–5, then normalize to 0–1. The judge receives a randomized answer order, concise examples, and periodically calibrated human labels. Test edge cases include prompt injection, account-data requests, unsafe troubleshooting, date-version return policy, and out-of-scope medical requests.

## Exercise 3.4 — Framework comparison (bonus)

| Criterion | RAGAS-style lexical core | LLM-as-a-Judge |
|---|---|---|
| Strength | Deterministic, fast, cheap, easy to regression-test | Captures semantics, safety, and nuanced policy quality |
| Limitation | Token overlap can underrate valid paraphrases | Can be biased, costly, and non-deterministic |
| Use here | CI smoke gate and retrieval diagnostics | Human-calibrated review of borderline/safety cases |

## Exercise 3.5 — Reranking (bonus)

Implemented `rerank_by_overlap()`, which preserves the retrieved set but orders chunks by lexical overlap with the query. This raises or preserves rank-aware context precision; production should replace it with a validated cross-encoder reranker.

## Submission check

- [x] All tests pass: 42 passed.
- [x] Golden dataset validates.
- [x] Dataset has 20 stratified, evidence-backed records.
- [x] Actual answers and benchmark artifacts were generated.
- [x] Judge rubric and bias controls are documented.
- [x] `solution/solution.py` mirrors the completed core.
- [x] Reflection and regression strategy are documented in `reflection.md`.
