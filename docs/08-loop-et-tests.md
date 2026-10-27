# 8. Loop et plan de recette

## Pourquoi Loop est à tester

Dans Loop, une tâche assignée à toi apparaît dans Microsoft To Do (liste « Assignées à moi ») et dans Planner, et la compléter à un endroit la complète à l'autre. Ce comportement est natif. Ce qui n'est pas garanti, c'est que les connecteurs Power Automate exposent ces tâches. Il faut le tester.

## Tests Loop (environ 20 minutes)

**T1. Visibilité native.** Dans la page Monthly eng sous Loop, crée une tâche et assigne-la à toi. Vérifie qu'elle apparaît dans To Do sous « Assignées à moi ». Coche-la dans To Do et vérifie qu'elle se coche dans Loop.

**T2. Visibilité Power Automate.** Crée un flux de test :
- *Microsoft To Do (Business) : Lister les listes de tâches (V2)* : vois si « Assignées à moi » apparaît dans le résultat.
- Sinon, *Planner : Lister mes tâches* (ou l'action équivalente) : vois si la tâche Loop y figure.
- Teste un trigger *Quand une tâche est créée* ou *terminée* sur la liste qui la contient.

**T3. Flag Outlook.** Vérifie ce que le connecteur Outlook expose sur l'état de l'indicateur : l'action *Obtenir un e-mail (V2)* retourne-t-elle un champ d'état de flag ? La liste « E-mails signalés » existe-t-elle dans le connecteur To Do ? Le résultat décide de l'option A, B ou C dans [05](05-flux-cloture.md).

## Décision selon T2

| Résultat de T2 | Ce que tu construis |
|---|---|
| La tâche Loop est lisible et mise à jour par un connecteur | Un flux déclenché par la création de tâche Loop crée l'item (type Monthly eng, source Loop). La clôture dans les deux sens est automatique. |
| Elle est lisible mais pas modifiable | Entrée automatique, mais clôture dans Loop à la main. L'app affiche « à cocher dans Loop » pour ces items. |
| Elle n'est pas exposée | Entrée manuelle : un bouton « Nouvelle tâche Monthly eng » dans l'app crée l'item (type Monthly eng, étapes du modèle). Tu coches la tâche dans Loop toi-même. |

Dans tous les cas, la clôture native Loop ↔ To Do reste active. Seule la synchronisation avec la liste Workload peut manquer.

## Plan de recette global

Dans cet ordre :

1. **Listes** ([02](02-listes-sharepoint.md)) : ligne de test créée, unicité de SourceId vérifiée.
2. **Flux mail** ([03](03-flux-mail-flagge.md)) : six cas du plan de test.
3. **Flux étapes** ([04](04-flux-etapes.md)) : étapes créées une seule fois.
4. **Flux clôture** ([05](05-flux-cloture.md)) : sens sortant d'abord, puis To Do.
5. **Flux Teams** ([06](06-flux-teams.md)).
6. **Power App** ([07](07-power-app.md)) : compteurs, avancement, clôture.
7. **Bout en bout** : flagge un mail Support, vérifie l'item et ses étapes, coche deux étapes, termine la tâche dans l'app, vérifie que le flag Outlook, la tâche To Do et l'item sont tous terminés.

## Points d'attention en production

- Surveille la première semaine l'historique des exécutions de chaque flux (échecs, boucles, doublons).
- Garde un œil sur les mails déplacés hors de la boîte de réception : leur identifiant peut changer et empêcher la clôture du flag.
- Si ton compte change de rôle ou de département, les connexions des flux doivent être renouvelées.
- Exporte régulièrement les flux (*Exporter* → package) pour pouvoir les restaurer.
