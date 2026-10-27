# 7. La Power App

Application **canvas** (format Tablette) branchée sur les listes SharePoint. La maquette [maquette/workload.html](../maquette/workload.html) donne le rendu visé. Les contrôles Power Apps sont moins arrondis, la structure reste la même.

Les formules ci-dessous n'ont pas été exécutées. Adapte les noms de contrôles et de colonnes à ce que tu as créé, et corrige selon les messages d'erreur de l'éditeur.

## Source de données

Ajoute les connexions SharePoint pour : Workload, Projets, Etapes, ModelesEtapes.

Paramètres → Général : porte la **limite de lignes de données** à 2000. Les listes restent petites, donc la délégation n'est pas un problème tant que tu restes sous ce seuil. Archive les tâches terminées depuis longtemps pour t'en tenir éloigné.

## Écran unique `scrMain`

Disposition : rail de vues à gauche, liste au centre, détail à droite.

### Chargement

`App.OnStart` :

```
Set(varVue, "today");
Set(varProjet, Blank());
Set(varSel, Blank());
```

`scrMain.OnVisible` : appelle une procédure de rafraîchissement.

```
Refresh(Workload); Refresh(Etapes);
ClearCollect(colEtapes, Etapes);
ClearCollect(colPct,
  ForAll(Workload As w,
    {
      ID: w.ID,
      ProjetId: w.Projet.Id,
      pct: If(w.Statut.Value = "Terminé", 100,
        With({ e: Filter(colEtapes, Tache.Id = w.ID) },
          If(CountRows(e) = 0, 0, Round(100 * CountRows(Filter(e, Fait)) / CountRows(e), 0))))
    }));
```

Rappelle ce bloc après chaque modification d'étape ou de statut (mets-le dans un bouton « Actualiser » ou dans une variable de contexte).

### Rail de vues

Boutons qui font `Set(varVue, "today")`, `"open"`, `"late"`, `"done"`, `"trier"`, `"models"`. Un bouton par projet : `Set(varProjet, ThisItem.Title); Set(varVue, "open")`. La galerie des projets a pour `Items` : `Filter(Projets, StatutProjet.Value = "Actif")`.

### Bandeau de chiffres

```
Ouvertes     = CountRows(Filter(Workload, Statut.Value <> "Terminé"))
En retard    = CountRows(Filter(Workload, Statut.Value <> "Terminé", Echeance < Today()))
Terminées (semaine) = CountRows(Filter(Workload, Statut.Value = "Terminé", TermineLe >= DateAdd(Today(), -7, TimeUnit.Days)))
Avancement global = Round(Average(colPct, pct), 0) & " %"
```

### Cartes d'avancement par projet

Galerie horizontale `galProjets`, `Items` = projets actifs. Dans chaque carte :

```
Avancement : Round(Average(Filter(colPct, ProjetId = ThisItem.ID), pct), 0)
Terminées / total :
  CountRows(Filter(Workload, Projet.Id = ThisItem.ID, Statut.Value = "Terminé")) & " sur " &
  CountRows(Filter(Workload, Projet.Id = ThisItem.ID))
```

Barre de progression : un rectangle dont `Width` = `Parent.Width * avancement / 100`.

### Liste des tâches `galTaches`

```
SortByColumns(
  Filter(Workload,
    (varVue = "today" && Statut.Value <> "Terminé" && Echeance <= Today()) ||
    (varVue = "open"  && Statut.Value <> "Terminé") ||
    (varVue = "late"  && Statut.Value <> "Terminé" && Echeance < Today()) ||
    (varVue = "done"  && Statut.Value = "Terminé") ||
    (varVue = "trier" && TypeTache.Value = "À trier" && Statut.Value <> "Terminé"),
    IsBlank(varProjet) || Projet.Value = varProjet,
    IsBlank(txtRecherche.Text) || txtRecherche.Text in Title),
  "Echeance", SortOrder.Ascending)
```

