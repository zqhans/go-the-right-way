---
title: "Dependency Injection"
weight: 50
bookFlatSection: true
# bookToc: true
# bookHidden: false
# bookCollapseSection: false
# bookComments: false
# bookSearchExclude: false
---

# Dependency Injection

**Notice:** generated with AI (google-labs-jules)

Dependency Injection (DI) is a design pattern where a component's dependencies are provided to it from an external source, rather than the component creating them itself. This promotes loose coupling, making code more modular, testable, and maintainable.

Go's simplicity and its approach to interfaces lend themselves well to various DI techniques, from manual injection to more automated solutions using tools.

## What is Dependency Injection?

Instead of a component doing this (tight coupling):

```go
type MyService struct {
    // ... other fields
}

func NewMyService() *MyService {
    // MyService creates its own dependency
    db := NewDatabaseConnection("connection_string")
    // ...
    return &MyService{...}
}
```

The dependency is "injected" (passed in):

```go
// Database is an interface
type Database interface {
    Query(query string) (string, error)
}

type MyService struct {
    db Database // MyService depends on an interface
    // ... other fields
}

// NewMyService receives its dependency
func NewMyService(db Database) *MyService {
    return &MyService{db: db}
}

// Usage:
// concreteDB := NewConcreteDatabase("connection_string")
// service := NewMyService(concreteDB)
```

## Benefits of DI in Go

- **Improved Testability:** Dependencies can be easily mocked or faked in unit tests. If `MyService` depends on the `Database` interface, you can pass a mock `Database` implementation during testing.
- **Increased Modularity & Loose Coupling:** Components are less dependent on concrete implementations of other components. You can swap out implementations of dependencies (e.g., change database backends) without changing the components that use them, as long as the new implementation satisfies the required interface.
- **Clearer Dependencies:** It's explicit what a component needs to function.
- **Better Code Organization:** Encourages breaking down applications into smaller, manageable components.

## Manual Dependency Injection Techniques

Go's language features make manual DI straightforward and often sufficient for many projects.

1.  **Constructor Injection:**
    Dependencies are provided through the component's constructor function (e.g., `NewMyService(db Database)` above). This is the most common and generally recommended method.

2.  **Interface Injection:**
    While Go doesn't have explicit setter injection like some OO languages, you can achieve a similar effect by having components satisfy interfaces that define methods for setting dependencies. However, constructor injection is usually preferred for mandatory dependencies.

    ```go
    type DatabaseSetter interface {
        SetDatabase(db Database)
    }

    func (s *MyService) SetDatabase(db Database) {
        s.db = db
    }
    ```
    This is less common for primary dependencies as it can leave an object in an invalid state if the setter is not called.

3.  **Global Variables / Service Locators (Anti-Pattern):**
    Using global variables or a service locator pattern, where components fetch their dependencies from a global registry, is generally discouraged. It obscures dependencies, makes testing harder, and leads to tighter coupling with the global state or locator.

## DI Containers and Tools

For larger applications, managing dependencies manually can become complex. DI containers or tools can automate the process of dependency resolution and injection.

- **[google/wire](https://github.com/google/wire):**
    A compile-time dependency injection tool from Google. Wire analyzes your code and generates plain Go code to wire up dependencies.
    - **Pros:** Compile-time safety (errors caught during compilation, not runtime), generated code is readable Go, no reflection or runtime overhead.
    - **Cons:** Can have a learning curve, requires running a code generation step.

    ```go
    // Example (conceptual, from Wire's docs)
    // file: wire.go (build constraint //go:build wireinject)
    // func InitializeEvent(phrase string) (Event, error) {
    // wire.Build(NewEvent, NewGreeter, NewMessage)
    // return Event{}, nil
    // }

    // After running `wire`, a `wire_gen.go` file is produced.
    ```

- **[uber-go/fx](https://github.com/uber-go/fx):**
    A framework for dependency injection in Go developed by Uber, based on reflection. It's more of an application framework that manages the lifecycle of components.
    - **Pros:** Manages application lifecycle (start/stop), based on providing and invoking functions, good for large applications.
    - **Cons:** Uses reflection (runtime DI), can be more "magical" than Wire.

- **[facebookincubator/inject](https://github.com/facebookincubator/inject):**
    A reflection-based dependency injection library.

**Choosing a DI Tool:**
- For many Go projects, especially smaller to medium-sized ones, **manual DI is often clear, simple, and sufficient.**
- If you find yourself writing a lot of boilerplate for wiring dependencies, or if you want compile-time safety for DI, **`google/wire`** is a strong contender.
- For very large applications where managing the lifecycle of many components is a concern, **`uber-go/fx`** might be suitable.

When considering a DI tool, always weigh its benefits against the added complexity and potential for "magic" it might introduce. Idiomatic Go often favors explicitness.
