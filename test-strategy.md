# Stratégie de test — QuickOrder

*Écrite avant d'avoir ouvert l'appli. Je la corrigerai après la première heure d'exploration, et je garderai les ratures visibles dans l'historique git. C'est normal.*

## 1. Contexte

Plateforme de livraison de repas (Python, Flask, SQLite), déjà développée. Le README de l'appli annonce que certaines anomalies sont **volontairement conservées** pour créer des situations de test réalistes : il y en a donc à trouver, et ma tâche est de les prouver. Je suis l'équipe QA avant la mise en prod. La question n'est pas « est-ce que ça marche ? » mais **« est-ce qu'on prend le risque d'ouvrir ? »**

- Application : `http://127.0.0.1:5000`
- API / Swagger : `http://127.0.0.1:5000/docs/` (clé `X-API-Key` : `quickorder-demo-key`)
- API de base : `http://127.0.0.1:5000/api` (la clé donne accès au compte **alice**)
- Comptes de démo : `alice@example.com` / `Password123!` · `bob@example.com` / `Password123!` · admin `admin@quickorder.local` / `Admin123!`
- Collection Postman fournie (`quickorder-api.postman_collection.json`, variables `baseUrl`, `apiKey`, `restaurantId`, `productId`, `orderId`) : importable dans Bruno ou Postman

### Environnement de campagne

- Lancement : Docker (`docker compose up --build`) ou Python + venv (`python seed.py` puis `python run.py`)
- ⚠️ La base est **modifiée par mes tests** (commandes créées, stocks décrémentés). Pour repartir d'un état connu : `python seed.py` ou `docker compose down -v` puis relancer. Je note dans le rapport l'état de la base au début de chaque session, sinon deux exécutions du même cas peuvent donner deux résultats.
- Vérification de départ : `/api/health` renvoie `status: ok`, les restaurants s'affichent, un checkout passe (check-list d'`INSTALLATION.md`, partie D)

## 2. Périmètre (ce que je teste)

- Authentification (connexion / déconnexion)
- Consultation des restaurants et des menus
- Panier : ajout, modification de quantité, suppression, calcul du total
- Passage de commande et confirmation
- Suivi / historique des commandes, annulation
- Règles métier : codes promo, frais de livraison, seuil minimum, disponibilité des plats / restaurants
- API REST (`/api`) : health, restaurants (filtres `q`, `min_rating`), produits (`vegetarian`), `coupons/validate`, `orders` (création, détail, annulation), `favorites` ; codes de retour, format des erreurs, authentification par clé
- Modes de paiement : carte, espèces (`cash_received`), PayPal
- Fonctions annexes vues en exploration (30 min, sans cas dédié) : inscription, adresses, favoris, avis, recommander, tri, panier mono-restaurant, accès admin
- Cohérence UI ↔ API (même commande = mêmes montants partout)
- Tests utilisateurs : 3 scénarios × 2 personnes minimum

## 3. Hors périmètre (ce que je ne teste pas, et pourquoi)

| Sujet | Raison |
|---|---|
| Tests de charge / performance chiffrée | Pas d'outil ni d'environnement dédié ; je noterai seulement les lenteurs visibles |
| Pentest approfondi | Hors compétence du projet ; je fais seulement des vérifications de base (saisie hostile, accès sans clé) |
| Paiement réel (aucun vrai débit) | Environnement de démo ; je teste seulement les règles (montant en espèces, comportement PayPal) |
| Espace admin (`admin@quickorder.local`) | Pas de parcours admin au brief ; seulement une exploration courte : un client peut-il y accéder ? |
| Tests automatisés existants (`pytest -q`) | Je ne les modifie pas ; les lancer est optionnel, ils ne remplacent pas ma campagne |
| Compatibilité navigateurs larges | Chrome uniquement ; un second navigateur si le temps le permet (non garanti) |
| Accessibilité normative (RGAA/WCAG complet) | Je note ce que je vois (contrastes, clavier), sans audit formel |
| Revue du code source | Je teste en boîte noire (comme un vrai recetteur) ; le code ne doit **pas être modifié** pendant la campagne initiale |