Filtres par type : une collection `colTypes` alimentée par les pastilles, et une ligne supplémentaire `CountRows(colTypes) = 0 || TypeTache.Value in colTypes.Type` dans le `Filter`.

Dans chaque ligne :
- Pastille de type : `ThisItem.TypeTache.Value` avec une couleur par type.
- Échéance en rouge si `ThisItem.Echeance < Today() && ThisItem.Statut.Value <> "Terminé"`.
- Mini-barre : `LookUp(colPct, ID = ThisItem.ID, pct)`.
- `OnSelect` : `Set(varSel, ThisItem)`.

### Volet de détail

Visible si `!IsBlank(varSel)`.

Étapes : galerie `galEtapes`

```
Items : SortByColumns(Filter(colEtapes, Tache.Id = varSel.ID), "Ordre")
Case à cocher OnCheck / OnUncheck :
  Patch(Etapes, LookUp(Etapes, ID = ThisItem.ID), { Fait: !ThisItem.Fait });
  Refresh(Etapes); ClearCollect(colEtapes, Etapes);
  /* puis recalcule colPct comme dans OnVisible */
```

Si une tâche « À faire » reçoit sa première case cochée, passe-la à En cours :
```
If(varSel.Statut.Value = "À faire",
  Patch(Workload, LookUp(Workload, ID = varSel.ID), { Statut: { Value: "En cours" } }))
```

Ajouter une étape :
```
Patch(Etapes, Defaults(Etapes),
  { Title: txtEtape.Text, Tache: { Id: varSel.ID, Value: varSel.Title },
    Fait: false, Ordre: CountRows(Filter(colEtapes, Tache.Id = varSel.ID)) + 1 });
Reset(txtEtape); Refresh(Etapes); ClearCollect(colEtapes, Etapes)
```

Bouton **Terminer partout** :

```
Patch(Workload, LookUp(Workload, ID = varSel.ID),
  { Statut: { Value: "Terminé" }, TermineLe: Now() });
Refresh(Workload);
Notify("Clôture envoyée à Outlook, To Do et Loop par le flux.", NotificationType.Success)
```

Le bouton ne touche ni aux étapes ni à Outlook : c'est le flux [05](05-flux-cloture.md) qui s'en charge. Les cases apparaîtront cochées au prochain rafraîchissement de `colEtapes` (ajoute un bouton Actualiser ou un `Timer` de 30 secondes).

Affichage de la zone « Sera clôturée dans » : Source Mail → Flag Outlook et To Do ; Teams ou manuel → To Do ; Loop → Loop. Choisis les libellés avec un `Switch(varSel.Source.Value, ...)`.

### Vue « À trier »

Quand `varVue = "trier"`, le volet de détail affiche deux listes déroulantes :

```
ddType.Items   : Choices(Workload.TypeTache)  (sans « À trier »)
ddProjet.Items : Filter(Projets, StatutProjet.Value = "Actif")
Bouton « Classer » :
  Patch(Workload, LookUp(Workload, ID = varSel.ID),
    { TypeTache: { Value: ddType.Selected.Value },
      Projet: { Id: ddProjet.Selected.ID, Value: ddProjet.Selected.Title } })
```

Le flux d'étapes [04](04-flux-etapes.md) crée alors les étapes du modèle choisi.

### Vue « Modèles d'étapes »

Galerie sur `ModelesEtapes`, filtrée par `TypeModele`, avec ajout, modification et suppression via `Patch` et `Remove`. Pour un premier jet, modifie plutôt la liste directement dans SharePoint.

## Publication et partage

Enregistre et publie. Épingle l'app dans Teams (onglet Power Apps) si tu veux l'ouvrir au même endroit que tes messages.

## Test

1. Les compteurs et les barres de projet correspondent à ce que montre la liste SharePoint.
2. Cocher une étape fait monter la barre de la tâche, puis celle du projet.
3. Terminer une tâche déclenche le flux de clôture : vérifie dans l'historique du flux.
4. Classer un item « À trier » lui crée ses étapes.
