# Cognitive Biases That Bayesian Thinking Counteracts

Bayesian reasoning provides systematic corrections for the most common cognitive distortions in technical decision-making.

---

## 1. Base Rate Neglect (基础概率忽视)

### What It Is
Ignoring the prior probability of an event and focusing only on the specific evidence. "This bug looks like a race condition" → assumes race condition, ignoring that race conditions account for only 5% of bugs in this codebase.

### Real-World Example
A test fails with a timeout error. The developer immediately assumes "network issue" because the error message mentions timeout. But historically, 80% of timeout errors in this codebase are caused by database locks, not network problems.

### Bayesian Fix
**Always start from base rates (Phase 1: Priors).** Before examining specific evidence, establish what's most common. The prior distribution anchors the analysis in reality.

**Explicit protocol**: Before collecting evidence, write down: "In the last 6 months, what fraction of similar issues were caused by each category?"

### Diagnostic Question
> "Am I treating this specific evidence as if it's the only information I have, or am I properly weighting it against what I already know?"

---

## 2. Anchoring Effect (锚定效应)

### What It Is
Over-weighting the first piece of information received. If the first diagnostic points to hypothesis A, subsequent evidence is interpreted through that lens.

### Real-World Example
A senior engineer says "this is probably a caching issue" in a meeting. The team then spends 4 hours investigating cache invalidation, even though the actual root cause is a database schema mismatch.

### Bayesian Fix
**Process evidence in order of strength, not arrival order (Phase 2).** Rank evidence by how well it discriminates between hypotheses (likelihood ratio), not by when you encountered it.

**Explicit protocol**: Collect ALL evidence first, THEN rank by likelihood ratio, THEN update sequentially from strongest to weakest.

### Diagnostic Question
> "If I had seen the evidence in reverse order, would I reach the same conclusion?"

---

## 3. Confirmation Bias (确认偏差)

### What It Is
Seeking and overweighting evidence that supports the preferred hypothesis, while ignoring or minimizing disconfirming evidence.

### Real-World Example
A developer believes the performance regression is caused by the new ORM version. They find one benchmark showing ORM overhead, but ignore three other benchmarks showing no difference and the fact that the regression is localized to one endpoint.

### Bayesian Fix
**Actively seek disconfirming evidence (Phase 2: Evidence collection).** For each hypothesis, ask: "What evidence would DISPROVE this?" Then go look for that evidence specifically.

**Explicit protocol**: For each hypothesis, generate at least one "disconfirmation probe" — a test or observation that would significantly lower the posterior if the hypothesis is false.

### Diagnostic Question
> "What evidence would make me change my mind? Have I actually looked for it?"

---

## 4. Overconfidence (过度自信)

### What It Is
Assigning probabilities that are too extreme (too close to 0% or 100%) given the actual strength of evidence. Studies show that when people say "90% confident," they're right only about 60-70% of the time.

### Real-World Example
After finding one log line that supports their theory, a developer declares "I'm 95% sure it's a memory leak." In reality, that log line is consistent with 3 other hypotheses.

### Bayesian Fix
**Use wider prior distributions (Phase 1).** Unless you have strong base rate data, avoid priors above 70% or below 10% for any single hypothesis. Let the evidence do the updating.

**Cromwell's Rule**: Never assign a prior of exactly 0 or 1 to any hypothesis. Leave room for surprise.

**Explicit protocol**: After producing posteriors, apply the "50% test" — would you be surprised if any hypothesis with >20% posterior turned out to be correct? If not, your posteriors may be overconfident.

### Diagnostic Question
> "If I had to bet real money on this, what odds would I actually accept? Am I as confident as my numbers suggest?"

---

## 5. Availability Heuristic (可得性启发)

### What It Is
Overweighting evidence that is recent, vivid, or easy to recall. A dramatic production incident last week gets disproportionate weight compared to months of stable operation.

### Real-World Example
The team had a bad experience with Redis cluster last month. Now, when evaluating caching options, they assign artificially high probability to "Redis cluster will fail again," despite the actual failure being caused by a configuration error that's been fixed.

### Bayesian Fix
**Use systematic evidence collection (Phase 2) with explicit quality hierarchy.** The evidence hierarchy (reproducible test > runtime observation > code inspection > analogy > opinion) helps discount vivid but low-quality evidence.

**Explicit protocol**: Rate each piece of evidence by its position in the quality hierarchy, not by how memorable it is.

### Diagnostic Question
> "Am I giving this evidence weight because it's strong, or because it's recent/memorable?"

---

## 6. Conjunction Fallacy (合取谬误)

### What It Is
Judging a specific combination of events as more probable than a more general one. "The bug is caused by a race condition in the payment handler" seems more likely than "The bug is caused by a race condition" — but it can't be, because the conjunction is a subset.

### Real-World Example
"This is definitely a memory leak in the WebSocket handler caused by improper cleanup of connection buffers" sounds more convincing than "There's a resource leak somewhere" — but the specific story is necessarily less probable than the general one.

### Bayesian Fix
**Maintain the full hypothesis space (Phase 0).** Ensure hypotheses are at the same level of specificity. If one hypothesis is a special case of another, they're not competitors — they're nested.

**Explicit protocol**: Check hypothesis mutual exclusivity. If H1 ⊂ H2, either split H2 into "H1 + everything else" or raise H1 to H2's level.

### Diagnostic Question
> "Is my favored hypothesis actually a special case of a simpler, more probable hypothesis?"

---

## Bias Interaction Map

```
                    Base Rate Neglect
                         │
                         ▼
    Anchoring ──→ Overconfidence ──→ Confirmation Bias
                         │                    │
                         ▼                    ▼
                 Availability Heuristic ←── Conjunction Fallacy
```

These biases reinforce each other. Bayesian thinking breaks the cycle by:
1. **Base rates** counter anchoring and availability
2. **Systematic evidence collection** counters confirmation bias
3. **Likelihood ratios** counter overconfidence and conjunction fallacy
4. **Sequential updating** makes the reasoning process auditable

---

## Quick Reference: Bias → Bayesian Phase Fix

| Bias | Which Phase Addresses It | How |
|------|------------------------|-----|
| Base rate neglect | Phase 1 (Priors) | Must state base rates before evidence |
| Anchoring | Phase 2 (Evidence) | Rank by likelihood ratio, not arrival order |
| Confirmation bias | Phase 2 (Evidence) | Must seek disconfirming evidence |
| Overconfidence | Phase 3 (Update) | Cromwell's Rule + wider priors |
| Availability | Phase 2 (Evidence) | Quality hierarchy discounts vivid but weak evidence |
| Conjunction fallacy | Phase 0 (Frame) | Mutual exclusivity check on hypotheses |
