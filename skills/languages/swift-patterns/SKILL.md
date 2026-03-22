---
name: swift-patterns
description: Comprehensive Swift patterns covering concurrency (6.2+), actor-based persistence, protocol DI, SwiftUI state management, and testing with Swift Testing.
---

# Swift Patterns

Modern Swift patterns for Apple platform development — structured concurrency, actor isolation, protocol-based dependency injection, SwiftUI view composition, and testable architecture.

---

## 1. Swift 6.2 Concurrency Model

In Swift 6.2, async functions stay on the calling actor by default — no implicit background offloading. Start with `@MainActor`-isolated types; introduce concurrency explicitly.

```swift
// Stays on MainActor — no data race
@MainActor
final class StickerModel {
    let photoProcessor = PhotoProcessor()

    func extractSticker(_ item: PhotosPickerItem) async throws -> Sticker? {
        guard let data = try await item.loadTransferable(type: Data.self) else { return nil }
        return await photoProcessor.extractSticker(data: data, with: item.itemIdentifier)
    }
}
```

Use `@concurrent` only for CPU-intensive work that needs true background execution:

```swift
nonisolated final class PhotoProcessor {
    // @concurrent offloads to thread pool — profile before adding
    @concurrent
    static func extractSubject(from data: Data) async -> Sticker { /* heavy work */ }
}
```

Use isolated conformances so `@MainActor` types can satisfy protocols without workarounds:

```swift
extension StickerModel: @MainActor Exportable {
    func export() { photoProcessor.exportAsPNG() }
}
```

Protect global/static state with `@MainActor`:

```swift
@MainActor
final class StickerLibrary {
    static let shared: StickerLibrary = .init()
}
```

**Rule:** Never use `DispatchQueue` or `NSLock` for new Swift concurrency code — use actors. Never apply `nonisolated` to suppress compiler errors without understanding the isolation consequence. Profile before offloading with `@concurrent`.

---

## 2. Actor-Based Persistence

Use actors to build thread-safe persistence layers — the compiler enforces serialized access:

```swift
public actor LocalRepository<T: Codable & Identifiable> where T.ID == String {
    private var cache: [String: T] = [:]
    private let fileURL: URL

    public init(directory: URL = .documentsDirectory, filename: String = "data.json") {
        self.fileURL = directory.appendingPathComponent(filename)
        self.cache = Self.loadSynchronously(from: fileURL)
    }

    public func save(_ item: T) throws {
        cache[item.id] = item
        try persistToFile()
    }

    public func delete(_ id: String) throws {
        cache[id] = nil
        try persistToFile()
    }

    public func find(by id: String) -> T? { cache[id] }
    public func loadAll() -> [T] { Array(cache.values) }

    private func persistToFile() throws {
        let data = try JSONEncoder().encode(Array(cache.values))
        try data.write(to: fileURL, options: .atomic)  // .atomic prevents partial writes
    }

    private static func loadSynchronously(from url: URL) -> [String: T] {
        guard let data = try? Data(contentsOf: url),
              let items = try? JSONDecoder().decode([T].self, from: data) else { return [:] }
        return Dictionary(uniqueKeysWithValues: items.map { ($0.id, $0) })
    }
}
```

Combine with `@Observable` ViewModel for reactive UI:

```swift
@Observable
final class QuestionListViewModel {
    private(set) var questions: [Question] = []
    private let repository: LocalRepository<Question>

    init(repository: LocalRepository<Question> = LocalRepository()) { self.repository = repository }

    func load() async { questions = await repository.loadAll() }

    func add(_ question: Question) async throws {
        try await repository.save(question)
        questions = await repository.loadAll()
    }
}
```

**Rule:** Use `Sendable` types for all data crossing actor boundaries. Use `.atomic` writes to prevent data corruption on crash. Load synchronously in `init` — async initializers add complexity for minimal benefit with local files. Keep the actor's public API domain-focused, not persistence-focused.

---

## 3. Protocol-Based Dependency Injection

Abstract external dependencies (file system, network, APIs) behind small, focused protocols. Each protocol handles exactly one concern:

```swift
public protocol FileSystemProviding: Sendable {
    func containerURL(for purpose: Purpose) -> URL?
}

public protocol FileAccessorProviding: Sendable {
    func read(from url: URL) throws -> Data
    func write(_ data: Data, to url: URL) throws
    func fileExists(at url: URL) -> Bool
}
```

Provide default (production) implementations and injectable mocks:

