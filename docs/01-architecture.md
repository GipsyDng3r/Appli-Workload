# 1. Architecture

## Schéma

```
Mails flaggés (catégorisés) ─┐
Teams (action sur message) ──┤                         ┌─► flag Outlook (terminé)
Loop (monthly eng) ──────────┼─► Flux ─► LISTE ◄───────┼─► To Do (tâches Teams)
Saisie manuelle ─────────────┘          « Workload »   └─► tâche Loop
                                            ▲
                                       Power App
```

La liste SharePoint **Workload** est la source de vérité. Tout le reste est un miroir ou une porte d'entrée.

## Règles de classement des mails flaggés

Les mails sont catégorisés par un flux existant **avant** d'être flaggés. Le flux d'entrée lit les catégories au moment du flag.

| Catégories du mail | Type | Projet |
|---|---|---|
| Support (même combinée à un projet) | Support | Hors projet |
| FA | Amélioration | Hors projet |
| Nom exact d'un projet de la liste Projets | Projet | Ce projet |
| Aucune, ou catégorie inconnue | À trier | Vide |

Ordre de priorité : Support, puis FA, puis un projet connu, sinon À trier.

- Le flux ne crée jamais de catégorie ni de projet. Un nouveau projet s'ajoute à la main dans la liste Projets, avec exactement le nom de la catégorie Outlook.
- Pour un item « À trier », le flux envoie un message Teams (bot Flow) ou un mail. Tu choisis ensuite le type et le projet dans la vue « À trier » de l'app.
- Le cas FA combiné à un projet n'a pas été tranché explicitement. La règle appliquée ici est FA seul, par cohérence avec Support.

## Avancement

- Une tâche a des **étapes** (cases à cocher) stockées dans la liste Étapes.
- Avancement d'une tâche = étapes cochées / total. Terminée = 100 %. Sans étape = 0 % tant qu'elle n'est pas terminée.
- Avancement d'un projet = moyenne des avancements de ses tâches, terminées comprises.
- Les étapes viennent d'un **modèle par type** (liste Modèles d'étapes), copié à la création. Modifier un modèle ne change pas les tâches existantes.

| Type | Étapes par défaut |
|---|---|
| Support | Qualifier la demande, Traiter, Répondre au demandeur |
| Amélioration | Cadrer, Réaliser, Tester, Déployer |
| Monthly eng | Préparer, Présenter, Suivre l'action |
| Projet | Aucune |

## Clôture partout

- **Depuis la liste ou l'app** : passer le Statut à Terminé déclenche un flux qui coche les étapes, termine le flag Outlook et le miroir To Do (tâches Teams), et tente de terminer la tâche Loop.
- **Depuis un autre outil** : possible selon ce que les connecteurs exposent. Voir [05](05-flux-cloture.md) et [08](08-loop-et-tests.md).
- Le flag d'un mail et la tâche To Do correspondante sont déjà synchronisés nativement par Microsoft : terminer l'un termine l'autre.

## Contraintes

- Uniquement des connecteurs standard : SharePoint, Office 365 Outlook, Microsoft Teams, Microsoft To Do (Business), Planner, Approvals.
- Les flux tournent sous ton compte (connexions personnelles). Si ton compte est désactivé, ils s'arrêtent.
- Une politique DLP peut interdire certaines combinaisons de connecteurs. À vérifier au premier test.
- Limites d'actions Power Automate des licences M365 standard : largement suffisantes pour un usage individuel, mais pas pour des milliers de mails par jour.
