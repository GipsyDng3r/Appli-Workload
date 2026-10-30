# 9. Power App : code complet à coller

Application canvas dont la mise en page **s'adapte à la taille de l'écran** : les containers se dimensionnent à partir de `Parent.Width` et `Parent.Height`, sans coordonnées fixes. Elle a été contrôlée numériquement à 1366 x 768, 1280 x 720 et 1920 x 907 : aucun contrôle ne déborde de son container. Style de la maquette [maquette/workload.html](../maquette/workload.html) : tableau de bord, Kanban, calendrier, analyse, vue « À trier » et volet de détail. Un seul écran `scrMain` : les vues sont des **containers** (`cntDash`, `cntTasks`, `cntCal`, `cntStats`, `cntTrier`) dont la propriété `Visible` dépend de la variable `varVue`. Chaque zone (cartes, compteurs, filtres, volet de détail) est un sous-container.

Typographie : police Segoe UI partout, échelle de tailles fixe : 22 pour le titre de page, 26 pour les chiffres des compteurs, 14 pour les titres de carte, 12 pour le texte courant, 10 à 11 pour les légendes. Pour grossir ou réduire toute l'échelle, fais un rechercher-remplacer sur les lignes `Size: =` du fichier. Les titres trop longs des listes et du calendrier sont tronqués avec « … » selon la largeur disponible.

Le même code est dans [power-app/scrMain.pa.yaml](../power-app/scrMain.pa.yaml), plus pratique à copier depuis GitHub (bouton « Copy raw file »).

## Ce qui n'est pas garanti

Je n'ai pas pu tester ce code dans Power Apps Studio. Deux niveaux de risque :
- **Le format de collage** (YAML) : il reprend la syntaxe réelle d'un export de Studio (`Control: Label@2.5.1`, `Properties:`, `Children:`), mais les numéros de version des contrôles peuvent différer chez toi. Si Studio en refuse un, voir « Si le collage échoue ». Le contrôle `Image` (`Image@2.2.3`) est celui dont la version est la moins sûre.
- **Les formules Power Fx** : elles ne dépendent pas du format et se corrigent contrôle par contrôle. Les points les plus délicats sont signalés dans « Points à surveiller ».

La structure du YAML a été vérifiée avec un analyseur YAML : indentation correcte, aucun nom de contrôle en double, aucune référence à un contrôle inexistant. Les contrôles sont rangés dans des containers en mise en page manuelle (`ManualLayout`) : les coordonnées X et Y d'un contrôle sont relatives à son container, et un container masqué masque tout son contenu. Les cartes sont des containers à fond blanc (plus de rectangles de fond).

## Différences avec la maquette

| Maquette | Power App |
|---|---|
| Glisser-déposer des cartes du Kanban | Boutons ◀ ▶ sur chaque carte : Power Apps n'a pas de glisser-déposer natif |
| Courbes et anneau dessinés en JavaScript | Images SVG générées par formule (`data:image/svg+xml`) |
| Barres en pilule hachurée | Barres pleines à angles droits (le contrôle Rectangle n'a pas de rayon d'arrondi) |
| Tiroir de détail animé | Volet fixe à droite, affiché quand une tâche est sélectionnée |
| Bouton « Simuler un mail flaggé » | Absent : ce sont les vrais flux qui créent les tâches. Un champ « Nouvelle tâche manuelle » le remplace |
| Données d'exemple | Données de tes listes SharePoint |
| Modèles d'étapes | Se modifient dans la liste `ModelesEtapes` |

## Avant de coller

