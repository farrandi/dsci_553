# Lecture 2: Conditional Probability, Bayes' Theorem, and Maximum a Posteriori Estimation

## Conditional Probability

$$P(A|B) = \frac{P(A \cap B)}{P(B)}$$

- $P(A^c|B) = 1 - P(A|B)$
- IF $A$ and $B$ are independent, then $P(A|B) = P(A)$
  - So $P(B|A) = P(B)$

## Bayes' Theorem

- **Prior**: $P(A)$
  - $P(A^c) = 1 - P(A)$
- **Conditional Probability**: $P(B|A)$ and $P(B|A^c)$
  - Likelihood: $\ell(A|B) = P(B|A)$
    - Means likelihood of $A$ given $B$
- **Posterior** (what we want to find): $P(A|B)$

$$P(A|B) = \frac{P(A \cap B)}{P(B)} = \frac{P(A)P(B|A)}{P(A)P(B|A) + P(A^c)P(B|A^c)}$$
$$\text{posterior} = \frac{\text{prior} \times \text{likelihood}}{\text{normalization constant}}$$

$$\text{posterior} \propto \text{prior} \times \text{likelihood}$$

- Once we have the **posterior**, we have everything we need to make decisions.

### Bayesian Inference

- **Properties**:
  1. Hidden variables of interest are random (prior distribution)
  2. Use **posterior** (conditional distribution of hidden variables given observation) to capture uncertainty
- E.g. posterior: $P(A|B) = 0.3$,
  - There is a 30% chance of $A$ if $B$ is true.

## Maximum a Posteriori Estimation (MAP) and Maximum Likelihood Estimation (MLE)

- MAP is a Bayesian approach to MLE

| MLE                                                               | MAP                                                               |
| ----------------------------------------------------------------- | ----------------------------------------------------------------- |
| Finding value that maximizes **likelihood**                       | Finding value that maximizes **posterior**                        |
| Only uses observed data                                           | Uses observed data and **prior**                                  |
| $\hat{\theta}_{\text{MLE}} = \text{argmax}_{\theta} P(D\|\theta)$ | $\hat{\theta}_{\text{MAP}} = \text{argmax}_{\theta} P(\theta\|D)$ |
