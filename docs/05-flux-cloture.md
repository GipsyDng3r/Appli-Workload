# 5. Terminer partout

Un seul flux fait la propagation sortante, quelle que soit l'origine de la clôture : il réagit au Statut de l'item Workload. Les clôtures venant d'autres outils se contentent de passer le Statut à Terminé.

## Flux « Workload - Clôture » (sortant)

Type : **Flux cloud automatisé**.

### Trigger

**SharePoint : Lorsqu'un élément est créé ou modifié**, liste Workload.

Conditions de déclenchement :

```
@equals(triggerOutputs()?['body/Statut/Value'],'Terminé')
@not(equals(triggerOutputs()?['body/Propage'],true))
```

Contrôle de la concurrence : activé, parallélisme = 1.

### Actions

**1. Cocher les étapes**
- *Obtenir les éléments*, liste Etapes, filtre `TacheId eq @{triggerOutputs()?['body/ID']} and Fait eq 0`.
- *Appliquer à chacun* : *Mettre à jour un élément* → `Fait = Oui`.

**2. Terminer le flag Outlook (si Source = Mail)**
- **Condition** : `Source/Value` est égal à `Mail` **et** `MessageId` n'est pas vide.
- Oui : **Office 365 Outlook : Signaler un e-mail (V2)** — ID de message = `MessageId`, État de l'indicateur = **Terminé**.
- Configure l'action « Exécuter après » d'une étape suivante pour continuer même si le signalement échoue (échec, expiration). Un mail déplacé dans un autre dossier peut changer d'identifiant : le flag n'est alors pas mis à jour. Dans ce cas, envoie-toi un message Teams du bot Flow indiquant le titre de la tâche.

Rappel : terminer le flag Outlook termine aussi la tâche To Do correspondante (synchronisation native Microsoft).

**3. Terminer le miroir To Do (Teams et manuel)**
- **Condition** : `ToDoTaskId` n'est pas vide.
- Oui : **Microsoft To Do (Business) : Mettre à jour une tâche (V3)** — liste = « Workload », ID de tâche = `ToDoTaskId`, statut = Terminé.

**4. Terminer la tâche Loop (si Source = Loop)**
- Dépend du test de [08](08-loop-et-tests.md). Si le connecteur expose les tâches assignées, mettre à jour la tâche avec statut Terminé ; sinon, cette étape est omise et tu coches la tâche dans Loop.

**5. Marquer comme propagée**
- *Mettre à jour un élément*, liste Workload : `Propage = Oui`, `TermineLe = utcNow()` si vide.

L'étape 5 est l'anti-boucle : elle relance le trigger, qui s'arrête à la condition `Propage = false`.

### Rouvrir une tâche

Non gérée en v1. Pour rouvrir, remets Statut à En cours, `Propage` à Non et `TermineLe` à vide, puis re-signale le mail dans Outlook si besoin. Un flux de réouverture symétrique est une évolution possible.

## Clôtures entrantes (depuis un autre outil)

### Depuis To Do (tâches Teams et manuelles) — fiable

Flux **« Workload - Clôture To Do »**
- Trigger : **Microsoft To Do (Business) : Quand une tâche est terminée**, liste « Workload ». **À vérifier** : le libellé exact varie selon la version du connecteur.
- Action : *Obtenir les éléments* dans Workload, filtre `ToDoTaskId eq '<id de la tâche>'`, puis *Mettre à jour un élément* → `Statut = Terminé`.
- Le flux sortant fait le reste (étapes, Propage).

### Depuis Outlook (flag terminé) — à tester

Il n'existe pas de trigger « flag terminé » dans le connecteur Outlook. Trois options, à choisir selon les tests T3 de [08](08-loop-et-tests.md) :

- **Option A.** Si le connecteur To Do expose la liste « E-mails signalés », un trigger *tâche terminée* sur cette liste. Le rapprochement se fait sur le titre de la tâche (objet du mail), donc il est fragile si deux mails ont le même objet.
- **Option B.** Un flux planifié toutes les 30 minutes : pour chaque item ouvert de Source Mail, lire l'e-mail (*Obtenir un e-mail (V2)*) et tester l'état de son indicateur si le connecteur l'expose. **À vérifier.**
- **Option C.** Ne pas synchroniser ce sens. Tu termines depuis l'app Workload (qui termine le flag), et si tu termines d'abord dans Outlook, tu coches aussi dans l'app. C'est le repli sans risque.

### Depuis Loop

Voir [08](08-loop-et-tests.md).

## Test

1. Termine un item de Source Mail dans l'app : ses étapes se cochent, le flag Outlook devient terminé, la tâche To Do correspondante aussi.
2. Termine un item de Source Teams : la tâche To Do du miroir se termine.
3. Termine la tâche To Do d'un item Teams : l'item Workload passe à Terminé et ses étapes se cochent.
4. Vérifie dans l'historique du flux que chaque exécution n'a lieu qu'une fois par item.
