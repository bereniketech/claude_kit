---
name: java-patterns
description: Comprehensive Java 17+ patterns for Spring Boot services — idioms, JPA, REST, Security (JWT/OAuth2), TDD, and verification.
---

# Java Patterns

Production-grade Java 17+ patterns for Spring Boot services covering code standards, data access, REST APIs, security, and testing.

---

## 1. Java Idioms and Code Standards

Use records for immutable DTOs, prefer `final` fields, and keep methods short and focused.

```java
// Records for data carriers
public record MarketDto(Long id, String name, MarketStatus status) {}

// PascalCase classes, camelCase methods/fields, UPPER_SNAKE_CASE constants
private static final int MAX_PAGE_SIZE = 100;
private final MarketRepository marketRepository;
```

Use `Optional` with map/flatMap — never call `.get()` directly:

```java
return marketRepository.findBySlug(slug)
    .map(MarketResponse::from)
    .orElseThrow(() -> new MarketNotFoundException(slug));
```

Use streams for transformations; keep pipelines short:

```java
List<String> names = markets.stream()
    .map(Market::name)
    .filter(Objects::nonNull)
    .toList();
```

**Rule:** Throw domain-specific unchecked exceptions (`MarketNotFoundException`), never broad `catch (Exception ex)` unless rethrowing centrally. Use `@NotNull`/`@NonNull` on inputs; use `@Nullable` only when unavoidable.

Project layout:
```
src/main/java/com/example/app/
  config/ controller/ service/ repository/ domain/ dto/ util/
src/test/java/... (mirrors main)
```

---

## 2. JPA Entity Design and Repositories

Annotate entities with `@Table` indexes, use `@Enumerated(EnumType.STRING)`, and enable JPA auditing:

```java
@Entity
@Table(name = "markets", indexes = {
  @Index(name = "idx_markets_slug", columnList = "slug", unique = true)
})
@EntityListeners(AuditingEntityListener.class)
public class MarketEntity {
  @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
  private Long id;

  @Column(nullable = false, unique = true, length = 120)
  private String slug;

  @Enumerated(EnumType.STRING)
  private MarketStatus status = MarketStatus.ACTIVE;

  @CreatedDate private Instant createdAt;
  @LastModifiedDate private Instant updatedAt;
}
```

Default to lazy loading; use `JOIN FETCH` to prevent N+1:

```java
@Query("select m from MarketEntity m left join fetch m.positions where m.id = :id")
Optional<MarketEntity> findWithPositions(@Param("id") Long id);
```

Use projections for lightweight reads:

```java
public interface MarketSummary { Long getId(); String getName(); MarketStatus getStatus(); }
Page<MarketSummary> findAllBy(Pageable pageable);
```

Paginate with `PageRequest`:

```java
PageRequest page = PageRequest.of(pageNumber, pageSize, Sort.by("createdAt").descending());
Page<MarketEntity> markets = repo.findByStatus(MarketStatus.ACTIVE, page);
```

**Rule:** Use Flyway or Liquibase for migrations — never rely on Hibernate auto DDL in production. Add composite indexes matching query patterns (`status, created_at`). Configure HikariCP: `maximum-pool-size=20`, `minimum-idle=5`.

---

## 3. Spring Boot REST and Service Layer

Keep controllers thin; inject services via constructor injection only:

```java
@RestController
@RequestMapping("/api/markets")
@Validated
class MarketController {
  private final MarketService marketService;

  MarketController(MarketService marketService) { this.marketService = marketService; }

  @GetMapping
  ResponseEntity<Page<MarketResponse>> list(
      @RequestParam(defaultValue = "0") int page,
      @RequestParam(defaultValue = "20") int size) {
    return ResponseEntity.ok(marketService.list(PageRequest.of(page, size)).map(MarketResponse::from));
  }

  @PostMapping
  ResponseEntity<MarketResponse> create(@Valid @RequestBody CreateMarketRequest request) {
    return ResponseEntity.status(HttpStatus.CREATED).body(MarketResponse.from(marketService.create(request)));
  }
}
```

Annotate service methods with `@Transactional`; use `readOnly = true` for reads:

```java
@Service
public class MarketService {
  @Transactional
  public Market create(CreateMarketRequest request) { ... }

  @Transactional(readOnly = true)
  public Page<Market> list(Pageable pageable) { ... }
}
```

Handle exceptions centrally:

```java
@ControllerAdvice
class GlobalExceptionHandler {
  @ExceptionHandler(MethodArgumentNotValidException.class)
  ResponseEntity<ApiError> handleValidation(MethodArgumentNotValidException ex) {
    String message = ex.getBindingResult().getFieldErrors().stream()
        .map(e -> e.getField() + ": " + e.getDefaultMessage())
        .collect(Collectors.joining(", "));
    return ResponseEntity.badRequest().body(ApiError.validation(message));
  }

  @ExceptionHandler(Exception.class)
  ResponseEntity<ApiError> handleGeneric(Exception ex) {
    return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).body(ApiError.of("Internal server error"));
  }
}
```

**Rule:** Enable `spring.mvc.problemdetails.enabled=true` (Spring Boot 3+). Use `@Async` + `@EnableAsync` for fire-and-forget operations. Prefer constructor injection — avoid field injection.

