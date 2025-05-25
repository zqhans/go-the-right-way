# Errors and Panics in Go

Go has a distinct approach to error handling that differs significantly from languages that rely heavily on exceptions. In Go, errors are values, and explicit error checking is a fundamental part of the language's philosophy. Panics are reserved for truly exceptional, unrecoverable situations.

## Errors as Values

In Go, functions that can fail return an `error` as their last return value. An `error` is a built-in interface type:

```go
type error interface {
    Error() string
}
```
Any type that implements this `Error() string` method satisfies the `error` interface.

**Idiomatic Error Handling:**

The standard way to handle errors is to check the returned `error` value immediately after calling a function:

```go
import (
    "fmt"
    "os"
)

func main() {
    file, err := os.Open("non_existent_file.txt")
    if err != nil {
        fmt.Println("Error opening file:", err)
        // Decide how to proceed: log, return, exit, etc.
        return
    }
    defer file.Close()
    // ... work with the file
}
```

**Creating Errors:**

1.  **`errors.New(text string) error`:**
    The `errors` package provides a simple way to create an error with a static error message.

    ```go
    import "errors"

    var ErrDivideByZero = errors.New("division by zero")

    func Divide(a, b int) (int, error) {
        if b == 0 {
            return 0, ErrDivideByZero
        }
        return a / b, nil
    }
    ```

2.  **`fmt.Errorf(format string, a ...interface{}) error`:**
    For creating errors with dynamic content, `fmt.Errorf` formats an error message just like `fmt.Printf`.

    ```go
    import "fmt"

    func ReadData(id string) ([]byte, error) {
        // ... attempt to read data ...
        if dataNotFound { // some condition
            return nil, fmt.Errorf("data not found for id %s", id)
        }
        // ...
    }
    ```

**Error Wrapping and Inspection (Go 1.13+):**

Go 1.13 introduced more robust ways to inspect and wrap errors, allowing you to provide more context without losing the original error information.

1.  **Wrapping with `%w`:**
    When using `fmt.Errorf`, the `%w` verb wraps an error, creating a new error that contains the original.

    ```go
    func processFile(path string) error {
        file, err := os.Open(path)
        if err != nil {
            return fmt.Errorf("failed to open file %s: %w", path, err) // Wrap os.Open error
        }
        defer file.Close()
        // ...
        return nil
    }
    ```

2.  **`errors.Is(err error, target error) bool`:**
    Checks if any error in `err`'s chain matches `target`. This is useful for checking against sentinel errors (like `ErrDivideByZero` or `io.EOF`).

    ```go
    if errors.Is(err, os.ErrNotExist) {
        fmt.Println("File does not exist.")
    }
    ```

3.  **`errors.As(err error, target interface{}) bool`:**
    Checks if any error in `err`'s chain matches `target`'s type, and if so, sets `target` to that error value. This is useful for accessing fields of a custom error type.

    ```go
    type MyCustomError struct {
        Msg  string
        Code int
    }

    func (e *MyCustomError) Error() string { return fmt.Sprintf("code %d: %s", e.Code, e.Msg) }

    // ...
    var customErr *MyCustomError
    if errors.As(err, &customErr) {
        fmt.Printf("Custom error occurred: Code %d, Message: %s
", customErr.Code, customErr.Msg)
    }
    ```

**Key Principles of Error Handling:**
- **Check every error:** Don't ignore errors.
- **Return errors to the caller:** Let the calling function decide how to handle the error, unless the current function has enough context to handle it meaningfully.
- **Provide context:** When returning or logging errors, add context to make debugging easier (e.g., what operation failed, what inputs were involved). Error wrapping helps with this.
- **Don't panic on expected errors:** Use `error` values for expected failures (e.g., file not found, network timeout, invalid input).

## Panics

Panics are a mechanism for signaling and handling truly exceptional, unrecoverable errors that should not occur during normal operation. When a panic occurs, the current function's execution stops, any deferred functions are executed, and then the program crashes with a stack trace.

**When Panics Occur:**
- **Runtime errors:** Indexing an array out of bounds, nil pointer dereference, concurrent map writes.
- **Explicitly calling `panic()`:** You can call the built-in `panic()` function with any value (often an error or a string).

```go
func main() {
    mightPanic()
    fmt.Println("This line might not be reached.")
}

func mightPanic() {
    data := []int{1, 2, 3}
    // This will panic: runtime error: index out of range [5] with length 3
    // fmt.Println(data[5])

    // Explicit panic
    panic("something went terribly wrong")
}
```

**`recover()` Mechanism:**

The `recover()` built-in function allows a goroutine to regain control after a panic. `recover()` is only useful inside a deferred function. If the current goroutine is panicking, a call to `recover()` captures the value given to `panic()` and normal execution resumes. If the goroutine is not panicking, `recover()` returns `nil`.

```go
import "fmt"

func main() {
    defer func() {
        if r := recover(); r != nil {
            fmt.Println("Recovered from panic:", r)
        }
    }()

    fmt.Println("Calling riskyFunction...")
    riskyFunction(true)
    fmt.Println("riskyFunction completed (if no panic or panic was recovered).")
    riskyFunction(false) // This call will be made if the first panic is recovered.
    fmt.Println("Program finished normally.")
}

func riskyFunction(shouldPanic bool) {
    fmt.Println("  Entering riskyFunction")
    if shouldPanic {
        panic("  Ah! A panic!")
    }
    fmt.Println("  Exiting riskyFunction normally")
}
```

**Use `panic` and `recover` Sparingly:**
- **Avoid using `panic` for normal error handling.** Errors are the standard way to report expected failures.
- `panic` is appropriate for unrecoverable situations, like an impossible state that indicates a programmer error (e.g., a required initialization step was missed).
- `recover` can be useful in server applications to prevent a single client request from crashing the entire server, or for cleaning up resources before exiting. However, it should not be used to simply ignore or hide bugs.

In summary, Go's `error` values provide a robust and clear way to handle expected issues, while `panic` and `recover` offer a mechanism for dealing with truly exceptional, program-terminating conditions.
