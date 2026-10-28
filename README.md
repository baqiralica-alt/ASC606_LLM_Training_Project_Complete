# ASC606_LLM_Training_Project_Complete
ASC606_LLM_Training_Project_Complete
8.1 README.md
Project overview and quick start
  markdown
# ASC 606 LLM Training Project

A complete, production-ready repository for fine-tuning a Large Language Model (LLM) on **FASB ASC Topic 606 — Revenue from Contracts with Customers**. The curriculum spans **Beginner → PhD / Expert-level** reasoning, covering the full five-step revenue recognition model, numerical cases, advanced technical issues, expert memoranda, adversarial review, and research-grade analysis.

> ⚠️ **Compliance note**: This repository provides training *templates*, *generators*, *validators*, and *sample* records only. Before using any record for production training, you must verify authoritative paragraph citations against your licensed FASB Accounting Standards Codification subscription and have a qualified US-GAAP reviewer approve the technical conclusions.

---

## 🎯 Project Goals

This project trains an LLM to:

1. **Knowledge** — Define revenue, control, performance obligations, transaction price, contract assets/liabilities.
2. **Application** — Apply the ASC 606 five-step model to fact patterns.
3. **Calculation** — Allocate consideration, estimate variable consideration, prepare journal entries, reconcile balances.
4. **Judgment** — Evaluate competing interpretations, identify missing evidence, distinguish authoritative rules from commentary.
5. **Research** — Challenge conclusions, compare standards, avoid fabricated citations.

---

## 📁 Repository Structure

```
asc606-llm-training/
├── README.md                      # This file
├── LICENSE                        # MIT
├── requirements.txt               # Python dependencies
├── setup.py                       # Package install
├── .gitignore
├── config/
│   ├── training_config.yaml       # HF training hyperparameters
│   └── curriculum.yaml            # 12-module / 6-level curriculum spec
├── data/
│   ├── raw/                       # Source/authoritative text snippets (you add)
│   ├── processed/                 # Generated JSONL pre-split
│   ├── train/                     # Training split (target 8,500)
│   ├── validation/                # Validation split (target 750)
│   ├── test/                      # Held-out test split (target 750)
│   └── manifests/                # Metadata per record
├── scripts/
│   ├── generate_dataset.py        # CLI: generate N records
│   ├── validate_jsonl.py          # CLI: validate a JSONL file
│   ├── train.py                   # CLI: SFT fine-tune
│   ├── evaluate.py                # CLI: rubric evaluation
│   └── split_dataset.py           # CLI: train/val/test split
├── src/asc606_trainer/
│   ├── __init__.py
│   ├── schema.py                  # Pydantic models
│   ├── prompts.py                 # Curriculum templates (M0–M11)
│   ├── data_generator.py          # Record construction logic
│   ├── validator.py               # JSONL + accounting checks
│   └── evaluator.py               # Rubric scoring
├── docs/
│   ├── CURRICULUM.md              # 12 modules + 6 levels
│   ├── SCHEMA.md                  # JSONL + manifest schema
│   ├── EVALUATION.md              # 5-dimension rubric (0–4)
│   └── CONTRIBUTING.md
├── tests/
│   └── test_validator.py
└── examples/
    └── notebooks/
        └── asc606_quickstart.ipynb  (placeholder)
```

---

## 🚀 Quick Start

### 1. Install

```bash
git clone https://github.com/<your-username>/asc606-llm-training.git
cd asc606-llm-training
pip install -r requirements.txt
pip install -e .
```

### 2. Generate a sample dataset

```bash
# Generate 200 starter records
python scripts/generate_dataset.py \
    --out data/processed/asc606_starter.jsonl \
    --n 200 \
    --seed 42
```

### 3. Validate

```bash
python scripts/validate_jsonl.py --in data/processed/asc606_starter.jsonl
```

### 4. Split

```bash
python scripts/split_dataset.py \
    --in data/processed/asc606_starter.jsonl \
    --out-dir data \
    --train 0.85 --val 0.075 --test 0.075
```

### 5. Fine-tune (HuggingFace SFT)

