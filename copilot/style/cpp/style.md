
# C++ Coding Standards - Condensed Guide for AI

## 1. General Principles

- **Express intent directly in code** – Use clear types, names, and standard library facilities rather than low-level constructs.
- **Prefer compile-time checking to run-time checking** – Use `static_assert`, `constexpr`, and strong types.
- **Don't leak resources** – Use RAII (Resource Acquisition Is Initialization) to manage all resources.
- **Optimize for the reader, not the writer** – Code is read more often than written.
- **Follow the golden rule**: When modifying existing code, match the existing style.

---

## 2. Naming Conventions

### LLVM Style

- **Types** (classes, structs, enums): Start with uppercase, camel case (`TextFileReader`).
- **Variables**: Lowercase with underscores (`table_name`).
- **Functions**: Verb phrases, camel case starting with lowercase (`openFile()`, `isFoo()`).
- **Constants**: `k` prefix followed by camel case (`kDaysInAWeek`).
- **Enumerators**: Start with uppercase, prefix with enum name if not in own namespace (`VK_Argument`).

### Google Style

- **Types** (classes, structs, enums): Camel case starting with uppercase (`MyExcitingClass`).
- **Variables**: Lowercase with underscores (`table_name`).
- **Functions**: Camel case starting with lowercase (`openFile()`, `isFoo()`).
- **Constants**: `k` prefix followed by camel case (`kDaysInAWeek`).
- **Enumerators**: Start with uppercase, prefix with enum name if not in own namespace (`VK_Argument`).

### Core Guidelines

- Use descriptive names; avoid abbreviations unless well-known.
- Make names match semantics and role.
- Use consistent capitalization.

---

## 3. File and Project Structure

### Headers

- **Self-contained headers**: Every header must compile on its own with header guards.
- **Header guards format**:
  - LLVM: `LLVM_PATH_TO_FILE_H` (all caps, path-based).
  - Google: `PROJECT_PATH_FILE_H_` (trailing underscore).
- **Include order** (LLVM & Google):
  1. Related/main header (for `.cpp` files).
  2. Local/private headers.
  3. Project headers.
  4. C system headers.
  5. C++ standard library headers.
  6. Other libraries' headers.
- **Minimize includes**: Use forward declarations where possible; include only what you use.
- **Keep internal headers private**: Don't expose implementation details in public headers.

### Source Files

- **File naming**: Use `.h` for headers, `.cpp` for implementations, `.inc` for non-self-contained includes (use sparingly).
- **One class per file** (typically): Exceptions for closely related helper classes.

---

## 4. Formatting and Style

### Line Width

- **80 columns** (LLVM, Google): Fit code within 80 columns to allow side-by-side viewing.

### Indentation

- **Spaces, not tabs** (all guides): Use spaces for indentation (typically 2 or 4 spaces).
- **No trailing whitespace**.

### Braces

- **Function/class braces**: Opening brace on same line (Google) or new line (LLVM varies by context).
- **Control flow statements**: Always use braces for `if`, `for`, `while`, `do-while` unless the entire statement fits on one line.

  ```cpp
  // Good
  if (condition) {
    DoSomething();
  }

  // OK if very brief
  if (x == kFoo) return new Foo();
  ```

- **No braces for simple single statements** (LLVM): Only if statement is simple, non-nested, and fits on one/two lines.

### Spacing

- **No spaces inside parentheses**: `if (condition)` not `if ( condition )`.
- **Space before open brace**: `if (condition) {`.
- **Space after control keywords**: `if (...)`, `for (...)`, `while (...)`.
- **No space before function call parentheses**: `func()` not `func ()`.
- **Pointer/reference alignment**: Attach to type like `type* var`, not to variable.

  ```cpp
  int* ptr;  // Good
  int *ptr;  // Bad
  ```

### Comments

- **Use C++-style comments** (`//`) for most cases.
- **Doxygen for documentation**: Use `///` for documentation comments.
  - `\brief` for summary, `\param` for parameters, `\returns` for return value.
- **File headers**: Include license, brief description, and reference to paper/algorithm if relevant.
- **Comment the "why", not the "what"**: Code should be self-documenting; comments explain intent and edge cases.

---

## 5. Language Features and Best Practices

### Modern C++ Standards

