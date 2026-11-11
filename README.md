# Volatilité implicite et modèle de Heston

Projet réalisé en M2 ISIFAR (Université Paris Cité). On calcule d'abord la volatilité implicite d'un call à partir de son prix, puis on simule le modèle de Heston par Monte Carlo pour retrouver le smile de volatilité qu'il produit.

Tout le code est dans `heston.ipynb`.

## 1. Volatilité implicite

Étant donné un prix observé $C$, on cherche la volatilité $\sigma$ telle que le prix de Black-Scholes soit égal à $C$. Deux méthodes sont codées :

| Fonction | Méthode |
|---|---|
| `vol_impli_newton` | Newton-Raphson, en utilisant la vega comme dérivée |
| `vol_impli_bisection` | Dichotomie : on encadre d'abord la solution en doublant ou divisant $\sigma$ par 2, puis on coupe l'intervalle en deux |

Test : on calcule le prix d'un call ($S_0 = 10$, $K = 6$, $T = 1$, $r = 0.05$, $\sigma = 0.2$), puis on inverse. La dichotomie retrouve $\sigma \approx 0.19999$.

## 2. Simulation du modèle de Heston

Le modèle s'écrit

$$dS_t = r S_t\, dt + \sqrt{v_t}\, S_t\, dW_t$$

$$dv_t = \kappa(\theta - v_t)\, dt + \xi \sqrt{v_t}\, dB_t$$

où les deux browniens $W$ et $B$ ont une corrélation $\rho$.

On discrétise avec un schéma d'Euler sur 5 000 pas de temps et 10 000 trajectoires. Comme ce schéma peut rendre la variance négative, on la remplace par 0 à chaque pas.

| Paramètre | Valeur | Rôle |
|---|---|---|
| $v_0$ | 0.04 | variance initiale (volatilité de 20 %) |
| $\theta$ | 0.04 | variance de long terme |
| $\kappa$ | 0.5 | vitesse de retour vers $\theta$ |
| $\xi$ | 0.3 | volatilité de la variance |
| $\rho$ | -0.5 | corrélation entre le spot et la variance |

Pour un call à la monnaie ($S_0 = K = 10$, $T = 1$, $r = 0.05$), on obtient 1.045 avec Black-Scholes ($\sigma = 0.2$) et 1.025 avec Heston.

## 3. Smile de volatilité

On fait varier le strike de $0.70\, S_0$ à $1.45\, S_0$ par pas de $0.05\, S_0$. Pour chaque strike, on calcule le prix avec Heston, puis on en déduit la volatilité implicite par dichotomie.

La volatilité implicite passe d'environ 26 % pour $K = 0.7\, S_0$ à environ 17 % autour de $K = 1.3\, S_0$, puis remonte légèrement. On retrouve le skew attendu quand $\rho$ est négatif : les baisses du spot s'accompagnent d'une hausse de la volatilité, ce qui renchérit les strikes bas.

## Utilisation

```python
# volatilité implicite à partir d'un prix
C = BS(S0=10, K=10, T=1, r=0.05, sigma=0.2)
sigma_impl = vol_impli_bisection(S0=10, K=10, T=1, r=0.05, C=C)

# prix d'un call dans le modèle de Heston
prix = Heston_MC(S0=10, v0=0.04, K=10, T=1, r=0.05,
                 kappa=0.5, theta=0.04, xi=0.3, rho=-0.5)
```

Bibliothèques : numpy, scipy, matplotlib.
