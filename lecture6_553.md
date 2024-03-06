# Lecture 6: Bayesian Binary Logistic Regression

## Bayesian Binary Logistic Regression

- Comparable to [logistic regression](https://mds.farrandi.com/block_3/561_regression/561_regression#logistic-regression)

### The Likelihood

- Let $Y_i \in \{0, 1\}$
- Response is assumed as:
  $$Y_i | \beta_0 \beta_1 \sim \text{Bernoulli}(\pi_i) $$

with link function:

$$log(\frac{\pi_i}{1-\pi_i}) = \beta_0 + \beta_1 X_i$$

That is our likelihood.

### The Priors

- Parameter of interest: $\beta_0, \beta_1$
- Let us assume as follows:
  - $\beta_0 \sim N(\mu=0, \sigma^2=100^2)$
  - $\beta_1 \sim N(\mu=0, \sigma^2=100^2)$
- Assume 0 because we do not know if there is any association, also variace is high to reflect the uncertainty.

### Coding in Stan

#### Coding the Prior

- No need `warmup` and `thin` as it is only prior.

```r
prior_stan_climate_change <- "parameters {
real beta_0;
real beta_1;
}
model {
beta_0 ~ normal(0, 100);
beta_1 ~ normal(0, 100);
}"
```

```r
prior_climate <- stan(
  model_code = prior_stan_climate_change,
  chains = 1,
  iter = 1000,
  warmup = 0, # Can be 0 since it is only prior
  thin = 1, # No need to thin as well
  seed = 553
)
```

#### Coding the Posterior

```r
posterior_stan_climate_change <- "data {
int<lower=0> n;                          // number of observations
vector[n] income;                        // regressor income
int<lower=0,upper=1> climate_change[n];  // setting the response variable as binary
}
parameters {
real beta_0;
real beta_1;
}
model {
beta_0 ~ normal(0, 100);
beta_1 ~ normal(0, 100);
climate_change ~ bernoulli_logit(beta_0 + beta_1 * income);
}"
```

```r
climate_dictionary <- list(
  n = nrow(pulse_training),
  income = pulse_training$income,
  climate_change = as.integer(pulse_training$climate_change)
)

posterior_climate <- stan(
  model_code = posterior_stan_climate_change,
  data = climate_dictionary,
  chains = 1,
  iter = 21000,
  warmup = 1000,
  thin = 20,
  seed = 553
)
```

### Comparing Bayesian vs Frequentist BLR

| Frequentist BLR                                   | Bayesian BLR                         |
| ------------------------------------------------- | ------------------------------------ |
| Estimates the MLE (Maximum Likelihood Estimation) | Estimates the posterior distribution |
| Estimates the standard errors                     | Estimates the posterior distribution |
| Estimates the confidence intervals                | Estimates the credible intervals     |

- A big advantage of Bayesian: did not need to derive any Maximum Likelihood steps
- Meaning of Bayesian coefficients:
  - e.g. estimate of $\beta_1$ is 0.009, 95% credible interval is (0.005, 0.0134)
  - For each unit increase in income, a subject is 1.009 times more likely to believe in climate change
  - There is a 95% posterior probability that the true value of $\beta_1$ lies between 0.005 and 0.0134
