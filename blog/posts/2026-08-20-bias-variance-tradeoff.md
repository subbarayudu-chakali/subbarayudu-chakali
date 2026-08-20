# The Bias–Variance Tradeoff, Explained Simply

When I started my M.Tech in AI & ML, the **bias–variance tradeoff** was one of the
first ideas that genuinely changed how I think about models. It's the lens that
explains *why* a model underfits or overfits — and what to do about it. These are
my notes, written in my own words so I actually remember them.

## The core idea

Every supervised model makes prediction errors. We can decompose the expected
error on unseen data into three parts:

$$
\text{Expected Error} = \underbrace{\text{Bias}^2}_{\text{too simple}} + \underbrace{\text{Variance}}_{\text{too sensitive}} + \underbrace{\sigma^2}_{\text{irreducible noise}}
$$

- **Bias** — error from wrong assumptions. A high-bias model is *too simple* to
  capture the real pattern. It **underfits**.
- **Variance** — error from sensitivity to the specific training data. A
  high-variance model memorizes noise and changes wildly with new data. It **overfits**.
- **Irreducible error** ($\sigma^2$) — noise in the data itself. No model can beat this.

The catch: pushing one down usually pushes the other up. That tension *is* the tradeoff.

## An intuition that stuck with me

> Bias is being **consistently wrong**. Variance is being **inconsistently right**.

A straight line fit to a curvy dataset is wrong the same way every time (bias).
A degree-15 polynomial threads every training point but flails between them (variance).

## Seeing it in code

A quick way I convinced myself this is real — fitting polynomials of increasing
degree and watching train vs. test error diverge:

```python
import numpy as np
from sklearn.pipeline import make_pipeline
from sklearn.preprocessing import PolynomialFeatures
from sklearn.linear_model import LinearRegression
from sklearn.model_selection import train_test_split
from sklearn.metrics import mean_squared_error

# Noisy sample from a true sine curve
rng = np.random.default_rng(0)
X = np.sort(rng.uniform(0, 1, 60)).reshape(-1, 1)
y = np.sin(2 * np.pi * X).ravel() + rng.normal(0, 0.15, X.shape[0])

Xtr, Xte, ytr, yte = train_test_split(X, y, test_size=0.3, random_state=0)

for degree in [1, 4, 15]:
    model = make_pipeline(PolynomialFeatures(degree), LinearRegression())
    model.fit(Xtr, ytr)
    train_mse = mean_squared_error(ytr, model.predict(Xtr))
    test_mse = mean_squared_error(yte, model.predict(Xte))
    print(f"degree={degree:>2}  train={train_mse:.3f}  test={test_mse:.3f}")
```

Typical output tells the whole story:

| Degree | Train MSE | Test MSE | Diagnosis |
| ------ | --------- | -------- | --------- |
| 1      | high      | high     | **Underfit** (high bias) |
| 4      | low       | low      | **Just right** |
| 15     | ~0        | high     | **Overfit** (high variance) |

Degree 15 nails the training data and falls apart on the test set — the signature
of variance.

## How to move along the curve

**To reduce bias (fix underfitting):**

- Use a more expressive model or add features
- Reduce regularization
- Train longer

**To reduce variance (fix overfitting):**

- Get more training data
- Add regularization (L1/L2, dropout, early stopping)
- Simplify the model or use ensembling (bagging averages away variance)

## What I'm taking away

The tradeoff isn't a formula to memorize — it's a **diagnostic habit**. When a model
disappoints, I now ask first: *is this bias or variance?* Comparing training error
against validation error usually answers it, and that answer points straight at the fix.

Next up in my notes: **regularization** — the main lever for dialing variance down
without throwing away model capacity.
