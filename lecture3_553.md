# Lecture 3: The Beta-Binomial Model

## The Bayesian Modelling

- **Big advantage**:
  - It formulates every problem in one common framework
- **Final goal**: Take samples from the posterior distribution
- Computer does most of the heavy lifting
  - All we need to do is good model design and critical analysis of the results
- **Characteristics:**

### Big Idea

1. **Question**: Pose a scientific question
2. **Design**: Formulate variables and create a probabilistic model for them. **Prior knowledge** is included here!
3. **Inference**: Get **posterior** samples from the model
4. **Check**: If the samples are from your posterior
5. **Analyze**: Use the samples to answer the question

### Pose a Scientific Question

1. **Inferential**: Using observed data $Y$ to make inferences about the population/ latent variable $\theta$
2. **Predictive**: Using observed data $Y$ to make predictions about future data $Y'$

## Beta-Binomial Model

- One of the most foundational Bayesian models
- Recall $Posterior \propto Likelihood \times Prior$

  - **Binomial**: The likelihood function
    - $Y | \pi \sim Binomial(n, \pi) \text{ where } \pi \in [0,1]$
  - Bayesian thinking: $Y$ is a random variable (population parameters are no longer fixed)
  - **Beta**: Prior distribution
    - $\pi \sim Beta(a, b)$

### Beta Distribution (The Prior)

$$\pi \sim Beta(a, b)$$

- **PDF**: $f(\pi) = \frac{\Gamma(a + b)}{\Gamma(a)\Gamma(b)} \pi^{a - 1} (1 - \pi)^{b - 1} \text{  for  } 0 \leq \pi \leq 1$
- **Mean**: $\frac{a}{a + b}$
- **Variance**: $\frac{ab}{(a + b)^2(a + b + 1)}$
- **Mode**: $\frac{a - 1}{a + b - 2} \text{ when } a, b > 1$

### Choosing the right Beta Prior

- One of the biggest challenges in Bayesian statistics
- Need to rely on subject matter prior knowledge
- e.g. Collect information from previous studies and plot a histogram of the data, then fit a beta distribution to it

- `bayesrule` package in R has a function `summarize_beta_binomial(a, b)` to summarize the beta distribution

- PDF of binomial distribution:
  - $f(y | \pi) = \binom{n}{y} \pi^y (1 - \pi)^{n - y}$

### Bayes Rules in Action

$$ Posterior \propto Likelihood \times Prior $$

$$ f(\pi | Y) \propto f(Y | \pi) \times f(\pi) $$

using the beta-binomial model:

$$ f(\pi | Y) \propto \binom{n}{y} \pi^y (1 - \pi)^{n - y} \times \frac{\Gamma(a + b)}{\Gamma(a)\Gamma(b)} \pi^{a - 1} (1 - \pi)^{b - 1}$$

Simplify (remove non-$\pi$ terms):

$$ f(\pi | Y) \propto \pi^{y + a - 1} (1 - \pi)^{n - y + b - 1} $$

We recognize this as the **kernel of a beta distribution**:

$$ f(\pi | Y) \propto Beta(a + y, b + n - y) $$

- **Kernel**: The part of the expression that depends on the variable of interest

### Posterior Distribution

- **Posterior**: $f(\pi | Y) = Beta(a + y, b + n - y)$

  - **Mean**: $\frac{a + y}{a + b + n}$
  - **Variance**: $\frac{(a + y)(b + n - y)}{(a + b + n)^2(a + b + n + 1)}$
  - **Mode**: $\frac{a + y - 1}{a + b + n - 2}$
    - Mode is the value of $\pi$ that maximizes the posterior distribution/ peak (MAP/ Maximum A Posteriori)

- Can also use `summarize_beta_binomial(a, b, n, y)` to summarize the posterior distribution
  - `a` and `b` are the parameters of the prior beta distribution
  - `n` is the number of trials
  - `y` is the number of successes
- Can also use `plot_beta_binomial(a, b, n, y)` to plot the prior and posterior distributions (also from `bayesrule` package)

#### Posterior Credible Interval

- **Credible Interval**: Range of plausible values for the parameter.
  - Width: measures variability of the posterior distribution
- Use function `qbeta` in R to calculate the quantiles of the beta distribution
- For a given a,b,n,y, the 95% credible interval is `qbeta(c(0.025, 0.975), shape1 = a + y, shape2 = b + n - y)`
- **95% CI means**:
  - There is a 95% posterior probability that the true value of $\pi$ is between $L$ and $U$
