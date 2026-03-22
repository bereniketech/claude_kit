---
name: android-patterns
description: Android and Kotlin Multiplatform patterns covering Clean Architecture, MVVM with StateFlow, Hilt/Koin DI, Room, Jetpack Compose, and Compose Multiplatform.
---

# Android Patterns

Production-grade Android and Kotlin Multiplatform patterns — Clean Architecture layering, MVVM with StateFlow, dependency injection, data persistence with Room/SQLDelight, and Compose UI.

---

## 1. Module Structure and Dependency Rules

Organize the project into strict layers with one-directional dependencies:

```
project/
├── app/              # Entry point, DI wiring, Application class
├── core/             # Shared utilities, base classes, error types
├── domain/           # UseCases, domain models, repository interfaces (pure Kotlin)
├── data/             # Repository implementations, DataSources, DB, network
├── presentation/     # Screens, ViewModels, UI models, navigation
├── design-system/    # Reusable Compose components, theme, typography
└── feature/          # Feature modules for larger projects (auth, settings, profile)
```

Dependency direction:
```
app → presentation, domain, data, core
presentation → domain, design-system, core
data → domain, core
domain → core (or nothing)
core → (nothing)
```

**Rule:** `domain` must never import Android framework classes, data layer types, or any framework. It contains pure Kotlin only. Never expose database entities or DTOs to the UI layer — always map to domain models. Never allow circular module dependencies.

---

## 2. Domain Layer — UseCases and Repository Interfaces

Each UseCase represents one business operation. Use `operator fun invoke` for clean call sites:

```kotlin
class GetItemsByCategoryUseCase(
    private val repository: ItemRepository
) {
    suspend operator fun invoke(category: String): Result<List<Item>> {
        return repository.getItemsByCategory(category)
    }
}

// Flow-based UseCase for reactive streams
class ObserveUserProgressUseCase(
    private val repository: UserRepository
) {
    operator fun invoke(userId: String): Flow<UserProgress> {
        return repository.observeProgress(userId)
    }
}
```

Domain models are plain Kotlin data classes — no framework annotations:

```kotlin
data class Item(
    val id: String,
    val title: String,
    val description: String,
    val tags: List<String>,
    val status: Status,
    val category: String
)

enum class Status { DRAFT, ACTIVE, ARCHIVED }
```

Repository interfaces live in domain; implementations live in data:

```kotlin
interface ItemRepository {
    suspend fun getItemsByCategory(category: String): Result<List<Item>>
    suspend fun saveItem(item: Item): Result<Unit>
    fun observeItems(): Flow<List<Item>>
}
```

**Rule:** Put business logic in UseCases, not ViewModels. Never use `GlobalScope` or unstructured coroutines — use `viewModelScope` or structured concurrency. Never put Android imports in domain module files.

---

## 3. Data Layer — Repository, Room, and Ktor

Repository implementation coordinates local and remote sources with mappers:

```kotlin
class ItemRepositoryImpl(
    private val localDataSource: ItemLocalDataSource,
    private val remoteDataSource: ItemRemoteDataSource
) : ItemRepository {

    override suspend fun getItemsByCategory(category: String): Result<List<Item>> {
        return runCatching {
            val remote = remoteDataSource.fetchItems(category)
            localDataSource.insertItems(remote.map { it.toEntity() })
            localDataSource.getItemsByCategory(category).map { it.toDomain() }
        }
    }

    override fun observeItems(): Flow<List<Item>> {
        return localDataSource.observeAll().map { entities -> entities.map { it.toDomain() } }
    }
}

// Mapper extension functions — keep near the data models
fun ItemEntity.toDomain() = Item(id, title, description, tags.split("|"), Status.valueOf(status), category)
fun ItemDto.toEntity() = ItemEntity(id, title, description, tags.joinToString("|"), status, category)
```

Room database for Android:

```kotlin
@Entity(tableName = "items")
data class ItemEntity(
    @PrimaryKey val id: String,
    val title: String,
    val description: String,
    val tags: String,
    val status: String,
    val category: String
)

@Dao
interface ItemDao {
    @Query("SELECT * FROM items WHERE category = :category")
    suspend fun getByCategory(category: String): List<ItemEntity>

    @Upsert
    suspend fun upsert(items: List<ItemEntity>)

    @Query("SELECT * FROM items")
    fun observeAll(): Flow<List<ItemEntity>>
}
```

Ktor HTTP client for KMP:

```kotlin
class ItemRemoteDataSource(private val client: HttpClient) {
    suspend fun fetchItems(category: String): List<ItemDto> {
        return client.get("api/items") { parameter("category", category) }.body()
    }
}

val httpClient = HttpClient {
    install(ContentNegotiation) { json(Json { ignoreUnknownKeys = true }) }
    install(Logging) { level = LogLevel.HEADERS }
    defaultRequest { url("https://api.example.com/") }
}
```

**Rule:** Split fat repository implementations into focused DataSources (local vs remote). Keep mappers as extension functions near the data models. Use `@Upsert` for insert-or-update patterns in Room.

---

## 4. Dependency Injection — Hilt (Android) and Koin (KMP)

Hilt for Android-only projects:

```kotlin
@Module
@InstallIn(SingletonComponent::class)
abstract class RepositoryModule {
    @Binds
    abstract fun bindItemRepository(impl: ItemRepositoryImpl): ItemRepository
}

@HiltViewModel
class ItemListViewModel @Inject constructor(
    private val getItems: GetItemsByCategoryUseCase
) : ViewModel()
```

Koin for KMP-friendly DI:

