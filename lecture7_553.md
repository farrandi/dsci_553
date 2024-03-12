# Lecture 7: Bayesian Hierarchical Models

## Complete Pooled Model

- **Complete Pooled Model**: A model that pools all the data together and estimates a single parameter for all the data

  - e.g. have a dataset of multiple rocket types and their launchs, and want to estimate the probability of a rocket launch succeeding

    - Only 1 $\pi$ for all the rocket types

$$
\text{likelihood:} \qquad X_i|\pi \sim \text{Binomial}(n_i, \pi) \quad \text{for } i = 1, \dots, 367\\
\text{prior:} \qquad \pi \sim \text{Beta}(a = 1, b = 1).
$$

- Does not allow us to infer any probability of an individual group

## Non-pooled Model

- **Non-pooled Model**: A model that estimates a parameter for each group of data
  - e.g. have a dataset of multiple rocket types and their launchs, and want to estimate the probability of a rocket launch succeeding
    - Have a $\pi_i$ for each rocket type

$$
\text{likelihood:} \qquad X_i|\pi_i \sim \text{Binomial}(n_i, \pi_i) \quad \text{for } i = 1, \dots, 367\\
\text{prior:} \qquad \pi_i \sim \text{Beta}(a = 1, b = 1).
$$

- Better when comparing groups with different sample sizes
  - Using MLE will not be reliable when sample sizes are small
  - e.g. Binomial distribution with $n=1$
- **Drawbacks**:
  - Cannot generalize to new groups
  - Cannot take valuable information from other groups

## Hierarchical Bayesian Model

- **Hierarchical Bayesian Model**: A model that estimates a parameter for each group of data, but also estimates a distribution of parameters for all the groups

  - e.g. have a dataset of multiple rocket types and their launchs, and want to estimate the probability of a rocket launch succeeding
    - Have a $\pi_i$ for each rocket type
    - Also have random variables $\pi_i \sim \text{Beta}(a, b)$
      - $a$ and $b$ are parameters and no longer hyperparameters
      - new hyperparameters are the priors for $a$ and $b$

$$
\text{likelihood:} \qquad X_i|\pi_i \sim \text{Binomial}(n_i, \pi_i) \quad \text{for } i = 1, \dots, 367\\
\text{priors:} \qquad \pi_i \sim \text{Beta}(a, b) \\
\quad a \sim \text{Gamma}(0.001, 0.001) \\
\qquad b \sim \text{Gamma}(0.001, 0.001).
$$

- Basically **combines the best** of of the complete pooled model and the non-pooled model:
  - Use valuable info from all groups to infer the success probability of a specific group
  - Can get posterior predictive distribution for new group

### Why Gamma Prior?

- For the $a$ and $b$ we need continous and non-negative priors
- Hence, we use the Gamma distribution
  \_ this tends to assign **small values** to $a$ and $b$ which is good for the prior

### Why will the estimates be more precise?

- Using Heirarchical Bayesian, we will get narrower Credible Intervals compared to the non-pooled model
- **Borrowing Strength**: The estimates will be more precise because we are using information from all the groups
  - In the case of rockets, its the $a$ and $b$ that are being shared
- Helps learn parameters and reduce posterior variance

### Get Success for all Groups

- Use posterior means of $a$ and $b$ to get the posterior means of $\pi_i$ for all groups (in rocket example)

### Prediction for New Group

- In Stan, need to add a new block `generated quantities` to get the posterior predictive distribution for a new group

```stan
generated quantities {
  real<lower=0,upper=1> pi_pred = beta_rng(a, b);
}
```

- This generates a new $\pi$ for a new group from the posterior distribution of $a$ and $b$
- Mean of `pi_pred` (new group) is similar to the posterior mean of $\pi$ for all groups
  - because it is the best bayesian model we can obtain without further covariates/ features
