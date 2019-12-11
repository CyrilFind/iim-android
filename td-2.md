# TD 2 - RecyclerView

L'objectif de ce TD est d'implementer un écran affichant une liste de tâches, de permettre de créer des nouvelles tâches, de les supprimer et de les partager dans une autre application.

## Dependances RecyclerView
Dans le `build.gradle`, ajouter :

```groovy
implementation "androidx.recyclerview:recyclerview:1.1.0"
```

## TasksFragment
- Créez `TasksFragment` qui va afficher la liste des tâches:

```kotlin
class TasksFragment : Fragment() {}
```
- Créer le layout associé `fragment_tasks.xml`
- Dans `TasksFragment`, overrider (surcharger) la méthode `onCreateView(...)` (commencez à taper ce nom de méthode et utilisez l'auto-completion de l'IDE pour vous aider) pour initialiser la `view` à l'aide de ce layout (c'est similaire au `onCreate` d'une Activity sauf qu'on doit retourner la `View` créée):

```kotlin
inflater.inflate(R.layout.fragment_tasks, container, false)
```
- Ajouter une balise `<fragment...>` à votre activité principale
- Utilisez `android:name` pour specifier la classe de votre Fragment (ex: `"com.cyrilfind.todo.TasksFragment"`)

## La liste des tâches

- Pour commencer, la liste des tâches sera simplement un tableau de `String`:

```kotlin
private val tasks = listOf("Task 1", "Task 2", "Task 3")
```

- Dans le layout associé à `TasksFragment`, placez une balise `<androidx.recyclerview.widget.RecyclerView...>`:

- Créer une nouvelle classe `TasksAdapter`

```kotlin
class TasksAdapter(private val tasks: List<String>) : RecyclerView.Adapter<TaskViewHolder>() {}
```

- À l'intérieur de `TasksAdapter`, créer la classe `TaskViewHolder`:

```kotlin
inner class TaskViewHolder(itemView: View) : RecyclerView.ViewHolder(itemView) {
	fun bind(taskTitle: String) {
	   // C'est ici qu'on reliera les données et les listeners une fois l'adapteur implémenté
	}
}
```

- Dans `TasksFragment`, overrider `onViewCreated` et y récupérer la `RecyclerView` du layout pour lui donner avec un `layoutManager` et un `adapter` (pour l'instant votre `TasksAdapter` ne va pas marcher)

**Rappel**: l'Adapteur recycle les cellules (`ViewHolder`) en y insérant les données des tâches visibles lorsqu'on scroll


- Créer le layout `item_task.xml` correspondant à une cellule (`TaskViewHolder`)

```xml
<LinearLayout 
  xmlns:android="http://schemas.android.com/apk/res/android"
  android:orientation="horizontal" 
  android:layout_width="match_parent"
  android:layout_height="wrap_content">

  <TextView
      android:id="@+id/task_title"
      android:background="@android:color/holo_blue_bright"
      android:layout_width="match_parent"
      android:layout_height="wrap_content" />
</LinearLayout>
```

## Implémentation du RecyclerViewAdapter

Dans le `TasksAdapter`, implémenter toutes les méthodes requises:

**Astuce**: Utilisez l'IDE pour faciliter l'implémentation des méthodes en cliquant sur le nom de votre classe (qui doit être soulignée en rouge) et cliquez sur l'ampoule jaune ou tapez `Alt` + `ENTER` (sinon, `CTRL` + `O` n'importe où dans la classe)

- `getItemCount` qui renvoie la taille de la liste de tâche à afficher
- `onCreateViewHolder` qui crée un nouveau `TaskViewHolder` en utilisant le layout `item_task.xml`: 

```kotlin
val itemView = LayoutInflater.from(parent.context).inflate(R.layout.item_task, parent, false)
val viewHolder = TaskViewHolder(itemView)
```

- `onBindViewHolder` qui insère la donnée dans la cellule (`TaskViewHolder`) en fonction de la position dans la liste.

Votre code doit compiler maintenant et vous devez voir 3 tâches

## Ajout de la data class Task

