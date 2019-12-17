# TD 4: L'Internet

## Avant de commencer

Les APIs qui nous allons utiliser exigent qu'une personne soit connectée, dans ce TD nous allons simuler que la personne est connectée, en passant un `token` dans les `headers` de nos requêtes HTTP.

- Nous allons utiliser ce site: https://android-tasks-api.herokuapp.com/api-docs/index.html
- Lisez rapidement la documentation de l'API: elle permet d'utiliser ses routes directement
- Cliquez sur `users/sign_up` puis sur "Try it out"
- Vous devriez voir un JSON prérempli dont vous devez remplir les données (vous pouvez mettre des infos bidon) avant de cliquer sur "Execute":

```json
{
    "firstname": "UN PRENOM",
    "lastname": "UN NOM",
    "email": "UN EMAIL",
    "password": "UN MDP",
    "password_confirmation": "LE MEME MDP"
}
```

- Copiez le token généré (vous pourrez le récuperer à nouveau en vous re-loggant)

## Accèder à l'internet

Afin de communiquer avec le réseau internet (wifi, ethernet ou mobile), il faut ajouter la permission dans le fichier `AndroidManifest`, juste après la balise `<manifest...>`

```xml
<uses-permission android:name="android.permission.INTERNET" />
```

## Ajout des dépendances

Dans le fichier `app/build.gradle`, ajouter : 

```groovy
  implementation "com.squareup.retrofit2:retrofit:2.6.2"
  implementation 'com.squareup.retrofit2:converter-moshi:2.6.2'
  implementation "com.squareup.moshi:moshi:1.8.0"
  implementation "com.squareup.moshi:moshi-kotlin:1.8.0"
  implementation "androidx.lifecycle:lifecycle-viewmodel-ktx:2.1.0"
  implementation "org.jetbrains.kotlinx:kotlinx-coroutines-core:1.3.2"
  implementation "androidx.lifecycle:lifecycle-runtime-ktx:2.2.0-alpha01"
  implementation "org.jetbrains.kotlin:kotlin-reflect:1.1.0"
```

## Retrofit

- Vous pouvez créer un package `network` qui contiendra les classes en rapport avec les échanges réseaux
- Créer un `object` `Api` (ses membres et méthodes seront donc `static`)
- Ajoutez y les constantes qui serviront à faire les requêtes:

```kotlin
object Api {
  private const val BASE_URL = "https://android-tasks-api.herokuapp.com/api/"
  private const val TOKEN = "COPIEZ VOTRE TOKEN ICI !"
}
```