1. Les quatre listes SharePoint existent avec les noms internes de colonnes de [02](02-listes-sharepoint.md) : `Workload` (`TypeTache`, `Source`, `Statut`, `Echeance`, `Projet`, `TermineLe`, `Propage`, `EtapesCreees`), `Projets` (`StatutProjet`), `Etapes` (`Tache`, `Fait`, `Ordre`).
2. Crée une application canvas vide au format **Tablette**.
3. Menu **Données** → ajoute tes listes SharePoint (connecteur SharePoint). **Le nom d'une source de données dans l'app est le nom de la liste.** Le code utilise `'[LME] Workload'`, `'[LME] Projets'` et `'[LME] Etapes'` (les apostrophes sont obligatoires à cause des crochets et de l'espace). Si tes listes portent un autre nom, remplace partout `[LME] ` par ton préfixe, ou retire-le avec un rechercher-remplacer sur `'[LME] ` puis sur `'` après `Workload`, `Projets` et `Etapes`. Un nom de source incorrect provoque des centaines d'erreurs en cascade dans l'App checker.
4. Paramètres → Général : porte la **limite de lignes de données** à 2000.
5. Vérifie que la **barre de formule Power Fx** est active : le collage de code en dépend.
6. Paramètres → Affichage : décoche « Adapter à l'écran » et « Verrouiller le format » si tu veux garder les coordonnées exactes.

## Coller

1. Copie tout le bloc YAML ci-dessous.
2. Dans l'arborescence, clic droit sur la liste des écrans → **Coller le code**. L'écran `scrMain` se crée avec ses contrôles et ses containers. Si tu avais déjà collé une version précédente, supprime l'ancien `scrMain` d'abord.
3. Dans Paramètres → Affichage, laisse « Adapter à l'écran » activé. Définis `scrMain` comme écran de démarrage, puis lance l'application (F5). L'`OnVisible` charge les données.

## Le code

```yaml
Screens:
  scrMain:
    Properties:
      Fill: =RGBA(238, 241, 247, 1)
      OnVisible: |-
        =Set(varVue, "dash");
        Set(varType, Blank());
        Set(varSel, Blank());
        Set(varAn, Year(Today()));
        Set(varMois, Month(Today()));
        ClearCollect(colTC,
          {T: "Support", C: RGBA(47, 98, 214, 1)},
          {T: "Amélioration", C: RGBA(123, 79, 196, 1)},
          {T: "Monthly eng", C: RGBA(184, 106, 0, 1)},
          {T: "Projet", C: RGBA(14, 138, 125, 1)},
          {T: "À trier", C: RGBA(107, 119, 140, 1)}
        );
        IfError(
          Refresh('[LME] Workload'),
          Notify("Refresh(Workload) : " & FirstError.Message, NotificationType.Error)
        );
        IfError(
          Refresh('[LME] Etapes'),
          Notify("Refresh(Etapes) : " & FirstError.Message, NotificationType.Error)
        );
        IfError(
          Refresh('[LME] Projets'),
          Notify("Refresh(Projets) : " & FirstError.Message, NotificationType.Error)
        );
        IfError(
          ClearCollect(colWL, '[LME] Workload'),
          Notify("ClearCollect(colWL) : " & FirstError.Message, NotificationType.Error)
        );
        IfError(
          ClearCollect(colEtapes, '[LME] Etapes'),
          Notify("ClearCollect(colEtapes) : " & FirstError.Message, NotificationType.Error)
        );
        IfError(
          ClearCollect(colPct,
            ForAll('[LME] Workload' As w,
              {
                ID: w.ID,
                ProjetId: w.Projet.Id,
                TypeT: w.TypeTache.Value,
                pct: If(w.Statut.Value = "Terminé", 100,
                  With({e: Filter(colEtapes, Tache.Id = w.ID)},
                    If(CountRows(e) = 0, 0, Round(100 * CountRows(Filter(e, Fait)) / CountRows(e), 0))))
              }
            )
          ),
          Notify("ClearCollect(colPct) : " & FirstError.Message, NotificationType.Error)
        );
        IfError(
          Set(varGlobal, Round(IfError(Average(colPct, pct), 0), 0)),
          Notify("Set(varGlobal) : " & FirstError.Message, NotificationType.Error)
        );
        IfError(
          Set(varNext, First(SortByColumns(Filter('[LME] Workload', Statut.Value <> "Terminé", TypeTache.Value <> "À trier"), "Echeance", SortOrder.Ascending))),
          Notify("Set(varNext) : " & FirstError.Message, NotificationType.Error)
        );
        IfError(
          ClearCollect(colTypes,
            ForAll(["Support", "Amélioration", "Monthly eng", "Projet"] As t,
              {
                T: t.Value,
                n: CountRows(Filter('[LME] Workload', TypeTache.Value = t.Value)),
                pct: Round(IfError(Average(Filter(colPct, TypeT = t.Value), pct), 0), 0)
              }
            )
          ),
          Notify("ClearCollect(colTypes) : " & FirstError.Message, NotificationType.Error)
        );
        IfError(
          ClearCollect(colProjCount,
            ForAll(Sequence(CountRows('[LME] Projets')),
              With({p: Index('[LME] Projets', Value)},
                {
                  k: Value,
                  name: p.Title,
                  n: CountRows(Filter('[LME] Workload', Projet.Id = p.ID, Statut.Value <> "Terminé"))
                }
              )
            )
          ),
          Notify("ClearCollect(colProjCount) : " & FirstError.Message, NotificationType.Error)
        );
        IfError(
          Set(varTotalOuv, Sum(colProjCount, n)),
          Notify("Set(varTotalOuv) : " & FirstError.Message, NotificationType.Error)
        );
        IfError(
          ClearCollect(colDonut,
            ForAll(colProjCount As c,
              {
                k: c.k,
                name: c.name,
                n: c.n,
                share: Round(100 * c.n / Max(varTotalOuv, 1), 0),
                start: Round(100 * Sum(Filter(colProjCount, k < c.k), n) / Max(varTotalOuv, 1), 0)
              }
            )
          ),
          Notify("ClearCollect(colDonut) : " & FirstError.Message, NotificationType.Error)
        );
        IfError(
          ClearCollect(colSem,
            ForAll(Sequence(8, 0),
              {
                i: Value,
                n: CountRows(Filter('[LME] Workload', Statut.Value = "Terminé",
                     TermineLe >= DateAdd(Today(), 1 - 7 * (8 - Value), TimeUnit.Days),
                     TermineLe < DateAdd(Today(), 1 - 7 * (7 - Value), TimeUnit.Days)))
              }
            )
          ),
          Notify("ClearCollect(colSem) : " & FirstError.Message, NotificationType.Error)
        );
        IfError(
          If(!IsBlank(varSel), Set(varSel, LookUp('[LME] Workload', ID = varSel.ID))),
          Notify("mise à jour de varSel : " & FirstError.Message, NotificationType.Error)
        )
    Children:
      - cntSidebar:
          Control: GroupContainer@1.5.0
          Variant: ManualLayout
          Properties:
            X: =0
            Y: =0
            Width: =232
            Height: =Parent.Height
            Fill: =RGBA(255, 255, 255, 1)
          Children:
            - lblLogo:
                Control: Label@2.5.1
                Properties:
                  X: =20
                  Y: =20
                  Width: =190
                  Height: =36
                  Text: ="Workload"
                  Size: =18
                  FontWeight: =FontWeight.Bold
                  Color: =RGBA(16, 26, 46, 1)
                  Font: =Font.'Segoe UI'
            - navDash:
                Control: Classic/Button@2.2.0
                Properties:
                  X: =12
                  Y: =84
                  Width: =208
                  Height: =40
                  Text: ="Tableau de bord"
                  Fill: =If(Coalesce(varVue, "dash") = "dash", RGBA(227, 235, 255, 1), RGBA(255, 255, 255, 1))
                  Color: =If(Coalesce(varVue, "dash") = "dash", RGBA(31, 79, 216, 1), RGBA(100, 114, 138, 1))
                  BorderColor: =Self.Fill
                  OnSelect: =Set(varVue, "dash"); Set(varSel, Blank())
                  Size: =13
                  Align: =Align.Left
                  Font: =Font.'Segoe UI'
            - navTasks:
                Control: Classic/Button@2.2.0
                Properties:
                  X: =12
                  Y: =130
                  Width: =208
                  Height: =40
                  Text: ="Tâches"
                  Fill: =If(varVue = "tasks", RGBA(227, 235, 255, 1), RGBA(255, 255, 255, 1))
                  Color: =If(varVue = "tasks", RGBA(31, 79, 216, 1), RGBA(100, 114, 138, 1))
                  BorderColor: =Self.Fill
                  OnSelect: =Set(varVue, "tasks"); Set(varSel, Blank())
                  Size: =13
                  Align: =Align.Left
                  Font: =Font.'Segoe UI'
            - navCal:
                Control: Classic/Button@2.2.0
                Properties:
                  X: =12
                  Y: =176
                  Width: =208
                  Height: =40
                  Text: ="Calendrier"
                  Fill: =If(varVue = "cal", RGBA(227, 235, 255, 1), RGBA(255, 255, 255, 1))
                  Color: =If(varVue = "cal", RGBA(31, 79, 216, 1), RGBA(100, 114, 138, 1))
                  BorderColor: =Self.Fill
                  OnSelect: =Set(varVue, "cal"); Set(varSel, Blank())
                  Size: =13
                  Align: =Align.Left
                  Font: =Font.'Segoe UI'
            - navStats:
                Control: Classic/Button@2.2.0
                Properties:
                  X: =12
                  Y: =222
                  Width: =208
                  Height: =40
                  Text: ="Analyse"
                  Fill: =If(varVue = "stats", RGBA(227, 235, 255, 1), RGBA(255, 255, 255, 1))
                  Color: =If(varVue = "stats", RGBA(31, 79, 216, 1), RGBA(100, 114, 138, 1))
                  BorderColor: =Self.Fill
                  OnSelect: =Set(varVue, "stats"); Set(varSel, Blank())
                  Size: =13
                  Align: =Align.Left
                  Font: =Font.'Segoe UI'
            - navTrier:
                Control: Classic/Button@2.2.0
                Properties:
                  X: =12
                  Y: =268
                  Width: =208
                  Height: =40
                  Text: ="À trier (" & CountRows(Filter('[LME] Workload', TypeTache.Value = "À trier", Statut.Value <> "Terminé")) & ")"
                  Fill: =If(varVue = "trier", RGBA(227, 235, 255, 1), RGBA(255, 255, 255, 1))
                  Color: =If(varVue = "trier", RGBA(31, 79, 216, 1), RGBA(100, 114, 138, 1))
                  BorderColor: =Self.Fill
                  OnSelect: =Set(varVue, "trier"); Set(varSel, Blank())
                  Size: =13
                  Align: =Align.Left
                  Font: =Font.'Segoe UI'
      - cntMain:
          Control: GroupContainer@1.5.0
          Variant: ManualLayout
          Properties:
            X: =232
            Y: =0
            Width: =Parent.Width - 232
            Height: =Parent.Height
          Children:
            - cntTopbar:
                Control: GroupContainer@1.5.0
                Variant: ManualLayout
                Properties:
                  X: =20
                  Y: =16
                  Width: =Parent.Width - 40
                  Height: =44
                Children:
                  - txtRecherche:
                      Control: Classic/TextInput@2.3.2
                      Properties:
                        X: =0
                        Y: =2
                        Width: =Parent.Width * 0.30
                        Height: =40
                        HintText: ="Rechercher une tâche"
                        Size: =12
                        Font: =Font.'Segoe UI'
                        Default: =""
                  - txtNouvelle:
                      Control: Classic/TextInput@2.3.2
                      Properties:
                        X: =Parent.Width * 0.30 + 12
                        Y: =2
                        Width: =Parent.Width * 0.26
                        Height: =40
                        HintText: ="Nouvelle tâche manuelle"
                        Size: =12
                        Font: =Font.'Segoe UI'
                        Default: =""
                  - btnAjouter:
                      Control: Classic/Button@2.2.0
                      Properties:
                        X: =Parent.Width * 0.56 + 24
                        Y: =2
                        Width: =110
                        Height: =40
                        Text: ="Ajouter"
                        Fill: =RGBA(31, 79, 216, 1)
                        Color: =RGBA(255, 255, 255, 1)
                        DisplayMode: =If(IsBlank(txtNouvelle.Text), DisplayMode.Disabled, DisplayMode.Edit)
                        OnSelect: |-
                          =Patch('[LME] Workload', Defaults('[LME] Workload'),
                            {
                              Title: txtNouvelle.Text,
                              TypeTache: {Value: "À trier"},
                              Source: {Value: "Manuel"},
                              Statut: {Value: "À faire"},
                              Echeance: DateAdd(Today(), 7, TimeUnit.Days),
                              Propage: false,
                              EtapesCreees: false
                            });
                          Reset(txtNouvelle);
                          Select(btnRefresh);
                          Set(varVue, "trier");
                          Notify("Tâche créée dans À trier : choisis son type et son projet.", NotificationType.Success)
                        Size: =12
                        Font: =Font.'Segoe UI'
                  - btnRefresh:
                      Control: Classic/Button@2.2.0
                      Properties:
                        X: =Parent.Width - 124
                        Y: =2
                        Width: =124
                        Height: =40
                        Text: ="Actualiser"
                        Fill: =RGBA(255, 255, 255, 1)
                        Color: =RGBA(16, 26, 46, 1)
                        OnSelect: |-
                          =IfError(
                            Refresh('[LME] Workload'),
                            Notify("Refresh(Workload) : " & FirstError.Message, NotificationType.Error)
                          );
                          IfError(
                            Refresh('[LME] Etapes'),
                            Notify("Refresh(Etapes) : " & FirstError.Message, NotificationType.Error)
                          );
                          IfError(
                            Refresh('[LME] Projets'),
                            Notify("Refresh(Projets) : " & FirstError.Message, NotificationType.Error)
                          );
                          IfError(
                            ClearCollect(colWL, '[LME] Workload'),
                            Notify("ClearCollect(colWL) : " & FirstError.Message, NotificationType.Error)
                          );
                          IfError(
                            ClearCollect(colEtapes, '[LME] Etapes'),
                            Notify("ClearCollect(colEtapes) : " & FirstError.Message, NotificationType.Error)
                          );
                          IfError(
                            ClearCollect(colPct,
                              ForAll('[LME] Workload' As w,
                                {
                                  ID: w.ID,
                                  ProjetId: w.Projet.Id,
                                  TypeT: w.TypeTache.Value,
                                  pct: If(w.Statut.Value = "Terminé", 100,
                                    With({e: Filter(colEtapes, Tache.Id = w.ID)},
                                      If(CountRows(e) = 0, 0, Round(100 * CountRows(Filter(e, Fait)) / CountRows(e), 0))))
                                }
                              )
                            ),
                            Notify("ClearCollect(colPct) : " & FirstError.Message, NotificationType.Error)
                          );
                          IfError(
                            Set(varGlobal, Round(IfError(Average(colPct, pct), 0), 0)),
                            Notify("Set(varGlobal) : " & FirstError.Message, NotificationType.Error)
                          );
                          IfError(
                            Set(varNext, First(SortByColumns(Filter('[LME] Workload', Statut.Value <> "Terminé", TypeTache.Value <> "À trier"), "Echeance", SortOrder.Ascending))),
                            Notify("Set(varNext) : " & FirstError.Message, NotificationType.Error)
                          );
                          IfError(
                            ClearCollect(colTypes,
                              ForAll(["Support", "Amélioration", "Monthly eng", "Projet"] As t,
                                {
                                  T: t.Value,
                                  n: CountRows(Filter('[LME] Workload', TypeTache.Value = t.Value)),
                                  pct: Round(IfError(Average(Filter(colPct, TypeT = t.Value), pct), 0), 0)
                                }
                              )
                            ),
                            Notify("ClearCollect(colTypes) : " & FirstError.Message, NotificationType.Error)
                          );
                          IfError(
                            ClearCollect(colProjCount,
                              ForAll(Sequence(CountRows('[LME] Projets')),
                                With({p: Index('[LME] Projets', Value)},
                                  {
                                    k: Value,
                                    name: p.Title,
                                    n: CountRows(Filter('[LME] Workload', Projet.Id = p.ID, Statut.Value <> "Terminé"))
                                  }
                                )
                              )
                            ),
                            Notify("ClearCollect(colProjCount) : " & FirstError.Message, NotificationType.Error)
                          );
                          IfError(
                            Set(varTotalOuv, Sum(colProjCount, n)),
                            Notify("Set(varTotalOuv) : " & FirstError.Message, NotificationType.Error)
                          );
                          IfError(
                            ClearCollect(colDonut,
                              ForAll(colProjCount As c,
                                {
                                  k: c.k,
                                  name: c.name,
                                  n: c.n,
                                  share: Round(100 * c.n / Max(varTotalOuv, 1), 0),
                                  start: Round(100 * Sum(Filter(colProjCount, k < c.k), n) / Max(varTotalOuv, 1), 0)
                                }
                              )
                            ),
                            Notify("ClearCollect(colDonut) : " & FirstError.Message, NotificationType.Error)
                          );
                          IfError(
                            ClearCollect(colSem,
                              ForAll(Sequence(8, 0),
                                {
                                  i: Value,
                                  n: CountRows(Filter('[LME] Workload', Statut.Value = "Terminé",
                                       TermineLe >= DateAdd(Today(), 1 - 7 * (8 - Value), TimeUnit.Days),
                                       TermineLe < DateAdd(Today(), 1 - 7 * (7 - Value), TimeUnit.Days)))
                                }
                              )
                            ),
                            Notify("ClearCollect(colSem) : " & FirstError.Message, NotificationType.Error)
                          );
                          IfError(
                            If(!IsBlank(varSel), Set(varSel, LookUp('[LME] Workload', ID = varSel.ID))),
                            Notify("mise à jour de varSel : " & FirstError.Message, NotificationType.Error)
                          )
                        Size: =12
                        Font: =Font.'Segoe UI'
            - cntDash:
                Control: GroupContainer@1.5.0
                Variant: ManualLayout
                Properties:
                  X: =20
                  Y: =72
                  Width: =Parent.Width - 40
                  Height: =Parent.Height - 88
                  Visible: =Coalesce(varVue, "dash") = "dash"
                Children:
                  - lblTitreDash:
                      Control: Label@2.5.1
                      Properties:
                        X: =0
                        Y: =0
                        Width: =Parent.Width
                        Height: =40
                        Text: ="Tableau de bord"
                        Size: =22
                        FontWeight: =FontWeight.Bold
                        Color: =RGBA(16, 26, 46, 1)
                        Font: =Font.'Segoe UI'
                  - cntDashKpi:
                      Control: GroupContainer@1.5.0
                      Variant: ManualLayout
                      Properties:
                        X: =0
                        Y: =48
                        Width: =Parent.Width
                        Height: =92
                      Children:
                        - cntKpiOuvertes:
                            Control: GroupContainer@1.5.0
                            Variant: ManualLayout
                            Properties:
                              X: =0
                              Y: =0
                              Width: =(Parent.Width - 36) / 4
                              Height: =92
                              Fill: =RGBA(21, 62, 170, 1)
                            Children:
                              - lblKpiOuvertesV:
                                  Control: Label@2.5.1
                                  Properties:
                                    Text: =CountRows(Filter('[LME] Workload', Statut.Value <> "Terminé"))
                                    Size: =26
                                    Color: =RGBA(255, 255, 255, 1)
                                    X: =16
                                    Y: =10
                                    Width: =Parent.Width - 32
                                    Height: =46
                                    Font: =Font.'Segoe UI'
                                    FontWeight: =FontWeight.Bold
                              - lblKpiOuvertesC:
                                  Control: Label@2.5.1
                                  Properties:
                                    Text: ="Tâches ouvertes"
                                    Size: =12
                                    Color: =RGBA(255, 255, 255, 0.85)
                                    X: =16
                                    Y: =58
                                    Width: =Parent.Width - 32
                                    Height: =26
                                    Font: =Font.'Segoe UI'
                        - cntKpiRetard:
                            Control: GroupContainer@1.5.0
                            Variant: ManualLayout
                            Properties:
                              X: =(Parent.Width - 36) / 4 * 1 + 12
                              Y: =0
                              Width: =(Parent.Width - 36) / 4
                              Height: =92
                              Fill: =RGBA(255, 255, 255, 1)
                            Children:
                              - lblKpiRetardV:
                                  Control: Label@2.5.1
                                  Properties:
                                    Text: =CountRows(Filter('[LME] Workload', Statut.Value <> "Terminé", Echeance < Today()))
                                    Size: =26
                                    Color: =RGBA(192, 51, 43, 1)
                                    X: =16
                                    Y: =10
                                    Width: =Parent.Width - 32
                                    Height: =46
                                    Font: =Font.'Segoe UI'
                                    FontWeight: =FontWeight.Bold
                              - lblKpiRetardC:
                                  Control: Label@2.5.1
                                  Properties:
                                    Text: ="En retard"
                                    Size: =12
                                    Color: =RGBA(100, 114, 138, 1)
                                    X: =16
                                    Y: =58
                                    Width: =Parent.Width - 32
                                    Height: =26
                                    Font: =Font.'Segoe UI'
                        - cntKpiSemaine:
                            Control: GroupContainer@1.5.0
                            Variant: ManualLayout
                            Properties:
                              X: =(Parent.Width - 36) / 4 * 2 + 24
                              Y: =0
                              Width: =(Parent.Width - 36) / 4
                              Height: =92
                              Fill: =RGBA(255, 255, 255, 1)
                            Children:
                              - lblKpiSemaineV:
                                  Control: Label@2.5.1
                                  Properties:
                                    Text: =CountRows(Filter('[LME] Workload', Statut.Value = "Terminé", TermineLe >= DateAdd(Today(), -6, TimeUnit.Days)))
                                    Size: =26
                                    Color: =RGBA(19, 122, 74, 1)
                                    X: =16
                                    Y: =10
                                    Width: =Parent.Width - 32
                                    Height: =46
                                    Font: =Font.'Segoe UI'
                                    FontWeight: =FontWeight.Bold
                              - lblKpiSemaineC:
                                  Control: Label@2.5.1
                                  Properties:
                                    Text: ="Terminées sur 7 jours"
                                    Size: =12
                                    Color: =RGBA(100, 114, 138, 1)
                                    X: =16
                                    Y: =58
                                    Width: =Parent.Width - 32
                                    Height: =26
                                    Font: =Font.'Segoe UI'
                        - cntKpiTrier:
                            Control: GroupContainer@1.5.0
                            Variant: ManualLayout
                            Properties:
                              X: =(Parent.Width - 36) / 4 * 3 + 36
                              Y: =0
                              Width: =(Parent.Width - 36) / 4
                              Height: =92
                              Fill: =RGBA(255, 255, 255, 1)
                            Children:
                              - lblKpiTrierV:
                                  Control: Label@2.5.1
                                  Properties:
                                    Text: =CountRows(Filter('[LME] Workload', TypeTache.Value = "À trier", Statut.Value <> "Terminé"))
                                    Size: =26
                                    Color: =RGBA(16, 26, 46, 1)
                                    X: =16
                                    Y: =10
                                    Width: =Parent.Width - 32
                                    Height: =46
                                    Font: =Font.'Segoe UI'
                                    FontWeight: =FontWeight.Bold
                              - lblKpiTrierC:
                                  Control: Label@2.5.1
                                  Properties:
                                    Text: ="À trier"
                                    Size: =12
                                    Color: =RGBA(100, 114, 138, 1)
                                    X: =16
                                    Y: =58
                                    Width: =Parent.Width - 32
                                    Height: =26
                                    Font: =Font.'Segoe UI'
                  - cntDashRow2:
                      Control: GroupContainer@1.5.0
                      Variant: ManualLayout
                      Properties:
                        X: =0
                        Y: =152
                        Width: =Parent.Width
                        Height: =(Parent.Height - 164) * 0.5
                      Children:
                        - cntDashProjets:
                            Control: GroupContainer@1.5.0
                            Variant: ManualLayout
                            Properties:
                              X: =0
                              Y: =0
                              Width: =(Parent.Width - 24) * 0.38
                              Height: =Parent.Height
                              Fill: =RGBA(255, 255, 255, 1)
                            Children:
                              - lblHProj:
                                  Control: Label@2.5.1
                                  Properties:
                                    X: =16
                                    Y: =12
                                    Width: =Parent.Width - 32
                                    Height: =26
                                    Text: ="Avancement par projet"
                                    FontWeight: =FontWeight.Semibold
                                    Color: =RGBA(16, 26, 46, 1)
                                    Size: =14
                                    Font: =Font.'Segoe UI'
                              - galProjets:
                                  Control: Gallery@2.15.0
                                  Variant: Horizontal
                                  Properties:
                                    X: =8
                                    Y: =44
                                    Width: =Parent.Width - 16
                                    Height: =Parent.Height - 52
                                    Items: =Filter('[LME] Projets', StatutProjet.Value = "Actif")
                                    TemplateSize: =Min(190, Self.Width / Max(CountRows(Filter('[LME] Projets', StatutProjet.Value = "Actif")), 1))
                                    TemplatePadding: =0
                                  Children:
                                    - lblPctProj:
                                        Control: Label@2.5.1
                                        Properties:
                                          X: =0
                                          Y: =0
                                          Width: =Parent.TemplateWidth
                                          Height: =26
                                          Text: =Round(IfError(Average(Filter(colPct, ProjetId = ThisItem.ID), pct), 0), 0) & " %"
                                          Align: =Align.Center
                                          FontWeight: =FontWeight.Semibold
                                          Color: =RGBA(16, 26, 46, 1)
                                          Size: =13
                                          Font: =Font.'Segoe UI'
                                    - recTube:
                                        Control: Rectangle@2.3.0
                                        Properties:
                                          X: =Parent.TemplateWidth / 2 - 24
                                          Y: =32
                                          Width: =48
                                          Height: =Max(20, Parent.TemplateHeight - 82)
                                          Fill: =RGBA(238, 241, 247, 1)
                                    - recFillProj:
                                        Control: Rectangle@2.3.0
                                        Properties:
                                          X: =Parent.TemplateWidth / 2 - 24
                                          Y: =32 + Max(20, Parent.TemplateHeight - 82) - Self.Height
                                          Width: =48
                                          Height: =Max(8, Max(20, Parent.TemplateHeight - 82) * IfError(Average(Filter(colPct, ProjetId = ThisItem.ID), pct), 0) / 100)
                                          Fill: =RGBA(31, 79, 216, 1)
                                    - lblNomProj:
                                        Control: Label@2.5.1
                                        Properties:
                                          X: =0
                                          Y: =Parent.TemplateHeight - 46
                                          Width: =Parent.TemplateWidth
                                          Height: =42
                                          Text: =ThisItem.Title
                                          Align: =Align.Center
                                          Size: =11
                                          Color: =RGBA(100, 114, 138, 1)
                                          Font: =Font.'Segoe UI'
                        - cntDashNext:
                            Control: GroupContainer@1.5.0
                            Variant: ManualLayout
                            Properties:
                              X: =(Parent.Width - 24) * 0.38 + 12
                              Y: =0
                              Width: =(Parent.Width - 24) * 0.3
                              Height: =Parent.Height
                              Fill: =RGBA(255, 255, 255, 1)
                            Children:
                              - lblHNext:
                                  Control: Label@2.5.1
                                  Properties:
                                    X: =16
                                    Y: =12
                                    Width: =Parent.Width - 32
                                    Height: =26
                                    Text: ="Prochaine échéance"
                                    FontWeight: =FontWeight.Semibold
                                    Color: =RGBA(16, 26, 46, 1)
                                    Size: =14
                                    Font: =Font.'Segoe UI'
                              - lblNextTitre:
                                  Control: Label@2.5.1
                                  Properties:
                                    X: =16
                                    Y: =46
                                    Width: =Parent.Width - 32
                                    Height: =66
                                    Text: =Coalesce(varNext.Title, "Rien à échéance")
                                    Size: =14
                                    FontWeight: =FontWeight.Semibold
                                    Color: =RGBA(31, 79, 216, 1)
                                    Font: =Font.'Segoe UI'
                              - lblNextMeta:
                                  Control: Label@2.5.1
                                  Properties:
                                    X: =16
                                    Y: =116
                                    Width: =Parent.Width - 32
                                    Height: =24
                                    Text: =If(IsBlank(varNext), "", varNext.TypeTache.Value & " · " & varNext.Source.Value & " · " & Text(varNext.Echeance, "dd/mm"))
                                    Size: =11
                                    Color: =RGBA(100, 114, 138, 1)
                                    Font: =Font.'Segoe UI'
                              - btnNextTerminer:
                                  Control: Classic/Button@2.2.0
                                  Properties:
                                    X: =16
                                    Y: =Parent.Height - 100
                                    Width: =Parent.Width - 32
                                    Height: =42
                                    Text: ="Terminer partout"
                                    Fill: =RGBA(31, 79, 216, 1)
                                    Color: =RGBA(255, 255, 255, 1)
                                    Visible: =!IsBlank(varNext)
                                    OnSelect: |-
                                      =Patch('[LME] Workload', LookUp('[LME] Workload', ID = varNext.ID),
                                        {Statut: {Value: "Terminé"}, TermineLe: Now()});
                                      Select(btnRefresh);
                                      Notify("Clôture envoyée à Outlook, To Do et Loop par le flux.", NotificationType.Success)
                                    Size: =12
                                    Font: =Font.'Segoe UI'
                              - btnNextOuvrir:
                                  Control: Classic/Button@2.2.0
                                  Properties:
                                    X: =16
                                    Y: =Parent.Height - 52
                                    Width: =Parent.Width - 32
                                    Height: =38
                                    Text: ="Ouvrir le détail"
                                    Fill: =RGBA(255, 255, 255, 1)
                                    Color: =RGBA(16, 26, 46, 1)
                                    Visible: =!IsBlank(varNext)
                                    OnSelect: =Set(varSel, varNext)
                                    Size: =12
                                    Font: =Font.'Segoe UI'
                        - cntDashListe:
                            Control: GroupContainer@1.5.0
                            Variant: ManualLayout
                            Properties:
                              X: =(Parent.Width - 24) * 0.68 + 24
                              Y: =0
                              Width: =(Parent.Width - 24) * 0.32
                              Height: =Parent.Height
                              Fill: =RGBA(255, 255, 255, 1)
                            Children:
                              - lblHList:
                                  Control: Label@2.5.1
                                  Properties:
                                    X: =16
                                    Y: =12
                                    Width: =Parent.Width - 32
                                    Height: =26
                                    Text: ="À traiter"
                                    FontWeight: =FontWeight.Semibold
                                    Color: =RGBA(16, 26, 46, 1)
                                    Size: =14
                                    Font: =Font.'Segoe UI'
                              - galATraiter:
                                  Control: Gallery@2.15.0
                                  Variant: Vertical
                                  Properties:
                                    X: =8
                                    Y: =44
                                    Width: =Parent.Width - 16
                                    Height: =Parent.Height - 52
                                    Items: =FirstN(SortByColumns(Filter('[LME] Workload', Statut.Value <> "Terminé"), "Echeance", SortOrder.Ascending), 5)
                                    TemplateSize: =54
                                    TemplatePadding: =0
                                    OnSelect: =Set(varSel, ThisItem)
                                  Children:
                                    - lblATTitre:
                                        Control: Label@2.5.1
                                        Properties:
                                          X: =8
                                          Y: =4
                                          Width: =Parent.TemplateWidth - 76
                                          Height: =24
                                          Text: |-
                                            =With({t: ThisItem.Title}, Left(t, RoundDown((Parent.TemplateWidth - 76) / 6, 0)) & If(Len(t) > RoundDown((Parent.TemplateWidth - 76) / 6, 0), "…", ""))
                                          Size: =12
                                          Color: =RGBA(16, 26, 46, 1)
                                          Font: =Font.'Segoe UI'
                                    - lblATMeta:
                                        Control: Label@2.5.1
                                        Properties:
                                          X: =8
                                          Y: =29
                                          Width: =Parent.TemplateWidth - 76
                                          Height: =20
                                          Text: =ThisItem.TypeTache.Value
                                          Size: =10
                                          Color: =Coalesce(LookUp(colTC, T = ThisItem.TypeTache.Value, C), RGBA(107, 119, 140, 1))
                                          Font: =Font.'Segoe UI'
                                    - lblATDue:
                                        Control: Label@2.5.1
                                        Properties:
                                          X: =Parent.TemplateWidth - 66
                                          Y: =14
                                          Width: =58
                                          Height: =24
                                          Text: =Text(ThisItem.Echeance, "dd/mm")
                                          Align: =Align.Right
                                          Size: =11
                                          Color: =If(ThisItem.Echeance < Today(), RGBA(192, 51, 43, 1), RGBA(16, 26, 46, 1))
                                          Font: =Font.'Segoe UI'
                  - cntDashRow3:
                      Control: GroupContainer@1.5.0
                      Variant: ManualLayout
                      Properties:
                        X: =0
                        Y: =152 + (Parent.Height - 164) * 0.5 + 12
                        Width: =Parent.Width
                        Height: =(Parent.Height - 164) * 0.5
                      Children:
                        - cntDashType:
                            Control: GroupContainer@1.5.0
                            Variant: ManualLayout
                            Properties:
                              X: =0
                              Y: =0
                              Width: =(Parent.Width - 24) * 0.4
                              Height: =Parent.Height
                              Fill: =RGBA(255, 255, 255, 1)
                            Children:
                              - lblHType:
                                  Control: Label@2.5.1
                                  Properties:
                                    X: =16
                                    Y: =12
                                    Width: =Parent.Width - 32
                                    Height: =26
                                    Text: ="Avancement par type"
                                    FontWeight: =FontWeight.Semibold
                                    Color: =RGBA(16, 26, 46, 1)
                                    Size: =14
                                    Font: =Font.'Segoe UI'
                              - galParType:
                                  Control: Gallery@2.15.0
                                  Variant: Vertical
                                  Properties:
                                    X: =8
                                    Y: =44
                                    Width: =Parent.Width - 16
                                    Height: =Parent.Height - 52
                                    Items: =colTypes
                                    TemplateSize: =Max(44, Self.Height / 4)
                                    TemplatePadding: =0
                                  Children:
                                    - lblTypeNom:
                                        Control: Label@2.5.1
                                        Properties:
                                          X: =8
                                          Y: =2
                                          Width: =Parent.TemplateWidth - 100
                                          Height: =26
                                          Text: =ThisItem.T & " · " & ThisItem.n
                                          Size: =12
                                          Color: =RGBA(16, 26, 46, 1)
                                          Font: =Font.'Segoe UI'
                                    - lblTypePct:
                                        Control: Label@2.5.1
                                        Properties:
                                          X: =Parent.TemplateWidth - 90
                                          Y: =2
                                          Width: =82
                                          Height: =26
                                          Text: =ThisItem.pct & " %"
                                          Align: =Align.Right
                                          Size: =12
                                          FontWeight: =FontWeight.Semibold
                                          Color: =RGBA(16, 26, 46, 1)
                                          Font: =Font.'Segoe UI'
                                    - recTypeBack:
                                        Control: Rectangle@2.3.0
                                        Properties:
                                          X: =8
                                          Y: =32
                                          Width: =Parent.TemplateWidth - 16
                                          Height: =8
                                          Fill: =RGBA(238, 241, 247, 1)
                                    - recTypeFill:
                                        Control: Rectangle@2.3.0
                                        Properties:
                                          X: =8
                                          Y: =32
                                          Width: =(Parent.TemplateWidth - 16) * ThisItem.pct / 100
                                          Height: =8
                                          Fill: =Coalesce(LookUp(colTC, T = ThisItem.T, C), RGBA(31, 79, 216, 1))
                        - cntDashGauge:
                            Control: GroupContainer@1.5.0
                            Variant: ManualLayout
                            Properties:
                              X: =(Parent.Width - 24) * 0.40 + 12
                              Y: =0
                              Width: =(Parent.Width - 24) * 0.3
                              Height: =Parent.Height
                              Fill: =RGBA(255, 255, 255, 1)
                            Children:
                              - lblHGauge:
                                  Control: Label@2.5.1
                                  Properties:
                                    X: =16
                                    Y: =12
                                    Width: =Parent.Width - 32
                                    Height: =26
                                    Text: ="Avancement global"
                                    FontWeight: =FontWeight.Semibold
                                    Color: =RGBA(16, 26, 46, 1)
                                    Size: =14
                                    Font: =Font.'Segoe UI'
                              - imgGauge:
                                  Control: Image@2.2.3
                                  Properties:
                                    X: =16
                                    Y: =44
                                    Width: =Parent.Width - 32
                                    Height: =Parent.Height - 56
                                    Image: ="data:image/svg+xml;utf8," & EncodeUrl("<svg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 120 74'><path d='M10 64 A50 50 0 0 1 110 64' fill='none' stroke='#eef1f7' stroke-width='12' stroke-linecap='round'/><path d='M10 64 A50 50 0 0 1 110 64' pathLength='100' fill='none' stroke='#1f4fd8' stroke-width='12' stroke-linecap='" & If(varGlobal > 0, "round", "butt") & "' stroke-dasharray='" & varGlobal & " 100'/><text x='60' y='58' text-anchor='middle' font-family='Segoe UI,Arial,sans-serif' font-size='15' font-weight='700' fill='#101a2e'>" & varGlobal & "%</text></svg>")
                        - cntDashDark:
                            Control: GroupContainer@1.5.0
                            Variant: ManualLayout
                            Properties:
                              X: =(Parent.Width - 24) * 0.70 + 24
                              Y: =0
                              Width: =(Parent.Width - 24) * 0.3
                              Height: =Parent.Height
                              Fill: =RGBA(12, 29, 74, 1)
                            Children:
                              - lblDark:
                                  Control: Label@2.5.1
                                  Properties:
                                    X: =20
                                    Y: =20
                                    Width: =Parent.Width - 40
                                    Height: =Parent.Height - 40
                                    Text: =CountRows(Filter('[LME] Workload', Statut.Value = "Terminé")) & " / " & CountRows('[LME] Workload') & Char(10) & "tâches clôturées partout"
                                    Size: =20
                                    FontWeight: =FontWeight.Bold
                                    Color: =RGBA(255, 255, 255, 1)
                                    Font: =Font.'Segoe UI'
            - cntTasks:
                Control: GroupContainer@1.5.0
                Variant: ManualLayout
                Properties:
                  X: =20
                  Y: =72
                  Width: =Parent.Width - 40
                  Height: =Parent.Height - 88
                  Visible: =varVue = "tasks"
                Children:
                  - lblTitreTasks:
                      Control: Label@2.5.1
                      Properties:
                        X: =0
                        Y: =0
                        Width: =Parent.Width
                        Height: =40
                        Text: ="Tâches"
                        Size: =22
                        FontWeight: =FontWeight.Bold
                        Color: =RGBA(16, 26, 46, 1)
                        Font: =Font.'Segoe UI'
                  - cntTasksFiltres:
                      Control: GroupContainer@1.5.0
                      Variant: ManualLayout
                      Properties:
                        X: =0
                        Y: =48
                        Width: =Parent.Width
                        Height: =36
                      Children:
                        - btnChipAll:
                            Control: Classic/Button@2.2.0
                            Properties:
                              X: =0
                              Y: =0
                              Width: =120
                              Height: =36
                              Text: ="Toutes"
                              Fill: =If(IsBlank(varType), RGBA(31, 79, 216, 1), RGBA(255, 255, 255, 1))
                              Color: =If(IsBlank(varType), RGBA(255, 255, 255, 1), RGBA(16, 26, 46, 1))
                              OnSelect: =Set(varType, Blank())
                              Size: =12
                              Font: =Font.'Segoe UI'
                        - btnChipSupport:
                            Control: Classic/Button@2.2.0
                            Properties:
                              X: =128
                              Y: =0
                              Width: =120
                              Height: =36
                              Text: ="Support"
                              Fill: =If(varType = "Support", RGBA(31, 79, 216, 1), RGBA(255, 255, 255, 1))
                              Color: =If(varType = "Support", RGBA(255, 255, 255, 1), RGBA(16, 26, 46, 1))
                              OnSelect: =Set(varType, If(varType = "Support", Blank(), "Support"))
                              Size: =12
                              Font: =Font.'Segoe UI'
                        - btnChipAmel:
                            Control: Classic/Button@2.2.0
                            Properties:
                              X: =256
                              Y: =0
                              Width: =120
                              Height: =36
                              Text: ="Amélioration"
                              Fill: =If(varType = "Amélioration", RGBA(31, 79, 216, 1), RGBA(255, 255, 255, 1))
                              Color: =If(varType = "Amélioration", RGBA(255, 255, 255, 1), RGBA(16, 26, 46, 1))
                              OnSelect: =Set(varType, If(varType = "Amélioration", Blank(), "Amélioration"))
                              Size: =12
                              Font: =Font.'Segoe UI'
                        - btnChipMonthly:
                            Control: Classic/Button@2.2.0
                            Properties:
                              X: =384
                              Y: =0
                              Width: =120
                              Height: =36
                              Text: ="Monthly eng"
                              Fill: =If(varType = "Monthly eng", RGBA(31, 79, 216, 1), RGBA(255, 255, 255, 1))
                              Color: =If(varType = "Monthly eng", RGBA(255, 255, 255, 1), RGBA(16, 26, 46, 1))
                              OnSelect: =Set(varType, If(varType = "Monthly eng", Blank(), "Monthly eng"))
                              Size: =12
                              Font: =Font.'Segoe UI'
                        - btnChipProjet:
                            Control: Classic/Button@2.2.0
                            Properties:
                              X: =512
                              Y: =0
                              Width: =120
                              Height: =36
                              Text: ="Projet"
                              Fill: =If(varType = "Projet", RGBA(31, 79, 216, 1), RGBA(255, 255, 255, 1))
                              Color: =If(varType = "Projet", RGBA(255, 255, 255, 1), RGBA(16, 26, 46, 1))
                              OnSelect: =Set(varType, If(varType = "Projet", Blank(), "Projet"))
                              Size: =12
                              Font: =Font.'Segoe UI'
                  - galCols:
                      Control: Gallery@2.15.0
                      Variant: Horizontal
                      Properties:
                        X: =0
                        Y: =94
                        Width: =Parent.Width
                        Height: =Parent.Height - 94
                        Items: |-
                          =Table({S: "À faire"}, {S: "En cours"}, {S: "Bloqué"}, {S: "Terminé"})
                        TemplateSize: =Self.Width / 4
                        TemplatePadding: =6
                      Children:
                        - recCol:
                            Control: Rectangle@2.3.0
                            Properties:
                              X: =0
                              Y: =0
                              Width: =Parent.TemplateWidth
                              Height: =Parent.TemplateHeight
                              Fill: =RGBA(255, 255, 255, 1)
                        - lblColHead:
                            Control: Label@2.5.1
                            Properties:
                              X: =14
                              Y: =10
                              Width: =Parent.TemplateWidth - 28
                              Height: =28
                              Text: =ThisItem.S & " (" & CountRows(Filter('[LME] Workload', Statut.Value = ThisItem.S, TypeTache.Value <> "À trier")) & ")"
                              FontWeight: =FontWeight.Semibold
                              Color: =Switch(ThisItem.S, "En cours", RGBA(31, 79, 216, 1), "Bloqué", RGBA(192, 51, 43, 1), "Terminé", RGBA(19, 122, 74, 1), RGBA(100, 114, 138, 1))
                              Size: =14
                              Font: =Font.'Segoe UI'
                        - galCartes:
                            Control: Gallery@2.15.0
                            Variant: Vertical
                            Properties:
                              X: =8
                              Y: =44
                              Width: =Parent.TemplateWidth - 16
                              Height: =Parent.TemplateHeight - 52
                              Items: |-
                                =SortByColumns(
                                  Filter('[LME] Workload',
                                    Statut.Value = ThisItem.S,
                                    TypeTache.Value <> "À trier",
                                    IsBlank(varType) || TypeTache.Value = varType,
                                    IsBlank(txtRecherche.Text) || txtRecherche.Text in Title
                                  ),
                                  "Echeance", SortOrder.Ascending
                                )
                              TemplateSize: =128
                              TemplatePadding: =0
                              OnSelect: =Set(varSel, ThisItem)
                            Children:
                              - recCarte:
                                  Control: Rectangle@2.3.0
                                  Properties:
                                    X: =0
                                    Y: =0
                                    Width: =Parent.TemplateWidth
                                    Height: =Parent.TemplateHeight - 8
                                    Fill: =RGBA(244, 246, 250, 1)
                              - lblCType:
                                  Control: Label@2.5.1
                                  Properties:
                                    X: =10
                                    Y: =6
                                    Width: =Parent.TemplateWidth - 20
                                    Height: =18
                                    Text: =ThisItem.TypeTache.Value
                                    Size: =10
                                    FontWeight: =FontWeight.Semibold
                                    Color: =Coalesce(LookUp(colTC, T = ThisItem.TypeTache.Value, C), RGBA(107, 119, 140, 1))
                                    Font: =Font.'Segoe UI'
                              - lblCTitre:
                                  Control: Label@2.5.1
                                  Properties:
                                    X: =10
                                    Y: =26
                                    Width: =Parent.TemplateWidth - 20
                                    Height: =48
                                    Text: =ThisItem.Title
                                    Size: =12
                                    FontWeight: =FontWeight.Semibold
                                    Color: =If(ThisItem.Statut.Value = "Terminé", RGBA(100, 114, 138, 1), RGBA(16, 26, 46, 1))
                                    Font: =Font.'Segoe UI'
                              - recCBack:
                                  Control: Rectangle@2.3.0
                                  Properties:
                                    X: =10
                                    Y: =78
                                    Width: =Parent.TemplateWidth - 20
                                    Height: =6
                                    Fill: =RGBA(227, 232, 240, 1)
                              - recCFill:
                                  Control: Rectangle@2.3.0
                                  Properties:
                                    X: =10
                                    Y: =78
                                    Width: =(Parent.TemplateWidth - 20) * Coalesce(LookUp(colPct, ID = ThisItem.ID, pct), 0) / 100
                                    Height: =6
                                    Fill: =RGBA(31, 79, 216, 1)
                              - lblCDue:
                                  Control: Label@2.5.1
                                  Properties:
                                    X: =10
                                    Y: =90
                                    Width: =Parent.TemplateWidth - 100
                                    Height: =24
                                    Text: =Text(ThisItem.Echeance, "dd/mm") & " · " & Coalesce(LookUp(colPct, ID = ThisItem.ID, pct), 0) & " %"
                                    Size: =11
                                    Color: =If(ThisItem.Statut.Value <> "Terminé" && ThisItem.Echeance < Today(), RGBA(192, 51, 43, 1), RGBA(100, 114, 138, 1))
                                    Font: =Font.'Segoe UI'
                              - btnPrev:
                                  Control: Classic/Button@2.2.0
                                  Properties:
                                    X: =Parent.TemplateWidth - 84
                                    Y: =88
                                    Width: =36
                                    Height: =28
                                    Text: ="◀"
                                    Fill: =RGBA(255, 255, 255, 1)
                                    Color: =RGBA(16, 26, 46, 1)
                                    Visible: =ThisItem.Statut.Value <> "À faire"
                                    OnSelect: |-
                                      =Patch('[LME] Workload', LookUp('[LME] Workload', ID = ThisItem.ID),
                                        {Statut: {Value: Switch(ThisItem.Statut.Value, "En cours", "À faire", "Bloqué", "En cours", "Terminé", "Bloqué", "À faire")}});
                                      If(ThisItem.Statut.Value = "Terminé",
                                        Patch('[LME] Workload', LookUp('[LME] Workload', ID = ThisItem.ID), {TermineLe: Blank(), Propage: false}));
                                      Select(btnRefresh)
                                    Size: =12
                                    Font: =Font.'Segoe UI'
                              - btnNext:
                                  Control: Classic/Button@2.2.0
                                  Properties:
                                    X: =Parent.TemplateWidth - 44
                                    Y: =88
                                    Width: =36
                                    Height: =28
                                    Text: ="▶"
                                    Fill: =RGBA(255, 255, 255, 1)
                                    Color: =RGBA(16, 26, 46, 1)
                                    Visible: =ThisItem.Statut.Value <> "Terminé"
                                    OnSelect: |-
                                      =With({nv: Switch(ThisItem.Statut.Value, "À faire", "En cours", "En cours", "Bloqué", "Bloqué", "Terminé", "À faire")},
                                        Patch('[LME] Workload', LookUp('[LME] Workload', ID = ThisItem.ID), {Statut: {Value: nv}});
                                        If(nv = "Terminé",
                                          Patch('[LME] Workload', LookUp('[LME] Workload', ID = ThisItem.ID), {TermineLe: Now()});
                                          Notify("Clôture envoyée à Outlook, To Do et Loop par le flux.", NotificationType.Success))
                                      );
                                      Select(btnRefresh)
                                    Size: =12
                                    Font: =Font.'Segoe UI'
            - cntCal:
                Control: GroupContainer@1.5.0
                Variant: ManualLayout
                Properties:
                  X: =20
                  Y: =72
                  Width: =Parent.Width - 40
                  Height: =Parent.Height - 88
                  Visible: =varVue = "cal"
                Children:
                  - lblTitreCal:
                      Control: Label@2.5.1
                      Properties:
                        X: =0
                        Y: =0
                        Width: =330
                        Height: =40
                        Text: =Text(Date(Coalesce(varAn, Year(Today())), Coalesce(varMois, Month(Today())), 1), "mmmm yyyy")
                        Size: =22
                        FontWeight: =FontWeight.Bold
                        Color: =RGBA(16, 26, 46, 1)
                        Font: =Font.'Segoe UI'
                  - btnMoisPrec:
                      Control: Classic/Button@2.2.0
                      Properties:
                        X: =340
                        Y: =3
                        Width: =40
                        Height: =34
                        Text: ="◀"
                        Fill: =RGBA(255, 255, 255, 1)
                        Color: =RGBA(16, 26, 46, 1)
                        OnSelect: |-
                          =Set(varAn, Coalesce(varAn, Year(Today())));
                          Set(varMois, Coalesce(varMois, Month(Today())));
                          If(varMois = 1,
                            Set(varAn, varAn - 1); Set(varMois, 12),
                            Set(varMois, varMois - 1))
                        Size: =12
                        Font: =Font.'Segoe UI'
                  - btnMoisSuiv:
                      Control: Classic/Button@2.2.0
                      Properties:
                        X: =388
                        Y: =3
                        Width: =40
                        Height: =34
                        Text: ="▶"
                        Fill: =RGBA(255, 255, 255, 1)
                        Color: =RGBA(16, 26, 46, 1)
                        OnSelect: |-
                          =Set(varAn, Coalesce(varAn, Year(Today())));
                          Set(varMois, Coalesce(varMois, Month(Today())));
                          If(varMois = 12,
                            Set(varAn, varAn + 1); Set(varMois, 1),
                            Set(varMois, varMois + 1))
                        Size: =12
                        Font: =Font.'Segoe UI'
                  - cntCalGrille:
                      Control: GroupContainer@1.5.0
                      Variant: ManualLayout
                      Properties:
                        X: =0
                        Y: =52
                        Width: =Parent.Width * 0.68
                        Height: =Parent.Height - 52
                      Children:
                        - galJoursEntete:
                            Control: Gallery@2.15.0
                            Variant: Horizontal
                            Properties:
                              X: =0
                              Y: =0
                              Width: =Parent.Width
                              Height: =28
                              Items: |-
                                =Table({j: "lun"}, {j: "mar"}, {j: "mer"}, {j: "jeu"}, {j: "ven"}, {j: "sam"}, {j: "dim"})
                              TemplateSize: =Self.Width / 7
                              TemplatePadding: =0
                            Children:
                              - lblJourEntete:
                                  Control: Label@2.5.1
                                  Properties:
                                    X: =0
                                    Y: =0
                                    Width: =Parent.TemplateWidth
                                    Height: =Parent.TemplateHeight
                                    Text: =Upper(ThisItem.j)
                                    Align: =Align.Center
                                    Size: =10
                                    Color: =RGBA(100, 114, 138, 1)
                                    Font: =Font.'Segoe UI'
                        - galJours:
                            Control: Gallery@2.15.0
                            Variant: Vertical
                            Properties:
                              X: =0
                              Y: =32
                              Width: =Parent.Width
                              Height: =Parent.Height - 32
                              Items: |-
                                =With({debut: DateAdd(Date(Coalesce(varAn, Year(Today())), Coalesce(varMois, Month(Today())), 1), -(Weekday(Date(Coalesce(varAn, Year(Today())), Coalesce(varMois, Month(Today())), 1), StartOfWeek.Monday) - 1), TimeUnit.Days)},
                                  ForAll(Sequence(42, 0), {d: DateAdd(debut, Value, TimeUnit.Days)})
                                )
                              WrapCount: =7
                              TemplateSize: =Self.Height / 6
                              TemplatePadding: =2
                            Children:
                              - recJour:
                                  Control: Rectangle@2.3.0
                                  Properties:
                                    X: =0
                                    Y: =0
                                    Width: =Parent.TemplateWidth
                                    Height: =Parent.TemplateHeight
                                    Fill: =If(ThisItem.d = Today(), RGBA(227, 235, 255, 1), RGBA(255, 255, 255, 1))
                                    BorderColor: =If(ThisItem.d = Today(), RGBA(31, 79, 216, 1), RGBA(227, 232, 240, 1))
                                    BorderThickness: =1
                              - lblNumJour:
                                  Control: Label@2.5.1
                                  Properties:
                                    X: =6
                                    Y: =4
                                    Width: =40
                                    Height: =22
                                    Text: =Day(ThisItem.d)
                                    Size: =11
                                    Color: =If(Month(ThisItem.d) = Coalesce(varMois, Month(Today())), RGBA(16, 26, 46, 1), RGBA(170, 180, 196, 1))
                                    Font: =Font.'Segoe UI'
                              - lblEv1:
                                  Control: Label@2.5.1
                                  Properties:
                                    X: =4
                                    Y: =28
                                    Width: =Parent.TemplateWidth - 8
                                    Height: =20
                                    Text: |-
                                      =With({t: Coalesce(Last(FirstN(Filter(colWL, Echeance = ThisItem.d), 1)).Title, "")}, Left(t, RoundDown((Parent.TemplateWidth - 8) / 5.4, 0)) & If(Len(t) > RoundDown((Parent.TemplateWidth - 8) / 5.4, 0), "…", ""))
                                    Size: =10
                                    Fill: =If(IsBlank(Self.Text), RGBA(0, 0, 0, 0), RGBA(227, 235, 255, 1))
                                    Color: =RGBA(31, 79, 216, 1)
                                    Font: =Font.'Segoe UI'
                              - lblEv2:
                                  Control: Label@2.5.1
                                  Properties:
                                    X: =4
                                    Y: =50
                                    Width: =Parent.TemplateWidth - 8
                                    Height: =20
                                    Text: |-
                                      =With({t: If(CountRows(Filter(colWL, Echeance = ThisItem.d)) >= 2, Last(FirstN(Filter(colWL, Echeance = ThisItem.d), 2)).Title, "")}, Left(t, RoundDown((Parent.TemplateWidth - 8) / 5.4, 0)) & If(Len(t) > RoundDown((Parent.TemplateWidth - 8) / 5.4, 0), "…", ""))
                                    Size: =10
                                    Fill: =If(IsBlank(Self.Text), RGBA(0, 0, 0, 0), RGBA(227, 235, 255, 1))
                                    Color: =RGBA(31, 79, 216, 1)
                                    Font: =Font.'Segoe UI'
                              - lblEvPlus:
                                  Control: Label@2.5.1
                                  Properties:
                                    X: =4
                                    Y: =72
                                    Width: =60
                                    Height: =18
                                    Text: =If(CountRows(Filter(colWL, Echeance = ThisItem.d)) > 2, "+" & (CountRows(Filter(colWL, Echeance = ThisItem.d)) - 2), "")
                                    Size: =10
                                    Color: =RGBA(100, 114, 138, 1)
                                    Font: =Font.'Segoe UI'
                  - cntCalCote:
                      Control: GroupContainer@1.5.0
                      Variant: ManualLayout
                      Properties:
                        X: =Parent.Width * 0.68 + 16
                        Y: =52
                        Width: =Parent.Width * 0.32 - 16
                        Height: =Parent.Height - 52
                      Children:
                        - cntCalAuj:
                            Control: GroupContainer@1.5.0
                            Variant: ManualLayout
                            Properties:
                              X: =0
                              Y: =0
                              Width: =Parent.Width
                              Height: =Parent.Height * 0.32 - 6
                              Fill: =RGBA(255, 255, 255, 1)
                            Children:
                              - lblHAuj:
                                  Control: Label@2.5.1
                                  Properties:
                                    X: =16
                                    Y: =12
                                    Width: =Parent.Width - 32
                                    Height: =26
                                    Text: ="Aujourd'hui"
                                    FontWeight: =FontWeight.Semibold
                                    Color: =RGBA(16, 26, 46, 1)
                                    Size: =14
                                    Font: =Font.'Segoe UI'
                              - galAuj:
                                  Control: Gallery@2.15.0
                                  Variant: Vertical
                                  Properties:
                                    X: =8
                                    Y: =44
                                    Width: =Parent.Width - 16
                                    Height: =Parent.Height - 52
                                    Items: =Filter('[LME] Workload', Echeance = Today(), Statut.Value <> "Terminé")
                                    TemplateSize: =52
                                    TemplatePadding: =0
                                    OnSelect: =Set(varSel, ThisItem)
                                  Children:
                                    - lblAujTitre:
                                        Control: Label@2.5.1
                                        Properties:
                                          X: =6
                                          Y: =4
                                          Width: =Parent.TemplateWidth - 12
                                          Height: =24
                                          Text: |-
                                            =With({t: ThisItem.Title}, Left(t, RoundDown((Parent.TemplateWidth - 12) / 6, 0)) & If(Len(t) > RoundDown((Parent.TemplateWidth - 12) / 6, 0), "…", ""))
                                          Size: =12
                                          Color: =RGBA(16, 26, 46, 1)
                                          Font: =Font.'Segoe UI'
                                    - lblAujMeta:
                                        Control: Label@2.5.1
                                        Properties:
                                          X: =6
                                          Y: =29
                                          Width: =Parent.TemplateWidth - 12
                                          Height: =20
                                          Text: =ThisItem.TypeTache.Value & " · " & Coalesce(LookUp(colPct, ID = ThisItem.ID, pct), 0) & " %"
                                          Size: =10
                                          Color: =RGBA(100, 114, 138, 1)
                                          Font: =Font.'Segoe UI'
                        - cntCalAvenir:
                            Control: GroupContainer@1.5.0
                            Variant: ManualLayout
                            Properties:
                              X: =0
                              Y: =Parent.Height * 0.32 + 6
                              Width: =Parent.Width
                              Height: =Parent.Height * 0.68 - 6
                              Fill: =RGBA(255, 255, 255, 1)
                            Children:
                              - lblHAvenir:
                                  Control: Label@2.5.1
                                  Properties:
                                    X: =16
                                    Y: =12
                                    Width: =Parent.Width - 32
                                    Height: =26
                                    Text: ="À venir"
                                    FontWeight: =FontWeight.Semibold
                                    Color: =RGBA(16, 26, 46, 1)
                                    Size: =14
                                    Font: =Font.'Segoe UI'
                              - galAvenir:
                                  Control: Gallery@2.15.0
                                  Variant: Vertical
                                  Properties:
                                    X: =8
                                    Y: =44
                                    Width: =Parent.Width - 16
                                    Height: =Parent.Height - 52
                                    Items: =FirstN(SortByColumns(Filter('[LME] Workload', Echeance > Today(), Statut.Value <> "Terminé"), "Echeance", SortOrder.Ascending), 8)
                                    TemplateSize: =54
                                    TemplatePadding: =0
                                    OnSelect: =Set(varSel, ThisItem)
                                  Children:
                                    - lblAvTitre:
                                        Control: Label@2.5.1
                                        Properties:
                                          X: =6
                                          Y: =4
                                          Width: =Parent.TemplateWidth - 76
                                          Height: =24
                                          Text: |-
                                            =With({t: ThisItem.Title}, Left(t, RoundDown((Parent.TemplateWidth - 76) / 6, 0)) & If(Len(t) > RoundDown((Parent.TemplateWidth - 76) / 6, 0), "…", ""))
                                          Size: =12
                                          Color: =RGBA(16, 26, 46, 1)
                                          Font: =Font.'Segoe UI'
                                    - lblAvMeta:
                                        Control: Label@2.5.1
                                        Properties:
                                          X: =6
                                          Y: =29
                                          Width: =Parent.TemplateWidth - 76
                                          Height: =20
                                          Text: =ThisItem.TypeTache.Value
                                          Size: =10
                                          Color: =Coalesce(LookUp(colTC, T = ThisItem.TypeTache.Value, C), RGBA(107, 119, 140, 1))
                                          Font: =Font.'Segoe UI'
                                    - lblAvDate:
                                        Control: Label@2.5.1
                                        Properties:
                                          X: =Parent.TemplateWidth - 66
                                          Y: =14
                                          Width: =58
                                          Height: =24
                                          Text: =Text(ThisItem.Echeance, "dd/mm")
                                          Align: =Align.Right
                                          Size: =11
                                          Color: =RGBA(16, 26, 46, 1)
                                          Font: =Font.'Segoe UI'
            - cntStats:
                Control: GroupContainer@1.5.0
                Variant: ManualLayout
                Properties:
                  X: =20
                  Y: =72
                  Width: =Parent.Width - 40
                  Height: =Parent.Height - 88
                  Visible: =varVue = "stats"
                Children:
                  - lblTitreStats:
                      Control: Label@2.5.1
                      Properties:
                        X: =0
                        Y: =0
                        Width: =Parent.Width
                        Height: =40
                        Text: ="Analyse"
                        Size: =22
                        FontWeight: =FontWeight.Bold
                        Color: =RGBA(16, 26, 46, 1)
                        Font: =Font.'Segoe UI'
                  - cntStatsKpi:
                      Control: GroupContainer@1.5.0
                      Variant: ManualLayout
                      Properties:
                        X: =0
                        Y: =48
                        Width: =Parent.Width
                        Height: =92
                      Children:
                        - cntSt1:
                            Control: GroupContainer@1.5.0
                            Variant: ManualLayout
                            Properties:
                              X: =0
                              Y: =0
                              Width: =(Parent.Width - 36) / 4
                              Height: =92
                              Fill: =RGBA(255, 255, 255, 1)
                            Children:
                              - lblSt1V:
                                  Control: Label@2.5.1
                                  Properties:
                                    Text: =CountRows(Filter('[LME] Workload', Statut.Value = "Terminé"))
                                    Size: =26
                                    Color: =RGBA(31, 79, 216, 1)
                                    X: =16
                                    Y: =10
                                    Width: =Parent.Width - 32
                                    Height: =46
                                    Font: =Font.'Segoe UI'
                                    FontWeight: =FontWeight.Bold
                              - lblSt1C:
                                  Control: Label@2.5.1
                                  Properties:
                                    Text: ="Tâches terminées"
                                    Size: =12
                                    Color: =RGBA(100, 114, 138, 1)
                                    X: =16
                                    Y: =58
                                    Width: =Parent.Width - 32
                                    Height: =26
                                    Font: =Font.'Segoe UI'
                        - cntSt2:
                            Control: GroupContainer@1.5.0
                            Variant: ManualLayout
                            Properties:
                              X: =(Parent.Width - 36) / 4 * 1 + 12
                              Y: =0
                              Width: =(Parent.Width - 36) / 4
                              Height: =92
                              Fill: =RGBA(255, 255, 255, 1)
                            Children:
                              - lblSt2V:
                                  Control: Label@2.5.1
                                  Properties:
                                    Text: =CountRows(Filter(colEtapes, Fait))
                                    Size: =26
                                    Color: =RGBA(14, 138, 125, 1)
                                    X: =16
                                    Y: =10
                                    Width: =Parent.Width - 32
                                    Height: =46
                                    Font: =Font.'Segoe UI'
                                    FontWeight: =FontWeight.Bold
                              - lblSt2C:
                                  Control: Label@2.5.1
                                  Properties:
                                    Text: ="Étapes cochées"
                                    Size: =12
                                    Color: =RGBA(100, 114, 138, 1)
                                    X: =16
                                    Y: =58
                                    Width: =Parent.Width - 32
                                    Height: =26
                                    Font: =Font.'Segoe UI'
                        - cntSt3:
                            Control: GroupContainer@1.5.0
                            Variant: ManualLayout
                            Properties:
                              X: =(Parent.Width - 36) / 4 * 2 + 24
                              Y: =0
                              Width: =(Parent.Width - 36) / 4
                              Height: =92
                              Fill: =RGBA(255, 255, 255, 1)
                            Children:
                              - lblSt3V:
                                  Control: Label@2.5.1
                                  Properties:
                                    Text: |-
                                      =With({fin: Filter('[LME] Workload', Statut.Value = "Terminé")},
                                        If(CountRows(fin) = 0, "0", Round(100 * CountRows(Filter(fin, Date(Year(TermineLe), Month(TermineLe), Day(TermineLe)) <= Echeance)) / CountRows(fin), 0)) & " %"
                                      )
                                    Size: =26
                                    Color: =RGBA(19, 122, 74, 1)
                                    X: =16
                                    Y: =10
                                    Width: =Parent.Width - 32
                                    Height: =46
                                    Font: =Font.'Segoe UI'
                                    FontWeight: =FontWeight.Bold
                              - lblSt3C:
                                  Control: Label@2.5.1
                                  Properties:
                                    Text: ="Clôturées dans les temps"
                                    Size: =12
                                    Color: =RGBA(100, 114, 138, 1)
                                    X: =16
                                    Y: =58
                                    Width: =Parent.Width - 32
                                    Height: =26
                                    Font: =Font.'Segoe UI'
                        - cntSt4:
                            Control: GroupContainer@1.5.0
                            Variant: ManualLayout
                            Properties:
                              X: =(Parent.Width - 36) / 4 * 3 + 36
                              Y: =0
                              Width: =(Parent.Width - 36) / 4
                              Height: =92
                              Fill: =RGBA(255, 255, 255, 1)
                            Children:
                              - lblSt4V:
                                  Control: Label@2.5.1
                                  Properties:
                                    Text: =CountRows(Filter('[LME] Workload', Statut.Value <> "Terminé", Echeance < Today()))
                                    Size: =26
                                    Color: =RGBA(192, 51, 43, 1)
                                    X: =16
                                    Y: =10
                                    Width: =Parent.Width - 32
                                    Height: =46
                                    Font: =Font.'Segoe UI'
                                    FontWeight: =FontWeight.Bold
                              - lblSt4C:
                                  Control: Label@2.5.1
                                  Properties:
                                    Text: ="En retard"
                                    Size: =12
                                    Color: =RGBA(100, 114, 138, 1)
                                    X: =16
                                    Y: =58
                                    Width: =Parent.Width - 32
                                    Height: =26
                                    Font: =Font.'Segoe UI'
                  - cntStatsRow:
                      Control: GroupContainer@1.5.0
                      Variant: ManualLayout
                      Properties:
                        X: =0
                        Y: =152
                        Width: =Parent.Width
                        Height: =Parent.Height - 152
                      Children:
                        - cntStatsCourbe:
                            Control: GroupContainer@1.5.0
                            Variant: ManualLayout
                            Properties:
                              X: =0
                              Y: =0
                              Width: =(Parent.Width - 12) * 0.64
                              Height: =Parent.Height
                              Fill: =RGBA(255, 255, 255, 1)
                            Children:
                              - lblHCourbe:
                                  Control: Label@2.5.1
                                  Properties:
                                    X: =16
                                    Y: =12
                                    Width: =Parent.Width - 32
                                    Height: =26
                                    Text: ="Tâches terminées par semaine (8 dernières semaines)"
                                    FontWeight: =FontWeight.Semibold
                                    Color: =RGBA(16, 26, 46, 1)
                                    Size: =14
                                    Font: =Font.'Segoe UI'
                              - imgCourbe:
                                  Control: Image@2.2.3
                                  Properties:
                                    X: =16
                                    Y: =48
                                    Width: =Parent.Width - 32
                                    Height: =Parent.Height - 64
                                    Image: ="data:image/svg+xml;utf8," & EncodeUrl("<svg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 640 190'><polygon fill='#1f4fd8' fill-opacity='0.15' points='" & Concat(colSem, Text(20 + i * 85) & "," & Text(170 - Round(150 * n / Max(Max(colSem, n), 1), 0)) & " ") & "605,170 20,170'/><polyline fill='none' stroke='#1f4fd8' stroke-width='3' points='" & Concat(colSem, Text(20 + i * 85) & "," & Text(170 - Round(150 * n / Max(Max(colSem, n), 1), 0)) & " ") & "'/>" & Concat(colSem, "<text x='" & Text(20 + i * 85) & "' y='186' font-family='Segoe UI,Arial,sans-serif' font-size='11' text-anchor='middle' fill='#64728a'>" & n & "</text>") & "</svg>")
                        - cntStatsDonut:
                            Control: GroupContainer@1.5.0
                            Variant: ManualLayout
                            Properties:
                              X: =(Parent.Width - 12) * 0.64 + 12
                              Y: =0
                              Width: =(Parent.Width - 12) * 0.36
                              Height: =Parent.Height
                              Fill: =RGBA(255, 255, 255, 1)
                            Children:
                              - lblHDonut:
                                  Control: Label@2.5.1
                                  Properties:
                                    X: =16
                                    Y: =12
                                    Width: =Parent.Width - 32
                                    Height: =26
                                    Text: ="Tâches ouvertes par projet"
                                    FontWeight: =FontWeight.Semibold
                                    Color: =RGBA(16, 26, 46, 1)
                                    Size: =14
                                    Font: =Font.'Segoe UI'
                              - imgDonut:
                                  Control: Image@2.2.3
                                  Properties:
                                    X: =(Parent.Width - Self.Width) / 2
                                    Y: =48
                                    Width: =Min(Parent.Width - 32, Parent.Height * 0.5)
                                    Height: =Min(Parent.Width - 32, Parent.Height * 0.5)
                                    Image: ="data:image/svg+xml;utf8," & EncodeUrl("<svg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 120 120'>" & Concat(colDonut, "<circle r='40' cx='60' cy='60' fill='none' pathLength='100' stroke='" & Switch(k, 1, "#1f4fd8", 2, "#0e8a7d", 3, "#b86a00", "#6b778c") & "' stroke-width='16' stroke-dasharray='" & share & " " & (100 - share) & "' stroke-dashoffset='" & -start & "' transform='rotate(-90 60 60)'/>") & "<text x='60' y='58' text-anchor='middle' font-family='Segoe UI,Arial,sans-serif' font-size='16' font-weight='700' fill='#101a2e'>" & varTotalOuv & "</text><text x='60' y='73' text-anchor='middle' font-family='Segoe UI,Arial,sans-serif' font-size='9' fill='#64728a'>ouvertes</text></svg>")
                              - lblLegende:
                                  Control: Label@2.5.1
                                  Properties:
                                    X: =16
                                    Y: =56 + Parent.Height * 0.5
                                    Width: =Parent.Width - 32
                                    Height: =Parent.Height - 56 - Parent.Height * 0.5 - 8
                                    Text: |-
                                      =Concat(colDonut, name & " : " & n, Char(10))
                                    Size: =12
                                    Color: =RGBA(16, 26, 46, 1)
                                    Font: =Font.'Segoe UI'
            - cntTrier:
                Control: GroupContainer@1.5.0
                Variant: ManualLayout
                Properties:
                  X: =20
                  Y: =72
                  Width: =Parent.Width - 40
                  Height: =Parent.Height - 88
                  Visible: =varVue = "trier"
                Children:
                  - lblTitreTrier:
                      Control: Label@2.5.1
                      Properties:
                        X: =0
                        Y: =0
                        Width: =Parent.Width
                        Height: =40
                        Text: ="À trier"
                        Size: =22
                        FontWeight: =FontWeight.Bold
                        Color: =RGBA(16, 26, 46, 1)
                        Font: =Font.'Segoe UI'
                  - lblSousTrier:
                      Control: Label@2.5.1
                      Properties:
                        X: =0
                        Y: =42
                        Width: =Parent.Width
                        Height: =26
                        Text: ="Mails sans catégorie reconnue et tâches manuelles. Choisis le type et le projet, les étapes du modèle se créent ensuite."
                        Size: =12
                        Color: =RGBA(100, 114, 138, 1)
                        Font: =Font.'Segoe UI'
                  - lblVideTrier:
                      Control: Label@2.5.1
                      Properties:
                        X: =0
                        Y: =110
                        Width: =Parent.Width
                        Height: =40
                        Text: ="Rien à trier."
                        Align: =Align.Center
                        Color: =RGBA(100, 114, 138, 1)
                        Visible: =CountRows(galTrier.AllItems) = 0
                        Size: =13
                        Font: =Font.'Segoe UI'
                  - galTrier:
                      Control: Gallery@2.15.0
                      Variant: Vertical
                      Properties:
                        X: =0
                        Y: =76
                        Width: =Parent.Width
                        Height: =Parent.Height - 76
                        Items: =Filter('[LME] Workload', TypeTache.Value = "À trier", Statut.Value <> "Terminé")
                        TemplateSize: =100
                        TemplatePadding: =6
                      Children:
                        - recTrier:
                            Control: Rectangle@2.3.0
                            Properties:
                              X: =0
                              Y: =0
                              Width: =Parent.TemplateWidth
                              Height: =Parent.TemplateHeight - 6
                              Fill: =RGBA(255, 255, 255, 1)
                        - lblTrierTitre:
                            Control: Label@2.5.1
                            Properties:
                              X: =16
                              Y: =10
                              Width: =Parent.TemplateWidth - 32
                              Height: =26
                              Text: =ThisItem.Title
                              FontWeight: =FontWeight.Semibold
                              Color: =RGBA(16, 26, 46, 1)
                              Size: =13
                              Font: =Font.'Segoe UI'
                        - lblTrierMeta:
                            Control: Label@2.5.1
                            Properties:
                              X: =16
                              Y: =38
                              Width: =Parent.TemplateWidth - 32
                              Height: =20
                              Text: =ThisItem.Source.Value & " · échéance " & Text(ThisItem.Echeance, "dd/mm")
                              Size: =11
                              Color: =RGBA(100, 114, 138, 1)
                              Font: =Font.'Segoe UI'
                        - ddTrierType:
                            Control: Classic/DropDown@2.3.1
                            Properties:
                              X: =16
                              Y: =62
                              Width: =Parent.TemplateWidth * 0.25
                              Height: =32
                              Items: =["Support", "Amélioration", "Monthly eng", "Projet"]
                              Size: =12
                              Font: =Font.'Segoe UI'
                        - ddTrierProjet:
                            Control: Classic/DropDown@2.3.1
                            Properties:
                              X: =Parent.TemplateWidth * 0.25 + 28
                              Y: =62
                              Width: =Parent.TemplateWidth * 0.30
                              Height: =32
                              Items: |-
                                =ForAll(Filter('[LME] Projets', StatutProjet.Value = "Actif"), {Value: Title, ID: ID})
                              Size: =12
                              Font: =Font.'Segoe UI'
                        - btnClasser:
                            Control: Classic/Button@2.2.0
                            Properties:
                              X: =Parent.TemplateWidth * 0.55 + 40
                              Y: =62
                              Width: =130
                              Height: =32
                              Text: ="Classer"
                              Fill: =RGBA(31, 79, 216, 1)
                              Color: =RGBA(255, 255, 255, 1)
                              OnSelect: |-
                                =Patch('[LME] Workload', LookUp('[LME] Workload', ID = ThisItem.ID),
                                  {
                                    TypeTache: {Value: ddTrierType.Selected.Value},
                                    Projet: {Id: ddTrierProjet.Selected.ID, Value: ddTrierProjet.Selected.Value}
                                  });
                                Select(btnRefresh);
                                Notify("Classée : les étapes du modèle vont être créées par le flux.", NotificationType.Success)
                              Size: =12
                              Font: =Font.'Segoe UI'
      - cntDrawer:
          Control: GroupContainer@1.5.0
          Variant: ManualLayout
          Properties:
            X: =Parent.Width - 420
            Y: =0
            Width: =420
            Height: =Parent.Height
            Visible: =!IsBlank(varSel)
            Fill: =RGBA(255, 255, 255, 1)
          Children:
            - btnFermer:
                Control: Classic/Button@2.2.0
                Properties:
                  X: =Parent.Width - 54
                  Y: =12
                  Width: =38
                  Height: =38
                  Text: ="✕"
                  Fill: =RGBA(255, 255, 255, 1)
                  Color: =RGBA(16, 26, 46, 1)
                  OnSelect: =Set(varSel, Blank())
                  Size: =12
                  Font: =Font.'Segoe UI'
            - lblDType:
                Control: Label@2.5.1
                Properties:
                  X: =20
                  Y: =18
                  Width: =320
                  Height: =22
                  Text: =varSel.TypeTache.Value
                  Size: =11
                  FontWeight: =FontWeight.Semibold
                  Color: =Coalesce(LookUp(colTC, T = varSel.TypeTache.Value, C), RGBA(107, 119, 140, 1))
                  Font: =Font.'Segoe UI'
            - lblDTitre:
                Control: Label@2.5.1
                Properties:
                  X: =20
                  Y: =44
                  Width: =372
                  Height: =66
                  Text: =varSel.Title
                  Size: =16
                  FontWeight: =FontWeight.Semibold
                  Color: =RGBA(16, 26, 46, 1)
                  Font: =Font.'Segoe UI'
            - lblDMeta:
                Control: Label@2.5.1
                Properties:
                  X: =20
                  Y: =114
                  Width: =380
                  Height: =44
                  Text: =varSel.Statut.Value & " · " & varSel.Source.Value & " · " & Coalesce(varSel.Projet.Value, "sans projet") & Char(10) & "Échéance " & Text(varSel.Echeance, "dd/mm/yyyy")
                  Size: =12
                  Color: =RGBA(100, 114, 138, 1)
                  Font: =Font.'Segoe UI'
            - lblDPct:
                Control: Label@2.5.1
                Properties:
                  X: =20
                  Y: =162
                  Width: =380
                  Height: =26
                  Text: =Coalesce(LookUp(colPct, ID = varSel.ID, pct), 0) & " %  ·  " & CountRows(Filter(colEtapes, Tache.Id = varSel.ID, Fait)) & " étape(s) sur " & CountRows(Filter(colEtapes, Tache.Id = varSel.ID))
                  FontWeight: =FontWeight.Semibold
                  Color: =RGBA(31, 79, 216, 1)
                  Size: =13
                  Font: =Font.'Segoe UI'
            - recDBack:
                Control: Rectangle@2.3.0
                Properties:
                  X: =20
                  Y: =194
                  Width: =380
                  Height: =8
                  Fill: =RGBA(238, 241, 247, 1)
            - recDFill:
                Control: Rectangle@2.3.0
                Properties:
                  X: =20
                  Y: =194
                  Width: =380 * Coalesce(LookUp(colPct, ID = varSel.ID, pct), 0) / 100
                  Height: =8
                  Fill: =RGBA(31, 79, 216, 1)
            - galEtapes:
                Control: Gallery@2.15.0
                Variant: Vertical
                Properties:
                  X: =20
                  Y: =212
                  Width: =380
                  Height: =Parent.Height - 212 - 250
                  Items: =SortByColumns(Filter(colEtapes, Tache.Id = varSel.ID), "Ordre")
                  TemplateSize: =38
                  TemplatePadding: =0
                Children:
                  - chkEtape:
                      Control: Classic/CheckBox@2.1.0
                      Properties:
                        X: =0
                        Y: =0
                        Width: =Parent.TemplateWidth
                        Height: =Parent.TemplateHeight
                        Text: =ThisItem.Title
                        Default: =ThisItem.Fait
                        OnCheck: |-
                          =Patch('[LME] Etapes', LookUp('[LME] Etapes', ID = ThisItem.ID), {Fait: true});
                          If(varSel.Statut.Value = "À faire",
                            Patch('[LME] Workload', LookUp('[LME] Workload', ID = varSel.ID), {Statut: {Value: "En cours"}}));
                          Select(btnRefresh)
                        OnUncheck: |-
                          =Patch('[LME] Etapes', LookUp('[LME] Etapes', ID = ThisItem.ID), {Fait: false});
                          Select(btnRefresh)
                        Size: =12
                        Font: =Font.'Segoe UI'
            - txtEtape:
                Control: Classic/TextInput@2.3.2
                Properties:
                  X: =20
                  Y: =Parent.Height - 232
                  Width: =290
                  Height: =36
                  HintText: ="Ajouter une étape"
                  Visible: =varSel.Statut.Value <> "Terminé"
                  Size: =12
                  Font: =Font.'Segoe UI'
                  Default: =""
            - btnAjouterEtape:
                Control: Classic/Button@2.2.0
                Properties:
                  X: =320
                  Y: =Parent.Height - 232
                  Width: =80
                  Height: =36
                  Text: ="Ajouter"
                  Visible: =varSel.Statut.Value <> "Terminé"
                  DisplayMode: =If(IsBlank(txtEtape.Text), DisplayMode.Disabled, DisplayMode.Edit)
                  OnSelect: |-
                    =Patch('[LME] Etapes', Defaults('[LME] Etapes'),
                      {
                        Title: txtEtape.Text,
                        Tache: {Id: varSel.ID, Value: varSel.Title},
                        Fait: false,
                        Ordre: CountRows(Filter(colEtapes, Tache.Id = varSel.ID)) + 1
                      });
                    Reset(txtEtape);
                    Select(btnRefresh)
                  Size: =12
                  Font: =Font.'Segoe UI'
            - btnStAFaire:
                Control: Classic/Button@2.2.0
                Properties:
                  X: =20
                  Y: =Parent.Height - 188
                  Width: =120
                  Height: =36
                  Text: ="À faire"
                  Fill: =If(varSel.Statut.Value = "À faire", RGBA(31, 79, 216, 1), RGBA(255, 255, 255, 1))
                  Color: =If(varSel.Statut.Value = "À faire", RGBA(255, 255, 255, 1), RGBA(16, 26, 46, 1))
                  Visible: =varSel.Statut.Value <> "Terminé"
                  OnSelect: |-
                    =Patch('[LME] Workload', LookUp('[LME] Workload', ID = varSel.ID), {Statut: {Value: "À faire"}});
                    Select(btnRefresh)
                  Size: =12
                  Font: =Font.'Segoe UI'
            - btnStEnCours:
                Control: Classic/Button@2.2.0
                Properties:
                  X: =148
                  Y: =Parent.Height - 188
                  Width: =120
                  Height: =36
                  Text: ="En cours"
                  Fill: =If(varSel.Statut.Value = "En cours", RGBA(31, 79, 216, 1), RGBA(255, 255, 255, 1))
                  Color: =If(varSel.Statut.Value = "En cours", RGBA(255, 255, 255, 1), RGBA(16, 26, 46, 1))
                  Visible: =varSel.Statut.Value <> "Terminé"
                  OnSelect: |-
                    =Patch('[LME] Workload', LookUp('[LME] Workload', ID = varSel.ID), {Statut: {Value: "En cours"}});
                    Select(btnRefresh)
                  Size: =12
                  Font: =Font.'Segoe UI'
            - btnStBloque:
                Control: Classic/Button@2.2.0
                Properties:
                  X: =276
                  Y: =Parent.Height - 188
                  Width: =120
                  Height: =36
                  Text: ="Bloqué"
                  Fill: =If(varSel.Statut.Value = "Bloqué", RGBA(31, 79, 216, 1), RGBA(255, 255, 255, 1))
                  Color: =If(varSel.Statut.Value = "Bloqué", RGBA(255, 255, 255, 1), RGBA(16, 26, 46, 1))
                  Visible: =varSel.Statut.Value <> "Terminé"
                  OnSelect: |-
                    =Patch('[LME] Workload', LookUp('[LME] Workload', ID = varSel.ID), {Statut: {Value: "Bloqué"}});
                    Select(btnRefresh)
                  Size: =12
                  Font: =Font.'Segoe UI'
            - lblSync:
                Control: Label@2.5.1
                Properties:
                  X: =20
                  Y: =Parent.Height - 144
                  Width: =380
                  Height: =50
                  Text: |-
                    =If(varSel.Statut.Value = "Terminé", "Clôture propagée par le flux.",
                      Switch(varSel.Source.Value,
                        "Mail", "Terminer clôt la liste, le flag Outlook et To Do.",
                        "Teams", "Terminer clôt la liste et la tâche To Do.",
                        "Loop", "Terminer clôt la liste. À cocher aussi dans Loop si non synchronisé.",
                        "Terminer clôt la liste."))
                  Size: =11
                  Color: =RGBA(100, 114, 138, 1)
                  Font: =Font.'Segoe UI'
            - btnTerminer:
                Control: Classic/Button@2.2.0
                Properties:
                  X: =20
                  Y: =Parent.Height - 84
                  Width: =380
                  Height: =48
                  Text: ="Terminer partout"
                  Fill: =RGBA(31, 79, 216, 1)
                  Color: =RGBA(255, 255, 255, 1)
                  Visible: =varSel.Statut.Value <> "Terminé"
                  OnSelect: |-
                    =Patch('[LME] Workload', LookUp('[LME] Workload', ID = varSel.ID),
                      {Statut: {Value: "Terminé"}, TermineLe: Now()});
                    Select(btnRefresh);
                    Notify("Clôture envoyée à Outlook, To Do et Loop par le flux.", NotificationType.Success)
                  Size: =12
                  Font: =Font.'Segoe UI'
            - btnRouvrir:
                Control: Classic/Button@2.2.0
                Properties:
                  X: =20
                  Y: =Parent.Height - 84
                  Width: =380
                  Height: =48
                  Text: ="Rouvrir"
                  Fill: =RGBA(255, 255, 255, 1)
                  Color: =RGBA(16, 26, 46, 1)
                  Visible: =varSel.Statut.Value = "Terminé"
                  OnSelect: |-
                    =Patch('[LME] Workload', LookUp('[LME] Workload', ID = varSel.ID),
                      {Statut: {Value: "En cours"}, TermineLe: Blank(), Propage: false});
                    Select(btnRefresh)
                  Size: =12
                  Font: =Font.'Segoe UI'
```

## Après chaque collage : exécuter `OnVisible`

Coller le code ne relance pas l'`OnVisible` de l'écran : les variables (`varVue`, `varAn`, `varMois`) et les collections restent vides dans la session de Studio en cours. Dans l'arborescence, clic droit sur `scrMain` → **Exécuter OnVisible**, ou relance l'application avec F5 depuis l'écran. Sans cela, le calendrier affichait l'erreur « The year argument to the Date function must be between 0 and 9999 ». Il est maintenant protégé par des `Coalesce`, mais les chiffres ne se chargent qu'après l'`OnVisible` ou un clic sur « Actualiser ».

Pour vérifier que les données sont chargées : menu **Affichage → Collections**, puis ouvre `colPct` (une ligne par tâche).

## Si les écrans sont vides

L'application affiche uniquement ce qui est dans tes listes. Si `[LME] Workload` contient peu de lignes, ou si `[LME] Etapes` est vide, les cartes sont vides ou à 0 %, ce qui est normal. Ajoute quelques lignes de test (voir [02](02-listes-sharepoint.md), section « Jeu d'essai »).

