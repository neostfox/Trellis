---
name: bayesian-thinking
description: >
  Systematic Bayesian reasoning for decision-making under uncertainty. Use when the user says
  "Bayesian analysis", "贝叶斯", "概率推理", "prior/posterior", "更新信念",
  "under uncertainty", "evidence-based reasoning", "credence updating",
  "what's the probability", "how confident should we be", or needs to make
  decisions with incomplete information, weigh competing hypotheses, or update
  beliefs based on new evidence. Also triggers on "conditional probability",
  "似然比", "贝叶斯更新", "先验概率", "后验概率", "假设检验",
  "which hypothesis is more likely", "conflicting evidence", or any situation
  where multiple explanations compete and evidence must be weighed.
license: MIT
metadata:
  author: oh-my-openclaw
  version: "1.0"
  composed_from:
    - "Eliezer Yudkowsky — An Intuitive Explanation of Bayes' Theorem"
    - "Julia Galef — The Scout Mindset (Bayesian updating as a thinking tool)"
    - "LessWrong Bayesian reasoning sequences"
    - "诺贝尔经济学奖 Daniel Kahneman — Thinking, Fast and Slow (锚定效应、基础概率忽视)"
---

# Bayesian Thinking

A systematic approach to reasoning under uncertainty — updating beliefs proportionally to evidence strength, quantifying confidence, and making calibrated decisions when information is incomplete.

## When to Use

- Multiple competing hypotheses exist and evidence is ambiguous
- Root cause analysis with uncertain diagnostic signals
- Architecture/design tradeoffs where outcomes are probabilistic
- Evaluating whether a fix actually addresses the bug or is a coincidence
- Estimating confidence in a technical decision before committing resources
- Debates where both sides have valid points and the right answer depends on context probabilities

## When NOT to Use

