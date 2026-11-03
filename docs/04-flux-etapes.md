# 4. Flux « Étapes depuis le modèle »

Crée les étapes d'une tâche à partir du modèle de son type. Se déclenche à la création d'un item, et aussi quand tu fais passer un item de « À trier » à un type.

Type de flux : **Flux cloud automatisé**. Nom : `Workload - Etapes`.

## Pourquoi des doublons apparaissent

Quatre causes possibles, de la plus fréquente à la plus rare :

1. **Deux exécutions en même temps.** Le trigger « créé ou modifié » se déclenche souvent deux fois pour un seul nouvel item : une fois à la création, une autre à une mise à jour qui suit de quelques secondes (la liste, l'application ou un autre flux touche l'item). Les deux exécutions lisent `EtapesCreees = Non` avant que la première ait fini, et créent chacune les étapes. Le contrôle de la concurrence ne protège que s'il est vraiment activé, ce qui n'est pas le cas par défaut.
2. **Un modèle en double.** Si les lignes de `ModelesEtapes` ont été collées deux fois (par exemple le jeu d'essai), chaque étape existe deux fois dans le modèle, donc deux fois dans la tâche. Vérifie cette liste en premier.
3. **Une nouvelle tentative automatique.** Si l'action « Créer un élément » reçoit une erreur réseau après avoir réussi, Power Automate la rejoue et crée une seconde ligne.
4. **Deux flux qui créent des étapes.** Un flux copié, ou la création des étapes ajoutée aussi dans le flux Teams.

Un drapeau lu au début de l'exécution ne peut pas empêcher la cause 1. Seule une contrainte d'unicité dans SharePoint la bloque à coup sûr : c'est ce que fait la colonne `Cle` ci-dessous.

## 0. Préparer la liste `Etapes`

Ajoute la colonne :

| Colonne | Type | Remarques |
|---|---|---|
| `Cle` | Texte, **indexée**, **valeurs uniques** | `<ID de la tâche>-<Ordre>`, par exemple `45-2`. Les étapes ajoutées à la main dans l'application la laissent vide : une valeur vide n'est jamais en conflit. |

Pour l'unicité : paramètres de la colonne → *Plus d'options* → *Appliquer des valeurs uniques* = Oui (SharePoint demande d'indexer la colonne). Une étape créée deux fois pour la même tâche et le même numéro est alors **refusée par SharePoint** : le doublon ne peut plus exister, même si deux exécutions se croisent.

## Trigger

**SharePoint : Lorsqu'un élément est créé ou modifié**
- Liste : Workload

**Conditions de déclenchement** (Paramètres du trigger → Conditions de déclenchement), une par ligne :

```
@not(equals(triggerOutputs()?['body/TypeTache/Value'],'À trier'))
@not(equals(triggerOutputs()?['body/EtapesCreees'],true))
```

**Contrôle de la concurrence** (Paramètres du trigger) : **activé**, degré de parallélisme = 1. Vérifie que l'interrupteur est bien sur *Activé* : il est désactivé par défaut. Ce réglage réduit les croisements, mais la colonne `Cle` est la vraie protection.

## Actions

### 1. Relire l'item

**SharePoint : Obtenir l'élément** — renomme `Item`
- Liste : Workload, ID : *ID* du trigger.

**Condition** : `outputs('Item')?['body/EtapesCreees']` est égal à `true`
- Oui : **Terminer** (Réussi).

### 2. Vérifier qu'il n'y a pas déjà des étapes

**SharePoint : Obtenir les éléments** — renomme `Existantes`
- Liste : Etapes
- Requête de filtre : `TacheId eq @{triggerOutputs()?['body/ID']}`
- Nombre maximal : 1

**Condition** : `length(body('Existantes')?['value'])` est supérieur à `0`
- Oui : **SharePoint : Mettre à jour un élément** (Workload, ID du trigger) → `EtapesCreees = Oui`, puis **Terminer** (Réussi).
- Non : continuer.

Cette étape couvre aussi une tâche à laquelle tu as déjà ajouté des étapes à la main : le modèle ne s'applique pas par-dessus.

### 3. Lire le modèle

**SharePoint : Obtenir les éléments** — renomme `Modele`
- Liste : ModelesEtapes
- Requête de filtre : `TypeModele eq '@{outputs('Item')?['body/TypeTache/Value']}'`
- Trier par : `Ordre asc`

### 4. Créer les étapes

Mets l'action suivante dans un **Étendue** (Scope) nommé `Creation`.

**Appliquer à chacun** sur `body('Modele')?['value']`, **parallélisme = 1** pour garder l'ordre :

**SharePoint : Créer un élément**
- Liste : Etapes
- Title : `@{items('Appliquer_à_chacun')?['Title']}`
- Tache Id : `@{triggerOutputs()?['body/ID']}`
- Fait : Non
- Ordre : `@{items('Appliquer_à_chacun')?['Ordre']}`
- **Cle** : `@{triggerOutputs()?['body/ID']}-@{items('Appliquer_à_chacun')?['Ordre']}`

Dans les **Paramètres** de cette action « Créer un élément », mets **Stratégie de nouvelle tentative = Aucune**. Cela supprime la cause 3.

Pour le type Projet, le modèle est vide : la boucle ne fait rien.

Si deux exécutions se croisent, la seconde reçoit un refus de SharePoint pour « valeur en double » sur `Cle` : c'est le comportement voulu, aucune ligne n'est créée.

### 5. Marquer comme fait

**SharePoint : Mettre à jour un élément**
- Liste : Workload, ID : *ID* du trigger
- EtapesCreees : Oui
- **Configurer l'exécution après** : cocher *a réussi* **et** *a échoué* pour l'étendue `Creation`. Ainsi, une exécution qui a croisé une autre et reçu un refus pour doublon marque quand même la tâche comme traitée.

Cette mise à jour relance le trigger, mais la condition `EtapesCreees = true` l'arrête aussitôt.

### 6. Éviter les exécutions en échec inutiles

Après l'action de l'étape 5, ajoute **Contrôle : Terminer** avec le statut *Réussi*, configuré pour s'exécuter après l'étendue `Creation` quand elle **a échoué**. Sans cela, chaque croisement apparaît comme un échec dans l'historique alors qu'il n'y a rien d'anormal.

## Nettoyer les doublons déjà créés

### Option 1 : repartir de zéro (recommandée pour un jeu d'essai)

1. Dans la liste `ModelesEtapes`, supprime les lignes en double (une seule ligne par type et par numéro).
2. Dans la liste `Etapes`, sélectionne toutes les lignes en mode Grille et supprime-les.
3. Dans `Workload`, remets `EtapesCreees` à **Non** sur les tâches concernées. Le flux recrée les étapes proprement, une seule fois.

Les étapes ajoutées à la main sont perdues avec cette méthode.

### Option 2 : supprimer uniquement les doublons

Flux manuel **« Workload - Nettoyer les étapes »**, à lancer une fois :

1. **Déclencher manuellement un flux**.
2. **SharePoint : Obtenir les éléments** sur `Etapes`, nombre maximal 5000, **Trier par** `ID asc`.
3. **Initialiser une variable** `Vues`, type Tableau, vide.
4. **Appliquer à chacun** sur `body('Obtenir_les_éléments')?['value']`, parallélisme = 1 :
   - **Composer** `Cle` : `concat(string(items('Appliquer_à_chacun')?['Tache']?['Id']),'|',toLower(trim(items('Appliquer_à_chacun')?['Title'])))`
   - **Condition** : `contains(variables('Vues'), outputs('Cle'))` est égal à `true`
     - Oui : **SharePoint : Supprimer un élément** (Etapes, `ID` de l'item).
     - Non : **Ajouter à une variable tableau** `Vues` avec `outputs('Cle')`.

Le flux garde la première étape de chaque titre pour chaque tâche et supprime les suivantes. Il supprimerait aussi une étape volontairement répétée (deux étapes du même nom dans la même tâche) : lance-le une seule fois, sur des données de test ou après vérification.

**À vérifier** : la forme exacte de la colonne de recherche `Tache` dans la sortie (`items(...)?['Tache']?['Id']`). Regarde la sortie de l'étape 2 dans l'historique d'exécution.

## Cas à connaître

- Si tu changes le type d'un item après la création des étapes, elles ne sont pas recréées : `EtapesCreees` est déjà à Oui. Pour appliquer un autre modèle, supprime les étapes et remets `EtapesCreees` à Non.
- Modifier la liste ModelesEtapes ne touche pas les tâches existantes. C'est voulu.
- Si tu supprimes une étape puis remets `EtapesCreees` à Non, le flux la recrée avec la même `Cle` : c'est possible parce que la ligne supprimée libère sa clé.

## Test

1. Crée un item Workload de type Support à la main : trois étapes apparaissent dans Etapes, liées à lui, avec les clés `<ID>-1`, `<ID>-2`, `<ID>-3`.
2. Crée un item À trier : aucune étape. Change son type en Amélioration : quatre étapes apparaissent.
3. Crée un item Projet : aucune étape.
4. Modifie le titre d'un item qui a déjà ses étapes : rien ne se duplique.
5. **Test de croisement** : crée un item de type Support dans l'application, puis change aussitôt son statut et son titre. Les trois étapes ne doivent exister qu'une fois. L'historique du flux peut montrer une exécution terminée sans rien créer : c'est normal.
6. Crée trois items d'affilée très vite : chacun a ses trois étapes, sans doublon.
