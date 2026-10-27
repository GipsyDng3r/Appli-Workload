# 6. Flux « Message Teams vers Workload »

Transforme un message Teams en tâche de support. Se lance à la demande depuis le menu `…` d'un message.

Type de flux : **Flux cloud instantané**. Nom : `Workload - Message Teams`.

## Trigger

**Microsoft Teams : Pour un message sélectionné**
- Disponible pour : toi seul (ou ton équipe, au choix).
- Le flux apparaît ensuite dans Teams : survol d'un message → `…` → *Autres actions* → le nom du flux.

**À vérifier** : le déclencheur peut être absent si l'administrateur a désactivé les extensions de message Power Automate dans Teams. Dans ce cas, repli : flagger le message Teams en le partageant dans Outlook, ou créer l'item à la main dans l'app.

## Actions

**1. Éviter les doublons**
- *SharePoint : Obtenir les éléments*, liste Workload, filtre `SourceId eq '@{triggerBody()?['messageLink']}'`, maximum 1.
- Condition : un item existe déjà → **Répondre** / **Terminer** sans rien créer.

**2. Chercher le projet par défaut**
- *Obtenir les éléments*, liste Projets, filtre `Title eq 'Hors projet'`, maximum 1.

**3. Créer l'item**
- *SharePoint : Créer un élément*, liste Workload :
  - Title : début du texte du message (`substring` limité à 120 caractères) ou un titre saisi dans une entrée du flux
  - TypeTache Value : `Support`
  - Source Value : `Teams`
  - Statut Value : `À faire`
  - Echeance : `@{addDays(utcNow(),5,'yyyy-MM-dd')}`
  - Projet Id : l'ID de « Hors projet »
  - LienSource : le lien du message (`messageLink`)
  - SourceId : le lien du message

Les noms de champs du trigger (`messageLink`, `messageText`, etc.) **sont à vérifier** dans le contenu dynamique.

**4. Créer le miroir To Do**
- *Microsoft To Do (Business) : Ajouter une tâche (V3)* — liste « Workload » (à créer une fois dans To Do), titre = Title de l'item, échéance = Echeance.
- *SharePoint : Mettre à jour un élément* : `ToDoTaskId` = identifiant de la tâche créée.

Le flux « Étapes » ([04](04-flux-etapes.md)) crée les étapes du modèle Support dès la création.

## Pourquoi un miroir To Do ici et pas pour les mails

Un mail flaggé a déjà sa tâche To Do, synchronisée par Microsoft avec le flag. Un message Teams n'en a pas, d'où le miroir. Il permet de voir ces tâches dans « Mon jour » de To Do et de les terminer depuis To Do (sens entrant géré dans [05](05-flux-cloture.md)).

## Test

1. Lance le flux sur un message : item Support, source Teams, trois étapes, tâche dans la liste To Do « Workload ».
2. Relance-le sur le même message : aucun doublon.
3. Termine l'item dans l'app : la tâche To Do se termine.
