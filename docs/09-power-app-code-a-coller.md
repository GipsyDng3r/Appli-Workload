# 9. Power App : code à coller

Ce document remplace la construction à la main de l'écran décrite en [07](07-power-app.md). Tu colles le code de l'écran dans Power Apps Studio, et les contrôles se créent avec leurs formules.

## Ce qui n'est pas garanti

Je n'ai pas pu tester ce code dans Power Apps Studio. Le format de collage est un sous-ensemble de YAML dont la syntaxe exacte évolue : un nom de type de contrôle, une propriété ou un niveau d'indentation peuvent être refusés au premier collage. Les messages d'erreur de Studio indiquent la ligne en cause. La section « Si le collage échoue » donne une méthode pour corriger sans réécrire.

## Avant de coller

1. Les quatre listes SharePoint existent avec les noms internes de colonnes de [02](02-listes-sharepoint.md) (`TypeTache`, `Statut`, `Source`, `Echeance`, `Projet`, `TermineLe`, `Tache`, `Fait`, `Ordre`, `StatutProjet`).
2. Crée une application canvas vide au format **Tablette**.
3. Menu **Données** → ajoute les sources `Workload`, `Projets` et `Etapes` (SharePoint, ton site). Les noms dans l'app doivent être exactement ceux-là (supprime un éventuel suffixe).
4. Paramètres → Général : porte la **limite de lignes de données** à 2000.
5. Vérifie que la **barre de formule Power Fx** est active (Paramètres → Mises à jour, ou option par défaut sur les nouvelles apps) : le collage de code en dépend.
6. Supprime l'écran vide `Screen1` après le collage, ou garde-le comme écran de secours.

## Coller

1. Copie tout le bloc YAML ci-dessous.
2. Dans l'arborescence (vue Arborescence à gauche), clic droit sur **l'application / la liste des écrans** → **Coller le code**. Un écran `scrMain` se crée avec tous ses contrôles.
3. Ouvre `scrMain`, ajuste si besoin, puis lance l'application (F5). L'écran charge les données à l'ouverture.

## Le code

