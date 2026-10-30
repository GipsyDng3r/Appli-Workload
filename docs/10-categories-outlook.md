# 10. Proposer les catégories Outlook comme projets

Objectif : quand tu crées une tâche à la main ou que tu tries un mail, la liste déroulante des projets propose **tes catégories Outlook**. Choisir une catégorie qui n'est pas encore un projet crée le projet à ce moment-là, sur ton action. Le flux de mail, lui, ne crée toujours aucun projet tout seul.

## Principe

```
Outlook (catégories) ─► flux « Catégories Outlook » ─► liste [LME] Categories ─► Power App
                          (quotidien)                    (copie locale)          (liste déroulante)
```

Le flux copie les catégories dans une petite liste SharePoint. L'application lit cette liste, ce qui est rapide et ne dépend pas d'un appel Outlook à chaque ouverture. Elle y ajoute les projets déjà existants dans `[LME] Projets`, sans doublon.

## Ce qui n'est pas garanti

Je n'ai pas pu confirmer qu'une action standard du connecteur Office 365 Outlook lise la liste des catégories. La voie A ci-dessous repose sur l'action « Envoyer une requête HTTP » de ce connecteur, qui ne demande que des licences standard ([guide Collab365](https://go.collab365.com/outlook-categories-in-power-automate), [guide de Matthew Devaney](https://www.matthewdevaney.com/how-to-set-an-email-category-in-power-automate/)). Ces guides décrivent la relecture des catégories d'un message, pas la liste maîtresse : l'adresse de la liste maîtresse et les autorisations requises sont à vérifier chez toi. Certains tenants bloquent aussi cette action. Les voies B et C existent pour ce cas.

## 1. Créer la liste `[LME] Categories`

| Colonne (nom interne) | Type | Remarques |
|---|---|---|
| Title | Texte | Nom exact de la catégorie Outlook |
| Couleur | Texte | Code de couleur renvoyé par Outlook (informatif) |
| Exclue | Oui/Non, défaut Non | Mets Oui pour les catégories qui ne sont pas des projets (par exemple Support et FA) : elles ne seront plus proposées |

Le flux ne modifie jamais `Exclue` : ton choix reste en place d'une exécution à l'autre.

## 2. Le flux « Workload - Catégories Outlook »

Type : **Flux cloud planifié**, une fois par jour. Tu peux aussi le lancer à la main depuis la page du flux.

### Voie A : liste maîtresse via Graph (à tester en premier)

1. **Office 365 Outlook : Envoyer une requête HTTP**, renomme `Categories_Outlook`
   - URI : `https://graph.microsoft.com/v1.0/me/outlook/masterCategories`
   - Méthode : GET
2. **Analyser JSON** sur `body('Categories_Outlook')` avec ce schéma :
   ```json
   {
     "type": "object",
     "properties": {
       "value": {
         "type": "array",
         "items": {
           "type": "object",
           "properties": {
             "displayName": { "type": "string" },
             "color": { "type": "string" }
           }
         }
       }
     }
   }
   ```
3. **SharePoint : Obtenir les éléments** sur `[LME] Categories`, nombre maximal 500, renomme `Existantes`.
4. **Sélectionner**, renomme `NomsExistants` : de `body('Existantes')?['value']`, mappage `item()?['Title']`.
5. **Appliquer à chacun** sur `body('Analyser_JSON')?['value']`, avec une **Condition** :
   - `contains(body('NomsExistants'), items('Appliquer_à_chacun')?['displayName'])` est égal à `false`
   - Oui : **SharePoint : Créer un élément**
     - Title : `@{items('Appliquer_à_chacun')?['displayName']}`
     - Couleur : `@{items('Appliquer_à_chacun')?['color']}`
     - Exclue : `@{or(equals(items('Appliquer_à_chacun')?['displayName'],'Support'),equals(items('Appliquer_à_chacun')?['displayName'],'FA'))}`

Une catégorie renommée dans Outlook apparaît comme une nouvelle ligne ; l'ancienne reste. Supprime-la à la main dans la liste.

### Voie B : catégories vues sur tes derniers mails

À utiliser si la voie A est refusée (erreur 403 ou action bloquée).

1. **Office 365 Outlook : Obtenir les e-mails (V3)** : dossier Boîte de réception, nombre maximal 250.
2. **Initialiser une variable** `Vues`, type Tableau, vide.
3. **Appliquer à chacun** sur les e-mails, puis un second **Appliquer à chacun** sur `items('Appliquer_à_chacun')?['categories']` avec la condition `contains(variables('Vues'), items('Appliquer_à_chacun_2'))` égal à `false`, puis **Ajouter à une variable tableau** `Vues`.
4. Reprends ensuite les étapes 3 à 5 de la voie A en remplaçant la source par `variables('Vues')` (la catégorie est directement le texte `items(...)`, sans `displayName` ni `color`).

Limite : seules les catégories déjà posées sur des mails récents apparaissent, pas celles que tu n'as pas encore utilisées.

### Voie C : saisie à la main

Remplis la liste `[LME] Categories` toi-même, en collant tes catégories depuis Outlook (Catégoriser → Toutes les catégories). C'est ce qui marche à coup sûr, et ça ne demande aucun flux.

### Test

1. Lance le flux à la main.
2. Ouvre son historique : l'action HTTP doit renvoyer le code 200 et le corps doit contenir `value` avec tes catégories.
3. Ouvre la liste `[LME] Categories` : une ligne par catégorie. Passe `Exclue` à Oui pour Support, FA, et toute catégorie qui n'est pas un projet.

## 3. Brancher l'application

1. Dans Power Apps Studio, ajoute la source `[LME] Categories` (menu Données).
2. Remplace `scrMain` par la variante [`power-app/scrMain.avec-categories.pa.yaml`](../power-app/scrMain.avec-categories.pa.yaml) : supprime l'ancien écran, puis clic droit sur la liste des écrans et **Coller le code**.
3. Clic droit sur `scrMain`, puis **Exécuter OnVisible**.

Ne colle pas cette variante avant d'avoir ajouté la source : tous les appels à `[LME] Categories` seraient en erreur.

## Ce que change la variante

- **Barre du haut** : une liste déroulante « projet » apparaît à côté du champ « Nouvelle tâche manuelle ». Elle propose « (sans projet) », puis tes catégories (hors celles marquées `Exclue`) et les projets actifs de `[LME] Projets`, sans doublon, triés par nom.
- **Création manuelle** : avec « (sans projet) », la tâche arrive dans « À trier », comme avant. Avec un projet choisi, elle est créée directement en type Projet, rattachée à ce projet, et tu arrives sur la vue Tâches.
- **Vue « À trier »** : la liste déroulante des projets propose les mêmes choix.
- **Création du projet** : si la catégorie choisie n'existe pas encore dans `[LME] Projets`, la ligne est créée à ce moment-là (statut Actif). C'est une action de ta part, pas une création automatique.
- **Chargement** : trois nouvelles collections (`colProjOptions`, `colProjChoix`, et la lecture de `[LME] Categories`) se remplissent à l'ouverture et sur « Actualiser ». Si une étape échoue, un bandeau rouge nomme la collection.

## Cohérence avec le flux de mail

Le flux « Mail flaggé » ([03](03-flux-mail-flagge.md)) classe un mail en projet seulement si sa catégorie existe déjà dans `[LME] Projets`. Une catégorie que tu choisis ici crée le projet : les prochains mails portant cette catégorie seront donc classés tout seuls.