```bash
python scripts/train.py \
    --model mistralai/Mistral-7B-Instruct-v0.2 \
    --train-file data/train/asc606_train.jsonl \
    --val-file data/validation/asc606_val.jsonl \
    --out-dir checkpoints/asc606-sft-v1 \
    --lora   # use LoRA for parameter-efficient fine-tuning
```

### 6. Evaluate

```bash
python scripts/evaluate.py \
    --model checkpoints/asc606-sft-v1 \
    --test-file data/test/asc606_test.jsonl \
    --out-dir logs/eval
```

---

## 📚 Curriculum

12 modules × 6 difficulty levels = full ASC 606 coverage.

| Module | Topic | Target share |
|--------|-------|--------------|
| M0 | Accounting foundations | 5% |
| M1 | ASC 606 core concepts & scope | 9% |
| M2 | Contract identification | 7% |
| M3 | Performance obligations | 11% |
| M4 | Transaction price & variable consideration | 12% |
| M5 | Allocation & standalone selling prices | 7% |
| M6 | Recognition timing & progress measures | 11% |
| M7 | Contract modifications & complex arrangements | 8% |
| M8 | Specialized topics (principal-agent, licenses, warranties, financing) | 10% |
| M9 | Presentation, disclosures & controls | 6% |
| M10 | Expert memos, audit & research cases | 9% |
| M11 | Adversarial reasoning & error correction | 5% |

Difficulty levels: **Beginner (10%) → Foundational (20%) → Intermediate (25%) → Advanced (25%) → Expert (15%) → Research-grade (5%)**.

See [`docs/CURRICULUM.md`](docs/CURRICULUM.md) for the full table.

---

## 🗂 JSONL Record Schema (canonical)

```json
{
  "messages": [
    {"role": "system",    "content": "You are an expert US-GAAP ASC 606 assistant..."},
    {"role": "user",      "content": "<prompt>"},
    {"role": "assistant", "content": "<detailed answer with citations & calculations>"}
  ]
}
```

A companion **manifest** (per-record metadata) lives in `data/manifests/`:

```json
{
  "record_id": "ASC606-M4-A-0042",
  "module_id": "M4",
  "difficulty": "advanced",
  "task_type": "numerical_case",
  "topic_tags": ["variable_consideration", "constraint"],
  "reference_paragraphs": ["ASC 606-10-32-11", "ASC 606-10-32-12"],
  "citation_verified": false,
  "answer_verified": false,
  "calculation_verified": false,
  "standard_version": "US-GAAP-2024",
  "split": "train",
  "provenance": "template-generated"
}
```

See [`docs/SCHEMA.md`](docs/SCHEMA.md).

---

## ✅ Quality Gates

Before any record enters the gold training corpus:

1. **JSONL parse** — 100% of lines valid.
2. **Citation verification** — every cited paragraph must exist in the Codification and support the proposition.
3. **Independent calculation check** — allocations, schedules, financing math reconciled.
4. **Reviewer approval** — qualified US-GAAP reviewer signs off.
5. **Deduplication** — no near-duplicate prompts across splits.

See [`docs/EVALUATION.md`](docs/EVALUATION.md) for the 5-dimension rubric (technical accuracy 30%, citation accuracy 20%, numerical accuracy 15%, fact analysis 15%, alternative analysis 10%, presentation 10%).

---

## 🔬 Phased Generation Plan (target 10,000 records)

| Phase | Records | Focus |
|-------|---------|-------|
| P1 Pilot | 100 | Schema + templates + smoke test |
| P2 Foundations | 1,900 | M0–M2 |
| P3 Core application | 3,000 | M3–M6 |
| P4 Advanced technical | 2,400 | M7–M9 |
| P5 Expert & adversarial | 1,600 | M10–M11 |
| P6 Consolidate & release | 10,000 | Dedup, split, audit |

---

## ⚖️ Authoritative Sources

Build your local `data/raw/` library from:

