# 🍳 QuickOrder — carnet d'inspection de nuit

> *Audit qualité avant mise en prod. Un QA, un navigateur, un Swagger, beaucoup de café.*
> — Samy

---

## Le principe (en vrac)

QuickOrder, c'est une appli de livraison de repas. Elle est déjà codée. Ma mission : **entrer dans la cuisine à 23h, goûter tous les plats, et dire au patron si on ouvre demain matin.**

Je ne répare rien. Je **prouve** : une capture, une requête, une réponse, une console rouge. Sans preuve, un bug n'existe pas (règle du brief, règle de la maison).

## Comment je parle des bugs

Les sévérités standard (bloquante / majeure / mineure / cosmétique) ont chacune un nom de cuisine, parce qu'à 2h du matin c'est plus facile à retenir :

| Standard | Dans ce carnet | Ça veut dire |
|---|---|---|
| Bloquante | 🔥 **Cramé** | On ne sert pas. Point. |
| Majeure | 🥵 **Brûlé** | On sert, mais quelqu'un va se plaindre fort |
| Mineure | 🧂 **Trop salé** | Mangeable, on le remarque |
| Cosmétique | 🦟 **Mouche dans la soupe** | Pas grave, mais ça se voit |

La **priorité** (P1 / P2 / P3) est une autre question : *quand* on le corrige. Un truc peut être cramé et P2, si personne n'y passe jamais. À discuter.

## Où en est le dépôt (honnêtement)

La campagne a été **arrêtée après quelques vérifications** (12/10/2026). Résultat réel :

- ✅ Écrit : stratégie, matrice de risques, 24 cas de test, protocole des tests utilisateurs, trame du plan d'amélioration, rapport d'exécution
- 🟡 Exécuté en partie : 3 cas (NOM-02, NOM-05, ERR-03), aucune anomalie vue sur ce périmètre
- ❌ **Non fait** : les 21 autres cas, les 8 bug reports demandés (aucun bug confirmé, aucun inventé), les tests avec 2 personnes, et le plan d'amélioration reste à l'état d'hypothèses
- 📋 Recommandation du rapport : **NO-GO faute de preuves** (parcours critiques non exécutés), pas parce que des défauts auraient été prouvés
- 🧭 Pour compléter : voir `docs/execution-report.md`, section 9 (environ 1 h)

Les endpoints, comptes et idées de tests viennent de `API.md`, `INSTALLATION.md`, `openapi.yaml` et de la collection Postman.

## Plan du dépôt

```
quality-audit-team-samy/
├── README.md                  ← vous êtes ici
└── docs/
    ├── test-strategy.md       périmètre, parcours critiques, types de tests
    ├── risk-matrix.md         risque × impact, avec mes paris de départ
    ├── test-cases.xlsx        24 cas + synthèse qui se calcule toute seule
    ├── execution-report.md    la campagne, les chiffres, le Go / No-Go
    ├── bug-reports/           un fichier = un bug (+ trame + pistes de chasse)
    ├── user-feedback.md       protocole + fiches de test avec de vraies gens
    └── improvement-plan.md    6+ améliorations, priorisées
```

## Ordre de bataille

1. Lancer l'appli (`docker compose up --build` ou `python seed.py` + `python run.py`), vérifier `/api/health`, ouvrir Swagger, mettre la clé `quickorder-demo-key`
2. Importer la collection Postman, relever les valeurs manquantes (minimum, frais, stock, restaurant fermé, coupon expiré) → compléter les `[à confirmer]`
3. Exécuter les cas par ordre de risque (le plus gros score d'abord, voir `risk-matrix.md`)
4. Chaque Fail → un fichier dans `bug-reports/` **tout de suite**, pas « plus tard »
5. Faire passer les 3 scénarios à 2 personnes
6. Remplir `execution-report.md`, trancher Go / No-Go
7. Seulement ensuite : branche `fix/...` si on veut proposer des corrections (le code ne bouge pas pendant la campagne initiale)

## Outils

Chrome + DevTools (Network, Console), Swagger (`/docs/`), Bruno ou Postman (collection fournie) pour rejouer les requêtes, un bloc-notes papier (oui).

Comptes de démo : `alice@example.com` / `Password123!`, `bob@example.com` / `Password123!`, admin `admin@quickorder.local` / `Admin123!`. La clé API correspond à **alice**.

⚠️ Mes tests modifient la base : je la remets à zéro (`python seed.py` ou `docker compose down -v`) quand j'ai besoin d'un état propre, et je le note.