- **Target C++20** : Use modern features for safety and clarity.
- **Use standard library features**: Prefer STL and standard algorithms over custom implementations.
- **Avoid vendor-specific extensions** unless necessary and well-encapsulated.

### Types and Constants

- **Prefer `constexpr` over macros** for constants.
- **Use `const` liberally**: On variables, parameters, and methods that don't modify state.
- **Use `constinit` for thread_local/global variables** to enforce constant initialization.
- **Use strong types**: Avoid `void*`, use typed pointers or templates. Avoid `int` for non-numeric data.
- **Avoid magic numbers**: Use named constants or enums.

### Variables

- **Initialize all variables**: Prefer initialization over declaration + assignment.

  ```cpp
  int i = f();  // Good
  int i; i = f();  // Bad
  ```

- **Declare variables in narrowest scope**: Close to first use.
- **Prefer brace initialization** `{}` for clarity and safety (but not for auto with single element).

### Pointers and References

- **Use `nullptr`, not `NULL` or `0`**: For null pointers.
- **Prefer references over pointers** when object cannot be null.
- **Mark non-null pointers**: Use `not_null<T*>` (GSL) or document in comments.
- **Never transfer ownership via raw pointer**: Use `unique_ptr`, `shared_ptr`, or return by value.
  - Use `owner<T*>` (GSL) for legacy code where ownership must be indicated.
- **Pointers vs references**:
  - Reference for non-null, no ownership transfer.
  - Pointer when `nullptr` is valid or for optional parameters.
  - Smart pointers for ownership.

### Resource Management (RAII)

- **Always use RAII**: Acquire resources in constructors, release in destructors.
- **Prefer smart pointers**:
  - `unique_ptr<T>` for exclusive ownership.
  - `shared_ptr<T>` for shared ownership (use sparingly; prefer `unique_ptr`).
  - `weak_ptr<T>` to break cycles.
- **Avoid `new` and `delete`**: Use `make_unique<T>()` and `make_shared<T>()`.
- **No `malloc`/`free`** in C++ code: They don't call constructors/destructors.
- **No static constructors**: Objects with static storage duration should be trivially destructible.
- **Avoid global mutable state**: Use `const` globals or pass by reference.

### Functions

- **Keep functions short** (< 40-50 lines ideal): Break up long functions.
- **Single responsibility**: Each function should do one thing.
- **Minimize arguments** (≤ 4 ideal): Use structs or objects to group related parameters.
- **Prefer return by value**: Use move semantics for large objects. Avoid output parameters.
- **Use `auto` judiciously**: Only when type is obvious or improves readability.
  - Always use `auto&` or `auto*` for references/pointers to avoid unintended copies.
- **Use early returns**: Reduce nesting by returning early on error/edge cases.

  ```cpp
  if (!condition) return;  // Early exit
  // Main logic
  ```

- **Don't use `else` after `return`**: Reduces indentation.

### Classes and Structs

- **Use `struct` for passive data**: No invariants, all members public.
- **Use `class` for objects with invariants**: Enforce invariants via private members and methods.
- **Constructors**:
  - Establish class invariants.
  - Use `explicit` for single-argument constructors (prevents implicit conversions).
  - Prefer in-class member initializers.
  - Use delegating constructors to avoid duplication.
- **Destructors**:
  - Must never fail (no exceptions).
  - Virtual destructor for base classes with virtual methods.
- **Rule of Five/Zero**:
  - If you define one of {destructor, copy constructor, copy assignment, move constructor, move assignment}, consider defining all five or zero (rely on defaults).
  - Prefer `= default` or `= delete` over custom implementations when possible.
- **Copy/Move semantics**:
  - Copyable types: Declare copy constructor and assignment operator.
  - Move-only types: Delete copy operations, declare move operations.
  - Neither: Delete all copy and move operations.
  - Make intent explicit in public section.

### Interfaces

- **Make interfaces explicit**: Avoid hidden dependencies via global state.
- **State preconditions and postconditions**: Use `Expects()` and `Ensures()` from GSL, or comments.
- **Prefer `span<T>` over pointer + size**: Encapsulates array and length.
- **Use exceptions for errors** (or error codes if exceptions unavailable): Don't use return codes for normal control flow.

### Error Handling

