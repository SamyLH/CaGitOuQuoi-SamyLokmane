# Où chercher (hypothèses, pas des bugs)

⚠️ **Ce fichier ne contient aucun bug trouvé.** Ce sont des endroits où les bugs aiment se cacher dans ce genre d'appli. Un bug n'entre dans ce dossier qu'une fois reproduit, avec preuve, dans son propre fichier `BUG-0NN.md`.

## Pistes données dans le sujet (`API.md`)
Clé absente / incorrecte · ID inexistant · quantité 0 / négative / supérieure au stock · produit hors stock · restaurant fermé · commande sous le minimum · coupon expiré · coupon avec minimum non atteint · `subtotal` modifié artificiellement sur `/coupons/validate` · paiement `cash` avec montant insuffisant · paiement PayPal · statuts HTTP · contenu JSON.
*(Le sujet dit où regarder, pas ce qui est cassé. À moi de prouver.)*

## Écarts entre `openapi.yaml` et le comportement réel (à vérifier)
La spec documente : quantité de 1 à 20 · 201 / 400 / 401 / 409 sur `POST /orders` · 404 coupon inexistant ou expiré · 403 sur `GET /orders/{id}` · 409 annulation impossible.
Elle ne documente **pas** : le 404 de `/products/{id}`, de `/orders/{id}` ou de `/restaurants/{id}/products` · un 400 sur `/coupons/validate` · un 403 sur `/orders/{id}/cancel`. Que renvoie l'appli dans ces cas ? Un écart entre spec et réalité est un bug de documentation ou de code, selon le cas.

## Séances exploratoires de 30 min (pas de cas dédié)
- Inscription : email déjà pris, mot de passe faible, champs vides
- Panier mono-restaurant : ajouter un produit du restaurant A puis du B
- Adresses de livraison, favoris, avis (note 0, 6, texte vide, XSS), « recommander » une ancienne commande
- Tri par note, prix de livraison, délai : l'ordre affiché est-il vraiment trié ?
- Accès admin avec un compte client (URL directe)
- Déconnexion puis bouton Retour du navigateur

## Calculs
- Total avec 3 articles à prix décimaux : le centime manque-t-il ?
- Promo en % appliquée avant ou après les frais de livraison ? Est-ce cohérent partout ?
- Quantité modifiée après application d'un code promo : le total se recalcule-t-il ?
- Même total dans le panier, la confirmation, l'historique et la réponse API ?

## Frontières
- Quantité 0, 1, 99, 100, -1, 1.5, `abc`
- Seuil de commande minimale : seuil −0,01 / seuil / seuil +0,01
- Champ commentaire : 0, 255, 256, 5000 caractères ; accents, emoji, guillemets, retours à la ligne

## API
- `/coupons/validate` avec un `subtotal` que le client peut falsifier : la commande recalcule-t-elle le vrai sous-total côté serveur ?
- `cash_received` : 0, négatif, juste en dessous du total, très grand
- La commande de bob est-elle visible ou annulable avec la clé d'alice ?
- `GET /api/favorites` : renvoie-t-il bien les favoris de l'utilisateur de la clé ?
- `/api/restaurants?q=` avec des caractères spéciaux ; `min_rating` négatif, 6, `abc`
- Même requête sans `X-API-Key`, avec une clé fausse, avec une clé vide
- `POST` avec corps vide, JSON cassé, champ manquant, type faux (`"quantity": "deux"`)
- `GET` d'un id qui n'existe pas, négatif, énorme, texte
- Mêmes erreurs : format JSON identique d'un endpoint à l'autre ?
- Une erreur renvoie-t-elle une trace technique, un chemin de fichier, un nom de table ?
- `POST` de la même commande deux fois de suite (double clic / rejeu) : doublon ?
- Un code 200 renvoyé alors que l'action a échoué ?

## États
- Annuler une commande déjà préparée / livrée / déjà annulée
- Commander dans un restaurant fermé ou un plat indisponible
- Retour arrière navigateur après confirmation de commande : recommande-t-on sans le vouloir ?
- Rafraîchir la page au milieu du paiement

## Interface
- Messages d'erreur : compréhensibles par quelqu'un qui n'est pas développeur ?
- Rien ne se passe après un clic : y a-t-il un retour visuel ?
- Console DevTools : erreurs rouges au chargement de chaque page ?
- Onglet Network : requêtes en 4xx/5xx que l'utilisateur ne voit pas ?
- Navigation clavier, zoom 200 %, petit écran

## Sécurité de base
- Saisie `<script>alert(1)</script>` et `' OR 1=1--` dans les champs texte : échappé ou exécuté ?
- Accéder à la commande d'un autre utilisateur en changeant l'id dans l'URL / l'API
