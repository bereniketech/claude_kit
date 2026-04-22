---
name: kotlin-patterns
description: Comprehensive Kotlin skill covering idioms, null safety, data/sealed classes, coroutines/flows, Ktor routing/auth, Exposed ORM, and testing with MockK/Kotest.
---

# Kotlin Development Patterns

Idiomatic Kotlin for backend services — from null-safe idioms and coroutines to Ktor APIs, Exposed ORM, and production-ready testing.

---

## 1. Kotlin Idioms and Null Safety

Prefer `val` over `var`. Use safe calls `?.` and Elvis `?:` instead of force-unwrapping `!!`. Use expression bodies for single-expression functions.

```kotlin
// Null safety
fun getUserEmail(userId: String): String =
    userRepository.findById(userId)?.email ?: "unknown@example.com"

// Immutable data class with copy()
data class User(val id: String, val name: String, val email: String)
fun updateEmail(user: User, email: String): User = user.copy(email = email)

// Value class for type safety
@JvmInline value class UserId(val value: String) {
    init { require(value.isNotBlank()) { "UserId cannot be blank" } }
}
```

Use `require`/`check`/`error` for preconditions. Use `when` as an expression. Avoid `GlobalScope` — always use structured concurrency.

**Rule:** Never force-unwrap with `!!`. Prefer nullable return or `Result` over exceptions for expected failure cases.

---

## 2. Data Classes, Sealed Classes, and Scope Functions

Use sealed classes for exhaustive, compiler-enforced hierarchies:

```kotlin
sealed class Result<out T> {
    data class Success<T>(val data: T) : Result<T>()
    data class Failure(val error: Throwable) : Result<Nothing>()
    data object Loading : Result<Nothing>()
}

fun <T> Result<T>.getOrNull(): T? = when (this) {
    is Result.Success -> data
    is Result.Failure, is Result.Loading -> null
}
```

Scope functions — use the right one:
- `let` — transform nullable or scoped result
- `apply` — configure an object, returns the object
- `also` — side effects, returns the object
- `run`/`with` — execute a block, returns result

**Rule:** Do not nest scope functions more than one level deep — chain safe calls instead.

---

## 3. Coroutines and Structured Concurrency

Use `coroutineScope` + `async` for parallel work. Use `supervisorScope` when child failures must not cancel siblings.

```kotlin
suspend fun loadDashboard(userId: String): Dashboard = coroutineScope {
    val user  = async { userService.getUser(userId) }
    val posts = async { postService.getUserPosts(userId) }
    Dashboard(user = user.await(), posts = posts.await())
}

suspend fun processItems(items: List<Item>) {
    items.forEach { item ->
        ensureActive()   // cooperative cancellation
        processItem(item)
    }
}

suspend fun acquireAndProcess() {
    val r = acquireResource()
    try { r.process() }
    finally { withContext(NonCancellable) { r.release() } }
}
```

**Rule:** Never use `GlobalScope`. Inject `CoroutineDispatcher` via DI for testability.

---

## 4. Flow for Reactive Streams

Use `StateFlow` for UI/shared state, `SharedFlow` for one-shot events, `Flow` for cold streams.

```kotlin
// Cold flow with error handling
fun observeUsers(): Flow<List<User>> = flow {
    while (currentCoroutineContext().isActive) {
        emit(userRepository.findAll())
        delay(5.seconds)
    }
}.catch { emit(emptyList()) }

// Debounce search
fun searchUsers(query: Flow<String>): Flow<List<User>> =
    query.debounce(300.milliseconds)
        .distinctUntilChanged()
        .filter { it.length >= 2 }
        .mapLatest { q -> userRepository.search(q) }
        .catch { emit(emptyList()) }
```

**Rule:** Use `SharingStarted.WhileSubscribed(5_000)` for `stateIn` — survives config changes without restarting upstream.

---

## 5. Ktor Routing and Middleware

Structure the application using plugin functions:

```kotlin
fun Application.module() {
    configureSerialization()
    configureAuthentication()
    configureStatusPages()
    configureDI()
    configureRouting()
}

fun Route.userRoutes() {
    val userService by inject<UserService>()
    route("/users") {
        get { call.respond(userService.getAll()) }
        get("/{id}") {
            val id = call.parameters["id"] ?: return@get call.respond(HttpStatusCode.BadRequest)
            call.respond(userService.getById(id) ?: return@get call.respond(HttpStatusCode.NotFound))
        }
        authenticate("jwt") {
            post {
                val req = call.receive<CreateUserRequest>()
                call.respond(HttpStatusCode.Created, userService.create(req))
            }
        }
    }
}
```

