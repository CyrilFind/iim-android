# TD 5: ViewModel

On va faire un peu de ménage puis afficher et uploader des images

⚠️ **Attention** ⚠️: Erreur dans le td précédent, vérifiez la version dans `app/build.gradle`:

```groovy   
implementation "androidx.lifecycle:lifecycle-viewmodel-ktx:2.2.0-rc03"
```

## Refactorisation avec TasksViewModel

Mettre toute la logique dans le fragment est une mauvaise pratique: les `ViewModel` permettent d'en extraire une partie.

- Créer une classe `TasksViewModel` qui hérite de `ViewModel` qui va gérer:  
    - La liste des `tasks` sous forme de `LiveData`
    - Le `repository`
    - Les coroutines avec `viewModelScope`

- Dans `TasksFragment`:
    - Récupérer le `viewModel` grâce à `ViewModelProvider`
    - Supprimer le `repository` et la list de `tasks`
    - Observer la valeur de `viewModel.tasks` et mettre à jour la liste de l'`adapter`
- Dans `TasksRepository`, 
    - Supprimer les fonctions qui utilisent `coroutineScope`
    - Garder seulement celles qui sont `suspend` et retirer `private`

- Procéder par étapes et inspirez vous de ce squelette (NE COPIEZ PAS TOUT!) pour refactoriser votre app (commencez juste par le chargement de la liste):

```kotlin
// Repository simplifié, avec seulement des méthodes "suspend"
class TasksRepository {
  private val service = Api.tasksService
    
  suspend fun loadTasks(): List<Task>? {
    val response = service.getTasks()
    return if (response.isSuccessful) response.body() else null
  }
}

// Le ViewModel met à jour la liste de task qui est une LiveData 
class TasksViewModel: ViewModel() {
  private val tasks = MutableLiveData<List<Task>?>()
  private val repository = Api.tasksRepository
  
  fun loadTasks() { 
        viewModelScope.launch { 
            tasks.postValue(repository.loadTasks())
        }
    }
}

// Le Fragment observe la LiveData et met à jour la liste de l'adapter:
class TasksFragment: Fragment() {
  val tasksAdapter = TaskApdapter()
  private val viewModel by lazy {
    ViewModelProvider(this).get(TasksViewModel::class.java)
  }

  override fun onViewCreated(...) {
    viewModel.tasks.observe(this, Observer { newList -> 
        adapter.list = newList
    })
  }

  override fun onResume(...) {
    viewModel.loadTasks()
  }
}

// L'adapter se notifie automatiquement à chaque fois qu'on modifie sa liste:
class TaskAdapter() : ... {
    var list: List<Task> by Delegates.observable(emptyList()) {
        _, _, _ -> notifyDataSetChanged()
    }
}

```

- Vérifier que ça fonctionne
- Permettre la suppression, l'ajout et l'édition des tasks du serveur avec cette archi.
