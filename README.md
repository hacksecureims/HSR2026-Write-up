# HSR2026-Write-up

Bienvenue sur le dépôt des write-ups des challenges HSR2026.

Ce dépôt contient :

* les write-ups officiels rédigés par les créateurs des challenges ;
* les write-ups proposés par les participants.

## Organisation des dossiers

Chaque challenge possède son propre dossier.

Exemple :

```text
Crypto/
└── RSA_101/
    ├── README.md
    ├── challenge.py
    ├── generate.py
    ├── Alice/
    │   ├── README.md
    │   └── solve.py
    └── Bob/
        ├── README.md
        └── exploit.sage
```

### Write-up officiel

Le fichier `README.md` situé directement dans le dossier du challenge contient le write-up officiel rédigé par le créateur du challenge.

Les fichiers utilisés pour la génération ou la résolution du challenge peuvent également être présents à cet emplacement :

```text
RSA_101/
├── README.md
├── challenge.py
├── generate.py
└── ...
```

### Write-ups des participants

Si vous souhaitez proposer votre propre write-up :

1. Créez un dossier à votre nom ou pseudo dans le dossier du challenge.
2. Placez votre write-up dans un fichier `README.md`.
3. Ajoutez tous les fichiers complémentaires que vous jugez utiles.

Exemple :

```text
RSA_101/
└── Alice/
    ├── README.md
    ├── solve.py
    ├── notes.txt
    └── screenshot.png
```

Cette organisation permet à plusieurs personnes de proposer leur propre solution sans risque de conflit.

## Comment contribuer

### 1. Fork du dépôt

Créez votre propre copie du dépôt en cliquant sur **Fork**.

### 2. Ajoutez votre contribution

Créez votre dossier dans le challenge concerné et ajoutez votre `README.md` ainsi que les éventuels fichiers annexes.

### 3. Committez vos modifications

```bash
git add .
git commit -m "Add write-up for <challenge>"
```

### 4. Poussez les changements

```bash
git push origin main
```

### 5. Créez une Pull Request

Depuis GitHub :

* Cliquez sur **Compare & Pull Request**
* Vérifiez que la cible est bien ce dépôt
* Soumettez votre Pull Request

### 6. Validation

Un mainteneur examinera votre contribution avant son intégration.

## Recommandations

Dans votre `README.md`, vous pouvez notamment inclure :

* Présentation du challenge
* Analyse
* Méthodologie
* Scripts utilisés
* Exploitation
* Flag obtenu
* Conclusion

## Merci

Merci à tous les contributeurs qui participent au partage des connaissances et à la documentation des challenges HSR2026.