---

## 4. Spring Security — JWT and Authorization

Validate JWT tokens in a filter; set `SecurityContextHolder` before chain proceeds:

```java
@Component
public class JwtAuthFilter extends OncePerRequestFilter {
  private final JwtService jwtService;

  public JwtAuthFilter(JwtService jwtService) { this.jwtService = jwtService; }

  @Override
  protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response,
      FilterChain chain) throws ServletException, IOException {
    String header = request.getHeader(HttpHeaders.AUTHORIZATION);
    if (header != null && header.startsWith("Bearer ")) {
      Authentication auth = jwtService.authenticate(header.substring(7));
      SecurityContextHolder.getContext().setAuthentication(auth);
    }
    chain.doFilter(request, response);
  }
}
```

Enable method security and deny by default:

```java
@EnableMethodSecurity
// In controller:
@PreAuthorize("hasRole('ADMIN')")
@GetMapping("/users")
public List<UserDto> listUsers() { ... }
```

Configure CORS at the security filter level, not per-controller:

```java
@Bean
public CorsConfigurationSource corsConfigurationSource() {
  CorsConfiguration config = new CorsConfiguration();
  config.setAllowedOrigins(List.of("https://app.example.com"));
  config.setAllowedMethods(List.of("GET", "POST", "PUT", "DELETE"));
  config.setAllowCredentials(true);
  UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
  source.registerCorsConfiguration("/api/**", config);
  return source;
}
```

Always hash passwords with BCrypt (cost factor 12):

```java
@Bean
public PasswordEncoder passwordEncoder() { return new BCryptPasswordEncoder(12); }
```

**Rule:** No secrets in source — use `${DB_PASSWORD}` env placeholders or Spring Cloud Vault. Never concatenate strings in SQL queries — use `:param` bindings. Disable CSRF for stateless Bearer-token APIs; keep enabled for browser session apps. Never log secrets, tokens, or PAN data.

---

## 5. TDD with JUnit 5 and Mockito

Follow RED → GREEN → REFACTOR. Write the test first, make it fail, implement the minimum to pass:

```java
@ExtendWith(MockitoExtension.class)
class MarketServiceTest {
  @Mock MarketRepository repo;
  @InjectMocks MarketService service;

  @Test
  void createsMarket() {
    when(repo.save(any())).thenAnswer(inv -> inv.getArgument(0));
    Market result = service.create(new CreateMarketRequest("name", "desc", Instant.now(), List.of("cat")));
    assertThat(result.name()).isEqualTo("name");
    verify(repo).save(any());
  }

  @Test
  void throwsWhenNotFound() {
    when(repo.findBySlug("x")).thenReturn(Optional.empty());
    assertThatThrownBy(() -> service.findBySlug("x"))
        .isInstanceOf(MarketNotFoundException.class);
  }
}
```

Test the web layer with `@WebMvcTest`:

```java
@WebMvcTest(MarketController.class)
class MarketControllerTest {
  @Autowired MockMvc mockMvc;
  @MockBean MarketService marketService;

  @Test
  void returnsCreated() throws Exception {
    when(marketService.create(any())).thenReturn(new Market(1L, "Test", MarketStatus.ACTIVE));
    mockMvc.perform(post("/api/markets")
        .contentType(MediaType.APPLICATION_JSON)
        .content("""{"name":"Test","description":"Desc","endDate":"2030-01-01T00:00:00Z","categories":["general"]}"""))
      .andExpect(status().isCreated())
      .andExpect(jsonPath("$.name").value("Test"));
  }
}
```

**Rule:** Prefer AssertJ (`assertThat`) over JUnit assertions. Use `@ParameterizedTest` for input variants. Avoid partial mocks; prefer explicit stubbing. Target 80%+ coverage enforced via JaCoCo.

---

## 6. Integration Testing and Verification

Use Testcontainers for real database tests:

```java
@SpringBootTest
@Testcontainers
class UserRepositoryIntegrationTest {
  @Container
  static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:16-alpine");

  @DynamicPropertySource
  static void configureProperties(DynamicPropertyRegistry registry) {
    registry.add("spring.datasource.url", postgres::getJdbcUrl);
    registry.add("spring.datasource.username", postgres::getUsername);
    registry.add("spring.datasource.password", postgres::getPassword);
  }

  @Autowired private UserRepository userRepository;

  @Test
  void findByEmail_existingUser_returnsUser() {
    userRepository.save(new User("Alice", "alice@example.com"));
    assertThat(userRepository.findByEmail("alice@example.com")).isPresent();
  }
}
```

Run the full verification pipeline before PRs:

```bash
# 1. Build
mvn -T 4 clean verify -DskipTests

# 2. Static analysis
mvn -T 4 spotbugs:check pmd:check checkstyle:check

# 3. Tests + coverage
mvn -T 4 test && mvn jacoco:report   # verify 80%+

# 4. Security scan
mvn org.owasp:dependency-check-maven:check

# 5. Format
mvn spotless:apply
```

**Rule:** Use `@DataJpaTest` with `@AutoConfigureTestDatabase(replace = NONE)` + Testcontainers to mirror production. Run ASan/sanitizer equivalents (SpotBugs, PMD) as part of CI. Treat warnings as defects in production systems.