## Si les chiffres restent à zéro

1. Au lancement (F5) ou après un clic sur « Actualiser », un ou plusieurs **bandeaux rouges** apparaissent avec le nom de l'étape qui échoue, par exemple `ClearCollect(colPct) : ...`. Le message dit en général quelle colonne ou quelle source de données pose problème.
2. Dans Studio, sélectionne `btnRefresh`, propriété `OnSelect` : les erreurs de formule sont soulignées en rouge, et le **Vérificateur d'application** (icône en haut à droite) les liste toutes.
3. Si le clic sur « Actualiser » ne produit ni bandeau ni donnée, c'est qu'une erreur de syntaxe empêche toute la formule de s'exécuter : cherche le soulignement rouge.
4. Envoie-moi le texte des bandeaux ou des erreurs, ou une nouvelle trace faite après le clic.

## Propriétés ajoutées pour l'interface

Ces propriétés n'étaient pas dans la version précédente. Si Studio en refuse une (message « Unknown property »), supprime simplement la ligne : le reste fonctionne.

- `Font: =Font.'Segoe UI'` sur les labels, boutons, champs, listes déroulantes et cases à cocher.
- `Default: =""` sur les champs de saisie : sans elle, un champ affiche « Text input » au lieu de son texte d'aide.
- `Align: =Align.Left` sur les boutons du menu.