- Deterministic problems with clear right/wrong answers
- Time-critical emergencies (act first, analyze later)
- When all relevant information is already known
- Trivial decisions (use Occam's Razor)

---

## Core Methodology: 6 Phases

### Phase 0: Frame the Decision — Hypothesis Space

Before analyzing anything, enumerate the competing hypotheses.

**Hypothesis test**: Each hypothesis must be:
- Falsifiable (there exists evidence that would disprove it)
- Mutually exclusive (hypotheses cannot all be true simultaneously)
- Collectively exhaustive (together they cover the possibility space)

**Gate**: Must produce ≥2 competing hypotheses. Each stated in one sentence with initial plausibility.

### Phase 1: Establish Priors — What We Already Believe

Assign prior probabilities to each hypothesis based on existing knowledge.

**Prior sources (in order of reliability):**
1. Base rates from historical data (past bugs in this area, known failure modes)
2. Domain expertise (established architectural constraints)
3. Structural reasoning (what must be true by design)
4. Default: uniform prior (no reason to favor any hypothesis)

**Gate**: Must produce prior probabilities that sum to ≤1. Each prior justified by source.

### Phase 2: Collect Evidence — Observations and Signals

Gather evidence that differentially supports hypotheses.

**Evidence quality hierarchy:**
1. Reproducible experiment / deterministic test
2. Runtime observation (logs, metrics, stack traces)
3. Code inspection (static analysis, code path tracing)
4. Analogical reasoning (similar systems behave similarly)
5. Expert opinion / documentation

**For each piece of evidence:**
- State the observation
- Compute likelihood: how probable is this observation IF hypothesis H is true vs IF H is false?
- Record the likelihood ratio

**Gate**: Must produce ≥3 pieces of evidence with likelihood assessments.

### Phase 3: Update Beliefs — Apply Bayes' Theorem

Update prior probabilities using collected evidence.

**Bayes' Theorem (practical form):**
P(H|E) = P(E|H) × P(H) / P(E)

Where:
- P(H) = prior probability of hypothesis
- P(E|H) = likelihood of evidence given hypothesis
- P(E) = total probability of evidence (across all hypotheses)
- P(H|E) = posterior probability (updated belief)

**Sequential updating**: Process evidence in order of strength (strongest first). After each update, the posterior becomes the new prior.

**Gate**: Must produce posterior probabilities for all hypotheses. Posteriors must sum to 1.

### Phase 4: Decide Under Uncertainty — Calibrated Action

Translate belief distribution into action, calibrated to confidence level.

**Confidence → Action mapping:**
| Posterior | Confidence | Action |
|----------|-----------|--------|
| >95% | Very High | Proceed decisively; commit resources |
| 80-95% | High | Proceed but build verification checkpoint |
| 60-80% | Moderate | Proceed tentatively; plan rollback; seek more evidence |
| 40-60% | Low | Do NOT commit; seek discriminating evidence first |
| <40% | Very Low | Reject or radically reframe the hypothesis |

**Gate**: Must produce an action recommendation with explicit confidence level and stopping condition.

### Phase 5: Review and Calibrate — Post-Hoc Analysis

After the outcome is known, check calibration.

**Three calibration questions:**
1. Was the posterior prediction accurate? (Did the most probable hypothesis turn out correct?)
2. Was any strong evidence missed during collection? (Information gap analysis)
3. Were the priors well-calibrated? (Overconfident? Underconfident?)

**Gate**: Answer all 3 questions. Identify at least one calibration improvement.

---

## Reasoning Discipline Protocol

### Phase Gates (Mandatory Artifacts)

| Phase | Must Produce | Min Depth |
|-------|-------------|-----------|
| 0: Frame | ≥2 hypotheses with initial plausibility | Each falsifiable and mutually exclusive |
| 1: Priors | Prior probabilities with justifications | Sum ≤1, each justified by source |
| 2: Evidence | ≥3 evidence items with likelihood ratios | Each: observation + P(E\|H) assessment |
| 3: Update | Posterior probabilities | Sum = 1, sequential updating shown |
| 4: Decide | Action recommendation + confidence | Explicit confidence % + stopping condition |
| 5: Calibrate | 3 calibration answers | At least 1 improvement identified |

**No artifact → no next phase.** If a gate is not met, stop and complete it.

### Progress Tracker (Anti-Drift)

```markdown
## 🧭 Bayesian Progress
- [x] Phase 0: Frame — ✅ 3 hypotheses
- [x] Phase 1: Priors — ✅ H1=60%, H2=30%, H3=10%
- [→] Phase 2: Evidence — 2/4 collected
- [ ] Phase 3: Update
- [ ] Phase 4: Decide
- [ ] Phase 5: Calibrate
```

**If conversation drifts** (user asks a tangent, discussion expands on a side topic), after addressing it, immediately output:

> 📍 Returning to Bayesian analysis: Phase N has M items remaining. Continuing.

### Common Bayesian Errors to Avoid

| Error | What It Looks Like | Fix |
|-------|-------------------|-----|
| Base rate neglect | Ignoring how common a bug type is | Always start from base rates |
| Anchoring on prior | Refusing to update despite strong evidence | Let likelihood ratios dominate |
| Confirmation bias | Only seeking evidence for preferred hypothesis | Actively seek disconfirming evidence |
| Overconfidence | Assigning 99% to a hypothesis with limited evidence | Use wider prior distributions |
| Neglect of alternative | Focusing on one hypothesis exclusively | Maintain full hypothesis space |
| Posterior overconfidence | Treating updated belief as certainty | Always state confidence interval |

---

## Trellis Integration

When used within a Trellis-managed project, the analysis artifacts integrate with the task system.

### File Placement

```
.trellis/tasks/{MM-DD-slug}/
├── bayesian-analysis.md         # ← Bayesian analysis output (Phases 0-5)
├── implement.jsonl              # ← bayesian-analysis.md auto-added
├── check.jsonl                  # ← bayesian-analysis.md auto-added
└── ...
```

### Brainstorm Integration

During `trellis-brainstorm`, when the task involves:
- Multiple competing approaches with unclear evidence
- Root cause analysis under uncertainty
- Risk assessment with incomplete data

Trigger Bayesian analysis as complement to first-principles:
1. FP Phase 2 (Assumptions) can feed into Bayesian Phase 1 (Priors)
2. Bayesian posteriors can inform FP Phase 4 (Reason Upward)

### Context Injection

After Bayesian analysis completes, add to context files:

```bash
python3 ./.trellis/scripts/task.py add-context "$TASK_DIR" implement "bayesian-analysis.md" "Hypothesis probabilities and evidence weights"
python3 ./.trellis/scripts/task.py add-context "$TASK_DIR" check "bayesian-analysis.md" "Verify implementation addresses highest-probability hypothesis"
```

---

## Output Format

When applying Bayesian thinking, structure the final output as:

```markdown
## Bayesian Analysis: [Topic]

### Hypotheses
1. H1: [description] — Prior: [X%] — Posterior: [Y%]
2. H2: [description] — Prior: [X%] — Posterior: [Y%]
3. H3: [description] — Prior: [X%] — Posterior: [Y%]

### Evidence Assessment
|Evidence|P(E\|H1)|P(E\|H2)|P(E\|H3)|Weight|
|---|---|---|---|---|
|[obs 1]|...|...|...|Strong/Moderate/Weak|

### Belief Updates
Prior → After Evidence 1 → After Evidence 2 → After Evidence 3 = Final Posterior

### Decision
- Recommended action: [action]
- Confidence: [X%]
- Stopping condition: [when to reconsider]
- Key risk: [what could change the posterior significantly]

### Calibration Review
- [review notes]
```

---

## Complementary Thinking Models

| Model | When to Use Instead | How It Complements |
|-------|--------------------|--------------------|
| First Principles | When you need to decompose to axioms, not weigh hypotheses | FP establishes ground truths → Bayesian quantifies confidence in conclusions |
| Pre-Mortem | When you need to find failure modes, not rank hypotheses | Pre-mortem generates hypotheses → Bayesian ranks them |
| OODA Loop | When speed matters more than precision | OODA for rapid iteration → Bayesian for periodic recalibration |

> Full model toolkit: first-principles-thinking references/thinking-models-toolkit.md

---

## References

| File | Content | When to Use |
|------|---------|-------------|
| `references/bayes-theorem-practical.md` | Practical Bayes' theorem with programming examples | When you need to compute actual likelihood ratios or sequential updates |
| `references/cognitive-biases-bayesian.md` | Cognitive biases that Bayesian thinking counteracts | When you suspect reasoning is distorted by heuristics |
| `references/decision-calibration.md` | Confidence calibration techniques and Brier Score | When you need to assess or improve prediction accuracy |