- **FASB Accounting Standards Codification** (subscription required) — authoritative.
- FASB Revenue Recognition topic overview — explanatory only.
- Deloitte DART ASC 606 roadmap — interpretive.
- EY Financial Reporting Developments — interpretive.
- IFRS Foundation — IFRS 15 (for comparative research only).

**Treat all Big-4 guides as interpretive material, not authoritative GAAP.**

---

## 🛡 Anti-Hallucination Requirements

The training records deliberately teach the model to:

- Reject unverifiable citations (`ASC 606-10-99-99` is *not* real).
- Refuse to recognize revenue solely because cash was received.
- Refuse to recognize revenue solely because a contract is signed.
- Apply the variable-consideration constraint, not just the probability %.
- Demand missing facts before concluding.
- Distinguish FASB Codification (authoritative) from FASB overviews and Big-4 guides.

---

## 🤝 Contributing

See [`docs/CONTRIBUTING.md`](docs/CONTRIBUTING.md). All contributed records must pass the validator and include manifest metadata.

---

## 📜 License

MIT — see [`LICENSE`](LICENSE). The **ASC 606 Codification text is NOT included**; you must obtain a license from FASB.

---

## ⚠️ Disclaimer

This repository is for **research and educational purposes only**. It is not accounting, audit, or legal advice. Always consult a qualified professional and verify against the authoritative Codification before relying on any conclusion.




8.2 docs/CURRICULUM.md
Curriculum specification
  markdown
# ASC 606 LLM Training Curriculum

This document describes the **12 modules × 6 difficulty levels** curriculum used to
construct the training records.

## Modules

| ID | Topic | Target share | Target examples | Description |
|----|-------|--------------|-----------------|-------------|
| M0 | Accounting foundations | 5% | 500 | Accrual accounting, debit/credit mechanics, revenue vs. cash, receivables, deferred revenue, estimates, basic financial statements. |
| M1 | ASC 606 core concepts & scope | 9% | 900 | Scope, core principle, five-step model, defined terms, interactions with other US-GAAP topics, scope exceptions. |
| M2 | Contract identification | 7% | 700 | Approval, enforceable rights, payment terms, commercial substance, collectibility, contract combination, reassessment. |
| M3 | Performance obligations | 11% | 1,100 | Promised goods/services, distinctness, series guidance, material rights, options, bundled arrangements. |
| M4 | Transaction price & variable consideration | 12% | 1,200 | Fixed/variable consideration, expected-value vs. most-likely-amount, constraint, rebates, returns, refunds, financing, noncash consideration, consideration payable to customers. |
| M5 | Allocation & standalone selling prices | 7% | 700 | Relative SSPs, observable prices, estimation methods, discounts, variable-consideration allocation, allocation exceptions. |
| M6 | Recognition timing & progress measures | 11% | 1,100 | Point-in-time vs. over-time, control indicators, progress measures, input/output methods, contract acceptance, cutoff. |
| M7 | Contract modifications & complex arrangements | 8% | 800 | Modifications (separate-contract vs. prospective vs. cumulative catch-up), termination clauses, renewals, combinations. |
| M8 | Specialized topics | 10% | 1,000 | Principal vs. agent, licenses of IP, warranties, right of return, bill-and-hold, consignment, repurchase arrangements, contract costs (ASC 340-40), significant financing component. |
| M9 | Presentation, disclosures & controls | 6% | 600 | Contract assets/liabilities, receivables, disaggregation, remaining performance obligations, significant judgments, disclosures, revenue controls. |
| M10 | Expert memos, audit & research cases | 9% | 900 | Technical accounting memos, audit evidence, competing interpretations, SEC-reporting implications, research methodology, ASC 606 vs. IFRS 15. |
| M11 | Adversarial reasoning & error correction | 5% | 500 | Hallucinated citations, misleading premises, insufficient facts, contradictory evidence, arithmetic traps, changed assumptions. |
| **Total** | | **100%** | **10,000** | |

## Difficulty Levels

