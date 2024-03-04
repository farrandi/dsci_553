# Lecture 5: Bayesian Normal Linear Regression and Hypothesis Testing

## Bayesian Normal Linear Regression

### Example 1: `bikeshare` data

- This data is from `bayesrules` package in R.
- We want to consider the effects of `temp_feel` (numeric) and `weekend` (boolean) on `rides` (count).

---

For OLS would use Poisson regression since output is count data.

`lm(rides ~ temp_feel + weekend, data = bikeshare)`

---

#### Set up Priors

For Bayesian, we need to specify **priors** for the coefficients and the variance.

$$Y_i = \beta_0 + \beta_1X_{i1} + \beta_2X_{i2}\sigma^2 + \epsilon_i$$

where

- $X_{i1}$ is `temp_feel`
- $X_{i2}$ is `weekend`: 1 if weekend, 0 if not
- $\beta_0$ is the intercept ~ $Gamma(7.5, 1)$
- $\beta_1$ is the coefficient for `temp_feel` ~ $\mathcal{N}(0, 1000^2)$
- $\beta_2$ is the coefficient for `weekend` ~ $\mathcal{N}(0, 1000^2)$
- $\sigma^2$ is the variance ~ $IG(0.001, 0.001)$
  - Inverse Gamma distribution (popular for variance priors > 0)

#### MCMC Simulation

- Use `stan` to sample from the posterior distribution.

```r
{r bikerides_stan}

bikerides_stan <- "data {
  int<lower=0> n;                               // training sample size
  vector[n] y;                                  // response vector
  vector[n] x_1;                                // regressor 1 vector
  vector[n] x_2;                                // regressor 2 vector
  real pred_x_1;                                // fixed value for regressor 1
  real pred_x_2;                                // fixed value for regressor 2
}

parameters {
  real<lower=0> beta_0;                         // intercept with lower bound
  real beta_1;                                  // regression coefficient 1
  real beta_2;                                  // regression coefficient 2
  real<lower=0> sigma;                          // common standard deviation with lower bound
  }

model {
  beta_0 ~ gamma(7.5, 1);                       // alpha = 7.5 and beta = 1
  beta_1 ~ normal(0, 1000);                     // mu_b1 = 0 and sigma_b1 = 1000
  beta_2 ~ normal(0, 1000);                     // mu_b2 = 0 and sigma_b2 = 1000
  sigma ~ inv_gamma(0.001, 0.001);              // eta = 0.001 and lambda = 0.001
  y ~ normal(beta_0 + beta_1 * x_1 + beta_2 * x_2, sigma);
}
generated quantities {
  real y_pred = normal_rng(beta_0 + beta_1 * pred_x_1 + beta_2 * pred_x_2, sigma);
}"
```

((TODO: add important block from 1.5))

Then, in R need a dictionary to pass into stan.

```r
model_matrix <- as.data.frame(model.matrix(rides ~ temp_feel + weekend, data = bikeshare_data))
model_matrix

bikerides_dictionary <- list(
  n = nrow(bikeshare_data),
  y = bikeshare_data$rides,
  x_1 = bikeshare_data$temp_feel,
  x_2 = model_matrix$weekendTRUE,
  pred_x_1 = 75,
  pred_x_2 = 1
)
```

Now compile the `Stan` model:

```r
posterior_bikeshare <- stan(
  model_code = bikerides_stan,
  data = bikerides_dictionary,
  chains = 1,
  iter = 40000,
  warmup = 20000,
  thin = 60,
  seed = 553,
)

# view the posterior summary
round(summary(posterior_bikeshare)$summary, 2)[-6, c("mean", "sd", "2.5%", "97.5%")]
```

- The 2.5% and 97.5% quantiles are the 95% credible intervals.
  - If the interval **contains 0**, then the coefficient is **not significant**.
  - If the interval is **large**, then the model is **not very certain** about the coefficient (i.e. model is not capturing the right systematic component).

### Example 2: Tinder data

- Recall $Posterior \propto Likelihood \times Prior$

#### The Prior

#### The Likelihood

#### The Posterior

## Bayesian Hypothesis Testing

- Using the Tinder example from before...

### One-Sided Hypothesis Testing

- **Claim**: In any city like Vancouver, more than 15% of the single people who use the Tinder app will eventually find a partner.
  - **Null Hypothesis**: $\pi \leq 0.15$
  - **Alternative Hypothesis**: $\pi > 0.15$ (associated with the claim)
- In Bayesian, we use the _posterior_ and **get probability for each hypothesis** (unlike frequentist).
- Use `pbeta` function to get the probability
  - If posterior $Beta(a'=24, b'=192)$ = $f(\pi | y=20)$
  - Then $H_0: P(\pi \leq 0.15 | y=20) = \int_0^{0.15} f(\pi | y=20) d\pi$
  - equal to `pbeta(0.15, 24, 192)`
    </br>
- In the Tinder example, we get 2 probabilities:
  - $P(H_0 | y=20) = P(\pi \leq 0.15 | y=20) = 0.957$
  - $P(H_a | y=20) = P(\pi > 0.15 | y=20) = 0.043$
- Can get **Posterior Odds** that $\pi > 0.15$ by dividing the two probabilities.

$$\frac{P(H_a | y=20)}{P(H_0 | y=20)} = \frac{0.043}{0.957} = 0.045$$

- **Interpretation**: $\pi$ is 22 times($\frac{1}{0.045}$) more likely to be less than or equal to 0.15 compared to being greater than 0.15 using our **posterior model**.

### Bayes Factor

- Do the same for **prior odds**

$$\text{Prior Odds} = \frac{P(H_a)}{P(H_0)}$$

$$\text{Bayes Factor} = \frac{\text{Posterior Odds}}{\text{Prior Odds}}$$

- Bayes Factor = 1: Plausibility of $H_a$ stays the same even after new data
- Bayes Factor > 1: Plausibility of $H_a$ increases after new data
- Bayes Factor < 1: Plausibility of $H_a$ decreases after new data

### Two-Sided Hypothesis Testing

- Lets say we want to figure out: whether or not 15% of the single people who use the Tinder app will eventually find a partner (in any city like Vancouver)
  - $H_0: \pi = 0.15$
  - $H_a: \pi \neq 0.15$

$$P(\pi = 0.15 | y=20) = \int_{0.15}^{0.15} f(\pi | y=20) d\pi = 0$$

- Does not work since =0, so add a range of values (e.g. $\pm 0.10$)

  - $H_0: \pi \in [0.05, 0.25]$
  - $H_a: \pi \notin [0.05, 0.25]$

- Then find the **credible interval** for the posterior distribution of $\pi$.

```r
qbeta(c(0.025, 0.975), 24, 192)
```

Then based on the results:

- If **credible interval** falls within the range of $H_0$, then we are in favor of $H_0$, with 95% probability.
