# Matrice de risques

**Score = probabilité × impact** (échelle 1 à 5 chacun).

> ⚠️ Les probabilités sont des **paris faits avant d'avoir testé**, d'après ce qu'on voit souvent dans une appli de livraison. Après la campagne, je réécris la colonne « Après » avec ce que j'ai réellement trouvé. L'écart entre les deux est la partie intéressante.

| ID | Risque | Proba | Impact | Score | Niveau | Cas qui le couvrent | Après campagne |
|---|---|:-:|:-:|:-:|---|---|---|
| R1 | Total de commande faux (arrondi, frais, promo) | 3 | 5 | **15** | 🔴 | NOM-03, LIM-05, REG-03, REG-06 | *à remplir* |
| R2 | Accès API insuffisamment protégé (clé absente / invalide acceptée) | 3 | 5 | **15** | 🔴 | ERR-02, NOM-05 | *à remplir* |
| R3 | Commande créée en double, ou non créée alors que confirmée | 2 | 5 | **10** | 🟠 | NOM-04, NOM-06, ERR-05, LIM-06 | *à remplir* |
| R4 | Code promo mal appliqué (cumul, expiré accepté, appliqué deux fois) | 4 | 3 | **12** | 🟠 | REG-01, REG-02, LIM-04 | *à remplir* |
| R5 | Erreurs non gérées : stack trace, 500, message technique visible | 4 | 3 | **12** | 🟠 | ERR-03, ERR-04, ERR-01 | *à remplir* |
| R6 | Annulation incohérente avec le statut de la commande | 3 | 4 | **12** | 🟠 | REG-05 | *à remplir* |
| R7 | Saisie non validée (quantité négative, champs longs, HTML/script) | 3 | 4 | **12** | 🟠 | LIM-01, ERR-06 | *à remplir* |
| R8 | Plat ou restaurant indisponible mais commandable | 3 | 4 | **12** | 🟠 | REG-04, LIM-02 | *à remplir* |
| R9 | Incohérence UI ↔ API (montants, statuts, stocks) | 3 | 4 | **12** | 🟠 | REG-06, NOM-06 | *à remplir* |
| R10 | Seuil minimum / livraison gratuite mal borné (off-by-one) | 3 | 3 | **9** | 🟠 | LIM-03, REG-03 | *à remplir* |
| R11 | Parcours confus pour un vrai utilisateur | 4 | 2 | **8** | 🟠 | Tests utilisateurs A, B, C | *à remplir* |
| R12 | Listes vides ou longues mal gérées | 2 | 2 | **4** | 🟢 | LIM-06 | *à remplir* |

## Lecture rapide

```
Impact
  5 │          R3          R1 R2
  4 │              R6 R7 R8 R9
  3 │       R10  R4 R5
  2 │ R12     R11
  1 │
    └──────────────────────────
      1    2    3    4    5   Probabilité
```
*(le placement exact est approximatif, c'est le tableau qui fait foi)*

## Ordre d'exécution décidé

1. R1, R2 (score 15)
2. R4, R5, R6, R7, R8, R9 (score 12)
3. R3 (10), R10 (9), R11 (8)
4. R12 (4) — si le temps le permet

## Ce que j'ai volontairement laissé de côté

Performance sous charge, sécurité poussée, accessibilité normative : voir `test-strategy.md`, section « hors périmètre ».
