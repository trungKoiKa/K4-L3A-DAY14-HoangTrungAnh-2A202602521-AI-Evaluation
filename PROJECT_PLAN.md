# Project completion plan

## Completed

1. Implemented the evaluation engine in `template.py` and the submission copy in `solution/solution.py`.
2. Added answer-side, retrieval-side, judge, benchmark, regression, failure-analysis, and bonus lexical-reranking functionality.
3. Built a 20-case stratified golden dataset (5 easy, 7 medium, 5 hard, 3 adversarial) with verbatim evidence and coverage of all ten corpus files.
4. Verified the unit-test suite and the dataset validator.

## Runbook

```powershell
python -m pytest tests/ -v
python validate_golden_dataset.py
Copy-Item .env.example .env
# Set OPENAI_API_KEY in .env, then:
python domain_assistant.py
python evaluate_answers.py
```

## Remaining user-owned runtime step

`domain_assistant.py` calls OpenAI and needs a valid `OPENAI_API_KEY`. No key was created, requested, or stored by this work. Once a key is configured, the last two commands create `artifacts/actual_answers.json` and `artifacts/benchmark_results.json`; use those factual outputs to complete the benchmark-score placeholders in `exercises.md` and `reflection.md`.

## Quality gate

Block a release when average faithfulness, relevance, or completeness drops by more than 0.05 from the approved baseline, or when a safety/adversarial case fails. Add every confirmed production failure as a new regression case before closing it.
