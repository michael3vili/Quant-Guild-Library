**Idea:** we never know if the model we built is specified it or parameterized it correctly, aka we dont know the true paramaters of the real world we wouldn't need to build a model. Kalman filters gives us an estimate of the true latent state of the data.

**specification vs parameteriazation:**
- specification: picking the right model $\mathcal{M}$ to infer about the data
    - EX: 
         - Linear Regression: $y = \beta_0 + \beta_1 x + \epsilon$, $\theta = (\beta_0, \beta_1)$
         - AR(1): $x_t = \phi x_{t-1} + \epsilon_t$, $\theta = \phi$
        - GARCH(1,1): $\sigma_t^2 = \alpha_0 + \alpha_1 \epsilon_{t-1}^2 + \beta_1 \sigma_{t-1}^2$, $\theta = (\alpha_0, \alpha_1, \beta_1)$
        - Ornstein-Uhlenbeck: $dx_t = \theta_1 (\theta_2 - x_t)dt + \theta_3 dW_t$, $\theta = (\theta_1, \theta_2, \theta_3)$
- parametization: picking the right parameters ($\mathcal{M}(\theta)$) for the chosen model

