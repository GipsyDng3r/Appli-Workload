# Appli-Workload

Suivi centralisé de la charge de travail (projets, support, fiches d'amélioration, tâches de la monthly eng) construit uniquement avec des briques Microsoft 365 standard : SharePoint Lists, Power Automate, Power Apps, Outlook, Teams, To Do. Aucune licence premium, aucun connecteur premium, pas de Dataverse.

Principe : une liste SharePoint est la source de vérité. Les flux Power Automate l'alimentent à partir des mails flaggés (classés par catégories Outlook), de Teams et de Loop. Terminer une tâche à un endroit la termine partout.

## Guide de construction

À suivre dans l'ordre. Ce dépôt ne contient aucun code exécutable : ce sont des instructions à appliquer dans le tenant.

| # | Document | Contenu |
|---|---|---|
| 1 | [docs/01-architecture.md](docs/01-architecture.md) | Principes, règles de classement, schéma |
| 2 | [docs/02-listes-sharepoint.md](docs/02-listes-sharepoint.md) | Les quatre listes et leurs colonnes |
| 3 | [docs/03-flux-mail-flagge.md](docs/03-flux-mail-flagge.md) | Mail flaggé vers la liste, selon les catégories |
| 4 | [docs/04-flux-etapes.md](docs/04-flux-etapes.md) | Étapes créées depuis les modèles par type |
| 5 | [docs/05-flux-cloture.md](docs/05-flux-cloture.md) | Terminer partout, dans les deux sens |
| 6 | [docs/06-flux-teams.md](docs/06-flux-teams.md) | Message Teams vers la liste |
| 7 | [docs/07-power-app.md](docs/07-power-app.md) | Écrans et formules Power Fx |
| 8 | [docs/08-loop-et-tests.md](docs/08-loop-et-tests.md) | Test Loop et plan de recette |
| 9 | [docs/09-power-app-code-a-coller.md](docs/09-power-app-code-a-coller.md) | Écran complet de la Power App à coller dans Studio |
| 10 | [docs/10-categories-outlook.md](docs/10-categories-outlook.md) | Proposer les catégories Outlook comme projets (flux, liste et variante de l'app) |

La maquette interactive de l'application est dans [maquette/workload.html](maquette/workload.html) (à ouvrir dans un navigateur).

## Avertissement

Les noms d'actions, de champs dynamiques et les formules ont été rédigés sans accès au tenant. Les libellés exacts varient selon la version des connecteurs et la langue de l'interface. Chaque point incertain est marqué **À vérifier** dans les documents.
