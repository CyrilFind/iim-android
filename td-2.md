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
- `onCreateViewHolder` qui returne un nouveau `TaskViewHolder`: vous aurez besoin d'un `itemView`, généré à partir du layout `item_task.xml`: 

```kotlin
val itemView = LayoutInflater.from(parent.context).inflate(R.layout.item_task, parent, false)
```

- `onBindViewHolder` qui insère la donnée dans la cellule (`TaskViewHolder`) en fonction de la position dans la liste.

- Lancez l'app: vous devez voir 3 tâches s'afficher 👏

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


## Ajout de tâche simple

- Changez la root view de `fragment_tasks.xml` en ConstraintLayout en faisant un clic droit dessus en mode design (si ce n'est pas déjà le cas)
- Ouvrez le volet "Resource Manager" à gauche, cliquez sur le "+" en haut à gauche puis "Vector Drawable" puis double cliquez sur le clipart du logo android et selectionnez une icone + (en cherchant "add" dans la barre de recherche) puis "finish" pour ajouter une icone à vos resource
- Par défaut l'icône est noire mais vous pouvez utiliser l'attribut `android:tint` du bouton pour la rendre blanche (tapez "white" et laissez l'IDE compléter)
- Ajouter un Floating Action Button (FAB) en bas à droite de ce layout et utilisez l'icone créée 
- Donnez des contraintes en bas et à droite de ce bouton
- Transformer votre liste de taches `tasks` en `mutableListOf(...)` afin de pouvoir la modifier 
- Utilisez `.setOnClickListener {}` sur le FAB pour ajouter une tâche à votre liste:

```kotlin
// Instanciation d'un objet task avec des données préremplies:
Task(id = "id_#${tasks.size + 1}", title = "task #${tasks.size + 1}")
```

- Dans cette callback, **notifier l'adapteur** (aidez vous des suggestions de l'IDE) pour que votre modification s'affiche


## Suppression d'une tache

Dans le layout de votre ViewHolder, ajouter un `ImageButton` qui servira à supprimer la tâche associée. Vous pouvez utiliser par exemple l'icone `@android:drawable/ic_menu_delete`

- Dans l'adapteur, ajouter une lambda `onDeleteClickListener` qui prends en arguments une `Task` et ne renvoie rien: `(Task) -> Unit`

```kotlin
// Déclaration d'une lambda comme variable:
var onDeleteClickListener: (Task) -> Unit = { task -> /* faire qqchose */ }

// Utilisation d'une lambda:
onDeleteClickListener.invoke(task)
```

- Utilisez cette lambda avec dans le `onClickListener` du bouton supprimer
- Dans le fragment, accéder au `onDeleteClickListener` depuis l'adapter et implémentez là: donnez lui comme valeur une lambda qui va supprimer la tache passée en argument de la liste 


## Ajout de tâche complet

- Créer la nouvelle `TaskActivity`, n'oubliez pas de la déclarer dans le manifest
- Créer un layout contenant 2 `EditText`, pour le titre et la description et un bouton pour valider
- Définir un `ADD_TASK_REQUEST_CODE` et changer l'action du FAB pour qu'il ouvre cette activité avec un `Intent`, en attendant un resultat:
 
```kotlin
val intent = Intent(this, TaskActivity::class.java)
startActivityForResult(intent, ADD_TASK_REQUEST_CODE)
```

- Dans le `onCreate` de la nouvelle activité, récupérer le bouton de validation puis setter son `onClickListener` pour qu'il crée une tâche:

```kotlin
Task(id = UUID.randomUUID().toString(), title = "New Task !")
```

- Faire en sorte que la `data class Task` hérite de `Serializable` pour pouvoir passer des objets `Task` dans les `intent`
- Passer cette task dans l'intent avec `putExtra(...)`
- Overrider `onActivityResult`dans le `TaskFragment` pour récupérer cette task et l'ajouter à la liste

```kotlin
val task = data!!.getSerializableExtra(TaskActivity.TASK_KEY) as Task 
```

- Faites en sorte que la nouvelle tache s'affiche à notre retour sur l'activité principale
- Maintenant, récupérez les valeurs entrées dans les `EditText` pour les donner à la création de votre tâche (vous devrez faire un `toString()`)

## Édition d'une tâche

- Ajouter une bouton permettant d'éditer en ouvrant l'activité `TaskActivity` pré-remplie avec les informations de la tâche
- Pour transmettre des infos d'une activité à l'autre, vous pouvez utiliser la méthode `putExtra` depuis une instance d'`intent`
- Inspirez vous de l'implémentation du bouton supprimer et du bouton ajouter
- Vous pouvez ensuite récuperer dans le `onCreate` de l'activité les infos que vous avez passées
- Vérifier que les infos éditées s'affichent bien à notre retour sur l'activité principale.

## Partager

- Ajouter la possibilité de partager du texte **depuis** les autres applications et ouvrir le formulaire de création de tâche pré-rempli ([Documentation][1])
- Ajouter la possibilité de partager du texte **vers** les autres applications avec un `OnLongClickListener` sur les tâches ([Documentation][2])


## Changements de configuration

Que se passe-t-il si vous tournez votre téléphone ? 🤔

Pour sauvegarder votre liste de task, implémentez la méthodes suivante:

```kotlin
override fun onSaveInstanceState(outState: Bundle)
```

Puis, pour récupérer cette list, utilisez l'argument `savedInstanceState` de `onCreateView`


[1]: https://developer.android.com/training/sharing/receive

[2]: https://developer.android.com/training/sharing/send