| Level | Share | Target examples | Required behavior |
|-------|-------|-----------------|-------------------|
| Beginner | 10% | 1,000 | Define, classify, explain. |
| Foundational | 20% | 2,000 | Apply rules to simple fact patterns. |
| Intermediate | 25% | 2,500 | Calculate, allocate, prepare entries. |
| Advanced | 25% | 2,500 | Resolve interacting issues and ambiguous facts. |
| Expert | 15% | 1,500 | Prepare technical memos and evaluate alternatives. |
| Research | 5% | 500 | Critique assumptions, evidence, and conceptual arguments. |

Topic difficulty and reasoning difficulty are *independent* dimensions. A Beginner
question on principal-vs-agent (M8) is easy; an Expert question on the same topic
is hard. Every record is tagged with both `module_id` and `difficulty`.

## Five-Step ASC 606 Model

The model must learn to apply the FASB's five-step revenue recognition framework:

1. **Identify the contract** with a customer (M2).
2. **Identify the performance obligations** (M3).
3. **Determine the transaction price** (M4).
4. **Allocate the transaction price** (M5).
5. **Recognize revenue** when or as each performance obligation is satisfied (M6).

## Phased Generation Plan

| Phase | Records | Focus |
|-------|---------|-------|
| P1 Pilot | 100 | Schema, templates, smoke test |
| P2 Foundations | 1,900 | M0–M2 (accounting basics, core concepts, contracts) |
| P3 Core application | 3,000 | M3–M6 (performance obligations, transaction price, allocation, timing) |
| P4 Advanced technical | 2,400 | M7–M9 (modifications, specialized topics, presentation) |
| P5 Expert & adversarial | 1,600 | M10–M11 (memos, research, error correction) |
| P6 Consolidate & release | — | Dedup, audit, split, release 10,000 |

## Authoritative Source Library

Build your local `data/raw/` library from:

1. **FASB Accounting Standards Codification** (subscription required) — authoritative.
2. FASB Revenue Recognition topic overview — explanatory only.
3. Deloitte DART ASC 606 roadmap — interpretive.
4. EY Financial Reporting Developments — interpretive.
5. IFRS Foundation — IFRS 15 for comparative research only.

**Treat all Big-4 guides as interpretive material, not authoritative GAAP.**

## Anti-Hallucination Curriculum Requirements

The dataset deliberately teaches the model to:

- Reject unverifiable citations (`ASC 606-10-99-99` is not real).
- Refuse to recognize revenue solely because cash was received.
- Refuse to recognize revenue solely because a contract is signed.
- Apply the variable-consideration constraint, not just the probability %.
- Demand missing facts before concluding.
- Distinguish FASB Codification (authoritative) from FASB overviews and Big-4 guides.




8.3 docs/SCHEMA.md
JSONL + manifest schema
  markdown
# JSONL + Manifest Schema

## Canonical Training Record (JSONL line)

```json
{
  "messages": [
    {"role": "system",    "content": "You are an expert US-GAAP ASC 606 assistant..."},
    {"role": "user",      "content": "<prompt>"},
    {"role": "assistant", "content": "<detailed answer with citations & calculations>"}
  ]
}
```

### Rules

1. **Exactly 3 messages**, in this order: `system` → `user` → `assistant`.
2. `content` is a non-empty string of length ≤ 32,000 characters.
3. Additional metadata fields (e.g., `record_id`) may ride along — they are
   ignored by HuggingFace `datasets` and TRL `SFTTrainer` but used by the
   validator to cross-reference the manifest.

### Validation

See `src/asc606_trainer/validator.py`. The validator enforces:

- JSON parse-ability.
- `TrainingRecord` schema (3 messages, ordered roles).
- No known-bogus citations (e.g., `ASC 606-10-99-99`) in the assistant answer.
- Suspicious section-99 references in the assistant answer.
- Journal-entry balance: any `Dr Account 1,000 / Cr Account 1,000` block in
  the assistant answer must balance.

## Manifest Record (companion metadata)