## Points à surveiller

1. **Colonnes de choix** : les formules utilisent `Statut.Value`, `TypeTache.Value`, `Source.Value`, `StatutProjet.Value`. Si tes colonnes sont de type texte et non choix, retire `.Value` partout (rechercher-remplacer `.Value`).
2. **Colonnes de recherche** : `Projet.Id`, `Tache.Id`, `Projet.Value` supposent des colonnes de type recherche (lookup). Si Power Apps affiche une erreur, regarde la forme réelle avec une galerie temporaire sur `Workload`.
3. **Liste déroulante des projets** : `ddTrierProjet` affiche une colonne nommée `Value`, fabriquée par `ForAll` à partir de `Projets` (la propriété `Value` n'existe pas sur ce contrôle).
4. **Calendrier** : la galerie `galJours` utilise `WrapCount: 7`. Si ta version ne la propose pas, remplace-la par une galerie horizontale de 7 colonnes sur 6 lignes, ou garde la vue « À venir » et supprime la grille.
5. **Images SVG** : `imgGauge`, `imgCourbe` et `imgDonut` construisent une image par formule. Les nombres sont arrondis en entiers pour éviter la virgule décimale des paramètres régionaux français. Si une image reste vide, vérifie que le contrôle est de type *Image* (et non Image moderne).
6. **Nombre de projets** : le graphique en anneau prévoit 4 couleurs. Au-delà, les projets suivants prennent la même teinte grise.
7. **Délégation** : un triangle jaune sur `in` (recherche) ou sur certains `Filter` est attendu. Tant que tes listes restent sous 2000 lignes, le résultat est correct.
8. **Performance** : le calendrier évalue des filtres sur 42 cases. Si l'app ralentit, remplace-les par une collection `colEcheances` calculée dans `btnRefresh`.

## Fonctionnement

| Élément | Rôle |
|---|---|
| `btnRefresh` | Recharge les listes et recalcule toutes les collections : `colEtapes`, `colPct` (avancement par tâche), `colTypes`, `colProjCount`, `colDonut`, `colSem`. Il est appelé par `Select(btnRefresh)` après chaque action. Sa formule est aussi copiée dans l'`OnVisible` de l'écran, pour ne pas dépendre d'un `Select` lancé pendant le chargement. Si tu modifies le chargement, modifie les deux. Chaque étape est enveloppée dans `IfError` : en cas d'échec, l'application affiche un bandeau rouge avec le nom de l'étape et le message d'erreur. |
| `cntSidebar`, `cntTopbar`, `cntDrawer` | Barre latérale, barre du haut, volet de détail (toujours présents ou liés à `varSel`) |
| `varVue` | Vue active : `dash`, `tasks`, `cal`, `stats`, `trier` |
| `varSel` | Tâche affichée dans le volet de détail |
| `varNext` | Prochaine échéance du tableau de bord |
| `varType` | Filtre par type dans la vue Tâches |
| `varAn` / `varMois` | Mois affiché dans le calendrier |

Les clôtures passent toutes par une mise à jour de `Statut` et `TermineLe` : le flux de [05](05-flux-cloture.md) fait le reste (étapes, flag Outlook, To Do, Loop). Les étapes se cochent et se décochent dans le volet de détail. L'app ne touche jamais directement à Outlook.

## Si le collage échoue

1. **Lis le message d'erreur.** Il cite le contrôle ou la ligne en cause.
2. **Compare avec un contrôle réel.** Insère un contrôle du même type depuis le menu Insérer, puis clic droit → **Afficher le code**. Compare la ligne `Control:` (type et numéro de version, par exemple `Gallery@2.15.0`), la ligne `Variant:` et l'indentation, puis recopie la bonne valeur dans le fichier.
3. **Corrige par rechercher-remplacer** dans un éditeur de texte (par exemple un nom de type), puis recolle.
4. **Colle par morceaux.** Colle un bloc à la fois (la barre latérale, puis le tableau de bord, etc.) dans un écran vide : retire les lignes `Screens:` et `scrMain:` et recule la liste des contrôles pour qu'elle commence en première colonne.
5. **Dernier recours.** Construis les contrôles à la main à partir de la maquette et ne colle que les formules : elles sont indépendantes du format de collage.
