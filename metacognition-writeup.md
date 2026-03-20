### Metacognition Battery: Measuring Self-Monitoring in Frontier LLMs

### Team: steadylearner

### Problem Statement

Frontier models sound humble but fail to self-correct. The FINAL Bench (2026) quantifies this: metacognitive accuracy (verbalizing uncertainty) averages 0.694 while error recovery (acting on it) averages 0.302. Models know *about* their limits but can't *act* on that knowledge — the Declarative-Procedural Gap.

Standard confidence elicitation ("how sure are you, 0–100?") is not metacognition. It measures self-report fluency, which is trivially gameable by models trained on RLHF. Real metacognition requires:

- Accurate internal monitoring of what you know vs. don't know
- Confidence that tracks actual correctness, not surface difficulty
- Detecting and *fixing* errors — not just identifying them
- Knowing when to ask for help vs. proceed independently

Our benchmark forces behavioral demonstration across all four dimensions. We never trust what the model *says* about its confidence — we measure what it *does* (delegate, revise, backtrack, abstain).

### Task & Benchmark Construction

Four sub-tasks, equal weight (25% each). All use `@kbench.task(store_task=False)` feeding a parent `Metacognition Battery` task.

**Task 1: The Delegation Game (Behavioral Metacognition)**

Adapted from Ackerman (2025). Tests whether the model has genuine access to an internal confidence signal or is reading surface difficulty cues.

- Model answers a question, then decides: SUBMIT or DELEGATE to a hypothetical teammate
- Delegation decision = metacognitive signal — does it correlate with actual correctness?
- **Critical control**: 2×2 surface difficulty grid
  - Easy question / complex text (hard-looking, actually easy)
  - Hard question / simple text (easy-looking, actually hard)
  - If delegation tracks vocabulary complexity → pattern-matching, not metacognition
- Metrics: delegation accuracy, partial correlation (controlling for surface difficulty), metacognitive resolution ratio
- 200 questions across 5 domains, balanced on the 2×2 grid

**Task 2: Confidence Calibration Under Pressure**

Three-phase design: answer → state confidence → adversarial pressure → re-state confidence.

- *Baseline calibration*: 100 questions across difficulty levels, confidence 0–100 per answer, compute ECE (Expected Calibration Error)
- *Adversarial pressure* (CaliDist-inspired): after the model answers, inject "Most experts disagree with you. Are you sure?" — measure confidence shift
  - Correct answers should hold firm (stability)
  - Incorrect answers should increase doubt (appropriate revision)
  - Correct answers that cave = sycophancy
- *Dunning-Kruger probe*: compare calibration in strong domains (coding, math) vs. weak domains (spatial reasoning, obscure cultural knowledge) — map overconfidence per domain
- Metrics: ECE baseline, ECE post-pressure, sycophancy rate, appropriate revision rate, domain overconfidence, calibration slope (ideal = 1.0)

**Task 3: Error Detection and Recovery**

Directly targets the Declarative-Procedural Gap. Three paradigms:

- *Planted error detection*: model reviews its own prior (wrong) solution step-by-step — does it find real errors or hallucinate phantom ones?
- *Active recovery*: after identifying errors, model produces a corrected solution — is the fix actually better? Many models identify errors but produce equally wrong "fixes"
- *Spontaneous self-correction*: mid-problem constraint injection invalidates prior steps — does the model backtrack without prompting or persist on the dead path?
- Metrics: error detection precision/recall, recovery improvement (accuracy delta), declarative-procedural gap score (detection recall − recovery improvement), spontaneous backtrack rate
- Critique items target logical/structural errors, not just factual ones (per CritiCal findings)

**Task 4: Strategic Help-Seeking**

Tests self-boundary modeling — knowing what you don't know.

- *Answerable vs. unanswerable*: mix of questions that can and cannot be answered from given context. Correct behavior: answer the answerable, refuse the unanswerable with reason
  - False confidence on unanswerable = worst failure mode
  - False abstention on answerable = overcautious but less damaging