```yaml
Screens:
  scrMain As screen:
    Fill: =RGBA(243, 245, 247, 1)
    OnVisible: |-
      =Set(varVue, "today");
      Set(varProjet, Blank());
      Set(varType, Blank());
      Set(varSel, Blank());
      Select(btnRefresh)
    Children:
    - recTop As rectangle:
        Fill: =RGBA(11, 110, 110, 1)
        X: =0
        Y: =0
        Width: =Parent.Width
        Height: =48
    - lblTitle As label:
        Text: ="Workload"
        Color: =RGBA(255, 255, 255, 1)
        Size: =18
        FontWeight: =FontWeight.Semibold
        X: =16
        Y: =0
        Width: =300
        Height: =48
    - btnRefresh As button:
        Text: ="Actualiser"
        Fill: =RGBA(255, 255, 255, 1)
        Color: =RGBA(11, 110, 110, 1)
        X: =Parent.Width - 136
        Y: =8
        Width: =120
        Height: =32
        OnSelect: |-
          =Refresh(Workload);
          Refresh(Etapes);
          ClearCollect(colEtapes, Etapes);
          ClearCollect(colPct,
            ForAll(Workload As w,
              {
                ID: w.ID,
                ProjetId: w.Projet.Id,
                pct: If(w.Statut.Value = "Terminé", 100,
                  With({e: Filter(colEtapes, Tache.Id = w.ID)},
                    If(CountRows(e) = 0, 0, Round(100 * CountRows(Filter(e, Fait)) / CountRows(e), 0))))
              }
            )
          )

    - recRail As rectangle:
        Fill: =RGBA(238, 241, 244, 1)
        X: =0
        Y: =48
        Width: =200
        Height: =Parent.Height - 48
    - btnVueToday As button:
        Text: ="Aujourd'hui (" & CountRows(Filter(Workload, Statut.Value <> "Terminé", Echeance <= Today())) & ")"
        Fill: =If(varVue = "today", RGBA(217, 239, 238, 1), RGBA(238, 241, 244, 1))
        Color: =RGBA(23, 33, 43, 1)
        BorderColor: =RGBA(238, 241, 244, 1)
        X: =8
        Y: =64
        Width: =184
        Height: =36
        OnSelect: =Set(varVue, "today")
    - btnVueOpen As button:
        Text: ="Toutes ouvertes (" & CountRows(Filter(Workload, Statut.Value <> "Terminé")) & ")"
        Fill: =If(varVue = "open", RGBA(217, 239, 238, 1), RGBA(238, 241, 244, 1))
        Color: =RGBA(23, 33, 43, 1)
        BorderColor: =RGBA(238, 241, 244, 1)
        X: =8
        Y: =104
        Width: =184
        Height: =36
        OnSelect: =Set(varVue, "open")
    - btnVueLate As button:
        Text: ="En retard (" & CountRows(Filter(Workload, Statut.Value <> "Terminé", Echeance < Today())) & ")"
        Fill: =If(varVue = "late", RGBA(217, 239, 238, 1), RGBA(238, 241, 244, 1))
        Color: =RGBA(23, 33, 43, 1)
        BorderColor: =RGBA(238, 241, 244, 1)
        X: =8
        Y: =144
        Width: =184
        Height: =36
        OnSelect: =Set(varVue, "late")
    - btnVueDone As button:
        Text: ="Terminées (" & CountRows(Filter(Workload, Statut.Value = "Terminé")) & ")"
        Fill: =If(varVue = "done", RGBA(217, 239, 238, 1), RGBA(238, 241, 244, 1))
        Color: =RGBA(23, 33, 43, 1)
        BorderColor: =RGBA(238, 241, 244, 1)
        X: =8
        Y: =184
        Width: =184
        Height: =36
        OnSelect: =Set(varVue, "done")
    - btnVueTrier As button:
        Text: ="À trier (" & CountRows(Filter(Workload, TypeTache.Value = "À trier", Statut.Value <> "Terminé")) & ")"
        Fill: =If(varVue = "trier", RGBA(217, 239, 238, 1), RGBA(238, 241, 244, 1))
        Color: =RGBA(23, 33, 43, 1)
        BorderColor: =RGBA(238, 241, 244, 1)
        X: =8
        Y: =224
        Width: =184
        Height: =36
        OnSelect: =Set(varVue, "trier")
    - btnTousProjets As button:
        Text: ="Tous les projets"
        Fill: =RGBA(255, 255, 255, 1)
        Color: =RGBA(11, 110, 110, 1)
        X: =8
        Y: =284
        Width: =184
        Height: =36
        Visible: =!IsBlank(varProjet)
        OnSelect: =Set(varProjet, Blank())

    - lblKpiOuvertes As label:
        Text: =CountRows(Filter(Workload, Statut.Value <> "Terminé")) & Char(10) & "Ouvertes"
        Fill: =RGBA(255, 255, 255, 1)
        Color: =RGBA(23, 33, 43, 1)
        BorderColor: =RGBA(221, 226, 232, 1)
        BorderThickness: =1
        PaddingLeft: =10
        X: =216
        Y: =64
        Width: =196
        Height: =60
    - lblKpiRetard As label:
        Text: =CountRows(Filter(Workload, Statut.Value <> "Terminé", Echeance < Today())) & Char(10) & "En retard"
        Fill: =RGBA(255, 255, 255, 1)
        Color: =RGBA(179, 38, 30, 1)
        BorderColor: =RGBA(221, 226, 232, 1)
        BorderThickness: =1
        PaddingLeft: =10
        X: =426
        Y: =64
        Width: =196
        Height: =60
    - lblKpiSemaine As label:
        Text: =CountRows(Filter(Workload, Statut.Value = "Terminé", TermineLe >= DateAdd(Today(), -7, TimeUnit.Days))) & Char(10) & "Terminées sur 7 jours"
        Fill: =RGBA(255, 255, 255, 1)
        Color: =RGBA(30, 122, 70, 1)
        BorderColor: =RGBA(221, 226, 232, 1)
        BorderThickness: =1
        PaddingLeft: =10
        X: =636
        Y: =64
        Width: =196
        Height: =60
    - lblKpiGlobal As label:
        Text: =Round(Coalesce(Average(colPct, pct), 0), 0) & " %" & Char(10) & "Avancement global"
        Fill: =RGBA(255, 255, 255, 1)
        Color: =RGBA(23, 33, 43, 1)
        BorderColor: =RGBA(221, 226, 232, 1)
        BorderThickness: =1
        PaddingLeft: =10
        X: =846
        Y: =64
        Width: =196
        Height: =60

    - galProjets As gallery.galleryHorizontal:
        Items: =Filter(Projets, StatutProjet.Value = "Actif")
        X: =216
        Y: =136
        Width: =830
        Height: =96
        TemplateSize: =270
        TemplatePadding: =6
        OnSelect: =Set(varProjet, ThisItem.Title); Set(varVue, "open")
        Children:
        - recProjCard As rectangle:
            Fill: =If(varProjet = ThisItem.Title, RGBA(217, 239, 238, 1), RGBA(255, 255, 255, 1))
            BorderColor: =RGBA(221, 226, 232, 1)
            BorderThickness: =1
            X: =0
            Y: =0
            Width: =Parent.TemplateWidth
            Height: =Parent.TemplateHeight
        - lblProjNom As label:
            Text: =ThisItem.Title
            FontWeight: =FontWeight.Semibold
            Color: =RGBA(23, 33, 43, 1)
            X: =8
            Y: =6
            Width: =Parent.TemplateWidth - 80
            Height: =28
        - lblProjPct As label:
            Text: =Round(Coalesce(Average(Filter(colPct, ProjetId = ThisItem.ID), pct), 0), 0) & " %"
            Align: =Align.Right
            FontWeight: =FontWeight.Semibold
            Color: =RGBA(11, 110, 110, 1)
            X: =Parent.TemplateWidth - 76
            Y: =6
            Width: =68
            Height: =28
        - recProjBarBack As rectangle:
            Fill: =RGBA(238, 241, 244, 1)
            X: =8
            Y: =42
            Width: =Parent.TemplateWidth - 16
            Height: =8
        - recProjBar As rectangle:
            Fill: =RGBA(11, 110, 110, 1)
            X: =8
            Y: =42
            Width: =(Parent.TemplateWidth - 16) * Coalesce(Average(Filter(colPct, ProjetId = ThisItem.ID), pct), 0) / 100
            Height: =8
        - lblProjCompte As label:
            Text: =CountRows(Filter(Workload, Projet.Id = ThisItem.ID, Statut.Value = "Terminé")) & " terminée(s) sur " & CountRows(Filter(Workload, Projet.Id = ThisItem.ID))
            Color: =RGBA(93, 107, 122, 1)
            Size: =11
            X: =8
            Y: =56
            Width: =Parent.TemplateWidth - 16
            Height: =26

    - txtRecherche As text:
        HintText: ="Rechercher une tâche"
        X: =216
        Y: =244
        Width: =240
        Height: =36
    - btnTypeSupport As button:
        Text: ="Support"
        Fill: =If(varType = "Support", RGBA(11, 110, 110, 1), RGBA(255, 255, 255, 1))
        Color: =If(varType = "Support", RGBA(255, 255, 255, 1), RGBA(23, 33, 43, 1))
        X: =464
        Y: =244
        Width: =100
        Height: =36
        OnSelect: =Set(varType, If(varType = "Support", Blank(), "Support"))
    - btnTypeAmel As button:
        Text: ="Amélioration"
        Fill: =If(varType = "Amélioration", RGBA(11, 110, 110, 1), RGBA(255, 255, 255, 1))
        Color: =If(varType = "Amélioration", RGBA(255, 255, 255, 1), RGBA(23, 33, 43, 1))
        X: =572
        Y: =244
        Width: =120
        Height: =36
        OnSelect: =Set(varType, If(varType = "Amélioration", Blank(), "Amélioration"))
    - btnTypeMonthly As button:
        Text: ="Monthly eng"
        Fill: =If(varType = "Monthly eng", RGBA(11, 110, 110, 1), RGBA(255, 255, 255, 1))
        Color: =If(varType = "Monthly eng", RGBA(255, 255, 255, 1), RGBA(23, 33, 43, 1))
        X: =700
        Y: =244
        Width: =120
        Height: =36
        OnSelect: =Set(varType, If(varType = "Monthly eng", Blank(), "Monthly eng"))
    - btnTypeProjet As button:
        Text: ="Projet"
        Fill: =If(varType = "Projet", RGBA(11, 110, 110, 1), RGBA(255, 255, 255, 1))
        Color: =If(varType = "Projet", RGBA(255, 255, 255, 1), RGBA(23, 33, 43, 1))
        X: =828
        Y: =244
        Width: =100
        Height: =36
        OnSelect: =Set(varType, If(varType = "Projet", Blank(), "Projet"))

    - galTaches As gallery.galleryVertical:
        Items: |-
          =SortByColumns(
            Filter(Workload,
              (varVue = "today" && Statut.Value <> "Terminé" && Echeance <= Today()) ||
              (varVue = "open" && Statut.Value <> "Terminé") ||
              (varVue = "late" && Statut.Value <> "Terminé" && Echeance < Today()) ||
              (varVue = "done" && Statut.Value = "Terminé") ||
              (varVue = "trier" && TypeTache.Value = "À trier" && Statut.Value <> "Terminé"),
              IsBlank(varProjet) || Projet.Value = varProjet,
              IsBlank(varType) || TypeTache.Value = varType,
              IsBlank(txtRecherche.Text) || txtRecherche.Text in Title
            ),
            "Echeance", SortOrder.Ascending
          )
        X: =216
        Y: =288
        Width: =830
        Height: =Parent.Height - 296
        TemplateSize: =64
        TemplatePadding: =0
        OnSelect: =Set(varSel, ThisItem)
        Children:
        - recLigne As rectangle:
            Fill: =If(varSel.ID = ThisItem.ID, RGBA(217, 239, 238, 1), RGBA(255, 255, 255, 1))
            BorderColor: =RGBA(221, 226, 232, 1)
            BorderThickness: =1
            X: =0
            Y: =0
            Width: =Parent.TemplateWidth
            Height: =Parent.TemplateHeight
        - recStripe As rectangle:
            Fill: =If(ThisItem.Statut.Value = "Terminé", RGBA(30, 122, 70, 1), Switch(ThisItem.TypeTache.Value, "Support", RGBA(43, 95, 180, 1), "Amélioration", RGBA(122, 75, 176, 1), "Monthly eng", RGBA(161, 92, 0, 1), "Projet", RGBA(11, 110, 110, 1), RGBA(93, 107, 122, 1)))
            X: =0
            Y: =0
            Width: =5
            Height: =Parent.TemplateHeight
        - lblTitre As label:
            Text: =ThisItem.Title
            Color: =If(ThisItem.Statut.Value = "Terminé", RGBA(93, 107, 122, 1), RGBA(23, 33, 43, 1))
            Wrap: =false
            X: =14
            Y: =6
            Width: =Parent.TemplateWidth - 220
            Height: =24
        - lblMeta As label:
            Text: =ThisItem.Source.Value & " · " & Coalesce(ThisItem.Projet.Value, "sans projet") & " · " & ThisItem.Statut.Value
            Color: =RGBA(93, 107, 122, 1)
            Size: =11
            X: =14
            Y: =30
            Width: =Parent.TemplateWidth - 220
            Height: =18
        - recBarBack As rectangle:
            Fill: =RGBA(238, 241, 244, 1)
            X: =16
            Y: =52
            Width: =200
            Height: =6
        - recBar As rectangle:
            Fill: =RGBA(11, 110, 110, 1)
            X: =16
            Y: =52
            Width: =200 * Coalesce(LookUp(colPct, ID = ThisItem.ID, pct), 0) / 100
            Height: =6
        - lblPctLigne As label:
            Text: =Coalesce(LookUp(colPct, ID = ThisItem.ID, pct), 0) & " %"
            Color: =RGBA(93, 107, 122, 1)
            Size: =10
            X: =222
            Y: =46
            Width: =50
            Height: =18
        - lblTypeLigne As label:
            Text: =ThisItem.TypeTache.Value
            Align: =Align.Center
            Size: =11
            FontWeight: =FontWeight.Semibold
            Color: =Switch(ThisItem.TypeTache.Value, "Support", RGBA(43, 95, 180, 1), "Amélioration", RGBA(122, 75, 176, 1), "Monthly eng", RGBA(161, 92, 0, 1), "Projet", RGBA(11, 110, 110, 1), RGBA(93, 107, 122, 1))
            X: =Parent.TemplateWidth - 200
            Y: =18
            Width: =110
            Height: =26
        - lblEcheanceLigne As label:
            Text: =If(ThisItem.Statut.Value = "Terminé", "fait " & Text(ThisItem.TermineLe, "dd/mm"), If(ThisItem.Echeance < Today(), "retard ", "") & Text(ThisItem.Echeance, "dd/mm"))
            Align: =Align.Right
            Size: =12
            Color: =If(ThisItem.Statut.Value <> "Terminé" && ThisItem.Echeance < Today(), RGBA(179, 38, 30, 1), RGBA(23, 33, 43, 1))
            X: =Parent.TemplateWidth - 90
            Y: =18
            Width: =80
            Height: =26

    - recSide As rectangle:
        Fill: =RGBA(255, 255, 255, 1)
        BorderColor: =RGBA(221, 226, 232, 1)
        BorderThickness: =1
        X: =1046
        Y: =48
        Width: =Parent.Width - 1046
        Height: =Parent.Height - 48
    - lblDTitre As label:
        Text: =varSel.Title
        FontWeight: =FontWeight.Semibold
        Size: =15
        Color: =RGBA(23, 33, 43, 1)
        Visible: =!IsBlank(varSel)
        X: =1062
        Y: =60
        Width: =288
        Height: =56
    - lblDMeta As label:
        Text: =varSel.TypeTache.Value & " · " & varSel.Source.Value & " · " & Coalesce(varSel.Projet.Value, "sans projet") & Char(10) & "Échéance " & Text(varSel.Echeance, "dd/mm/yyyy") & " · " & varSel.Statut.Value
        Size: =12
        Color: =RGBA(93, 107, 122, 1)
        Visible: =!IsBlank(varSel)
        X: =1062
        Y: =118
        Width: =288
        Height: =40
    - lblDPct As label:
        Text: =Coalesce(LookUp(colPct, ID = varSel.ID, pct), 0) & " %  ·  " & CountRows(Filter(colEtapes, Tache.Id = varSel.ID, Fait)) & " étape(s) sur " & CountRows(Filter(colEtapes, Tache.Id = varSel.ID))
        FontWeight: =FontWeight.Semibold
        Color: =RGBA(11, 110, 110, 1)
        Visible: =!IsBlank(varSel)
        X: =1062
        Y: =162
        Width: =288
        Height: =26
    - recDBarBack As rectangle:
        Fill: =RGBA(238, 241, 244, 1)
        Visible: =!IsBlank(varSel)
        X: =1062
        Y: =192
        Width: =288
        Height: =8
    - recDBar As rectangle:
        Fill: =RGBA(11, 110, 110, 1)
        Visible: =!IsBlank(varSel)
        X: =1062
        Y: =192
        Width: =288 * Coalesce(LookUp(colPct, ID = varSel.ID, pct), 0) / 100
        Height: =8
    - galEtapes As gallery.galleryVertical:
        Items: =SortByColumns(Filter(colEtapes, Tache.Id = varSel.ID), "Ordre")
        Visible: =!IsBlank(varSel)
        X: =1062
        Y: =210
        Width: =288
        Height: =200
        TemplateSize: =36
        TemplatePadding: =0
        Children:
        - chkEtape As checkbox:
            Text: =ThisItem.Title
            Default: =ThisItem.Fait
            OnCheck: |-
              =Patch(Etapes, LookUp(Etapes, ID = ThisItem.ID), {Fait: true});
              Select(btnRefresh);
              If(varSel.Statut.Value = "À faire",
                Patch(Workload, LookUp(Workload, ID = varSel.ID), {Statut: {Value: "En cours"}});
                Set(varSel, LookUp(Workload, ID = varSel.ID))
              )
            OnUncheck: |-
              =Patch(Etapes, LookUp(Etapes, ID = ThisItem.ID), {Fait: false});
              Select(btnRefresh)
            X: =0
            Y: =0
            Width: =Parent.TemplateWidth
            Height: =Parent.TemplateHeight
    - txtEtape As text:
        HintText: ="Ajouter une étape"
        Visible: =!IsBlank(varSel) && varSel.Statut.Value <> "Terminé"
        X: =1062
        Y: =418
        Width: =200
        Height: =34
    - btnAjouterEtape As button:
        Text: ="Ajouter"
        Visible: =!IsBlank(varSel) && varSel.Statut.Value <> "Terminé"
        X: =1270
        Y: =418
        Width: =80
        Height: =34
        DisplayMode: =If(IsBlank(txtEtape.Text), DisplayMode.Disabled, DisplayMode.Edit)
        OnSelect: |-
          =Patch(Etapes, Defaults(Etapes),
            {
              Title: txtEtape.Text,
              Tache: {Id: varSel.ID, Value: varSel.Title},
              Fait: false,
              Ordre: CountRows(Filter(colEtapes, Tache.Id = varSel.ID)) + 1
            });
          Reset(txtEtape);
          Select(btnRefresh)
    - lblSync As label:
        Text: =If(varSel.Statut.Value = "Terminé", "Clôture propagée par le flux.", Switch(varSel.Source.Value, "Mail", "Terminer clôt la liste, le flag Outlook et To Do.", "Teams", "Terminer clôt la liste et la tâche To Do.", "Loop", "Terminer clôt la liste. À cocher aussi dans Loop si non synchronisé.", "Terminer clôt la liste."))
        Size: =11
        Color: =RGBA(93, 107, 122, 1)
        Visible: =!IsBlank(varSel)
        X: =1062
        Y: =458
        Width: =288
        Height: =44
    - ddType As dropdown:
        Items: =Filter(Choices(Workload.TypeTache), Value <> "À trier")
        Visible: =!IsBlank(varSel) && varSel.TypeTache.Value = "À trier"
        X: =1062
        Y: =510
        Width: =288
        Height: =34
    - ddProjet As dropdown:
        Items: =Filter(Projets, StatutProjet.Value = "Actif")
        Value: ="Title"
        Visible: =!IsBlank(varSel) && varSel.TypeTache.Value = "À trier"
        X: =1062
        Y: =550
        Width: =288
        Height: =34
    - btnClasser As button:
        Text: ="Classer"
        Visible: =!IsBlank(varSel) && varSel.TypeTache.Value = "À trier"
        X: =1062
        Y: =590
        Width: =288
        Height: =36
        OnSelect: |-
          =Patch(Workload, LookUp(Workload, ID = varSel.ID),
            {
              TypeTache: {Value: ddType.Selected.Value},
              Projet: {Id: ddProjet.Selected.ID, Value: ddProjet.Selected.Title}
            });
          Set(varSel, LookUp(Workload, ID = varSel.ID));
          Select(btnRefresh)
    - btnTerminer As button:
        Text: ="Terminer partout"
        Fill: =RGBA(11, 110, 110, 1)
        Color: =RGBA(255, 255, 255, 1)
        Visible: =!IsBlank(varSel) && varSel.Statut.Value <> "Terminé"
        X: =1062
        Y: =Parent.Height - 64
        Width: =288
        Height: =44
        OnSelect: |-
          =Patch(Workload, LookUp(Workload, ID = varSel.ID),
            {Statut: {Value: "Terminé"}, TermineLe: Now()});
          Set(varSel, LookUp(Workload, ID = varSel.ID));
          Select(btnRefresh);
          Notify("Clôture envoyée à Outlook, To Do et Loop par le flux.", NotificationType.Success)
```

