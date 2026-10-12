# Rapport d'exécution de la campagne

**Statut : campagne interrompue. Rapport rédigé le 12/10/2026 avec ce qui a réellement été observé, pas avec ce qui était prévu.**

## 1. Contexte

| | |
|---|---|
| Date de campagne | 12/10/2026 (session unique, courte) |
| Testeur | Samy |
| Application | QuickOrder V3.1, lancée via `docker compose up --build`, base fraîchement seedée (état propre) |
| Point de départ | `/api/health` : `status: ok`, horodatage 2026-10-12T20:16:35 (UTC probable) |
| Navigateur / outils | Chrome + DevTools (console), Swagger (`/docs/`) |
| Serveur observé (en-têtes) | Werkzeug/3.1.3, Python/3.12.15 (serveur de développement Flask) |

## 2. Chiffres

| Mesure | Valeur |
|---|:-:|
| Cas prévus | 24 |
| Cas exécutés **en entier** | **0** |
| Cas exécutés **en partie** | 3 (NOM-02, NOM-05, ERR-03) |
| ✅ Pass | 0 |
| ❌ Fail | 0 |
| ⛔ Blocked | 0 |
| ⏭️ Skipped | 24 (dont les 3 partiels, voir ci-dessous) |
| Taux de réussite | non calculable (aucun cas terminé) |

**Pourquoi Skipped ?** Décision du testeur d'arrêter la campagne après les premières vérifications (manque de temps). Aucun cas n'a été bloqué par l'application.

### Ce qui a été réellement observé

| Cas | Observation | Portée |
|---|---|---|
| NOM-02 | Liste de 6 restaurants affichée sur le site ; console DevTools vide (aucune erreur JavaScript) | Partielle : menu non ouvert, onglet Network non relevé |
| NOM-05 | `GET /api/health` sans clé : 200, `{"service":"quickorder","status":"ok"}` · `GET /api/restaurants` avec clé : JSON, `count: 6`, champs conformes à OpenAPI · `GET /api/restaurants/1` : Burger Factory, valeurs identiques à la liste | Partielle : filtres `q`, `min_rating`, `category` non testés, code HTTP non noté |
| ERR-03 | `GET /api/restaurants/999` : « not found » | Partielle : code HTTP à confirmer ; `/products/999`, `/restaurants/999/products`, `/orders/999` non testés |
| (cohérence) | Les 6 restaurants du site (noms, frais, minimums, notes, Night Sushi fermé) correspondent au JSON de l'API | Constat, sans anomalie |

**Aucune anomalie n'a été constatée sur ce périmètre, qui est très réduit.**

## 3. Bugs

| Sévérité | Nombre confirmé |
|---|:-:|
| 🔥 Cramé | 0 |
| 🥵 Brûlé | 0 |
| 🧂 Trop salé | 0 |
| 🦟 Mouche dans la soupe | 0 |

**Aucun bug n'est confirmé.** Le brief en demande au moins 8, reproductibles : cet objectif **n'est pas atteint**. Je n'en ai pas inventé.

Pistes **non prouvées** (à tester, sans valeur de bug tant qu'elles ne sont pas reproduites) :
- Le site annonce « ATLAS15 : -15 % » alors que les paramètres de démo visibles dans le log de démarrage indiquent 10 %.
- Le site annonce « livraison gratuite dès 30 € avec TOKYO30 » ; le log montre une réduction de 100 % plafonnée à 3,49 €. Le comportement réel au panier n'a pas été vérifié.
- Les en-têtes révèlent les versions du serveur (risque faible, serveur de développement).

*(Transparence : les paramètres des coupons sont apparus dans le log de démarrage, pas dans une observation de l'application en fonctionnement.)*

## 4. Risques : paris vs réalité

Aucun risque de `risk-matrix.md` n'a pu être confirmé ou écarté. Les deux risques de score 15 restent **ouverts** :
- **R1 (totaux faux)** : aucun panier ni commande testé.
- **R2 (accès API non protégé)** : la route publique `/health` se comporte comme documenté, mais l'appel **sans clé** sur les routes protégées n'a pas été testé.

Zones non couvertes : panier, checkout, coupons, frais de livraison, paiement (carte, espèces, PayPal), annulation, commandes, authentification, favoris, avis, administration.

## 5. Retours utilisateurs

Non réalisés (aucune session avec 2 personnes × 3 scénarios). Voir `user-feedback.md` : le protocole est prêt, les fiches sont vides.

## 6. Recommandation

- [ ] GO
- [ ] GO sous conditions
- [x] **NO-GO (faute de preuves)**

**Justification.** Ce NO-GO ne repose pas sur des défauts prouvés : aucun n'a été trouvé. Il repose sur le fait que les parcours critiques définis dans la stratégie (P1 : commande de bout en bout, P2 : cohérence des totaux) n'ont **pas été exécutés** : aucune commande n'a été passée ni vérifiée. Les critères posés avant la campagne (`test-strategy.md` §9) exigent des parcours P1/P2 validés pour recommander la mise en production. Avec 3 vérifications partielles sur 24 cas, aucune assurance de qualité ne peut être donnée. De plus, le README de l'application indique que des anomalies sont volontairement conservées : l'absence de bugs trouvés reflète le périmètre testé, pas la qualité de l'application.

## 7. Revalidation

Sans objet : aucun bug confirmé, aucun correctif disponible.

## 8. Limites de cet audit

- Campagne arrêtée volontairement après environ 3 vérifications ; couverture réelle très faible.
- Tests réalisés par une seule personne, en une session, sans second regard.
- Aucune preuve capturée (captures, requêtes sauvegardées) dans `docs/bug-reports/evidence/`.
- Les résultats « partiels » ne valent pas des Pass : ils indiquent seulement qu'aucune anomalie n'est apparue sur ce qui a été regardé.

## 9. Pour compléter (ordre conseillé, environ 1 h)

1. Finir NOM-02 et NOM-05 (5 min) → 2 vrais Pass.
2. ERR-02 : appels sans clé sur `/restaurants`, `/orders`, `POST /orders` (risque R2, 5 min).
3. NOM-03, NOM-04, REG-01 : un panier, un coupon, une commande, en comparant au calcul manuel (risque R1, 20 min).
4. REG-04 (Night Sushi fermé), LIM-01 (quantité 0 et 21), LIM-06 (espèces insuffisantes) : bons candidats à bugs.
5. Seulement alors : réécrire la section 3 et la recommandation.