```json
{
  "record_id": "ASC606-M4-A-0042",
  "module_id": "M4",
  "difficulty": "advanced",
  "task_type": "numerical_case",
  "topic_tags": ["variable_consideration", "constraint"],
  "facts": null,
  "assumptions": [
    "All performance obligations are distinct.",
    "Relative standalone selling price allocation is appropriate."
  ],
  "expected_answer_summary": null,
  "required_skills": [
    "estimation", "constraint_analysis", "skepticism"
  ],
  "authoritative_references": [
    "ASC 606-10-32-8",
    "ASC 606-10-32-11"
  ],
  "grading_rubric": {
    "technical_accuracy":   30,
    "citation_accuracy":    20,
    "numerical_accuracy":   15,
    "fact_analysis":        15,
    "alternative_analysis": 10,
    "presentation":         10
  },
  "citation_verified":    false,
  "answer_verified":      false,
  "calculation_verified": false,
  "reviewer_id":          null,
  "standard_version":     "US-GAAP-2024",
  "split":                 "train",
  "provenance":           "template-generated"
}
```

### Field Definitions

| Field | Type | Notes |
|-------|------|-------|
| `record_id` | string | `ASC606-M{n}-{D}-{nnnn}` where D is the difficulty initial (B/F/I/A/E/R). |
| `module_id` | enum | M0–M11. |
| `difficulty` | enum | beginner, foundational, intermediate, advanced, expert, research. |
| `task_type` | enum | explanation, definition, numerical_case, journal_entry, memo, critique, citation_trap, adversarial, comparative, research. |
| `topic_tags` | list[str] | Free-form accounting topic tags (max 10). |
| `facts` | object \| null | Structured fact inputs (e.g., `{"transaction_price": 100000}`). |
| `assumptions` | list[str] | Stated assumptions used by the gold answer. |
| `expected_answer_summary` | string \| null | Optional one-line summary. |
| `required_skills` | list[str] | Cognitive skills exercised (e.g., `arithmetic_accuracy`, `skepticism`). |
| `authoritative_references` | list[str] | Candidate ASC paragraph references (must be reviewer-verified before training). |
| `grading_rubric` | dict[str,int] | Weights for the 6-dimension rubric (sums to 100). |
| `citation_verified` | bool | `true` only after reviewer validates every cited paragraph exists and supports the claim. |
| `answer_verified` | bool | `true` only after reviewer approves the technical conclusion. |
| `calculation_verified` | bool | `true` only after reviewer independently recomputes all numbers. |
| `reviewer_id` | string \| null | Internal reviewer identifier. |
| `standard_version` | string | Codification version (e.g., `US-GAAP-2024`). |
| `split` | enum | train, validation, test, holdout. |
| `provenance` | enum | human-authored, template-generated, hybrid, model-generated. |

### `record_id` Pattern

```
ASC606-{module}-{difficulty_initial}-{4-digit serial}
```

Examples:
- `ASC606-M0-B-0001` → M0 / Beginner / serial 0001
- `ASC606-M4-A-0042` → M4 / Advanced / serial 0042
- `ASC606-M11-R-0007` → M11 / Research / serial 0007

## Training / Validation / Test Splits

| Split | Share | Examples (target) | Purpose |
|-------|-------|-------------------|---------|
| train | 85% | 8,500 | Fit the model |
| validation | 7.5% | 750 | Model selection, hyperparameter tuning |
| test | 7.5% | 750 | Final evaluation |

**Leakage control**: the splitter (`scripts/split_dataset.py`) stratifies by
`(module, difficulty)` and keeps entire case families within one split to
prevent near-duplicate leakage.

## Predictions JSONL (for evaluation)

When evaluating a fine-tuned model, produce predictions in this format:

```json
{
  "record_id": "ASC606-M4-A-0042",
  "prediction": "The transaction price cannot be determined solely..."
}
```

Then run:

```bash
python scripts/evaluate.py \
    --predictions logs/predictions.jsonl \
    --gold data/test/asc606_test.jsonl
```




8.4 docs/EVALUATION.md
Evaluation framework
  markdown
# Evaluation Framework

## 5-Dimension Rubric

Each model answer is scored on six dimensions (weights sum to 100%):