## Après le collage

1. **Premier lancement.** Appuie sur « Actualiser » si les chiffres sont à zéro : il remplit les collections `colEtapes` et `colPct`.
2. **Erreurs rouges sur les noms de colonnes.** Passe la souris sur le contrôle : Studio indique la colonne introuvable. Les noms de colonne attendus sont ceux de [02](02-listes-sharepoint.md). Corrige soit dans la formule, soit dans la liste.
3. **Délégation.** Un triangle jaune sur `in` (recherche) ou sur un `Filter` est normal tant que les listes restent sous 2000 lignes.
4. **Les cases d'étapes** se rafraîchissent après le flux de clôture : ajoute un bouton « Actualiser » visible, ou un `Timer` de 30 secondes qui appelle `Select(btnRefresh)`.
5. **Vue « Modèles d'étapes »** : volontairement absente de l'app. Modifie les modèles directement dans la liste `ModelesEtapes`.

## Si le collage échoue

Le format de collage est strict. Méthode de correction, dans l'ordre :

1. **Lis le message d'erreur.** Il cite le contrôle ou la ligne en cause.
2. **Compare avec un contrôle réel.** Insère un contrôle du même type (un bouton, une galerie verticale, une galerie horizontale, une case à cocher, une liste déroulante) depuis le menu Insérer. Clic droit sur le contrôle → **Afficher le code**. Compare avec mon code : le nom du type après `As` (par exemple `gallery.galleryVertical`), la présence du mot `Children:` et l'indentation.
3. **Corrige par rechercher-remplacer** dans un éditeur de texte, par exemple le nom d'un type de contrôle, puis colle à nouveau.
4. **Repli en deux étapes.** Crée un écran vide, puis colle seulement la liste des contrôles : retire les lignes `Screens:`, `scrMain As screen:`, `Fill`, `OnVisible` et `Children:`, et supprime l'indentation en trop de façon à ce que chaque ligne `- nom As type:` commence en première colonne. Recopie ensuite l'`OnVisible` à la main dans la propriété `OnVisible` de l'écran.
5. **Dernier recours.** Construis les contrôles à la main à partir de la maquette et colle uniquement les formules de ce document dans leurs propriétés. Les formules sont indépendantes du format de collage.

## Variables et collections utilisées

| Nom | Rôle |
|---|---|
| `varVue` | Vue active : today, open, late, done, trier |
| `varProjet` | Nom du projet filtré, ou vide |
| `varType` | Type filtré, ou vide |
| `varSel` | Item Workload sélectionné |
| `colEtapes` | Copie locale de la liste Etapes |
| `colPct` | Avancement calculé par tâche (ID, ProjetId, pct) |
