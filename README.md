# Tennis Resa — dépôt d'add-ons Home Assistant

Dépôt d'add-ons Home Assistant pour **Tennis Resa** : réservation automatique des courts
de tennis sur [Balle Jaune](https://ballejaune.com), avec une interface de gestion dans la
barre latérale Home Assistant.

## Ajouter ce dépôt

**Paramètres → Modules complémentaires → Boutique → ⋮ → Dépôts**, puis ajouter :

```
https://github.com/cedricteck/bj-resa-addon
```

**Tennis Resa** apparaît alors dans la boutique. Installation et options :
**[bj_resa/DOCS.md](bj_resa/DOCS.md)**.

Nécessite Home Assistant **OS** ou **Supervised** — les add-ons n'existent pas sur les
installations HA Container ni HA Core.

## Contenu

Ce dépôt ne contient que la déclaration de l'add-on : `repository.yaml` et
`bj_resa/config.yaml`. L'add-on tourne depuis une **image préconstruite** publiée sur
`ghcr.io`, donc le Supervisor n'a besoin ni des sources ni d'un Dockerfile, et ne compile
rien.

Le code source est maintenu dans un dépôt séparé.

## Sécurité

Aucun identifiant n'est stocké ici ni dans l'image : `user` et `password` sont saisis dans
les options de l'add-on, côté Home Assistant, et transmis à l'application par variables
d'environnement au démarrage.