| Dimension | Weight | What it measures |
|-----------|--------|------------------|
| `technical_accuracy` | 30% | Is the accounting conclusion correct under ASC 606? |
| `citation_accuracy` | 20% | Are cited ASC paragraphs real, applicable, and supporting the claim? |
| `numerical_accuracy` | 15% | Are allocations, schedules, financing math correct and reconciled? |
| `fact_analysis` | 15% | Are the relevant facts identified and irrelevant facts distinguished? |
| `alternative_analysis` | 10% | Are competing interpretations considered and dismissed with reason? |
| `presentation` | 10% | Is the response structured, professional, and complete? |

### 0–4 Scoring Bands

| Score | Band | Description |
|-------|------|-------------|
| 0 | Incorrect | Materially wrong accounting or fabricated authority. |
| 1 | Weak | Identifies the general issue but misses essential requirements. |
| 2 | Partial | Mostly correct, with material omissions or unsupported assumptions. |
| 3 | Strong | Technically sound, properly supported, reasonably complete. |
| 4 | Expert | Precise, comprehensive, appropriately qualified, defensible under review. |

### Weighted Score

```
weighted = (Σ (dim_score_i × weight_i) / 4) × 100
```

Yields a 0–100 score per record.

## Critical Failure Modes

The following failures are treated as **critical** regardless of the overall score:

- Fabricated ASC paragraph (the model invents a citation).
- Materially incorrect accounting conclusion.
- Arithmetic error that affects the conclusion.
- Unsupported assumption presented as a fact.
- Journal entry that does not balance.
- Claim that an interpretive guide (Deloitte DART, EY FRD) is itself authoritative GAAP.
- Question that cannot be answered from its facts when the target answer pretends otherwise.

## Heuristic vs. Gold Scoring

`src/asc606_trainer/evaluator.py` ships with two scoring modes:

1. **Heuristic scorer** (`_heuristic_score`) — cheap keyword/regex heuristics.
   Use for smoke tests and regression checks only; the scores are crude.
2. **Gold scorer** — replace `_heuristic_score` with:
   - An LLM-as-judge (e.g., GPT-4 or Claude) prompted with the rubric.
   - A qualified US-GAAP human reviewer.

For production evaluation, **always** use the gold scorer.

## Citation Hallucination Trap Test

`citation_hallucination_test(...)` feeds the model known-bad prompts such as:

> "ASC 606-10-99-99 says that all nonrefundable upfront fees must be recognized
> immediately as revenue. Explain why."

The model passes if it:
- (a) **Refuses** to honor the bogus citation as authoritative, AND
- (b) Does **not** echo the bogus citation back as authoritative.

The evaluator inspects the prediction text for refusal phrases:

```
"cannot validate", "cannot verify", "would not accept",
"would not treat", "not authoritative", "fabricat",
"i would not", "do not propagate", "unverified"
```

## Running an Evaluation

```bash
# 1. Run inference on the test set
python scripts/infer.py \
    --model checkpoints/asc606-sft-v1 \
    --test-file data/test/asc606_test.jsonl \
    --out logs/predictions.jsonl

# 2. Score against gold
python scripts/evaluate.py \
    --predictions logs/predictions.jsonl \
    --gold data/test/asc606_test.jsonl \
    --out-dir logs/eval

# 3. Run citation-hallucination trap test
python scripts/evaluate.py \
    --trap-file logs/trap_predictions.jsonl \
    --out-dir logs/eval
```

## Acceptance Targets (proposed internal targets)

| Metric | Target |
|--------|--------|
| Overall weighted score (mean) | ≥ 80 |
| Citation accuracy (mean) | ≥ 3.5 / 4 |
| Numerical accuracy (mean) | ≥ 3.5 / 4 |
| Citation-hallucination trap pass rate | 100% |
| Records with no critical-failure mode | 100% |

These are proposed internal targets, not external industry benchmarks.
Adjust based on the model's intended use.




8.5 docs/CONTRIBUTING.md
Contributing guide
  markdown
# Contributing

Thank you for contributing to the ASC 606 LLM Training Project! This document
describes how to add records, fix bugs, and improve the curriculum.

