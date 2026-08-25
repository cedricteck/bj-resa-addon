# Tennis Resa — add-on Home Assistant

Réservation automatique des courts de tennis sur Balle Jaune, avec une interface de
gestion intégrée à la barre latérale Home Assistant.

## Installation

**Nécessite Home Assistant OS ou Supervised** : les add-ons n'existent pas sur les
installations HA Container ni HA Core.

1. **Paramètres → Modules complémentaires → Boutique → ⋮ → Dépôts**
2. Ajouter `https://github.com/cedricteck/bj-resa-addon`
3. Installer **Tennis Resa** dans la liste qui apparaît, puis **Démarrer**.

L'add-on utilise une **image préconstruite** (`image:` dans `config.yaml`) : la machine
Home Assistant ne compile rien, elle tire l'image publiée sur `ghcr.io`. Il n'y a pas de
bouton *Reconstruire* à utiliser.

## Configuration

| Option | Défaut | Rôle |
| --- | --- | --- |
| `user` | — | Identifiant Balle Jaune, tel qu'affiché par le club (obligatoire) |
| `password` | — | Mot de passe Balle Jaune (obligatoire, masqué) |
| `timezone` | `Europe/Paris` | Fuseau du jour cible et du déclenchement |
| `max_booking_day_away` | `2` | Fenêtre de réservation du club, en jours |
| `guests` | `1` | Nombre d'invités par réservation |
| `cron` | `50 29 22 * * *` | Déclenchement (6 champs : sec min h j m jsem) |
| `race_attempts` | `20` | Tentatives à l'ouverture du créneau |
| `race_delay_ms` | `500` | Délai entre deux tentatives |
| `proxy_url` | *(vide)* | Proxy HTTP explicite. Vide = détection automatique |
| `log_level` | `info` | Verbosité de l'application |

L'identifiant peut comporter des accents sans précaution : les options sont transmises en
JSON, contrairement à `application.properties` que Spring Boot lit en ISO-8859-1.

Les identifiants ne sont **jamais** stockés dans l'image ni dans ce dépôt : ils sont lus
depuis ces options et transmis à l'application par variables d'environnement.

`max_booking_day_away` est **auto-corrigé** : si Balle Jaune sert un jour plus proche que
demandé, c'est celui-là qui fait foi et un avertissement apparaît dans les journaux.

Ces options ne contiennent **pas** les demandes de réservation : elles s'éditent dans
l'interface et vivent dans `/data/requests.json`, persisté par le Supervisor et inclus
dans les sauvegardes Home Assistant.

## Utilisation

Après démarrage, **Tennis Resa** apparaît dans la barre latérale (icône raquette).
L'accès passe par l'ingress : l'authentification est celle de Home Assistant, et aucun
port n'est publié sur le réseau local.

- **Demandes auto** — les créneaux à réserver automatiquement. Une demande *ponctuelle*
  disparaît dès qu'elle est honorée ; une demande *récurrente* (jour de la semaine) est
  rejouée chaque semaine. Le bouton *Lancer un run maintenant* permet de tester sans
  attendre l'heure du cron.

  Une demande ponctuelle n'est honorée que si Balle Jaune sert bien le jour visé : lancée
  en journée, avant l'ouverture de 22h30, elle est **ajournée** plutôt que réservée pour
  aujourd'hui. Le run du soir retente jusqu'à l'ouverture.

  Le champ **Partenaire** accepte un nom libre, cherché dans l'annuaire du club à
  l'enregistrement : tape assez de lettres pour lever l'ambiguïté (prénom + nom). Si
  plusieurs membres correspondent, les candidats sont listés. Laisse le champ vide pour
  réserver avec un invité — c'est alors le champ *Invités* qui compte.
- **Planning du club** — l'état réel des courts pour chaque jour de la fenêtre ouverte.
  Un créneau vert se réserve en un clic ; les vôtres sont surlignés et annulables. Le
  code d'ouverture du portillon s'affiche après une réservation réussie.

  La barre **Réserver avec** propose vos favoris Balle Jaune (le cœur, coché depuis leur
  site) et une recherche pour tout autre membre. Le partenaire retenu est porté par l'URL :
  il reste actif en changeant de jour et après une réservation, pour en enchaîner plusieurs.

## Journaux utiles

| Message | Signification |
| --- | --- |
| `Compte Balle Jaune : '...'` | Identifiant effectivement chargé — à vérifier en cas de refus de login |
| `ballejaune.user n'a pas été résolu` | L'option `user` (ou `password`) est vide : l'add-on s'arrête au lieu d'attendre 22h30 |
| `Accès à ballejaune.com via le proxy ...` | Un proxy a été détecté et est utilisé |
| `Balle Jaune est en version N` | Le site a été déployé : rejouer les tests de parsing |
| `Le ... n'est pas ouvert : Balle Jaune a servi le jour courant` | Normal en journée : le jour visé n'ouvre qu'à 22h30 |
| `N demande(s) ajournée(s) : le ... n'est pas ouvert` | Le jour visé n'est pas encore ouvert ; le run retente jusqu'à l'ouverture |
| `Balle Jaune a servi le ... alors qu'on demandait ...` | Fenêtre de réservation réellement plus courte que la configuration |
| `Contrôle de transport OK` | Connectivité, TLS et parsing du formulaire de login validés |

## Limites

- L'application n'a **aucune notion d'utilisateur**. Avec `panel_admin: false`, tout
  utilisateur Home Assistant peut réserver et annuler sur le compte du club.
- Balle Jaune ne publie pas d'API documentée : l'add-on s'appuie sur les endpoints du
  site. Un déploiement de leur côté peut nécessiter une mise à jour — le canari de
  version le signale dans les journaux.
