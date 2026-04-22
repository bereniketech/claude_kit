---
name: cpp-patterns
description: Modern C++ (C++17/20) patterns covering RAII, smart pointers, move semantics, templates/concepts, concurrency, and testing with GoogleTest/CMake.
---

# C++ Patterns

Production-grade modern C++ patterns based on the C++ Core Guidelines — RAII, type safety, immutability, templates with concepts, safe concurrency, and a complete testing workflow.

---

## 1. Core Principles

Apply these cross-cutting rules to all C++ code:

1. **RAII everywhere** — bind resource lifetime to object lifetime (P.8, R.1)
2. **Immutability by default** — start with `const`/`constexpr`; mutability is the exception (P.10, Con.1-5)
3. **Type safety** — use the type system to prevent errors at compile time (P.4, I.4)
4. **Value semantics** — prefer returning by value and scoped objects over pointer passing (R.3-5, F.20)
5. **Express intent** — names, types, and concepts should communicate purpose (P.3, F.1)

Naming conventions (underscore_style):

```cpp
namespace my_project {
    constexpr int max_buffer_size = 4096;  // not ALL_CAPS — not a macro
    class tcp_connection { ... };          // underscore_style class
    std::string host_;                     // trailing underscore for members
}
```

**Rule:** Never use `using namespace std;` in headers at global scope (SF.7). Headers must have include guards and be self-contained. Use `#pragma once` or `#ifndef` guards (SF.8, SF.11).

---

## 2. RAII and Resource Management

Never call raw `new`/`delete` or `malloc`/`free`. Use smart pointers for ownership:

```cpp
auto widget = std::make_unique<Widget>("config");  // unique ownership (R.21)
auto cache  = std::make_shared<Cache>(1024);        // shared ownership — only when needed

// Raw pointer = non-owning observer (R.3)
void render(const Widget* w) { if (w) w->draw(); }
render(widget.get());
```

RAII pattern for C resources:

```cpp
class FileHandle {
public:
    explicit FileHandle(const std::string& path)
        : handle_(std::fopen(path.c_str(), "r")) {
        if (!handle_) throw std::runtime_error("Failed to open: " + path);
    }

    ~FileHandle() { if (handle_) std::fclose(handle_); }

    FileHandle(const FileHandle&) = delete;
    FileHandle& operator=(const FileHandle&) = delete;
    FileHandle(FileHandle&& other) noexcept : handle_(std::exchange(other.handle_, nullptr)) {}
    FileHandle& operator=(FileHandle&& other) noexcept {
        if (this != &other) {
            if (handle_) std::fclose(handle_);
            handle_ = std::exchange(other.handle_, nullptr);
        }
        return *this;
    }

private:
    std::FILE* handle_;
};
```

**Rule:** Apply Rule of Zero when the compiler can generate all special members. Apply Rule of Five when you manage a resource manually — define or delete all five (C.20, C.21). Prefer `unique_ptr` over `shared_ptr` unless sharing ownership is required (R.21).

---

## 3. Classes and Type Safety

Use `class` when there is an invariant; `struct` when members vary independently (C.2). Mark single-argument constructors `explicit` (C.46). Use `override` and `final` — never both `virtual` and `override` on the same function (C.128):

```cpp
class Shape {
public:
    virtual ~Shape() = default;          // C.35: public virtual destructor
    virtual double area() const = 0;
};

class Circle : public Shape {
public:
    explicit Circle(double r) : radius_(r) {}
    double area() const override { return 3.14159 * radius_ * radius_; }
private:
    double radius_;
};
```

Prefer `enum class` over plain `enum`; use lowercase enumerator names:

```cpp
enum class LogLevel { debug, info, warning, error };  // Enum.3 + Enum.5
```

Immutability by default:

```cpp
class Sensor {
public:
    explicit Sensor(std::string id) : id_(std::move(id)) {}
    const std::string& id() const { return id_; }   // Con.2: const member function
    void record(double value) { reading_ = value; }  // non-const only when mutation needed
private:
    const std::string id_;     // Con.4: never changes after construction
    double reading_{0.0};      // ES.20: always initialize
};
```

**Rule:** Always initialize objects at declaration (ES.20). Use `{}` initializer syntax (ES.23). Use `nullptr` — never `0` or `NULL` (ES.47). Never C-style casts — use `static_cast`, `const_cast` etc. (ES.48). Never cast away `const` (ES.50).

---

## 4. Functions and Parameter Passing

Follow the parameter passing rules:

```cpp
void print(int x);                        // F.16: cheap types by value
void analyze(const std::string& data);    // F.16: expensive types by const&
void transform(std::string s);            // sink parameter: by value (will move)

// F.20 + F.21: return values, not output parameters
struct ParseResult { std::string token; int position; };
ParseResult parse(std::string_view input);   // GOOD
// void parse(std::string_view, std::string&, int&);  // BAD — avoid output params
```

Use `constexpr` and `noexcept` where applicable:

```cpp
constexpr int factorial(int n) noexcept {  // F.4 + F.6
    return (n <= 1) ? 1 : n * factorial(n - 1);
}
static_assert(factorial(5) == 120);
```

**Rule:** Never return `T&&` from a function (F.45). Never use `va_arg`/C-style variadics (F.55). Prefer pure functions (F.8). Keep functions short and focused — one logical operation (F.2, F.3).