- Créer une instance de [Moshi](https://github.com/square/moshi) pour parser le JSON renvoyé par le serveur:

```kotlin
object Api {
  // ...
  private val moshi = Moshi.Builder().build()
}
```

- Créer le client HTTP en ajoutant un intercepteur pour ajouter le `header` d'authentification avec votre `Token`:

```kotlin
object Api {
  // ...
  private val okHttpClient by lazy {
    OkHttpClient.Builder()
      .addInterceptor { chain ->
        val newRequest = chain.request().newBuilder()
          .addHeader("Authorization", "Bearer $TOKEN")
          .build()
        chain.proceed(newRequest)
      }
      .build()
  }
}
```

- Créer une instance de retrofit qui permettra d'implémenter les interfaces que nous allons créer ensuite:

```kotlin
object Api {
  // ...
  private val retrofit = Retrofit.Builder()
    .baseUrl(BASE_URL)
    .client(okHttpClient)
    .addConverterFactory(MoshiConverterFactory.create(moshi))
    .build()
}
```

### UserService

- Créez l'interface `UserService` pour requêter les infos de l'utilisateur (importez `Response` avec <kbd>alt + enter</kbd> et choisissez la version `retrofit`):

```kotlin
interface UserService {
    @GET("users/info")
    suspend fun getInfo(): Response<UserInfo>
}
```

- Utilisez retrofit pour créer une implémentation de ce service (grace aux annotations):

```kotlin
object Api {
  // ...
  val userService: UserService by lazy { retrofit.create(UserService::class.java) }
}

```

### UserInfo

Exemple de json renvoyé par la route `/info`:

```json
{
  "email": "email",
  "firstname": "john",
  "lastname": "doe"
}
```

Créer la `data class` `UserInfo` avec des annotations Moshi pour récupérer ces données:

```kotlin
data class UserInfo(
    @field:Json(name = "email")
    val email: String,
    @field:Json(name = "firstname")
    val firstName: String,
    @field:Json(name = "lastname")
    val lastName: String
)
```

### Affichage

- Dans `fragment_tasks.xml`, ajoutez une `TextView` au dessus de la liste de tâche si vous n'en avez pas
- Overrider la méthode `onResume` pour y récuperer les infos de l'utilisateur, une erreur va s'afficher mais ne paniquez pas, on la résoud au point suivant:

```kotlin
val userInfo = Api.userService.getInfo().body()
```

- La méthode `getInfo()` étant déclarée comme `suspend`, vous aurez besoin de la lancer dans un `couroutineScope`.
Pour cela on peut utiliser `GlobalScope`, mais une meilleure façon est d'en créer un "vrai" pour pouvoir le `cancel()` après:

```kotlin
// Pour créer:
private val coroutineScope = MainScope()
// Pour utiliser:
coroutineScope.launch {...}
// Dans onDestroy():
coroutineScope.cancel()
```

Vous pouvez aussi utiliser `lifeCycleScope` en ayant ajouté `implementation "androidx.lifecycle:lifecycle-runtime-ktx:2.2.0-alpha01"` 

**NB:** Une vraiment bonne façon est d'utiliser les scopes fournis par android, notamment: `viewModelScope`, mais pour l'instant on implémente tout dans les fragments comme des 🐷

- Afficher les données dans votre `TextView`

```kotlin
    my_text_view.text = "${userInfo.firstName} ${userInfo.lastName}"
```
- Lancez l'app et vérifiez que vos infos s'affichent ! 

## TasksFragment

Il est temps de récuperer les tâches depuis le serveur !

- Créer un nouveau service `TaskService`

```kotlin
interface TasksService {
    @GET("tasks")
    suspend fun getTasks(): Response<List<Task>>
}
```

- Utiliser l'instance de retrofit comme précédemment pour créer une instance de `TaskService` dans l'objet `Api`

- Modifier `Task` pour la rendre lisible par Moshi (i.e. faire comme pour `UserInfo`)

## TasksRepository

Le Repository va chercher des data dans une ou plusieurs sources de données (ex: DB locale et API distante)

Créer la classe `TasksRepository`avec:

- une méthode publique `getTasks` qui renvoie des LiveData (auquel va s'abonner le fragment)
- une méthode privée `loadTasks` qui récupère la liste en asynchrone

```kotlin
class TasksRepository {
    private val tasksService = Api.tasksService
	private val coroutineScope = MainScope()

    fun getTasks(): LiveData<List<Task>?> {
        val tasks = MutableLiveData<List<Task>?>()
        coroutineScope.launch { tasks.postValue(loadTasks()) }
        return tasks
    }

    private suspend fun loadTasks(): List<Task>? {
        val tasksResponse = tasksService.getTasks()
        return if (tasksResponse.isSuccessful) tasksResponse.body() else null
    }
}
```

## LiveData

- Dans `TasksFragment`, ajouter une instance de `TasksRepository` 
- Modifier votre code en conséquence: dans `onResume()`, "abonnez" le fragment àla réponse du repository et mettez à jour la liste et l'`adapter` avec le résultat (importer le `Observer` de la lib `lifecycle`):


```kotlin
private val tasksRepository = TasksRepository()
private val tasks = mutableListOf<Task>()

// Dans onResume()
tasksRepository.getTasks().observe(this, Observer {
	if (it != null) {
	  tasks.clear()
	  tasks.addAll(it)
	  adapter.notifyDataSetChanged()
	}
})
```

## Compléter TasksService

Modifier `TasksService` et ajoutez y les routes suivantes:

```kotlin
@GET("tasks")
suspend fun getTasks(): Response<List<Task>>

@DELETE("tasks/{id}")
suspend fun deleteTask(@Path("id") id: String): Response<String>

@POST("tasks")
suspend fun createTask(@Body task: Task): Response<Task>

@PATCH("tasks/{id}")
suspend fun updateTask(@Body task: Task): Response<Task>
```

## Suppression, Ajout et Édition d'une tâche

**Remarque:** Vous pouvez créer des tâches dans l'interface web, en spécifiant votre token dans avec le bouton "Authorize" en haut

- Modifier l'action lorsqu'on clique sur le bouton de suppression et effectuer un call réseau afin de la supprimer dans le serveur puis supprimer la dans la liste locale `tasks`

- Avant de fermer l'Activity qui permet de créer/editer des tâches, effectuer un call réseau et vérifier qu'il n'y a pas d'erreurs avant de la fermer et de réafficher l'écran des tâches

## TasksViewModel

Mettre toute la logique dans le fragment est une trés mauvaise pratique: les `ViewModel` permettent d'extraire une partie logique du fragment.

Créer une classe `TasksViewModel` qui hérite de `ViewModel`: elle contiendra la liste des tâches, l'adapteur et le Repository, ainsi que les coroutines, supprimer la fonction `getTasks` du `TasksRepository`

Vous pourrez la récupérer dans le fragment grâce au `ViewModelProvider`, voici un squelette de l'implémentation globale:

```kotlin
class TasksFragment: Fragment() {
  val tasksAdapter = ...
  private val tasksViewModel by lazy {
    ViewModelProvider(this).get(TasksViewModel::class.java)
  }

  override fun onCreateView(...) {
    tasksViewModel.tasks.observe(this, Observer { ... })
  }

  override fun onResume(...) {
    tasksViewModel.loadTasks()
  }
}

class TasksViewModel: ViewModel() {
  private val tasks = MutableLiveData<List<Data>
  private val repository = ...
  
  fun loadTasks() { 
        viewModelScope { repository.loadTasks() }
    }
}

class TasksRepository {
  private val tasksService = Api.tasksService
    
  suspend fun loadTasks(): List<Task>? {
    val tasksResponse = tasksService.getTasks()
    return if (tasksResponse.isSuccessful) tasksResponse.body() else null
  }
}
```
