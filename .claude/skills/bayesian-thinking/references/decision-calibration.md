# Decision Calibration — Measuring and Improving Prediction Accuracy

Calibration means your confidence matches your accuracy. If you say "80% confident" across 100 predictions, about 80 should be correct. Most people are poorly calibrated — they're right much less often than their confidence suggests.

---

## Calibration Curve

### What It Shows

A calibration curve plots predicted probability (x-axis) against observed frequency (y-axis). Perfect calibration = diagonal line.

```
Observed %
100 ┤                                    ╭───────
    │                              ╭─────╯
 80 ┤                        ╭─────╯
    │                  ╭─────╯       ╰──── Overconfident
 60 ┤            ╭─────╯                  (predicted > actual)
    │      ╭─────╯
 40 ┤╭─────╯
    │╯
 20 ┤  Underconfident
    │   (predicted < actual)
  0 ┤──────┬──────┬──────┬──────┬──────┬──────
    0     20     40     60     80    100   Predicted %
```

### How to Build One

1. Record each prediction with a confidence percentage
2. After outcomes are known, group predictions by confidence bucket (e.g., all 70-80% predictions)
3. Calculate actual hit rate per bucket
4. Plot predicted vs actual

### Reading the Curve

- **On the diagonal**: Well-calibrated. Your confidence matches reality.
- **Above diagonal**: Underconfident. You're right more often than you think.
- **Below diagonal**: Overconfident. You're wrong more often than you think. (This is the most common pattern for engineers.)

---

## Brier Score

### Formula

```
Brier Score = (1/N) × Σ (predicted_prob - actual_outcome)²
```

Where:
- `predicted_prob` = your predicted probability (0 to 1)
- `actual_outcome` = 1 if event happened, 0 if not
- `N` = number of predictions

### Interpretation

| Score | Meaning |
|-------|---------|
| 0.0 | Perfect prediction |
| 0.25 | Random guessing (coin flip) on binary events |
| 0.33 | Always predicting the base rate |
| >0.25 | Worse than random — something is systematically wrong |

### Example Calculation

5 predictions about which hypothesis is correct:

| # | Predicted | Actual | (p - a)² |
|---|-----------|--------|-----------|
| 1 | 0.8 | 1 (correct) | 0.04 |
| 2 | 0.6 | 0 (wrong) | 0.36 |
| 3 | 0.7 | 1 (correct) | 0.09 |
| 4 | 0.9 | 1 (correct) | 0.01 |
| 5 | 0.5 | 0 (wrong) | 0.25 |

Brier Score = (0.04 + 0.36 + 0.09 + 0.01 + 0.25) / 5 = 0.15

This is better than random (0.25) but shows room for improvement — predictions 2 and 5 were wrong.

---

## Calibration Exercises

### Exercise 1: Bug Diagnosis Log

After each bug investigation, record:

```
Date: 2024-01-15
Issue: API returning 500 on /users endpoint
Hypotheses + Priors:
  H1: Database connection pool exhaustion (60%)
  H2: Invalid input validation edge case (30%)
  H3: Authentication token expiry (10%)
Evidence collected + likelihoods: [record]
Final posteriors:
  H1: 85%, H2: 10%, H3: 5%
Actual cause: H1 (connection pool exhaustion)
Calibration: 85% predicted → correct ✓
```

After 20+ entries, compute your Brier Score and build a calibration curve.

### Exercise 2: Architecture Decision Review

For each architecture decision with >70% confidence, set a calendar reminder for 3 months later:

1. Did the chosen approach work as expected?
2. Were the predicted benefits realized?
3. Were the predicted costs accurate?
4. What was the actual outcome vs predicted?

### Exercise 3: Pre-Mortem Calibration

Before implementation, state:
- "I'm X% confident this approach will work without major revision"
- "I'm X% confident we'll finish within the estimated timeline"
- "I'm X% confident no critical edge case was missed"

After completion, compare predictions to reality.

---

## Common Calibration Patterns in Software Engineering

### Overconfidence in Bug Diagnosis

Engineers typically show 85-95% confidence in their diagnosis, but are correct only 60-70% of the time.

**Fix**: Before stating high confidence, explicitly list 2 alternative hypotheses you haven't ruled out.

### Overconfidence in Time Estimates

The planning fallacy: engineers estimate based on best-case scenarios, ignoring historical base rates.

**Fix**: Use base rates from past similar tasks as priors. If the last 5 similar tasks took 3-5 days, your prior for "1 day" should be very low.

### Underconfidence in Novel Situations

When facing unfamiliar problems, engineers often assign near-uniform priors (50/50 between two options) even when structural reasoning favors one.

**Fix**: Use structural priors even without historical data. If option A violates fewer constraints than option B, it deserves a higher prior.

---

## Calibration Improvement Protocol

### Weekly Review (5 minutes)

1. Look at last week's high-confidence decisions (≥80%)
2. How many were correct?
3. If <70% correct: you're overconfident → widen priors
4. If >90% correct: you might be underconfident → trust your analysis more

### Monthly Review (20 minutes)

1. Compute Brier Score for the month
2. Build calibration curve
3. Identify systematic bias direction
4. Adjust prior-setting heuristics

### Calibration Targets

| Experience Level | Target Brier Score | Target Calibration |
|-----------------|-------------------|-------------------|
| Beginner | <0.20 | Within 15% of diagonal |
| Intermediate | <0.15 | Within 10% of diagonal |
| Expert | <0.10 | Within 5% of diagonal |

---

## Integration with Bayesian Phases

| Phase | Calibration Check |
|-------|------------------|
| Phase 1: Priors | "Are my priors consistent with historical base rates?" |
| Phase 3: Update | "Are my posteriors consistent with the evidence strength?" |
| Phase 4: Decide | "Does my confidence match what the posterior actually says?" |
| Phase 5: Calibrate | "Was my prediction accurate? What can I improve?" |
