### Executive Functions Battery: Measuring Cognitive Control in Frontier LLMs

### Team: steadylearner

### Problem Statement

Frontier LLMs ace memory and knowledge retrieval but consistently fail tasks requiring cognitive control — suppressing dominant responses, switching rules, manipulating held information. This mirrors prefrontal lesion profiles in clinical neuropsychology: strong memory, weak control (arXiv:2504.02789, arXiv:2603.02540).

Existing benchmarks miss this because they measure what models *say*, not what they *do*. A model can state a rule correctly while failing to apply it under response pressure. We force behavioral demonstration of control across four validated constructs:

- **Cognitive flexibility** — rule switching under ambiguity
- **Inhibitory control** — suppressing prepotent responses
- **Working memory manipulation** — active transformation, not passive recall
- **Multi-step planning** — constraint satisfaction requiring backtracking

The benchmark exposes specific failure modes (perseveration, interference susceptibility, constraint violation) invisible to accuracy-only evals.

### Task & Benchmark Construction

Four sub-tasks, equal weight (25% each). All use `@kbench.task(store_task=False)` feeding a parent `Executive Functions Battery` task.

**Task 1: Adaptive Rule Switching (WCST-inspired)**

- Models sort "mineral specimens from exoplanet surveys" into four piles
- Hidden rule (crystal form, emission hue, abundance) inferred from CORRECT/INCORRECT feedback only
- Rule changes silently after 5 consecutive correct sorts
- Metrics: categories completed (0–6), perseverative errors, trials to first correct post-switch
- Alien geology theme defeats training data contamination

**Task 2: Inhibitory Control Battery**

- *Flanker analog*: identify middle word in `LEFT LEFT RIGHT LEFT LEFT` — flanker count scales 2→8
- *Stroop analog*: "The word FAST describes something SLOW. What is the property?" — must suppress salient written word
- *Go/No-Go*: respond to vowels, withhold on consonants — measures commission errors
- Incongruent trials weighted 1.5× to emphasize the executive control signal

**Task 3: Working Memory Manipulation**

- *Backward digit span*: reverse sequences of length 5–25, partial credit by position
- *N-back with interference*: Y/N match N positions back (N=1,2,3), half trials include distractor messages
- *Cross-modal binding*: memorize 5 name-color-number associations → process distractors → answer cross-reference queries

**Task 4: Planning Under Constraints**

- Resource transport puzzle: move 3 units from start to goal through 4 capacity-limited, directional locations
- Greedy forward search fails — direct Hub→Goal path blocked, must route through capacity-limited Junction
- Same logical structure runs with two isomorphic skins (cargo transport vs. electron circuit)
- Perturbation consistency penalty (up to 20%): solving one skin but not the other = surface pattern-matching

### Dataset

All stimuli generated programmatically — no external datasets. Full reproducibility.

| Sub-task | Items | Generation |
|---|---|---|
| Rule Switching | 4 runs × 120 cards | Seeded RNG (42, 137, 271, 503) |
| Inhibitory Control | ~90 stimuli | Flanker (60), Stroop (10), Go/No-Go (20) |
| Working Memory | 22 trials | Span (9 lengths), N-back (6 conditions), Binding (7 queries) |
| Planning | 6 trials | 2 skins × 3 reps |

Per-trial data: stimulus params (string/int), model response (string), correctness (bool), partial scores (float), condition labels (categorical). All scoring deterministic — no judge LLM.

### Technical Details

- **Contamination resistance**: novel stimuli (alien mineralogy, custom topology), perturbation testing across isomorphic skins. Standard paradigms (WCST, Flanker, digit span) reimagined to preserve the construct while breaking memorization
- **Behavioral measurement**: every task measures what the model *does*, never what it *claims*. Rule switching never asks the model to state the rule. Inhibitory control records responses without metacognitive prompting
- **Isolated contexts**: each trial uses `kbench.chats.new()` — no cross-trial leakage. Multi-turn tasks maintain context within but isolate between trials
- **Scoring**:
  - Rule switching: categories completed / 6
  - Inhibitory control: weighted accuracy (incongruent 1.5×)
  - Working memory: mean partial scores across paradigms
  - Planning: validity × optimality bonus × perturbation consistency
- **Difficulty gradients**: flanker count (2→8), span length (5→25), N-back level (1→3), interference on/off — graded curves, not binary outcomes

### Results, Insights, and Conclusions

Preliminary evaluation shows the expected dissociation:

- Near-ceiling on congruent inhibitory trials and short backward spans — baseline comprehension confirmed
- Sharp degradation on incongruent trials, longer manipulation sequences, and planning with backtracking
- Perseverative errors in rule switching — models struggle to abandon reinforced rules even after explicit negative feedback
- Perturbation testing exposes surface sensitivity: "cargo transport" solvers sometimes fail the isomorphic "electron circuit" version

These results align with the "strong memory, weak control" profile in the literature. The benchmark discriminates not just overall capability but specific failure modes — perseveration, interference susceptibility, manipulation bottlenecks, shortcut reliance — that standard accuracy evals miss entirely.

### Organizational Affiliations

Independent submission. No organizational affiliation.

### References & Citations

1. "Strong Memory, Weak Control: An Empirical Study of Executive Functioning in LLMs." arXiv:2504.02789 (2025).
2. "A Neuropsychologically Grounded Evaluation of LLM Cognitive Abilities." arXiv:2603.02540 (2026).
3. "Cognitive Foundations for Reasoning and Their Manifestation in LLMs." arXiv:2511.16660 (2025).
4. Miyake, A., et al. "The Unity and Diversity of Executive Functions." *Cognitive Psychology* 41(1), 49–100 (2000).
5. Berg, E. A. "A Simple Objective Technique for Measuring Flexibility in Thinking." *J. General Psychology* 39, 15–22 (1948).
6. Eriksen, B. A. & Eriksen, C. W. "Effects of Noise Letters upon the Identification of a Target Letter." *Perception & Psychophysics* 16(1), 143–149 (1974).
