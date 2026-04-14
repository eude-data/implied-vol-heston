# Volatilité Implicite & Modèle de Heston 📉

Extraction de la volatilité implicite à partir de prix de marché, et simulation du smile de volatilité via le modèle de Heston à volatilité stochastique.

---

## Contenu

### `heston.ipynb`

#### 1. Volatilité implicite

Deux algorithmes d'inversion de la formule Black-Scholes pour extraire $\sigma_{impl}$ à partir d'un prix observé $C_{mkt}$ :

| Méthode | Description |
|---|---|
| `vol_impli_newton` | Méthode de Newton–Raphson (convergence quadratique via vega) |
| `vol_impli_bisection` | Méthode de bissection (convergence garantie sur $[\sigma_{min}, \sigma_{max}]$) |

#### 2. Modèle de Heston — Monte Carlo

Simulation par schéma d'Euler du système d'EDS :

$$dS_t = r\, S_t\, dt + \sqrt{v_t}\, S_t\, dW_t^S$$
$$dv_t = \kappa(\theta - v_t)\, dt + \xi\, \sqrt{v_t}\, dW_t^v, \quad d\langle W^S, W^v \rangle_t = \rho\, dt$$

Paramètres utilisés :

| Paramètre | Valeur | Signification |
|---|---|---|
| $\kappa$ | 0.5 | Vitesse de retour à la moyenne |
| $\theta$ | $0.04$ | Variance long terme |
| $\xi$ | 0.3 | Vol de la vol |
| $\rho$ | -0.5 | Corrélation spot / variance |

#### 3. Smile de volatilité implicite

Calcul du skew en faisant varier le strike $K \in [0.7\,S_0,\; 1.5\,S_0]$ :
- Pour chaque $K$, on price via `Heston_MC` puis on inverse par bissection
- Tracé de $\sigma_{impl}(K/S_0)$ — reproduction du smile asymétrique (skew négatif pour $\rho < 0$)

---

## Concepts clés

- **Volatilité implicite** : inversion numérique de Black-Scholes
- **Modèle de Heston** : variance stochastique, corrélation spot/vol
- **Smile / skew de volatilité** : asymétrie induite par $\rho < 0$
- Schéma d'**Euler-Maruyama** pour la discrétisation des EDS couplées

---

## Stack

```python
langages   = ["Python"]
librairies = ["numpy", "scipy", "matplotlib"]
```

---

## Usage

```python
# Vol implicite à partir d'un prix de marché
C_mkt = BS(S0=10, K=10, T=1, r=0.05, sigma=0.2)
sigma_impl = vol_impli_bisection(S0=10, K=10, T=1, r=0.05, C=C_mkt)

# Pricing Heston Monte Carlo
prix = Heston_MC(S0=10, v0=0.04, K=10, T=1, r=0.05,
                 kappa=0.5, theta=0.04, xi=0.3, rho=-0.5)
```
