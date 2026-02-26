**Idea:** we never know if the model we built is specified it or parameterized it correctly, aka we dont know the true paramaters of the real world we wouldn't need to build a model. Kalman filters gives us an estimate of the true latent state of the data.

**specification vs parameteriazation:**
- specification: picking the right model $\mathcal{M}$ to infer about the data
    - EX: 
         - Linear Regression: $y = \beta_0 + \beta_1 x + \epsilon$, $\theta = (\beta_0, \beta_1)$
         - AR(1): $x_t = \phi x_{t-1} + \epsilon_t$, $\theta = \phi$
        - GARCH(1,1): $\sigma_t^2 = \alpha_0 + \alpha_1 \epsilon_{t-1}^2 + \beta_1 \sigma_{t-1}^2$, $\theta = (\alpha_0, \alpha_1, \beta_1)$
        - Ornstein-Uhlenbeck: $dx_t = \theta_1 (\theta_2 - x_t)dt + \theta_3 dW_t$, $\theta = (\theta_1, \theta_2, \theta_3)$
- parametization: picking the right parameters ($\mathcal{M}(\theta)$) for the chosen model

since we dont know the true latent state, we dont know if we used the right model and if we used the right parameters for the model

**how the kalman filter works:**
Kalman filter takes your prediction for time t, and compares it to real, observed data at time t, it then adjusts the next prediction based on how different predicted and observed data at time t was. this becomes a chain pretty much, since prediction at time t was based on data from t-1, t-2, .... t-n
equation:
$$\hat x_{t|t} = \hat x_{t|t-1} + K \times (innovation_t)$$
- where $innovation_t: y_t - \hat x_{t|t-1}$
  - $y_t:$ Observed value at time t
    - $y_t = x_t + v_t$
    - $x_t$: real latent/hidden state
    - $v_t$: data noise 
  - $\hat x_{t|t-1}$: predicted value at time t, based on info from time t-1
- and $K$: kalman gain $\frac{P_{t|t-1}}{P_{t|t-1}+R}$
  - where $P_{t|t-1} = F^2 \times P_{t-1|t-1} + Q$
    - $P$: prediction uncertainty, initalized as 1
    - $Q$: flexibility of the latent signal (parameter)
    - $F$: state transition
  - and $R$ is uncertainty in measurement aka noise (aka how noisy data is) (parameter) 
- and hidden/latent state: $x_t = F(x_{t-1}) + Bu_t + w_t$ 
  - where $w_t$: model error (different from observation/measurement noise)
    - $w_t \sim \mathcal{N}(0, Q)$
  - and $F$: the state transition, aka how the true state moves from t-1 to t
    - $F=1$: state from t-1 fully translates to state in t
    - $0 < F < 1$: state from t-1 fades (aka mean reversion/damping)
    - $F=0$: no impact from t-1 to t (aka data is white noise)
    - $F>1$: growth/expoision from t-1 to t
  - and $B$: optional input, $u_t$: long term mean (for mean reversions)   
- Important: $\hat x_t$ is predicted via a model that we specify
- Updating our prediction uncertainty: $P_{t|t}=(1-K) P_{t|t-1}$
  - Every time we see a new data point, our prediction certainty goes down by 1-K


--- 
**Two ways to estimate $F$ and $B$**:
1) Using data regression AR(1):
   - $F(x_{t-1})+Bu_t + w_t = \beta_1 (x_{t-1}) + \beta_0 + \epsilon_t$
   - aka the equation is pretty much a AR(1)
   - BUT you can only regress on the hidden/latent state ($x_t$) NOT $y_t$
     - so if HAVE to have hidden state data, you have to use one of these:
       1) Maximum Likelihood Estimation (MLE) with kalman filter
       2) Expectation Maximization (EM) algorithm 
       -    these will get expanded on *************
2) Using physics / SDEs
   - start with a model or diffrential equation of how you think the process revolves, and derive F and B mathmatically
   - EX: if we believe the process is mean reverting (OU), then we will use the mean reversion differential equation $dx_t = k(\theta - x_t)dt + \sigma dW_t$ where $\theta$ = equilibrium value, $k$ = speed of pull, and $\sigma dW_t$ = random shocks
     - solving the SDE of time step $\Delta t$:
     - $x_t = e^{-k \Delta t}x_{t-1} + \theta (1-e^{-k \Delta t})+ \epsilon_t$
     - and $F=e^{-k \Delta t}$
     - and $B = \theta(1 - e^{-k \Delta t})$
  3) Hybrid, using both regression and SDE
      - you make a beleif of the sturcture of the data, then you use regression to calibrate the parameters
      - EX: if we assume data is OU
        - our structure is: $x_t = \phi x_{t-1}+w_t$
          - to find $\phi$, we estimate it via regression from the data
          - once we have a value of $\phi$, we plug that into our OU parameter equation $k=-\frac{\ln \phi}{\Delta t}$
            - this equation is just the rearranged form of $F=e^{-k \Delta t}$ from the physics OU equation in bullet 2
             - we just replace F with $\phi$ and solve for k
               - $\phi = e^{-k \Delta t}$ -> $\ln \phi = -k \Delta t$ -> $- \frac{\ln \phi}{\Delta t} = k$
           - once we solve for k, it tells us the speed of the mean reversion, which we can use for other stuff, like getting the value of Q


**Dual filtering:** means you are filtering both the data and the model parameters
- this is important because in the real world, market dynamics change over time, you cannot have fixed F, Q, R bc it will become inaccurate over time as the world changes
- in dual filtering, you are observing $x_t$ AND $\phi_t$ as states
- so now:
  - $x_t = \phi_t x_{t-1} + w_t$
  - AND $\phi_t = \phi_{t-1} + \eta_t$
