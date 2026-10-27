# 4. Flux « Étapes depuis le modèle »

Crée les étapes d'une tâche à partir du modèle de son type. Se déclenche à la création d'un item, et aussi quand tu fais passer un item de « À trier » à un type.

Type de flux : **Flux cloud automatisé**. Nom : `Workload - Etapes`.

## Trigger

**SharePoint : Lorsqu'un élément est créé ou modifié**
- Liste : Workload

**Conditions de déclenchement** (Paramètres du trigger → Conditions de déclenchement), une par ligne :

```
@not(equals(triggerOutputs()?['body/TypeTache/Value'],'À trier'))
@not(equals(triggerOutputs()?['body/EtapesCreees'],true))
```

**Contrôle de la concurrence** (Paramètres du trigger) : activé, degré de parallélisme = 1. Cela évite que deux exécutions créent les étapes en double.

## Actions

### 1. Relire l'item

**SharePoint : Obtenir l'élément** — renomme `Item`
- Liste : Workload, ID : *ID* du trigger.

**Condition** : `outputs('Item')?['body/EtapesCreees']` est égal à `true`
- Oui : **Terminer** (Réussi).

La relecture protège du cas où une exécution précédente a déjà tout créé.

### 2. Lire le modèle

**SharePoint : Obtenir les éléments** — renomme `Modele`
- Liste : ModelesEtapes
- Requête de filtre : `TypeModele eq '@{outputs('Item')?['body/TypeTache/Value']}'`
- Trier par : `Ordre asc`

### 3. Créer les étapes

**Appliquer à chacun** sur `body('Modele')?['value']`, **parallélisme = 1** pour garder l'ordre :

**SharePoint : Créer un élément**
- Liste : Etapes
- Title : `@{items('Appliquer_à_chacun')?['Title']}`
- Tache Id : `@{triggerOutputs()?['body/ID']}`
- Fait : Non
- Ordre : `@{items('Appliquer_à_chacun')?['Ordre']}`

Pour le type Projet, le modèle est vide : la boucle ne fait rien.

### 4. Marquer comme fait

**SharePoint : Mettre à jour un élément**
- Liste : Workload, ID : *ID* du trigger
- EtapesCreees : Oui

Cette mise à jour relance le trigger, mais la condition `EtapesCreees = true` l'arrête aussitôt.

## Cas à connaître

- Si tu changes le type d'un item après la création des étapes, elles ne sont pas recréées : `EtapesCreees` est déjà à Oui. Pour appliquer un autre modèle, supprime les étapes et remets `EtapesCreees` à Non.
- Modifier la liste ModelesEtapes ne touche pas les tâches existantes. C'est voulu.

## Test

1. Crée un item Workload de type Support à la main : trois étapes apparaissent dans Etapes, liées à lui.
2. Crée un item À trier : aucune étape. Change son type en Amélioration : quatre étapes apparaissent.
3. Crée un item Projet : aucune étape.
4. Modifie le titre d'un item qui a déjà ses étapes : rien ne se duplique.
