# Bayes' Theorem — Practical Application Guide

## The Theorem

### Intuitive Version

Bayes' Theorem answers: **"Given what I just observed, how should I update my belief?"**

It's a ratio: how compatible is the evidence with each hypothesis, weighted by how likely each hypothesis was before you saw the evidence.

### Mathematical Form

```
P(H|E) = P(E|H) × P(H) / P(E)
```

Where:
- `P(H)` = prior (how likely was H before seeing evidence?)
- `P(E|H)` = likelihood (how expected is this evidence if H is true?)
- `P(E)` = marginal likelihood (how expected is this evidence overall?)
- `P(H|E)` = posterior (how likely is H now, after seeing evidence?)

### Odds Form (for two hypotheses)

When comparing H1 vs H2:

```
Posterior Odds = Prior Odds × Likelihood Ratio

O(H1:H2 | E) = O(H1:H2) × P(E|H1) / P(E|H2)
```

The odds form is often easier to compute: multiply prior odds by the likelihood ratio.

---

## Programming Scenarios

### Scenario 1: Bug Diagnosis

**Situation**: A flaky test fails intermittently. Three possible causes.

| Hypothesis | Prior | Reasoning |
|------------|-------|-----------|
| H1: Race condition | 50% | Most common cause of flaky tests in this codebase (historical base rate) |
| H2: External API timeout | 30% | Third-party dependency, known to be slow |
| H3: Test setup bug | 20% | Less common but happens |

**Evidence 1**: Test fails more often under load.

Likelihood assessment:
- P(E1|H1) = 0.8 (race conditions worsen under contention)
- P(E1|H2) = 0.4 (API timeouts somewhat load-related)
- P(E1|H3) = 0.1 (test setup bugs usually consistent)

Likelihood ratio H1:H2 = 0.8/0.4 = 2:1 (favors H1)
Likelihood ratio H1:H3 = 0.8/0.1 = 8:1 (strongly favors H1)

**Evidence 2**: Adding `sleep(100)` between setup and assertion reduces failure rate.

Likelihood assessment:
- P(E2|H1) = 0.9 (timing fix = classic race condition indicator)
- P(E2|H2) = 0.2 (sleep doesn't help API timeouts much)
- P(E2|H3) = 0.3 (might help if setup needs time, but unlikely)

**Sequential Update** (using odds form, starting from prior):

After E1:
- H1: 50% × 0.8 / P(E1) → recalculated
- H2: 30% × 0.4 / P(E1) → recalculated
- H3: 20% × 0.1 / P(E1) → recalculated

After E2:
- H1 posterior ≈ 80% (race condition)
- H2 posterior ≈ 12%
- H3 posterior ≈ 8%

**Decision**: Fix the race condition (80% confidence). Add a synchronization primitive as verification checkpoint.

### Scenario 2: Performance Regression Attribution

**Situation**: API latency increased 40% after deployment.

| Hypothesis | Prior |
|------------|-------|
| H1: Database query plan change | 40% |
| H2: New feature added N+1 queries | 35% |
| H3: Network congestion | 25% |

**Evidence**: DB query log shows same query taking 3x longer. Network I/O bytes unchanged.

Likelihood:
- P(E|H1) = 0.9 (directly explains slow queries)
- P(E|H2) = 0.3 (N+1 would show more queries, not slower ones)
- P(E|H3) = 0.1 (network issues don't slow DB CPU time)

**Posterior**: H1 ≈ 78%, H2 ≈ 15%, H3 ≈ 7% → Investigate query plan.

### Scenario 3: Architecture Decision Under Uncertainty

**Situation**: Choosing between monolith and microservices for a new module.

| Hypothesis | Prior |
|------------|-------|
| H1: Monolith optimal | 60% |
| H2: Microservices optimal | 40% |

**Evidence**: Team is 3 people, no DevOps hire planned. Module has 2 external integrations.

Likelihood:
- P(E|H1) = 0.8 (small team → monolith simpler)
- P(E|H2) = 0.3 (microservices need DevOps overhead)

**Posterior**: H1 ≈ 80% → Proceed with monolith, design module boundaries for future extraction.

---

## Likelihood Ratio Reference Table

| Likelihood Ratio | Strength | Interpretation |
|-----------------|----------|---------------|
| 1:1 | None | Evidence doesn't discriminate |
| 2:1 to 5:1 | Weak | Slight preference |
| 5:1 to 10:1 | Moderate | Notable but not decisive |
| 10:1 to 100:1 | Strong | Compelling evidence |
| >100:1 | Very Strong | Near-conclusive |

## Sequential Update Algorithm

```python
def sequential_bayesian_update(priors: dict, evidence_list: list[dict]) -> dict:
    """
    priors: {H1: 0.5, H2: 0.3, H3: 0.2}
    evidence_list: [{H1: likelihood1, H2: likelihood2, H3: likelihood3}, ...]
    """
    posteriors = priors.copy()
    
    for evidence in evidence_list:
        # Calculate marginal likelihood P(E)
        p_e = sum(posteriors[h] * evidence[h] for h in posteriors)
        
        if p_e == 0:
            continue  # Avoid division by zero (evidence impossible under all hypotheses)
        
        # Update each hypothesis
        for h in posteriors:
            posteriors[h] = (evidence[h] * posteriors[h]) / p_e
    
    return posteriors
```

## Common Pitfalls

1. **Forgetting the denominator**: P(E) includes ALL hypotheses, not just the one you're evaluating.
2. **Double-counting evidence**: Don't use correlated evidence as independent updates.
3. **Prior too close to 0 or 1**: A prior of exactly 0 or 1 can never be updated (Cromwell's Rule).
4. **Ignoring base rates**: The most common cause is often the correct cause — don't let vivid evidence override solid base rates.
5. **Confusing P(E|H) with P(H|E)**: "The probability of seeing this evidence IF the hypothesis is true" ≠ "The probability the hypothesis is true given this evidence."
