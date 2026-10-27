# 2. Les listes SharePoint

Crée les listes dans un site SharePoint dont tu es propriétaire (un site d'équipe privé convient). Utilise « Listes » (Microsoft Lists) ou la page « Contenu du site ».

**Astuce nommage.** Crée chaque colonne avec un nom interne sans espace ni accent (ex. `TypeTache`), puis renomme le nom affiché si tu veux. Le nom interne ne change plus après la création, et c'est lui qu'utilisent les flux et Power Apps.

## Liste `Projets`

| Colonne (nom interne) | Type | Remarques |
|---|---|---|
| Title | Texte | **Nom exact de la catégorie Outlook** (casse comprise) |
| StatutProjet | Choix : Actif, Terminé | Défaut : Actif |

Ajoute une ligne `Hors projet` (Actif). Elle sert de projet par défaut aux types Support et Amélioration.

## Liste `Workload`

| Colonne | Type | Remarques |
|---|---|---|
| Title | Texte | Intitulé de la tâche |
| TypeTache | Choix : Support, Amélioration, Monthly eng, Projet, À trier | |
| Source | Choix : Mail, Teams, Loop, Manuel | |
| Statut | Choix : À faire, En cours, Bloqué, Terminé | Défaut : À faire |
| Priorite | Choix : Haute, Normale, Basse | Optionnel |
| Echeance | Date | |
| Projet | Recherche vers `Projets` (colonne Title) | Vide pour « À trier » |
| LienSource | Lien hypertexte ou texte | URL du mail, du message Teams ou de la tâche Loop |
| MessageId | Texte | Identifiant Outlook du mail, pour terminer le flag |
| SourceId | Texte, **indexée**, **valeurs uniques** | Identifiant stable de la source (voir ci-dessous) |
| ToDoTaskId | Texte | Identifiant de la tâche miroir dans To Do (Teams, manuel) |
| TermineLe | Date et heure | Posée à la clôture |
| Propage | Oui/Non, défaut Non | La clôture a-t-elle été propagée ? Sert d'anti-boucle |
| EtapesCreees | Oui/Non, défaut Non | Les étapes du modèle ont-elles été créées ? |

`SourceId` :
- mail : `Internet Message Id` du mail (identifiant stable) ;
- Teams : lien ou identifiant du message ;
- Loop : identifiant de la tâche si exposé, sinon vide.

Pour l'unicité : paramètres de la colonne → *Plus d'options* → *Appliquer des valeurs uniques* = Oui (SharePoint demande d'indexer la colonne). Un second flux qui tenterait de créer le même item échoue au lieu de créer un doublon.

Vues à créer :
- **Ouvertes** : filtre Statut différent de Terminé, tri Échéance croissant.
- **À trier** : filtre TypeTache = À trier et Statut différent de Terminé.
- **En retard** : Échéance < [Aujourd'hui] et Statut différent de Terminé.
- **Par projet** : regroupement par Projet.
- **Terminées** : Statut = Terminé, tri TermineLe décroissant.

## Liste `Etapes`

| Colonne | Type | Remarques |
|---|---|---|
| Title | Texte | Libellé de l'étape |
| Tache | Recherche vers `Workload` (colonne Title) | Suppression en cascade : *Appliquer le comportement relationnel* → *Supprimer en cascade* |
| Fait | Oui/Non, défaut Non | |
| Ordre | Nombre | Tri dans l'app |

## Liste `ModelesEtapes`

| Colonne | Type | Remarques |
|---|---|---|
| Title | Texte | Libellé de l'étape |
| TypeModele | Choix : Support, Amélioration, Monthly eng, Projet | |
| Ordre | Nombre | |

Contenu initial :

| TypeModele | Ordre | Title |
|---|---|---|
| Support | 1 | Qualifier la demande |
| Support | 2 | Traiter |
| Support | 3 | Répondre au demandeur |
| Amélioration | 1 | Cadrer |
| Amélioration | 2 | Réaliser |
| Amélioration | 3 | Tester |
| Amélioration | 4 | Déployer |
| Monthly eng | 1 | Préparer |
| Monthly eng | 2 | Présenter |
| Monthly eng | 3 | Suivre l'action |

Le type Projet n'a aucune ligne : tu ajoutes les étapes à la main.

Astuce : une liste SharePoint permet de coller plusieurs lignes depuis Excel avec « Modifier en mode Grille ».

## Vérification

- Ajoute une ligne de test dans Workload à la main, puis supprime-la.
- Vérifie que tu peux créer deux lignes avec le même SourceId : la seconde doit être refusée.
