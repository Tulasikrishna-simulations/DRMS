# DRMS — Dynamic Relative Monetary System

> *"Currency should reflect performance, not power."*

**[→ Live Site](https://tulasikrishna-simulations.github.io/DRMS)**

---

I am 18, from Hospete, Karnataka. During a drop year I decided the global monetary system needed a redesign. I had no supervisor, no lab, no institution. No economist to ask — where in Hospete would I find one? I couldn't afford JEE coaching; I studied from YouTube. I found stochastic calculus in a borrowed Cengage book at 2am. I found the identifiability problem the same way.

Every gap documented here — I found myself, usually after already building on top of it for weeks. A PhD student gets these corrections in their first meeting. I got them by reading papers instead of solving mock tests and realising something I built three weeks ago has a problem I now know the name for.

This is not presented as a finished work. It is presented as genuine work, honestly documented.

If you find something I missed — and you probably will — I would be extremely grateful for any feedback.

---

## The Problem

The global monetary system has three structural failures:

**Single-Reserve Dependency** — The US dollar's dominance means one nation's monetary policy propagates through every economy holding dollar-denominated debt. They didn't vote for this.

**Structural Imbalances** — Nations are locked into dollar debt cycles that constrain domestic fiscal policy independent of their own economic performance or discipline.

**Geopolitical Contamination** — The Russian ruble didn't collapse in 2022 because Russia became economically incompetent. It collapsed because geopolitical decisions were transmitted through financial infrastructure. Currency values reflected political anger, not economic reality. That is a design flaw.

DRMS addresses these through one architectural shift: currency value computed algorithmically from measurable economic indicators, updated continuously, no central authority required.

---

## The Mathematics

### Three Models

**Model 1 — Linear (discarded)**
```
dC/dt = k₁·T(t)·G(t) − k₂·I(t)·D(t)
```
Trust and growth are multiplied not added — growth without trust doesn't compound. Problem: C(t) unbounded, can go negative. A negative currency strength is not an economic insight.

**Model 2 — Proportional (discarded)**
```
dC/dt = C(t) · [k₁·T(t)·G(t) − k₂·I(t)·D(t)]
```
Growth proportional to current strength. Correct direction. Problem: exponential growth with no ceiling for viable economies.

**Model 3 — Stability-Constrained (primary)**
```
dC/dt = C(t)·[k₁·T(t)·G(t) − k₂·I(t)·D(t)] − λ·[C(t)]²
```
The −λC² term introduces quadratic resistance capping currency at a finite equilibrium. Why n=2: n=1 gives no finite non-zero equilibrium; n≥3 is excessively aggressive; n=2 is the minimal nonlinearity that works. This is a Bernoulli ODE of order n=2 with exact closed-form solution.

### Closed-Form Solution (Bernoulli substitution, u = 1/C)

```
C(t) = 1 / [λ/α + (1/C₀ − λ/α)·e^(−αt)]
```

where α = k₁TG − k₂ID (net performance signal).

As t → ∞ with α > 0: C(t) → α/λ = C* (global attractor).

*Honest note: this closed form assumes α is constant. In the actual model T, G, I, D are time-varying, so α = α(t) and no simple closed form exists. The closed form is exact for constant parameters. Numerical integration is the primary simulation method. This tension was documented as Mistake #11.*

### Viability Condition

```
k₁·T(t)·G(t) > k₂·I(t)·D(t)
```

When this fails, the only stable equilibrium is C = 0. The currency collapses regardless of initial conditions.

### Stability Analysis

Fixed points: C₀ = 0 (trivial), C* = α/λ (non-trivial).

Linearising f(C) = αC − λC²:
```
f′(C) = α − 2λC
f′(0) = α         → UNSTABLE when α > 0, STABLE when α < 0
f′(C*) = α − 2α = −α → STABLE when α > 0 (global attractor)
```

The factor of 2 from differentiating C² matters. Skipping it was Mistake #2.

### Sensitivity Analysis

For the base model C* = α/λ:
```
∂C*/∂k₁ = TG/λ      (larger trust×growth → stronger sensitivity to k₁)
∂C*/∂k₂ = −ID/λ     (larger inflation×risk → stronger drag)
∂C*/∂λ  = −α/λ²     (stronger damping → less sensitive equilibrium)
∂C*/∂T  = k₁G/λ
∂C*/∂I  = −k₂D/λ
```

With dynamic damping λ(C) = λ₀ + λ₁C, C* becomes:
```
C* = (−λ₀ + √(λ₀² + 4λ₁α)) / (2λ₁)
```
Sensitivity requires implicit differentiation. This is a genuine new result from the v4 structural corrections.

---

## Variables

| Variable | Meaning | Domain |
|---|---|---|
| C(t) | Currency strength — state variable | ℝ⁺ |
| T(t) | Trust index — governance quality | [0, 1] |
| G(t) | Real GDP growth rate | Rate |
| I(t) | Inflation rate | Rate |
| D(t) | Decision risk — policy instability | [0, 1] |
| k₁(t) | Growth sensitivity — empirically fitted in v4 | ℝ⁺ |
| k₂(t) | Inflation-risk sensitivity — empirically fitted in v4 | ℝ⁺ |
| λ | Stability factor | ℝ⁺ |

k₁ = 40, k₂ = 30, λ = 0.5 are v1/v2 demonstration values manually chosen. Version 4 fits k₁(t), k₂(t), λ(t) empirically per country from World Bank data via least-squares minimisation.

---

## DRMS v4 — Master Equation

```
dCᵢ/dt = Cᵢ(k₁(t)·Tᵢ·Geff,ᵢ − k₂(t)·Iᵢ·Dᵢ + k₃Mᵢ) − λ(Cᵢ)·Cᵢ²
         + β·Rᵢ(t)
         + Σⱼ γᵢⱼ(t)·(Cⱼ − Cᵢ)·Cᵢ
         + σᵢ·Cᵢ·dWₜ + Jᵢ·dNₜ
```

Where Geff = ln(1 + Gnorm) and Gnorm = G / G_global_median.

| Term | Meaning |
|---|---|
| `k₁(t)·Tᵢ·Geff,ᵢ` | Growth-trust driver. k₁(t) time-varying, fitted from data. Geff compresses extreme growth values (the Ronaldo analogy: a beginner improves faster than prime Ronaldo not because he is better but because he has more room). |
| `−k₂(t)·Iᵢ·Dᵢ` | Inflation-risk drag. k₂(t) time-varying, fitted from data. One honest note: k₁ and k₂ always appear as products k₁TG and k₂ID. If trust and growth are correlated — and they often are — the data cannot always tell them apart. This is the identifiability problem. It affects every structural macroeconomic model ever written. |
| `+k₃Mᵢ` | Market sentiment. DRMS models structural equilibrium dynamics not market microstructure. Meese and Rogoff (1983) showed no structural model beats a random walk at short horizons. DRMS doesn't claim to. It claims structural fundamentals determine long-run equilibrium regimes. |
| `−λ(C)·C²` | Adaptive stabilisation. λ(C) = λ₀ + λ₁C — damping that increases with scale. |
| `+β·Rᵢ(t)` | Recovery injection. Countries collapse but they also recover. |
| `+Σⱼ γᵢⱼ(t)·(Cⱼ−Cᵢ)·Cᵢ` | Network coupling. Asymmetric, time-varying. US influencing India ≠ India influencing US. |
| `+σᵢCᵢdWₜ + JᵢdNₜ` | Stochastic shocks + jump diffusion. Brownian motion for continuous noise; Poisson jumps for abrupt crashes. The 2022 ruble collapse was not a continuous process. |

---

## The Twelve Documented Errors

Most researchers quietly fix mistakes. Version 2 documented every error found, ranked by severity, corrected fully.

| # | Error | Severity |
|---|---|---|
| 9 | Corrections report used k₁ = k₂ = 35; actual values are k₁ = 40, k₂ = 30. Bangladesh misclassified as non-viable. | CRITICAL |
| 1 | Integrating factor: wrote e^(−αt), correct is e^(+αt). Final answer survived because errors cancelled. | HIGH |
| 2 | Stability linearisation skipped the substitution step. Factor of 2 from differentiating C² was missing. | HIGH |
| 3 | Alternated between c(t) and C(t). Case is semantics not style. Missing time arguments on T, G, I, D. | HIGH |
| 4 | C₀=0 declared unconditionally unstable. It's stable when α < 0. Argentina proves this directly. | MEDIUM |
| 5 | Negative C* (Argentina = −35.5) reported as a meaningful result. C(t) ≥ 0 by domain restriction. | MEDIUM |
| 6 | Argentina inflation: 50% in code, 80% in PDF. Neither correct. IMF average 2015–2023: 66%. | MEDIUM |
| 7 | Euler Δt = 1.0 near stability boundary. Instability masked by clamping to [−300, 300]. | MEDIUM |
| 8 | integral() used Math.max(0, d.C), silently discarding negative values. | LOW |
| 10 | Bifurcation threshold stated as ~4%. Correct: I_crit = k₁TG/(k₂D) = 16.1%. Factor of four error. | MEDIUM |
| 11 | Closed form presented for time-varying α(t). Closed form only valid for constant α. | MEDIUM |
| 12 | Corrections report changed parameters without source citations. Cannot verify if C* changed due to formula or data. | MEDIUM |

---

## 18-Country Corrected Results

k₁ = 40, k₂ = 30, λ = 0.5 (v1/v2 values). Argentina inflation corrected to 66% (IMF IFS 2015–2023).

| Country | T | G% | I% | D | α | C* | Status |
|---|---|---|---|---|---|---|---|
| China | 0.55 | 7.0 | 2.5 | 0.30 | 1.315 | 2.630 | ✓ VIABLE |
| India | 0.65 | 6.5 | 5.5 | 0.35 | 1.113 | 2.225 | ✓ VIABLE |
| Bangladesh | 0.42 | 6.5 | 6.0 | 0.50 | 0.192 | 0.384 | ✓ VIABLE |
| South Korea | 0.75 | 3.0 | 2.0 | 0.20 | 0.780 | 1.560 | ✓ VIABLE |
| Australia | 0.82 | 2.5 | 2.5 | 0.17 | 0.693 | 1.385 | ✓ VIABLE |
| Germany | 0.85 | 2.5 | 3.0 | 0.15 | 0.715 | 1.430 | ✓ VIABLE |
| USA | 0.78 | 2.3 | 2.5 | 0.20 | 0.568 | 1.135 | ✓ VIABLE |
| Canada | 0.82 | 2.0 | 2.2 | 0.18 | 0.537 | 1.074 | ✓ VIABLE |
| Saudi Arabia | 0.55 | 2.5 | 2.0 | 0.40 | 0.310 | 0.620 | ✓ VIABLE |
| UK | 0.76 | 1.8 | 3.0 | 0.22 | 0.349 | 0.698 | ✓ VIABLE |
| France | 0.74 | 1.5 | 2.0 | 0.25 | 0.294 | 0.588 | ✓ VIABLE |
| Japan | 0.80 | 1.0 | 0.5 | 0.18 | 0.293 | 0.586 | ✓ VIABLE |
| Brazil | 0.48 | 2.0 | 7.0 | 0.55 | −0.771 | → 0 | ✗ COLLAPSE |
| Russia | 0.35 | 1.5 | 8.0 | 0.55 | −1.110 | → 0 | ✗ COLLAPSE |
| Turkey | 0.42 | 4.0 | 15.0 | 0.60 | −2.028 | → 0 | ✗ COLLAPSE |
| Pakistan | 0.35 | 4.0 | 12.0 | 0.65 | −1.780 | → 0 | ✗ COLLAPSE |
| Nigeria | 0.45 | 3.5 | 18.0 | 0.55 | −2.340 | → 0 | ✗ COLLAPSE |
| Argentina ◆ | 0.30 | 2.0 | 66.0 | 0.75 | −14.610 | → 0 | ✗ COLLAPSE |

◆ Argentina inflation corrected to 66%.

The meaningful question is not whether DRMS predicts exchange rates better than a random walk — Meese-Rogoff (1983) shows no structural model does at short horizons. The meaningful question is whether DRMS correctly classifies long-run viability regimes. Turkey, Pakistan, Argentina, Nigeria all correctly collapse. Bangladesh, corrected from the prior report's wrong k values, correctly survives.

---

## Version History

### v1 — Core Framework
Three ODE models derived. 4-country simulation (scipy RK45). k₁ = 40, k₂ = 30, λ = 0.5 manually chosen. These values were never formally stated in v1 — Version 2 confirmed them by back-calculation from the results table.

### v2 — Honest Audit
12 errors documented and corrected. Most consequential: the corrections report itself used k₁ = k₂ = 35 — wrong. Bangladesh misclassified as a direct result. 18-country corrected table. Argentina inflation corrected to 66%.

### v3 — Full Framework
- Trust decomposed: T = w₁P + w₂S + w₃F + w₄R, Σwᵢ = 1 (probability simplex constraint)
- Growth normalised: Gnorm = G/G_global_median, Geff = ln(1 + Gnorm)
- Manipulation resistance: T_final = η·T_reported + (1−η)·T_verified
- Stochastic term: σᵢCᵢdWₜ (Wiener process, found in Cengage calculus at 2am)
- Network coupling: Σⱼ γᵢⱼ(Cⱼ − Cᵢ)Cᵢ
- Recovery term: β·R(t)
- Optimal control: max ∫₀ᵀ U(C(t), u(t)) dt — derived by analogy with Fermat's principle

### v4 — Structural Corrections + Empirical Calibration
- k₁(t), k₂(t), λ(t) become time-varying, fitted from World Bank data
- Regularisation: Σ(C−Y)² + µ(k₁² + k₂² + λ²) to prevent overfitting
- Dynamic damping: λ(C) = λ₀ + λ₁C
- Market sentiment term: +k₃M
- Jump diffusion: +Jᵢ·dNₜ (Poisson process for abrupt crises)
- C_critical threshold replacing literal C → 0
- Endogenous feedback: T, I, G, D governed by coupled ODEs
- Dimensional consistency: C̃ = C/C_ref, t̃ = t/t_ref
- Validation against baselines: naive mean, random walk, OLS on same inputs

---

## Known Open Problems

These are not hidden. They are stated because finding them matters more than pretending they don't exist.

**Identifiability** — k₁ and k₂ always appear as products k₁TG and k₂ID. If trust and growth are correlated across time, they are not separately identified from exchange rate data alone. Resolving this requires instrumental variables, panel variation, or structural constraints from economic theory.

**Econometric rigor** — Confidence intervals on fitted parameters require bootstrap resampling. Stationarity tests (ADF) on exchange rate data before fitting. Causal structure between T, G, I, D and C requires instrumental variables or a natural experiment. These are pending.

**Stochastic stability** — Lyapunov stability for the deterministic case is provable (V(C) = (C−C*)² is a candidate). For the full stochastic SDE, rigorous stability requires Itô's lemma applied to the coupled system. Not yet completed.

**Existence and uniqueness** — For the base Model 3 ODE: provable by Picard-Lindelöf since the right-hand side is locally Lipschitz for C > 0. For the full endogenous coupled system: not yet proven. Stated explicitly as open.

**Meese-Rogoff** — No structural FX model beats a random walk at short horizons (1983, still holds). DRMS is a long-run equilibrium model and should be evaluated as one. Short-run prediction is not the claim.

---

## Running the Empirical Pipeline

```bash
pip install requests pandas numpy scipy matplotlib scikit-learn

# Test single country
python drms_empirical.py India

# Full 18-country pipeline
python drms_empirical.py
```

Fetches real data from World Bank API (no API key required). Outputs country-specific fitted k values, RMSE, Pearson correlation, and baseline comparison against naive mean, random walk, and OLS.

---

## Repository

```
DRMS/
├── index.html           # Full research website — self-contained, all 18 notebook pages embedded
├── drms_empirical.py    # v4 empirical calibration pipeline
├── README.md
```

---

## Other Research

- **[ETPT](https://tulasikrishna-simulations.github.io/etpt-portfolio)** — Evolutionary Technology Path Theory: why every intelligent species builds a completely different kind of technology. Started with a panda.
- **[NTI-AD](https://tulasikrishna-simulations.github.io/adsach-)** — Advertising Integrity Framework: separating product truth from advertising manipulation in Indian FMCG.
- **[RSF](https://tulasikrishna-simulations.github.io/RSF-simulation-/)** — Relativistic Spacetime Framework: independent derivation of Morris-Thorne metric.

---

*Sadhana Ravichandra Tulasi Krishna · Hospete, Karnataka · 2025*
*Independent Researcher · Drop Year · No Institutional Affiliation*