Use `StatusPages` for centralized error handling — map exceptions to HTTP status codes. Use `install(CORS)` with explicit allowed hosts and methods.

---

## 6. Ktor Authentication (JWT) and Koin DI

```kotlin
fun Application.configureAuthentication() {
    install(Authentication) {
        jwt("jwt") {
            verifier(JWT.require(Algorithm.HMAC256(jwtSecret)).withIssuer(issuer).build())
            validate { cred ->
                if (cred.payload.audience.contains(audience)) JWTPrincipal(cred.payload) else null
            }
            challenge { _, _ -> call.respond(HttpStatusCode.Unauthorized, "Invalid token") }
        }
    }
}

val appModule = module {
    single<UserRepository> { ExposedUserRepository(get()) }
    single { UserService(get()) }
    single { AuthService(get(), get()) }
}
```

Test authenticated routes with `bearerAuth(token)` inside `testApplication`.

---

## 7. Exposed ORM

Use `newSuspendedTransaction` for all coroutine-safe database operations. Define tables as objects extending `UUIDTable`. Use the repository pattern to isolate business logic from data access.

```kotlin
object UsersTable : UUIDTable("users") {
    val name      = varchar("name", 100)
    val email     = varchar("email", 255).uniqueIndex()
    val role      = enumerationByName<Role>("role", 20)
    val createdAt = timestampWithTimeZone("created_at").defaultExpression(CurrentTimestampWithTimeZone)
}

class ExposedUserRepository(private val db: Database) : UserRepository {
    override suspend fun findById(id: UUID): User? =
        newSuspendedTransaction(db = db) {
            UsersTable.selectAll().where { UsersTable.id eq id }
                .map { it.toUser() }.singleOrNull()
        }

    override suspend fun create(req: CreateUserRequest): User =
        newSuspendedTransaction(db = db) {
            UsersTable.insert {
                it[name]  = req.name
                it[email] = req.email
                it[role]  = req.role
            }.resultedValues!!.first().toUser()
        }
}
```

Use HikariCP for connection pooling (`isAutoCommit = false`, `TRANSACTION_READ_COMMITTED`). Use Flyway for versioned migrations at startup.

**Rule:** Always escape LIKE patterns to prevent wildcard injection. Use `batchInsert` for bulk operations.

---

## 8. Testing with Kotest and MockK

Follow RED-GREEN-REFACTOR. Choose a spec style and apply it consistently across the project.

```kotlin
// FunSpec — JUnit-like, most common
class UserServiceTest : FunSpec({
    val repository = mockk<UserRepository>()
    val service = UserService(repository)

    beforeTest { clearMocks(repository) }

    test("getUser returns user when found") {
        val expected = User(id = "1", name = "Alice", email = "a@x.com")
        coEvery { repository.findById("1") } returns expected
        service.getUser("1") shouldBe expected
        coVerify(exactly = 1) { repository.findById("1") }
    }

    test("getUser throws when not found") {
        coEvery { repository.findById("999") } returns null
        shouldThrow<UserNotFoundException> { service.getUser("999") }
    }
})
```

Use `coEvery`/`coVerify` for suspend functions. Use `slot<T>()` to capture arguments. Use `spyk` for partial mocking.

Test coroutines with `runTest`, flows with Turbine (`.test { awaitItem() }`), and time with `advanceTimeBy`.

Test Ktor routes with `testApplication`:

```kotlin
test("POST /users creates user") {
    testApplication {
        application { install(Koin) { modules(testModule) }; configureSerialization(); configureRouting() }
        val client = createClient { install(ContentNegotiation) { json() } }
        val response = client.post("/users") {
            contentType(ContentType.Application.Json)
            setBody(CreateUserRequest("Alice", "alice@example.com"))
        }
        response.status shouldBe HttpStatusCode.Created
    }
}
```

Use in-memory H2 (`MODE=PostgreSQL`) for Exposed repository tests.

**Rule:** Configure Kover with `minBound(80)` to fail builds below 80% coverage. Run `./gradlew koverVerify` in CI.