- **Use exceptions for exceptional conditions** (Core Guidelines).
  - LLVM/Google: **Don't use exceptions** (practical constraints in large codebases).
  - If not using exceptions: Use error codes, `optional<T>`, `expected<T, E>`, or assertions.
- **Use `noexcept`**: Mark functions that don't throw (especially move constructors, swap, destructors).
- **Use `assert` liberally**: Check preconditions and invariants. Include descriptive messages.

  ```cpp
  assert(x >= 0 && "x must be non-negative");
  ```

- **Use `llvm_unreachable` (LLVM)** for code paths that should never execute (better than `assert(false)`).

### Templates and Generic Programming

- **Document template parameters with concepts** (C++20): Makes requirements explicit and improves error messages.

  ```cpp
  template<typename T>
    requires std::integral<T>
  T add(T a, T b) { return a + b; }
  ```

- **Avoid complex template metaprogramming**: Keep templates simple unless necessary.
- **Prefer `auto` for lambdas and deduced types** in generic code.

### Type Casts

- **Avoid C-style casts**: Use C++ casts (`static_cast`, `const_cast`, `reinterpret_cast`, `dynamic_cast`).
- **Prefer `static_cast`** for safe conversions.
- **Avoid `dynamic_cast`** on pointers/references (use virtual functions or visitor pattern instead).
- **Use `const_cast` sparingly**: Only when interfacing with const-incorrect APIs.

### Enums

- **Prefer `enum class` over plain `enum`**: Strongly typed and scoped.

  ```cpp
  enum class Color { Red, Green, Blue };  // Good
  enum Color { Red, Green, Blue };         // Avoid
  ```

- **Name enum types with `Kind` suffix** if used as discriminator (LLVM).

### Inheritance

- **Prefer composition over inheritance**: Use inheritance for "is-a" relationships only.
- **Use `public` inheritance**: Private inheritance is rare; use member variables instead.
- **Mark overrides**: Use `override` (or `final`) for virtual function overrides, never `virtual`.

  ```cpp
  void foo() override;  // Good
  virtual void foo();   // Bad for override
  ```

- **Avoid multiple implementation inheritance**: Multiple interface inheritance (pure abstract classes) is OK.

### Operators

- **Overload operators judiciously**: Only when meaning is obvious and consistent.
- **Prefer non-member functions for symmetric operators**: E.g., `operator+`.
- **Don't overload `&&`, `||`, `,` (comma)**, or unary `&`.
- **Use `operator<=>` (C++20)** for comparison instead of defining all six comparison operators.

### Lambdas

- **Use lambdas where appropriate**: For short, local functions.
- **Prefer explicit captures** over default captures when lambda escapes scope.

  ```cpp
  [&foo] { ... }  // Explicit capture
  [&] { ... }     // Default by reference (use only for short-lived lambdas)
  ```

- **Avoid capturing `this` by value `[=]`** in non-static member functions: Captures `this` pointer, not the object.

### Concurrency

- **Avoid data races**: Don't access mutable non-local data from multiple threads without synchronization.
- **Prefer `thread_local` over other thread-local mechanisms**: Use `constinit` for non-function-local `thread_local`.
- **Use standard library concurrency primitives**: `std::mutex`, `std::lock_guard`, `std::atomic`, etc.

### Miscellaneous

- **Use range-based `for` loops** where possible:

  ```cpp
  for (const auto& elem : container) { ... }
  ```

- **Prefer preincrement `++i`** over postincrement `i++`: More efficient for non-primitive types.
- **Don't use `using namespace std`**: Pollutes namespace. Use explicit `std::` prefix or specific `using` declarations.
- **Provide a virtual method anchor** (LLVM): At least one out-of-line virtual method if class has vtable (to avoid vtable duplication).
- **Avoid `iostream`** in library headers (LLVM): Uses static constructors. Prefer `raw_ostream` (LLVM) or other alternatives.
- **Use `std::endl` sparingly**: Flushes stream; prefer `'\n'` if flush not needed.
- **Don't use `inline` keyword in class definitions**: Member functions defined in class are implicitly inline.

---

## 6. Safety and Correctness

