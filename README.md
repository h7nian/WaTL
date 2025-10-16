# Wasserstein Transfer Learning (WaTL) - Code Documentation

This repository contains the implementation of **Wasserstein Transfer Learning (WaTL)** algorithm, as described in the manuscript.

## Repository Structure

```
Codes/
├── README.md                    # This file
├── Simulation/
│   ├── Simulation.R            # Main simulation script (Section 5)
│   └── SimulationFunc.R        # Helper functions for simulation
└── RealData/
    ├── RealData.R              # Main real data analysis script (Section 6)
    └── RealDataFunc.R          # Helper functions for real data analysis
```

---

## Algorithm Implementation

### Algorithm 1: Wasserstein Transfer Learning (WaTL)

#### Step 1: Weighted Auxiliary Estimator

**Manuscript:**
$$\widehat{f}(x) = \frac{1}{n_0+n_{\mathcal{A}}}\sum_{k=0}^{K}n_k\widehat{f}^{(k)}(x)$$

**Code implementation:**

**Simulation (Global Fréchet):**
- `SimulationFunc.R`, function `compute_f1_hat()`, lines 278-333
- Aggregates target ($k=0$) and all sources ($k=1,\ldots,K$)
- Weighted by sample sizes $n_k$

**Real Data (Local Fréchet):**
- `RealDataFunc.R`, function `compute_f1_hat()`, lines 279-341
- Uses local linear weights instead of global weights
- Aggregates target and all sources

#### Step 2: Bias Correction Using Target Data

**Manuscript:**
$$\widehat{f}_0(x) = \argmin_{g\in L^2(0,1)}\frac{1}{n_0}\sum_{i=1}^{n_0}s_{iG}^{(0)}(x)\|F^{-1}_{\nu_i^{(0)}}-g\|_2^2 + \lambda\|g-\widehat{f}(x)\|_2$$

**Code implementation:**
- `SimulationFunc.R` / `RealDataFunc.R`, function `compute_f_L2()`
- Uses gradient descent to minimize the objective
- Regularization parameter $\lambda$ selected via cross-validation
  - Simulation: `lambda_candidates <- seq(0, 3, by = 0.1)` (Setting 1)
  - Real Data: Custom grid in `RealData.R`, lines 109-114

#### Step 3: Projection to Wasserstein Space

**Manuscript:**
$$\widehat{m}_G^{(0)}(x) = \argmin_{\mu \in \mathcal{W}}\|\mathbf{F}^{-1}_{\mu}-\widehat{f}_0(x)\|_2$$

**Code implementation:**
- **Simulation:** Implicitly satisfied (quantile functions already monotone in data generation)
- **Real Data:** `RealDataFunc.R`, function `Project()`, lines 348-373
  - Uses OSQP solver to enforce monotonicity constraint
  - Called in `RealData.R`, line 171

---

## Running the Code

### Simulation

```bash
Rscript Simulation/Simulation.R <M> <n_t> <seed> <setting> <tau>
```

**Arguments:**
- `M`: Grid size for quantile functions (e.g., 100)
- `n_t`: Target sample size (200-800)
- `seed`: Random seed
- `setting`: Data generation setting (1 or 2)
- `tau`: Source sample multiplier (100 or 200)

**Example:**
```bash
Rscript Simulation/Simulation.R 100 200 42 1 100
```

### Real Data

```bash
Rscript RealData/RealData.R <seed> <race> <M> <rate> <gender>
```

**Arguments:**
- `seed`: Random seed
- `race`: Race index (1=Black, 2=White)
- `M`: Grid size for quantile functions (e.g., 100)
- `rate`: Source data sampling rate (0-1, typically 1.0)
- `gender`: Gender (0=Female, 1=Male)

**Example:**
```bash
Rscript RealData/RealData.R 42 1 100 1.0 0
```