---

## 5. Templates and Concepts (C++20)

Constrain all template parameters with concepts (T.10, T.11):

```cpp
#include <concepts>

// Standard concept
template<std::integral T>
T gcd(T a, T b) {
    while (b != 0) { a = std::exchange(b, a % b); }
    return a;
}

// Shorthand concept syntax (T.13)
void sort(std::ranges::random_access_range auto& range) {
    std::ranges::sort(range);
}

// Custom concept
template<typename T>
concept Serializable = requires(const T& t) {
    { t.serialize() } -> std::convertible_to<std::string>;
};

template<Serializable T>
void save(const T& obj, const std::string& path);
```

Use `using` instead of `typedef` (T.43). Use standard library containers over C arrays (SL.con.1, SL.con.2). Use `std::string_view` for non-owning string parameters (SL.str.2):

```cpp
std::string build_greeting(std::string_view name) {
    return "Hello, " + std::string(name) + "!";
}
std::cout << result << '\n';  // SL.io.50: '\n' not std::endl
```

**Rule:** Prefer `std::vector` by default for sequences. Never specialize function templates — overload instead (T.144). Avoid template metaprogramming where `constexpr` suffices (T.120).

---

## 6. Safe Concurrency

Use RAII locks — always name lock guards (CP.20, CP.44):

```cpp
class ThreadSafeQueue {
public:
    void push(int value) {
        std::lock_guard<std::mutex> lock(mutex_);  // CP.44: named guard
        queue_.push(value);
        cv_.notify_one();
    }

    int pop() {
        std::unique_lock<std::mutex> lock(mutex_);
        cv_.wait(lock, [this] { return !queue_.empty(); });  // CP.42: always wait with condition
        const int value = queue_.front();
        queue_.pop();
        return value;
    }

private:
    std::mutex mutex_;
    std::condition_variable cv_;
    std::queue<int> queue_;
};

// CP.21: std::scoped_lock for multiple mutexes (deadlock-free)
void transfer(Account& from, Account& to, double amount) {
    std::scoped_lock lock(from.mutex_, to.mutex_);
    from.balance_ -= amount;
    to.balance_ += amount;
}
```

**Rule:** Never use `volatile` for synchronization — it is for hardware I/O only (CP.8). Never detach threads (CP.26). Never hold a lock while calling callbacks — deadlock risk (CP.22). Avoid lock-free programming without deep expertise (CP.100). Minimize explicit sharing of writable data (CP.3).

---

## 7. GoogleTest and CMake Testing

TDD loop: RED (failing test) → GREEN (minimal fix) → REFACTOR.

Basic unit test and fixture:

```cpp
#include <gtest/gtest.h>
#include <gmock/gmock.h>

TEST(CalculatorTest, AddsTwoNumbers) {
    EXPECT_EQ(Add(2, 3), 5);
}

class UserStoreTest : public ::testing::Test {
protected:
    void SetUp() override {
        store = std::make_unique<UserStore>(":memory:");
        store->Seed({{"alice"}, {"bob"}});
    }
    std::unique_ptr<UserStore> store;
};

TEST_F(UserStoreTest, FindsExistingUser) {
    auto user = store->Find("alice");
    ASSERT_TRUE(user.has_value());
    EXPECT_EQ(user->name, "alice");
}
```

Mock interactions with GoogleMock:

```cpp
class MockNotifier : public Notifier {
public:
    MOCK_METHOD(void, Send, (const std::string& message), (override));
};

TEST(ServiceTest, SendsNotifications) {
    MockNotifier notifier;
    Service service(notifier);
    EXPECT_CALL(notifier, Send("hello")).Times(1);
    service.Publish("hello");
}
```

CMake setup with FetchContent and `gtest_discover_tests`:

```cmake
cmake_minimum_required(VERSION 3.20)
project(example LANGUAGES CXX)
set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

include(FetchContent)
FetchContent_Declare(googletest
  URL https://github.com/google/googletest/archive/refs/tags/v1.17.0.zip)
FetchContent_MakeAvailable(googletest)

add_executable(example_tests tests/calculator_test.cpp src/calculator.cpp)
target_link_libraries(example_tests GTest::gtest GTest::gmock GTest::gtest_main)

enable_testing()
include(GoogleTest)
gtest_discover_tests(example_tests)
```

```bash
cmake -S . -B build -DCMAKE_BUILD_TYPE=Debug
cmake --build build -j
ctest --test-dir build --output-on-failure

# Coverage (GCC)
cmake -S . -B build-cov -DENABLE_COVERAGE=ON
cmake --build build-cov -j && ctest --test-dir build-cov
lcov --capture --directory build-cov --output-file coverage.info
genhtml coverage.info --output-directory coverage

# Sanitizers
cmake -S . -B build-asan -DENABLE_ASAN=ON && cmake --build build-asan -j
ctest --test-dir build-asan --output-on-failure
```

**Rule:** Use `ASSERT_*` for preconditions, `EXPECT_*` for multiple checks. Prefer dependency injection and fakes over global state. Never use `sleep` for synchronization — use condition variables or latches. Run ASan/UBSan/TSan in CI. Keep tests deterministic and isolated.