- Dans un nouveau fichier, créer la `data class` `Task` avec 3 attributs: un id, un titre et une description. 
- Ajouter une valeur par défaut à la description.
- Dans le `TasksFragment`, remplacer la liste `tasks` par

 ```kotlin       
private val tasks = listOf(
	Task(id = "id_1", title = "Task 1", description = "description 1"), 
	Task(id = "id_2", title = "Task 2"), 
	Task(id = "id_3", title = "Task 3")
)
```

- Corriger votre code en conséquence afin qu'il compile de nouveau
- Enfin afficher la description en dessous du titre
- Admirez avec fierté le travail accompli 🤩

## Suppression d'une tache

Dans le layout de votre ViewHolder, ajouter un bouton afin de pouvoir supprimer la tâche associée. Vous pouvez utiliser par exemple l'icone `@android:drawable/ic_menu_delete`

- Transformer votre liste de taches `tasks` en `mutableListOf(...)` afin de pouvoir la modifier 
- Dans l'adapteur, ajouter une lambda `onDeleteClickListener` qui prends en arguments une `Task` et ne renvoie rien: `(Task) -> Unit`
- Relier cette callback au `onClickListener` de l'image que vous avez ajoutée précédemment
- Dans le fragment, implementer le `onDeleteClickListener`, il doit supprimer la tache passée en argument de la liste **et notifier l'adapteur**.

## Création d'une nouvelle tache

- Ajouter un Floating Action Button (FAB) dans le layout de l'activité principale
- Dans le `onClickListener` du FAB, ajoutez la possibilité de créer une `Task` rapidement: 

```kotlin
Task(id = task.count, title = "task #${task.count}")
```


#### Étape Bonus: Changements de configuration

Que se passe-t-il si vous tournez votre téléphone ? 🤔

Pour régler ce problème, implémentez les méthodes suivantes:

```kotlin
override fun onSaveInstanceState(outState: Bundle)
override fun onActivityCreated(savedInstanceState: Bundle?)
```

Il faut également ajouter l'annotation `@Parcelize` à la classe `Task` afin qu'elle implémente`Parceleable` automatiquement 

Pour cela il faut d'abord ajouter la dépendance suivante:

```groovy
implementation "org.jetbrains.kotlinx:kotlinx-serialization-runtime:0.9.1"
```

## Ajout Complet

- Créer la nouvelle `TaskActivity`, n'oubliez pas de la déclarer dans le manifest
- Créer un layout contenant 2 `EditText`, pour le titre et la description et un bouton pour valider
- Changer l'action du FAB pour qu'il ouvre cette activité avec un `Intent`:

```kotlin
val intent = Intent(this, TaskActivity::class.java)
startActivity(intent)
```
- Lorsqu'on valide le formulaire, cela ajoute dans une nouvelle tache dans la liste et ferme l'activity
- N'oubliez pas de donner un `id` à votre tache avant de l'ajouter !
- Lorsqu'on clique sur le bouton "Back", la `TaskActivity` doit se fermer: ajouter une popup de confirmation si l'utilisateur a commencer à taper des informations
- Faites en sorte que la nouvelle tache s'affiche à notre retour sur l'activité principale.

## Édition d'une tache

- Ajouter une bouton permettant d'éditer
- Au lieu de supprimer la tache, ouvrir l'activité `TaskActivity` pré-remplie avec les informations de la tâche.
- Pour transmettre des infos d'une activité à l'autre, vous pouvez utiliser la méthode `putExtra` depuis une instance d'`Intent`
- Vous pouvez ensuite récuperer dans le onCreate de l'activité les infos que vous avez passées
- Vérifier que les infos éditées s'affichent bien à notre retour sur l'activité principale.

## Partager

- Ajouter la possibilité de partager du texte **depuis** les autres applications et ouvrir le formulaire de création de tâche pré-rempli ([Documentation][1])
- Ajouter la possibilité de partager du texte **vers** les autres applications avec un `OnLongClickListener` sur les tâches ([Documentation][2])

[1]: https://developer.android.com/training/sharing/receive

[2]: https://developer.android.com/training/sharing/send