- and filtered values will be:
  - $x_{t|t} = x_{t|t-1} + K_x(innovation)$
  - $\phi_{t|t} = \phi_{t|t-1} + K_{\phi}(innovation)$
    - where $innovation = y_t - x_{t|t-1}$
    - and $K_t = P_{t|t-1} H^\top \left( H P_{t|t-1} H^\top + R \right)^{-1}$
      - have chatgpt explain this ^ its just a matrix operation, since we are using both x and $\phi$


---

**Example kalman filter:** VIX mean reversion
- since vix mean reverts, we will use a mean reversion model (Ornstein-Uhlenbeck) as $x_t$, and build kalman filter on top of that to model VIX time series
- Our model speiicification ($\mathcal{M}$): Ornstein-Uhlenbeck
- the mean reverting process as $AR(1)$: 
$$x_t = \phi x_{t-1}+B + \epsilon_t$$
$$\phi = e^{-k \Delta t}$$
$$b = (1 - e^{-k \Delta t}) \theta
\rightarrow b = (1 - \phi) \theta$$
- Our parameter selection ($\Theta$): done via $AR(1)$ regression $\Theta = (\phi, b)$ from real data where $\phi = \hat \beta_1$ and $b = \hat \beta_0$ 
-  once we regress and find our beta values, we then solve for k and theta:
  $$k = \frac{- \ln \hat \beta_1}{\Delta t}$$
  $$\theta = \frac{\hat \beta_0}{1 - \hat \beta_1}$$
- once we have our k and $\theta$ values, we will plug them into our structure equation:
$$\hat x_{t | t-1} = e^{-k \Delta t} \hat x_{t-1|t-1} + \theta (1-e^{-k \Delta t})$$
$$P_{t|t-1} = (e^{-k \Delta t})^2 P_{t-1|t-1} + Q$$
- then we calculate our innovation, which is observed minus predicted value at time t
$$innovation = y_t - \hat x_{t|t-1}$$
- then we calculate the kalman gain:
$$K = \frac{P_{t|t-1}}{P_{t|t-1}+R}$$
- then we update our state estimate $\hat x_{t|t}$ and our error covariance $P_{t|t}$
$$\hat x_{t|t} = \hat x_{t|t-1} + K(innovation)$$
$$P_{t|t} = (1-K)P_{t|t-1}$$
- problem: over time, the true parameters $\Theta$ and model $\mathcal{M}$ may change, which will make our kalman filter more inaccurate, therefore we must extend the classic kalman filter approach to update the model $\Theta$ or $\mathcal{M}$ over time
  - this is because we did not use dual filtering, we only assumes $x_t$ as a state

code example:

```python
import pandas as pd
import numpy as np
import statsmodels.api as sm

df = pd.read_csv('vix.csv', parse_dates=['Date'])
df = df.sort_values('Date')

df['VIX_lag'] = df['VIX'].shift(1)
df = df.dropna()

y = df['VIX']
X = sm.add_constant(df['VIX_lag'])

model = sm.OLS(y, X).fit()

beta_0 = model.params['const']
beta_1 = model.params['VIX_lag']
resid_std = np.std(model.resid)

dt = 1/252  # annualized interpretation

kappa = -np.log(beta_1) / dt
theta = beta_0 / (1 - beta_1)

print("--- Calibrated OU Parameters ---")
print(f"kappa: {kappa:.4f}")
print(f"theta: {theta:.4f}")
```

---

**Example Kalman Filter:** simple noise reduction

goal: reduce noise from a random walk, aka there is no expansion or mean reversion, aka F=1 and there is no $B_{ut}$
true state: 
$$x_t = F(x_{t-1}) + Bu_t + w_t \rightarrow x_t = x_{t-1} + w_t$$
- where $w_t \sim \mathcal{N}(0, Q)$ 
- Q = how much latent state varies/bounces
observed value:
$$y_t = x_t + v_t$$
- where $v_t \sim \mathcal{N}(0, R)$
- R = measurement noise (microsturcture, bid/ask bounce, etc)

1) initializing:
  - $x_0 = y_0$  # first obv value, or can be mean value
  - $P_0 =$ (10 to 100)  # initial uncertainty
2) predict:
  - $\hat x_{t|t-1} = x_{t-1|t-1}$
  - $P_{t|t-1} = P_{t-1 | t-1} + Q$
3) update based on observation:
- $innovation = y_t - \hat x_{t|t-1}$
- kalman gain: $K_t = \frac{P_{t|t-1}}{P_{t|t-1} + R}$
- State update: $\hat x_{t|t} = \hat x_{t|t-1} + K(innovation)$
- Uncertainty update: $P_{t|t} = (1 - K_t) P_{t|t-1}$

code example:
```python
import numpy as np

def kalman_denoise(y, Q=1e-4, R=1e-2, x0=None, P0=1.0):
    """
    Simple 1D Kalman filter for denoising: local level model.
    y: 1D array of observations
    Q: process noise variance
    R: measurement noise variance
    """
    y = np.asarray(y, dtype=float)
    n = len(y)

    x_hat = np.zeros(n)
    P = P0

    # init
    x = y[0] if x0 is None else float(x0)

    for t in range(n):
        # Predict
        x_pred = x
        P_pred = P + Q

        # Update
        innovation = y[t] - x_pred
        K = P_pred / (P_pred + R)
        x = x_pred + K * innovation
        P = (1 - K) * P_pred

        x_hat[t] = x

    return x_hat
```