## 4. Parcours critiques

Un parcours est critique si **s'il casse, on ne gagne plus d'argent ou on perd la confiance.**

| # | Parcours | Pourquoi critique |
|---|---|---|
| P1 | Connexion → choisir un restaurant → remplir le panier → commander → confirmation | C'est le produit. S'il casse, il n'y a plus d'entreprise |
| P2 | Total correct à chaque étape (panier, frais, promo, confirmation) | Un total faux = client perdu ou argent perdu |
| P3 | Suivre une commande et l'annuler | Moment d'angoisse du client ; erreurs = litiges |
| P4 | Appels API (commande, lecture) avec et sans clé | Partenaires et front dépendent de l'API |
| P5 | Application des règles métier (promo, seuil, fermeture) | Règles ignorées = pertes ou réclamations |

## 5. Risques

Détaillés dans [`risk-matrix.md`](risk-matrix.md). Les plus gros paris avant d'avoir vu l'appli : totaux incohérents, accès API insuffisamment protégé, règles de promo / annulation mal appliquées, erreurs non gérées qui fuitent des détails techniques.

## 6. Types de tests

| Type | Quoi | Outils |
|---|---|---|
| Fonctionnel nominal | Les parcours qui doivent marcher | Navigateur |
| Valeurs limites | 0, 1, max, max+1, seuils, longueurs | Navigateur + API |
| Cas d'erreur | Mauvaises entrées, droits manquants, ressources absentes | Swagger, DevTools |
| Règles métier | Promo, frais, annulation, disponibilité | Navigateur + API |
| Exploratoire (sessions de 30 min) | Chercher ce que je n'ai pas prévu | Navigateur, DevTools |
| API | Codes HTTP, schéma JSON, auth, cohérence avec l'UI | Swagger, Bruno/Postman |
| Conformité à la spec | Comparer le comportement réel de l'API à `openapi.yaml` (codes, champs, limites comme quantité 1 à 20) ; noter aussi ce que la spec ne documente pas | Swagger, Bruno/Postman |
| Tests utilisateurs | Où les vraies personnes bloquent | Protocole `user-feedback.md` |
| Revalidation | Rejouer les cas Fail quand un correctif existe | Mêmes cas |

## 7. Priorisation

Score = **probabilité (1–5) × impact (1–5)**. Détails dans la matrice de risques.

- 15–25 : on teste en premier, et deux fois si besoin
- 8–14 : campagne normale
- 1–7 : si le temps le permet

## 8. Critères d'entrée / sortie

**Entrée** : appli accessible, Swagger ouvert, comptes connus, clé API OK.

**Sortie** : tous les cas exécutés (Pass / Fail / Blocked / Skipped justifié), chaque Fail relié à un bug, tests utilisateurs faits, rapport rédigé.

## 9. Critères Go / No-Go (fixés avant de tester, pour ne pas les adapter au résultat)

- **No-Go** si : au moins 1 bug 🔥 Cramé ouvert **ou** un parcours P1/P2 en échec
- **Go sous conditions** si : aucun Cramé, mais des 🥵 Brûlé ouverts avec correctif planifié
- **Go** si : uniquement des 🧂 / 🦟 ouverts

*(Ces seuils sont mon choix, pas ceux du brief. Je les assume et je les défends dans le rapport.)*

## 10. Règle d'or

Le code source de QuickOrder **ne bouge pas** pendant la campagne initiale. Tout correctif éventuel vit dans une branche séparée, après.

## 11. Preuves

Pour chaque Fail : capture d'écran **ou** requête + réponse API **ou** log console, rangée dans `docs/bug-reports/evidence/<ID-du-bug>/`.