- **No undefined behavior**: Avoid null pointer dereference, out-of-bounds access, use-after-free, data races.
- **Use `span<T>` for array parameters**: Prevents decay to pointer and loss of size information.
- **Check array bounds**: Use `.at()` for checked access or range-based `for`.
- **Avoid pointer arithmetic**: Use iterators or `span`.
- **Handle all error cases**: Don't ignore return values (especially from I/O and allocation functions).
- **Test for null before dereferencing**: Or use `not_null<T*>`.
- **Avoid signed/unsigned mix**: Prefer same signedness; use `gsl::index` or `std::size_t` for indices.

---

## 7. Performance

- **Don't optimize prematurely**: Write clear, correct code first. Measure before optimizing.
- **Avoid unnecessary copies**: Use move semantics, pass by const reference, or `std::move` for transfer.
- **Reserve container capacity** if size is known: E.g., `vector::reserve()`.
- **Prefer stack allocation** over heap: Faster and automatically managed.
- **Inline small, frequently-called functions**: But let compiler decide (don't overuse `inline` keyword).
- **Don't evaluate `end()` in loop condition**: Cache it before the loop.

  ```cpp
  for (auto it = v.begin(), e = v.end(); it != e; ++it) { ... }
  ```

---

## 8. Code Organization

### Namespaces

- **Use namespaces to avoid global scope pollution**: Especially in libraries.
- **Name namespaces after project/module**: Use nested namespaces for hierarchy.

  ```cpp
  namespace myproject::mymodule { ... }
  ```

- **Don't indent namespace contents** (LLVM): Saves indentation levels.
- **Use anonymous namespaces** for internal linkage in `.cpp` files: Prefer over `static`.
  - Keep anonymous namespaces small; use for class declarations only (LLVM).
- **Use `namespace::qualifiers`** in definitions: Don't reopen namespaces.

  ```cpp
  void myproject::MyClass::foo() { ... }  // Good
  namespace myproject { void MyClass::foo() { ... } }  // Avoid
  ```

### Access Control

- **Make data members `private`**: Provide accessors if needed.
- **Use `protected` sparingly**: Prefer `private` + `protected` accessors.
- **Declare `public`, `protected`, `private` in that order**: In class definitions.

### Declaration Order (Google)

Within each section (`public`, `protected`, `private`):

1. Types and type aliases
2. Static constants
3. Factory functions
4. Constructors and assignment operators
5. Destructor
6. All other methods
7. Data members

---

## 9. Documentation

- **Document all public interfaces**: Classes, functions, and complex algorithms.
- **Use Doxygen-style comments**: `///` with `\brief`, `\param`, `\return`, `\note`, etc.
- **Explain the "why" and edge cases**: Not the "what" (code should be self-explanatory).
- **Document preconditions and postconditions**: Use `\pre` and `\post`, or `Expects()`/`Ensures()`.
- **Document ownership and lifetime**: Especially for pointers and resources.
- **Keep documentation up-to-date**: Update comments when code changes.

---

## 10. Testing and Tools

- **Write unit tests**: For all non-trivial functionality.
- **Use static analysis tools**: `clang-tidy`, `cppcheck`, `cpplint`, etc.
- **Use sanitizers**: AddressSanitizer, ThreadSanitizer, UndefinedBehaviorSanitizer.
- **Format code automatically**: Use `clang-format` with project-specific `.clang-format` file.
- **Enable compiler warnings**: Treat warnings as errors (`-Werror`, `/WX`).

---

## 11. Specific Restrictions and Exceptions

### LLVM-Specific

- **No RTTI or exceptions**: Disabled in LLVM. Use `isa<>`, `cast<>`, `dyn_cast<>` for type queries.
- **Use LLVM data structures**: Prefer `llvm::SmallVector`, `llvm::DenseMap` over STL equivalents when appropriate.
- **Use `raw_ostream`** instead of `iostream`.
- **Use `llvm::sort`** instead of `std::sort`: Randomizes equal elements in debug builds to catch non-determinism.

### Google-Specific

- **No exceptions**: Exceptions disabled in Google codebase.
- **Ownership via smart pointers**: Use `std::unique_ptr`, `std::shared_ptr`.
- **Avoid non-standard extensions**: Ensure portability.
- **Use Abseil libraries**: For additional utilities beyond standard library.

### Core Guidelines

- **Emphasize safety**: Type safety, bounds safety, lifetime safety.
- **Use GSL (Guidelines Support Library)**: For `span`, `not_null`, `owner`, `Expects`, `Ensures`, etc.
- **Gradual adoption**: Apply rules incrementally to existing codebases.

---

## 12. Common Pitfalls and How to Avoid Them

| Pitfall | Solution |
|---------|----------|
| **Memory leaks** | Use RAII, smart pointers, avoid `new`/`delete`. |
| **Null pointer dereference** | Use `not_null<T*>`, check before deref, or use references. |
| **Out-of-bounds access** | Use `span`, `.at()`, range-based `for`. |
| **Use-after-free** | Follow ownership rules, use smart pointers. |
| **Data races** | Use mutexes, atomics, or immutable data. |
| **Implicit conversions** | Use `explicit` constructors and conversion operators. |
| **Object slicing** | Use pointers/references for polymorphism, avoid copying base. |
| **Dangling references** | Don't return references to local variables or temporaries. |
| **Signed/unsigned mismatch** | Use consistent types, cast explicitly when necessary. |
| **Integer overflow** | Use larger types, check bounds, or use checked arithmetic. |
| **Undefined initialization order** | Avoid complex initialization of globals/statics; use `constexpr`. |
| **Unnecessary copies** | Pass by const reference, use move semantics. |
| **Forgetting `virtual` destructor** | Always virtual for base classes with virtual methods. |
| **Forgetting `override`** | Always mark overrides to catch errors. |
| **Static constructor bloat** | Avoid global objects with non-trivial constructors. |

---

## 13. Quick Reference Cheat Sheet

### Do

- Use RAII for all resource management
- Use smart pointers (`unique_ptr`, `shared_ptr`)
- Use `const` everywhere applicable
- Use `auto` when type is obvious or improves readability
- Use range-based `for` loops
- Use `enum class` instead of plain `enum`
- Use `override` for virtual function overrides
- Use `= default` and `= delete` for special members
- Use `nullptr` instead of `NULL` or `0`
- Use `static_assert` for compile-time checks
- Use early returns to reduce nesting
- Initialize all variables at declaration
- Prefer stack allocation over heap
- Use `explicit` for single-argument constructors
- Mark non-throwing functions `noexcept`

### Don't

- Don't use `new` and `delete` directly
- Don't use raw pointers for ownership
- Don't use C-style casts
- Don't use `goto`
- Don't use macros for constants or functions
- Don't use `using namespace` in headers
- Don't use global mutable state
- Don't use exceptions in LLVM/Google codebases
- Don't use RTTI (`dynamic_cast`, `typeid`) in LLVM
- Don't leak resources
- Don't ignore return values (especially errors)
- Don't have naked array parameters (use `span`)
- Don't use `malloc`/`free`
- Don't return references/pointers to local variables
- Don't mix signed and unsigned arithmetic carelessly

---

## 14. Example: Good vs. Bad Code

### Bad Example

```cpp
// Bad: many issues
void processData(int* data, int size) {  // Raw pointer + separate size
    if (data == NULL) return;  // NULL instead of nullptr
    for (int i = 0; i < size; i++) {  // C-style loop, postincrement
        data[i] = data[i] * 2;  // Unchecked access
    }
    delete[] data;  // Manual delete, unclear ownership
}

void use() {
    int* arr = new int[100];  // Manual new
    processData(arr, 100);  // Ownership unclear, potential double-delete
}
```

### Good Example

```cpp
// Good: modern C++ with clear ownership
void processData(span<int> data) {  // span encapsulates pointer + size
    for (auto& value : data) {  // Range-based for, preincrement implicit
        value *= 2;  // Clear, safe access
    }
    // No manual delete needed
}

void use() {
    auto arr = make_unique<int[]>(100);  // Smart pointer, clear ownership
    processData(span<int>(arr.get(), 100));  // Explicit span
    // Automatic cleanup when arr goes out of scope
}
```

---

## 15. Conclusion

This condensed guide combines rules from LLVM, Google, and C++ Core Guidelines. Key themes:

1. **Safety first**: RAII, smart pointers, strong types, bounds checking.
2. **Clarity**: Expressive names, clear interfaces, minimal nesting.
3. **Modern C++**: Use C++17/20 features; avoid C-style constructs.
4. **Performance**: Write clear code first, optimize based on measurements.
5. **Consistency**: Follow existing codebase style; use automated tools.

Adapt these rules to your project's specific needs, constraints, and coding standards.