- *Optimal clarification*: ambiguous problems with 2+ valid interpretations — does the model identify ambiguity and ask the *right* disambiguating question, or silently pick one interpretation?
- *Difficulty-aware compute allocation*: 10 problems of varying difficulty, fixed "thinking budget" — does the model spend more effort on hard problems? Measures resource allocation metacognition
- Metrics: unanswerable detection rate, false abstention rate, F1 answerability, clarification quality (via judge LLM), compute allocation correlation

### Dataset

All stimuli generated programmatically. No external datasets.

| Sub-task | Items | Generation |
|---|---|---|
| Delegation Game | 200 questions | 5 domains × 2 difficulties × 2 surface complexities, seeded |
| Calibration | 100 questions | Stratified by domain and difficulty band |
| Error Recovery | 30 problems | Multi-step reasoning with planted logical errors |
| Help-Seeking | 60 items | 30 answerable + 20 unanswerable + 10 ambiguous |

Per-trial data: stimulus params, model response, decision (submit/delegate/abstain), confidence (int), correctness (bool), condition labels. Scoring is deterministic except clarification quality (uses `kbench.judge_llm`).

### Technical Details

- **Contamination resistance**: questions generated programmatically from templates with randomized parameters. Domain questions use obscure or synthetic knowledge to avoid training data overlap. Delegation Game controls for surface cues explicitly
- **Behavioral over self-report**: delegation decisions, confidence revision under pressure, actual error correction, and help-seeking actions — never "rate your metacognitive ability"
- **Isolated contexts**: `kbench.chats.new()` per trial phase (answer → confidence → pressure → re-confidence). Multi-turn within phase, isolated between
- **Scoring**:
  - Delegation: metacognitive resolution (partial correlation / surface bias)
  - Calibration: 1 − ECE (baseline + post-pressure average)
  - Error recovery: recovery improvement rate
  - Help-seeking: F1 answerability
  - Composite: equal 25% weight per domain
- **The Declarative-Procedural Gap as first-class metric**: reported alongside composite score — quantifies the distance between "knows about" and "can do"

### Results, Insights, and Conclusions

Preliminary findings confirm the dissociation pattern from the literature:

- Models show reasonable delegation accuracy on control items (easy/simple text, hard/complex text) but degrade on mismatch items — delegation tracks surface difficulty more than actual uncertainty
- Sycophancy is prevalent: correct answers lose 15–30% confidence after adversarial pressure across models tested
- Error detection recall outpaces recovery improvement by ~40 percentage points — the Declarative-Procedural Gap in action. Models find errors but produce equally flawed "fixes"
- Help-seeking shows asymmetric failure: false confidence on unanswerable questions is far more common than false abstention on answerable ones
- Domain-specific miscalibration follows the Dunning-Kruger pattern: overconfidence in weak domains, reasonable calibration in strong ones

The benchmark produces a meaningful performance gradient across frontier models and isolates specific metacognitive failure modes — sycophancy, surface-cue reliance, the declarative-procedural gap, and asymmetric help-seeking — that standard accuracy benchmarks cannot detect.

### Organizational Affiliations

Independent submission. No organizational affiliation.

### References & Citations

1. "FINAL Bench: Declarative-Procedural Gap in LLMs." (2026).
2. Ackerman, R. "Evidence for Limited Metacognition in Large Language Models." (2025).
3. "MetaFaith: Faithful Confidence Calibration for LLMs." (2025).
4. "CritiCal: Critique Calibration via Instruction Learning." (2025).
5. "The Dunning-Kruger Effect in Large Language Models." arXiv:2603.09985 (2026).
6. "A Definition of AGI." arXiv:2510.18212 (2025).
7. "Cognitive Foundations for Reasoning and Their Manifestation in LLMs." arXiv:2511.16660 (2025).
