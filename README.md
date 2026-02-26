# Automated Life Pricing & Table Graduation Pipeline
A Python pipeline that takes raw, noisy claims data from a hypothetical international subsidiary, smooths it into a usable mortality table, and prices a 10-Year Term Life policy.
## 📌 Project Objective
This project serves as a proof-of-concept for modernizing actuarial pricing workflows. It demonstrates the transition from legacy spreadsheet-based models to a vectorized Python pipeline for Individual Life Pricing. The pipeline handles raw portfolio data simulation, mortality experience studies, table graduation, and final premium/reserve calculations in accordance with the principles of SOA Exam FAM.

## 🛠️ Methodology & Implementation

### Phase 1: Portfolio Simulation & Competing Risks
To test the pricing engine, a synthetic portfolio of 10,000 policyholders was generated with realistic demographic and underwriting attributes (Age, Gender, Smoker Status, Diabetic Status).
* **Time-to-Death ($T_d$):** Simulated using Inverse Transform Sampling based on a Gompertz baseline hazard $\mu_x = B \cdot c^x$.
* **Excess Mortality:** Applied multiplicative factors to the baseline hazard for substandard risks (+50% for Smokers, +40% for Diabetics).
* **Time-to-Lapse ($T_l$):** Modeled as a competing risk using an Exponential distribution to mimic early policy surrenders.

### Phase 2: Exposure Calculation & Whittaker-Henderson Graduation
Raw mortality rates ($q_x$) were calculated by computing the exact central exposure to risk ($E_x$), accounting for fractional years lived by policyholders who lapsed. 
Due to the stochastic noise inherent in finite portfolios, a **Whittaker-Henderson** graduation algorithm was implemented via `scipy.optimize` to construct a smooth pricing table.

The algorithm minimizes the loss function $M$:
$$M = \sum W_x (q_x - \hat{q}_x)^2 + h \sum (\Delta^2 \hat{q}_x)^2$$
Where:
* $W_x = E_x$ (Weights prioritizing ages with higher exposure)
* $h = 250,000$ (Smoothing parameter)
* $\Delta^2 \hat{q}_x$ (Second differences enforcing curve regularity)


<img src="https://github.com/OuaisBien/life-insurance/blob/main/mortality.png">

### Phase 3: Actuarial Pricing (Temporaire Décès)
Using the smoothed mortality curve, the engine prices a 10-Year Term Life policy. The calculation relies on the **Equivalence Principle**, equating the Present Value of Future Benefits (PVFB) to the Present Value of Future Premiums (PVFP) at a constant discount rate ($i=3\%$).

**Actuarial Present Value of Death Benefit:**
$$A_{x:\overline{n|}}^1 = \sum v^{t+1} \cdot \hspace{0.1cm}_tp_x \cdot q_{x+t}$$

**Actuarial Present Value of Premiums (Annuity Due):**
$$\ddot{a}_{x:\overline{n|}} = \sum v^t \cdot \hspace{0.1cm}_tp_x$$

**Net Premium (Prime Pure):**
$$P = SA \cdot \frac{A_{x:\overline{n|}}^1}{\ddot{a}_{x:\overline{n|}}}$$

### Phase 4: Prospective Reserving (Provision Mathématique)
To project the balance sheet impact, the engine calculates the mathematical reserve ($V_t$) at the end of each policy year. The reserve accumulates in early years when the level premium exceeds the underlying mortality risk and depletes to zero at maturity.

$$V_t = SA \cdot A_{x+t:\overline{n-t}|}^1 - P \cdot \ddot{a}_{x+t:\overline{n-t}|}$$


*(Insert your generated reserve curve plot here)*

## 💻 Technologies Used
* **Python:** Core logic and vectorization.
* **NumPy & Pandas:** Data manipulation, fractional exposure calculations, and array operations.
* **SciPy (`scipy.optimize`):** Minimization solver (L-BFGS-B) for the Whittaker-Henderson graduation.
* **Matplotlib:** Log-scale mortality visualization and reserve curve plotting.