```swift
public struct DefaultFileAccessor: FileAccessorProviding {
    public func read(from url: URL) throws -> Data { try Data(contentsOf: url) }
    public func write(_ data: Data, to url: URL) throws { try data.write(to: url, options: .atomic) }
    public func fileExists(at url: URL) -> Bool { FileManager.default.fileExists(atPath: url.path) }
}

public final class MockFileAccessor: FileAccessorProviding, @unchecked Sendable {
    public var files: [URL: Data] = [:]
    public var readError: Error?

    public func read(from url: URL) throws -> Data {
        if let error = readError { throw error }
        guard let data = files[url] else { throw CocoaError(.fileReadNoSuchFile) }
        return data
    }
    public func write(_ data: Data, to url: URL) throws { files[url] = data }
    public func fileExists(at url: URL) -> Bool { files[url] != nil }
}
```

Inject via default parameters — production uses defaults, tests inject mocks:

```swift
public actor SyncManager {
    private let fileSystem: FileSystemProviding
    private let fileAccessor: FileAccessorProviding

    public init(
        fileSystem: FileSystemProviding = DefaultFileSystemProvider(),
        fileAccessor: FileAccessorProviding = DefaultFileAccessor()
    ) {
        self.fileSystem = fileSystem
        self.fileAccessor = fileAccessor
    }
}
```

**Rule:** Only mock external boundaries — file system, network, external APIs. Never mock internal types with no external dependencies. Require `Sendable` conformance on protocols used across actor boundaries. Never use `#if DEBUG` conditionals instead of proper DI.

---

## 4. Testing with Swift Testing

Use Swift Testing (`import Testing`) with `@Test`, `#expect`, and `@Suite`:

```swift
import Testing

@Test("Sync manager handles missing container")
func testMissingContainer() async {
    let mockFileSystem = MockFileSystemProvider(containerURL: nil)
    let manager = SyncManager(fileSystem: mockFileSystem)

    await #expect(throws: SyncError.containerNotAvailable) {
        try await manager.sync()
    }
}

@Test("Sync manager reads data correctly")
func testReadData() async throws {
    let mockFileAccessor = MockFileAccessor()
    mockFileAccessor.files[testURL] = testData

    let manager = SyncManager(fileAccessor: mockFileAccessor)
    let result = try await manager.loadData()

    #expect(result == expectedData)
}

@Test("Sync manager handles read errors gracefully")
func testReadError() async {
    let mockFileAccessor = MockFileAccessor()
    mockFileAccessor.readError = CocoaError(.fileReadCorruptFile)
    let manager = SyncManager(fileAccessor: mockFileAccessor)

    await #expect(throws: SyncError.self) {
        try await manager.sync()
    }
}
```

Use `#Preview` macros with injected mock repositories for fast SwiftUI iteration:

```swift
#Preview("Empty state") {
    ItemListView(viewModel: ItemListViewModel(repository: EmptyMockRepository()))
}
```

**Rule:** Test error handling paths that are hard to trigger in real environments. Design mocks with configurable `errorToThrow` properties.

---

## 5. SwiftUI State Management and View Composition

Choose the simplest state wrapper that fits the scope:

| Wrapper | Use Case |
|---|---|
| `@State` | View-local value types (toggles, form fields) |
| `@Binding` | Two-way reference to parent's `@State` |
| `@Observable` class + `@State` | Owned model with multiple properties |
| `@Bindable` | Two-way binding to an `@Observable` property |
| `@Environment` | Shared dependencies injected via `.environment()` |

Use `@Observable` (not `ObservableObject`) — it tracks property-level changes:

```swift
@Observable
final class ItemListViewModel {
    private(set) var items: [Item] = []
    private(set) var isLoading = false
    var searchText = ""
    private let repository: any ItemRepository

    init(repository: any ItemRepository = DefaultItemRepository()) { self.repository = repository }

    func load() async {
        isLoading = true
        defer { isLoading = false }
        items = (try? await repository.fetchAll()) ?? []
    }
}

struct ItemListView: View {
    @State private var viewModel: ItemListViewModel

    var body: some View {
        List(viewModel.items) { item in ItemRow(item: item) }
            .searchable(text: $viewModel.searchText)
            .overlay { if viewModel.isLoading { ProgressView() } }
            .task { await viewModel.load() }
    }
}
```

Use type-safe `NavigationStack` with a `Router` observable:

```swift
@Observable
final class Router {
    var path = NavigationPath()
    func navigate(to destination: Destination) { path.append(destination) }
    func popToRoot() { path = NavigationPath() }
}

enum Destination: Hashable {
    case detail(Item.ID)
    case settings
}
```

**Rule:** Never use `ObservableObject`/`@Published`/`@StateObject`/`@EnvironmentObject` in new code — use `@Observable`. Never perform I/O inside `body` — use `.task {}`. Use `LazyVStack`/`LazyHStack` for large collections. Conform expensive views to `Equatable` to skip unnecessary re-renders. Never use `AnyView` — prefer `@ViewBuilder` or `Group`.