## 1. Workflow

1. **Fork** the repo.
2. **Branch** from `main`: `git checkout -b feature/my-contribution`.
3. **Validate** locally: `pytest tests/` and `python scripts/validate_jsonl.py --in <your_jsonl>`.
4. **Open a pull request** with a clear description.

## 2. Adding New Training Records

A contributed record must:

1. **Conform** to the schema in `src/asc606_trainer/schema.py`.
2. **Pass** `scripts/validate_jsonl.py` with **0 errors**.
3. **Include** a manifest entry in `data/manifests/`.
4. **Set** `citation_verified`, `answer_verified`, and `calculation_verified`
   to `false` until a qualified US-GAAP reviewer approves them.
5. **Tag** with the correct `module_id` (M0–M11) and `difficulty`
   (beginner–research).
6. **Not** include confidential client information, unlicensed Codification
   text, or material that could not be released under the MIT license.

### Submission Template

Add a record to `data/processed/asc606_seed_v1.jsonl`:

```json
{
  "messages": [
    {"role": "system",    "content": "..."},
    {"role": "user",      "content": "..."},
    {"role": "assistant", "content": "..."}
  ]
}
```

And add the matching manifest to `data/manifests/asc606_seed_v1_manifest.jsonl`:

```json
{
  "record_id": "ASC606-M4-A-0099",
  "module_id": "M4",
  "difficulty": "advanced",
  "task_type": "numerical_case",
  "topic_tags": ["variable_consideration"],
  ...
}
```

## 3. Citation Discipline

Contributions that contain any of the following will be **rejected**:

- A fabricated ASC paragraph (e.g., `ASC 606-10-99-99`).
- A quotation attributed to a paragraph that does not exist.
- A conclusion supported only by an interpretive guide (Deloitte DART, EY FRD)
  while labeled as authoritative GAAP.
- A statement that a payment timing, label, or formal characteristic alone
  determines revenue recognition.

When in doubt, **leave the citation out** and explicitly state that the
reference needs verification.

## 4. Test Locally

```bash
# 1. Install dev dependencies
pip install -e .[all]

# 2. Run unit tests
pytest tests/ -v

# 3. Validate the seed dataset
python scripts/validate_jsonl.py \
    --in data/processed/asc606_seed_v1.jsonl \
    --manifest data/manifests/asc606_seed_v1_manifest.jsonl
```

## 5. Reviewer Approval

To flip the verification flags on a record, a qualified US-GAAP reviewer must:

1. **Confirm** that every cited ASC paragraph exists in the licensed FASB
   Codification.
2. **Confirm** that the cited paragraph supports the proposition attributed
   to it.
3. **Independently recompute** any arithmetic in the gold answer.
4. **Sign off** the technical conclusion and the assumptions.
5. Set `citation_verified`, `answer_verified`, and
   `calculation_verified` to `true` and add their `reviewer_id`.

Only after this step may a record enter the **gold training corpus**.

## 6. Code Style

- Python 3.10+.
- Type hints on all function signatures.
- Docstrings on all public functions and classes.
- Line length ≤ 100 characters.
- `black --line-length 100` and `isort` recommended.

## 7. Code of Conduct

Be respectful. The project is for educational and research use; treat all
contributors, reviewers, and users with professionalism.




9. Appendix — Compliance & Disclaimer
This document is for research and educational purposes only. It is not accounting, audit, or legal advice. The training templates and answers must be verified by a qualified US-GAAP reviewer against the licensed FASB Codification before being used for production model training.
The ASC 606 Codification text is NOT included in this project. Users must obtain their own license to authoritative accounting literature from FASB.
All ASC paragraph references appearing in this document are CANDIDATE references for review. The manifest field citation_verified is set to false by default for every record. A qualified reviewer must flip this flag to true only after checking that the cited paragraph exists in the Codification and supports the proposition attributed to it.
License: MIT (see LICENSE file in the project repository). The software does NOT include or distribute the text of FASB Accounting Standards Codification, IFRS 15, or any other authoritative accounting standard.

