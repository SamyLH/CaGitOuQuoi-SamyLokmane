# Plan d'amélioration

**Statut : 🟡 hypothèses de départ.**
Pour chaque ligne, la colonne « Preuve » doit pointer vers un bug (`BUG-0NN`) ou une fiche utilisateur (`UX-0N`). **Une amélioration sans preuve est rayée** : le brief demande de prouver, pas de supposer. Les lignes ci-dessous sont mes candidates avant campagne ; certaines tomberont.

## Grille de priorisation

- **Impact** (1–5) : gain pour l'utilisateur ou le business
- **Effort** (1–5) : coût estimé (1 = un après-midi, 5 = un chantier)
- **Valeur** = Impact − Effort *(grossier, volontairement)*
- **MoSCoW** : Must / Should / Could / Won't (cette fois)

## Candidates

| # | Amélioration | Problème visé | Preuve | Impact | Effort | MoSCoW | Justification |
|---|---|---|---|:-:|:-:|:-:|---|
| 1 | Format d'erreur API unique (code, message clair, aucune trace technique) | Erreurs incohérentes / techniques | `BUG-___` | 4 | 2 | Must | `___` |
| 2 | Validation côté serveur de toutes les entrées (quantités, longueurs, types) | Valeurs aberrantes acceptées | `BUG-___` | 5 | 2 | Must | `___` |
| 3 | Récapitulatif détaillé avant validation : sous-total, promo, frais, total | Total incompréhensible ou incohérent | `BUG-___` / `UX-__` | 4 | 2 | Should | `___` |
| 4 | Afficher les plats / restaurants indisponibles **avant** l'ajout au panier | Commande impossible découverte trop tard | `UX-__` | 3 | 2 | Should | `___` |
| 5 | Retour visuel à chaque action (chargement, succès, échec) | « Je ne sais pas si ça a marché » | `UX-__` | 3 | 1 | Should | `___` |
| 6 | Protéger la création de commande contre le rejeu (idempotence) | Doublons de commande | `BUG-___` | 5 | 3 | Could | `___` |
| 7 | Suite de non-régression API automatisée (Bruno CLI ou pytest) | Retomber dans les mêmes bugs | tous | 4 | 3 | Could | `___` |

## Ordre proposé

1. Tout ce qui est **Must** et lié à un bug Cramé / Brûlé
2. Les **Should** à effort ≤ 2
3. Le reste, après revalidation

## Ce que je ne proposerai pas

Tout ce que je ne peux pas relier à une preuve. *(à remplir en fin de campagne)*

## Branche de correctifs (optionnel)

Seulement après la campagne initiale : `fix/<bug-id>-<mot-clé>`. Chaque correctif revalidé avec le cas de test d'origine, résultat noté dans `execution-report.md` §7.
