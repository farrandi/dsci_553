# Lecture 4: Markov Chain Monte Carlo, Stan, and Complex Bayesian Models

## Designing the Model

- Bayesian model is a big joint probability distribution $P(Y, Y', \theta)$
  - Observations: $Y$
  - Latent variables: $\theta$
  - Predictions: $Y'$
- Did inferential approach in Beta-Binomial model (previous section). Now another approach:

$$P(Y, Y', \theta) = P(Y, Y' | \theta) \times P(\theta)$$

1. Generate $\theta$ from the **prior** $P(\theta)$
2. Generate $Y, Y'$ given $\theta$ from **likelihood** $P(Y, Y' | \theta)$

- **Constructing the Likelihood**:
  1. Formulate the data type (int, real, categorical, etc.) and support (positive, negative, etc.)
  2. Figure out which variables are fixed covariates and which are random variables
  3. Pick a distribution that match the type and support. The distribution will have some unknown parameters - need a prior for these parameters

## Markov Chain Monte Carlo (MCMC)

- **Goal**: Generate samples from the posterior distribution
- **Problem**: The posterior is often intractable (can't be solved analytically)
- **Solution**: Use MCMC to generate samples from the posterior

- **Monte Carlo Algorithm**
  - Need closed analytical form of the posterior $f(\theta | Y)$ (e.g. Beta-Binomial model or Gamma-Poisson model)
  - Build independent MC sample $\{\Theta_1, \Theta_2, \ldots, \Theta_n\}$ from $f(\Theta | Y)$ by:
    1. Drawing $\Theta_i$ from $f(\Theta | Y)$
    2. Go there
- Is a **random walk** in the space of $\theta$
- Called a Markov Chain because the next state depends only on the current state $\theta^{(t)} \rightarrow \theta^{(t+1)}$

### Metropolis-Hastings Algorithm

- Allows us to obtain an approximation of the posterior distribution $f(\Theta | Y)$ via MC $\{\Theta_1, \Theta_2, \ldots, \Theta_n\}$.
- Next $\Theta_{t+1}$ is selected by:
  1. Proposing a new value $\Theta'$ from a proposal distribution $q(\Theta' | \Theta_t)$ (e.g. Uniform, Normal, etc.)
  2. Decide whether to accept or reject $\Theta'$ based on acceptance probability $\alpha$:
     $$\alpha = min\left(1, \frac{f(\Theta') \ell(\Theta' | Y)}{f(\Theta_t) \ell(\Theta_t| Y)}\right)$$
     - Then obtain the next via bernoulli trial with probability $\alpha$ for success $\Theta^{(t+1)} = \Theta'$

### Sampling MCMC using rstan

- **Some considerations**:
  - **Warm-up**: Discard the first $n$ samples to allow the chain to converge
  - **Thinning**: Only keep every $n$th sample to reduce autocorrelation
    - skip the first $n$ samples and then keep every $n$th sample

$$\text{Num of approx posterior samples} = \frac{\text{iter} - \text{warmup}}{\text{thin}}$$

### Example: Gamma Poisson Model

- Prior: $\lambda \sim Gamma(s, r)$
- Likelihood: $Y_i | \lambda \sim Poisson(\lambda)$

$$\text{Posterior} \propto \text{Likelihood} \times \text{Prior}$$

$$f(\lambda | Y) \propto \ell(\lambda | Y) \times f(\lambda)$$

- Posterior: $\lambda | Y \sim Gamma(s + \sum Y_i, r + n)$
  </br>
- Our prior: $\lambda \sim Gamma(s=150, r=40)$

```r
{stan output.var='gamma_poisson_stan'}
data {
  int<lower=1> n; // number of rows in training data
  int<lower=0> count[n]; // array of observed counts (integer)
  real<lower=0> s; // prior shape Gamma parameter
  real<lower=0> r; // prior rate Gamma parameter

}

parameters {
  real<lower=0> lambda; // parameter of interest
}

model {
  lambda ~ gamma(s,r); // prior distribution of lambda
  count ~ poisson(lambda); // Poisson likelihood, can be complex formula too
}
```

```r
bird_dictionary <- list(
  n = nrow(observed_evidence),
  count = as.integer(observed_evidence$count),
  s = 150,
  r = 40
)

posterior_lambda <- sampling(
  object = gamma_poisson_stan,
  data = bird_dictionary,
  chains = 1,
  iter = 10000,
  warmup = 1000,
  thin = 5,
  seed = 553,
  algorithm = "NUTS"
)

posterior_lambda <- as.data.frame(posterior_lambda)
```

```r
library(ggplot2)
library(bayesplot)

# Plot the prior, likelihood, and posterior
plot <- plot_gamma_poisson(
    shape = 150, rate = 40,
    sum_y = sum(observed_evidence$count),
    n = nrow(observed_evidence),
)

# Plot the posterior distribution from stan
posterior_plot <- posterior_lambda %>%
  ggplot(aes(x = lambda)) +
  geom_histogram(aes(y = after_stat(count)),
                 bins = 30,
                 fill = "lightblue",
                 color = "black",
                 alpha = 0.5)
```

((See lab 2 for more examples))