```kotlin
val domainModule = module {
    factory { GetItemsByCategoryUseCase(get()) }
    factory { ObserveUserProgressUseCase(get()) }
}

val dataModule = module {
    single<ItemRepository> { ItemRepositoryImpl(get(), get()) }
    single { ItemLocalDataSource(get()) }
    single { ItemRemoteDataSource(get()) }
}

val presentationModule = module {
    viewModelOf(::ItemListViewModel)
    viewModelOf(::DashboardViewModel)
}
```

**Rule:** Use convention plugins in KMP projects to reduce build file duplication. Use `viewModelOf` with Koin instead of manually declaring ViewModel factories.

---

## 5. MVVM with ViewModel and StateFlow

Use a single data class for screen state. Expose as `StateFlow`; collect with `collectAsStateWithLifecycle`:

```kotlin
data class ItemListState(
    val items: List<Item> = emptyList(),
    val isLoading: Boolean = false,
    val error: String? = null,
    val searchQuery: String = ""
)

class ItemListViewModel(
    private val getItems: GetItemsUseCase
) : ViewModel() {
    private val _state = MutableStateFlow(ItemListState())
    val state: StateFlow<ItemListState> = _state.asStateFlow()

    fun onSearch(query: String) {
        _state.update { it.copy(searchQuery = query) }
        loadItems(query)
    }

    private fun loadItems(query: String) {
        viewModelScope.launch {
            _state.update { it.copy(isLoading = true) }
            getItems(query).fold(
                onSuccess = { items -> _state.update { it.copy(items = items, isLoading = false) } },
                onFailure = { e -> _state.update { it.copy(error = e.message, isLoading = false) } }
            )
        }
    }
}
```

For complex screens, use a sealed event interface instead of multiple callback lambdas:

```kotlin
sealed interface ItemListEvent {
    data class Search(val query: String) : ItemListEvent
    data class Delete(val itemId: String) : ItemListEvent
    data object Refresh : ItemListEvent
}

// In ViewModel
fun onEvent(event: ItemListEvent) {
    when (event) {
        is ItemListEvent.Search -> onSearch(event.query)
        is ItemListEvent.Delete -> deleteItem(event.itemId)
        is ItemListEvent.Refresh -> loadItems(_state.value.searchQuery)
    }
}
```

Use sealed interfaces for domain error propagation:

```kotlin
sealed interface AppError {
    data class Network(val message: String) : AppError
    data class Database(val message: String) : AppError
    data object Unauthorized : AppError
}
```

**Rule:** Never use `mutableStateOf` in ViewModels when `MutableStateFlow` with `collectAsStateWithLifecycle` is safer for lifecycle. Use `viewModelScope.launch` for all coroutines in ViewModels.

---

## 6. Jetpack Compose and Compose Multiplatform

Collect state from ViewModel; pass it to stateless composables:

```kotlin
@Composable
fun ItemListScreen(viewModel: ItemListViewModel = koinViewModel()) {
    val state by viewModel.state.collectAsStateWithLifecycle()
    ItemListContent(state = state, onEvent = viewModel::onEvent)
}

@Composable
private fun ItemListContent(state: ItemListState, onEvent: (ItemListEvent) -> Unit) {
    // Stateless — easy to preview and test
}
```

Type-safe navigation with `@Serializable` routes (Compose Navigation 2.8+):

```kotlin
@Serializable data object HomeRoute
@Serializable data class DetailRoute(val id: String)

@Composable
fun AppNavHost(navController: NavHostController = rememberNavController()) {
    NavHost(navController, startDestination = HomeRoute) {
        composable<HomeRoute> {
            HomeScreen(onNavigateToDetail = { id -> navController.navigate(DetailRoute(id)) })
        }
        composable<DetailRoute> { backStackEntry ->
            DetailScreen(id = backStackEntry.toRoute<DetailRoute>().id)
        }
    }
}
```

Mark UI models stable/immutable to enable skippable recomposition:

```kotlin
@Immutable
data class ItemUiModel(val id: String, val title: String, val progress: Float)
```

Use stable keys in lazy lists and `derivedStateOf` to defer reads:

```kotlin
LazyColumn {
    items(items = items, key = { it.id }) { item -> ItemRow(item = item) }
}

val showScrollToTop by remember {
    derivedStateOf { listState.firstVisibleItemIndex > 5 }
}
```

Use slot-based APIs for flexible, reusable composables:

```kotlin
@Composable
fun AppCard(
    modifier: Modifier = Modifier,
    header: @Composable () -> Unit = {},
    content: @Composable ColumnScope.() -> Unit,
    actions: @Composable RowScope.() -> Unit = {}
) {
    Card(modifier = modifier) {
        Column { header(); Column(content = content); Row(content = actions) }
    }
}
```

For KMP platform-specific UI, use `expect`/`actual`:

```kotlin
// commonMain
@Composable
expect fun PlatformStatusBar(darkIcons: Boolean)

// androidMain
@Composable
actual fun PlatformStatusBar(darkIcons: Boolean) {
    val systemUiController = rememberSystemUiController()
    SideEffect { systemUiController.setStatusBarColor(Color.Transparent, darkIcons) }
}
```

**Rule:** Never pass `NavController` deep into composables — pass lambda callbacks instead. Never perform heavy computation inside `@Composable` functions — move to ViewModel or `remember {}`. Never create new object instances in composable parameters — causes unnecessary recomposition. Apply modifier order: layout → shape → drawing → interaction. Use `@Immutable` or `@Stable` to help the Compose compiler skip unchanged composables.
