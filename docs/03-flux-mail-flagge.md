# 3. Flux « Mail flaggé vers Workload »

Crée un item dans Workload quand tu flagges un mail, d'après ses catégories Outlook. Les catégories sont déjà posées par ton flux de catégorisation avant le flag, donc il n'y a ni temporisation ni relecture.

Type de flux : **Flux cloud automatisé**. Nom : `Workload - Mail flaggé`.

## Trigger

**Office 365 Outlook : Quand un e-mail est signalé (V4)**
- Dossier : Boîte de réception (ou le dossier où arrivent les mails à traiter).

**À vérifier** : le contenu dynamique du trigger doit proposer *Catégories*, *Objet*, *ID de message*, *ID de message Internet*. Si *Catégories* est absent, ajoute juste après le trigger l'action *Obtenir un e-mail (V2)* avec l'ID de message et utilise ses sorties à la place.

## Actions

### 1. Éviter les doublons

**SharePoint : Obtenir les éléments** — renomme l'action `Doublon`
- Liste : Workload
- Requête de filtre : `SourceId eq '@{triggerOutputs()?['body/internetMessageId']}'`
- Nombre maximal : 1

**Condition** : `length(body('Doublon')?['value'])` est supérieur à `0`
- Oui : **Terminer** avec le statut *Réussi*. Rien d'autre.
- Non : continuer.

Si tu reflagges un mail déjà terminé, rien ne se passe. La réouverture n'est pas gérée en v1.

### 2. Lire les projets

**SharePoint : Obtenir les éléments** — renomme `Projets`
- Liste : Projets
- Requête de filtre : `StatutProjet eq 'Actif'`

**Composer** — renomme `Categories`
- Entrée : `triggerOutputs()?['body/categories']`

**Filtrer un tableau** — renomme `ProjetsCorrespondants`
- De : `body('Projets')?['value']`
- Condition (mode avancé) : `@contains(outputs('Categories'), item()?['Title'])`

La comparaison de `contains` sur un tableau est sensible à la casse : la catégorie Outlook et le titre du projet doivent être identiques.

### 3. Calculer le type

**Composer** — renomme `Type`

```
@if(contains(outputs('Categories'),'Support'),'Support',
 if(contains(outputs('Categories'),'FA'),'Amélioration',
 if(greater(length(body('ProjetsCorrespondants')),0),'Projet','À trier')))
```

### 4. Calculer le projet

**SharePoint : Obtenir les éléments** — renomme `HorsProjet`
- Liste : Projets, filtre `Title eq 'Hors projet'`, nombre maximal 1.

**Composer** — renomme `ProjetId`

```
@if(equals(outputs('Type'),'Projet'),
    first(body('ProjetsCorrespondants'))?['ID'],
    if(equals(outputs('Type'),'À trier'),null,
       first(body('HorsProjet')?['value'])?['ID']))
```

### 5. Créer l'item

**SharePoint : Créer un élément**
- Liste : Workload
- Title : *Objet* du mail
- TypeTache Value : `@{outputs('Type')}` (saisis l'expression dans le champ personnalisé de la liste de choix)
- Source Value : `Mail`
- Statut Value : `À faire`
- Echeance : `@{addDays(utcNow(),5,'yyyy-MM-dd')}`
- Projet Id : `@{outputs('ProjetId')}`
- LienSource : lien web du mail. **À vérifier** : si *Lien web* n'est pas dans le contenu dynamique, utilise `https://outlook.office.com/mail/deeplink/read/@{encodeUriComponent(triggerOutputs()?['body/id'])}`
- MessageId : `@{triggerOutputs()?['body/id']}`
- SourceId : `@{triggerOutputs()?['body/internetMessageId']}`

Le flux d'étapes ([04](04-flux-etapes.md)) se déclenche sur cette création.

### 6. Te prévenir pour « À trier »

**Condition** : `outputs('Type')` est égal à `À trier`
- Oui : **Microsoft Teams : Publier un message dans un chat ou un canal**
  - Publier en tant que : Flow bot
  - Publier dans : Discuter avec le Flow bot
  - Destinataire : ton adresse
  - Message : `Mail sans catégorie reconnue : @{triggerOutputs()?['body/subject']}. Choisis son type et son projet dans la vue « À trier ».`

## Gestion d'erreur

- Paramètres du flux → active l'e-mail en cas d'échec (Power Automate te prévient par mail).
- Si l'action de création échoue pour cause d'unicité de SourceId, c'est un doublon : c'est le comportement voulu.

## Test

1. Pose la catégorie Support sur un mail, flagge-le : un item Support, projet Hors projet, apparaît.
2. Catégorie FA : item Amélioration.
3. Catégorie du nom d'un projet existant : item Projet rattaché.
4. Catégories Support et un projet : un item Support.
5. Aucune catégorie : item À trier, projet vide, message Teams reçu.
6. Reflagge le même mail : aucun second item.
