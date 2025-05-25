# Testing in Go

Go has built-in support for testing, making it easy to write and run unit tests, benchmarks, and example tests. The `go test` command automates the execution of tests.

## The `testing` Package

The core of Go's testing infrastructure is the `testing` package.

**Key Conventions:**
- Test files are named `*_test.go` (e.g., `myapp_test.go`).
- Test files reside in the same package as the code they are testing.
- Test functions are named `TestXxx` where `Xxx` starts with an uppercase letter and describes the test (e.g., `TestMyFunction`).
- Test functions take one argument: `t *testing.T`.

**Basic Unit Test Example:**

Suppose you have a function in `math.go`:
```go
// In math.go
package mymath

func Add(a, b int) int {
    return a + b
}
```

Your test file `math_test.go` would look like this:
```go
// In math_test.go
package mymath

import "testing"

func TestAdd(t *testing.T) {
    result := Add(2, 3)
    expected := 5
    if result != expected {
        t.Errorf("Add(2, 3) = %d; want %d", result, expected)
    }

    result = Add(-1, 1)
    expected = 0
    if result != expected {
        t.Errorf("Add(-1, 1) = %d; want %d", result, expected)
    }
}
```

**Running Tests:**
- `go test`: Runs all tests in the current package.
- `go test ./...`: Runs all tests in the current directory and its subdirectories.
- `go test -v`: Verbose output, shows individual test results.
- `go test -run TestAdd`: Runs only tests matching the regex (e.g., `TestAdd` function).
- `go test -cover`: Shows test coverage.
- `go test -coverprofile=coverage.out && go tool cover -html=coverage.out`: Generates an HTML coverage report.

**Common `*testing.T` Methods:**
- `t.Errorf(format string, args ...interface{})`: Reports a test failure and continues execution.
- `t.Fatalf(format string, args ...interface{})`: Reports a test failure and stops execution of that test function.
- `t.Logf(format string, args ...interface{})`: Logs information (useful for debugging, only shown with `-v`).
- `t.SkipNow()`: Skips the current test.
- `t.Helper()`: Marks the calling function as a test helper function. When `t.Errorf` or `t.Fatalf` is called from a helper, the line number reported will be from the calling test function, not the helper.

## Table-Driven Tests

Table-driven tests are a common and effective way to test multiple scenarios with the same test logic.

```go
func TestAddTableDriven(t *testing.T) {
    tests := []struct {
        name   string
        a, b   int
        want int
    }{
        {"positive numbers", 2, 3, 5},
        {"negative and positive", -1, 1, 0},
        {"both negative", -2, -3, -5},
        {"zero values", 0, 0, 0},
    }

    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) { // t.Run creates subtests
            ans := Add(tt.a, tt.b)
            if ans != tt.want {
                t.Errorf("Add(%d, %d) = %d; want %d", tt.a, tt.b, ans, tt.want)
            }
        })
    }
}
```
Using `t.Run` creates subtests, which provides better organization and allows for more granular control (e.g., `go test -run TestAddTableDriven/positive_numbers`).

## Example Tests (`ExampleXxx`)

Example functions serve as both documentation and runnable tests. They are named `ExampleXxx` and reside in `*_test.go` files.
- If an example function includes an "Output:" comment at the end, `go test` will execute the function and compare its standard output with the comment.

```go
// In math_test.go
package mymath

import "fmt"

func ExampleAdd() {
    sum := Add(5, 10)
    fmt.Println(sum)
    // Output: 15
}

func ExampleAdd_negative() {
    sum := Add(-5, -10)
    fmt.Println(sum)
    // Output: -15
}
```
These examples will appear in the `godoc` documentation.

## Benchmarking (`BenchmarkXxx`)

Benchmark functions measure the performance of your code.
- Named `BenchmarkXxx`.
- Take one argument: `b *testing.B`.
- The code to be benchmarked is run inside a loop `for i := 0; i < b.N; i++`. `b.N` is adjusted by `go test` until the benchmark runs for a stable amount of time.

```go
// In math_test.go
package mymath

import "testing"

func BenchmarkAdd(b *testing.B) {
    for i := 0; i < b.N; i++ {
        Add(100, 200)
    }
}
```
**Running Benchmarks:**
- `go test -bench=.`: Runs all benchmarks in the current package.
- `go test -bench=Add`: Runs benchmarks matching the regex.
- `go test -benchmem`: Shows memory allocation statistics.

## Test Setup and Teardown

- **`TestMain(m *testing.M)`:** A special function that can be used for setup and teardown for all tests in a package.

```go
func TestMain(m *testing.M) {
    fmt.Println("Setting up tests for package mymath...")
    // Perform setup here (e.g., connect to a test database)

    exitCode := m.Run() // Run all other tests

    fmt.Println("Tearing down tests for package mymath...")
    // Perform teardown here (e.g., disconnect from test database)

    os.Exit(exitCode)
}
```
- For individual test setup/teardown, you can use helper functions or struct methods if your tests are more complex.

## Third-Party Testing Libraries and Tools

While Go's built-in `testing` package is powerful, the ecosystem offers additional tools:

- **`testify` ([github.com/stretchr/testify](https://github.com/stretchr/testify)):** A popular assertion library (`testify/assert`, `testify/require`) and mocking library (`testify/mock`).
  ```go
  // Example with testify/assert
  // import "github.com/stretchr/testify/assert"
  // func TestAddWithAssert(t *testing.T) {
  //     assert.Equal(t, 5, Add(2, 3), "2 + 3 should be 5")
  // }
  ```
- **`gomock` ([github.com/golang/mock](https://github.com/golang/mock)):** A mocking framework from Google.
- **`ginkgo` and `gomega` ([onsi.github.io/ginkgo](https://onsi.github.io/ginkgo/)):** For Behavior-Driven Development (BDD) style tests.
- **`httptest` ([pkg.go.dev/net/http/httptest](https://pkg.go.dev/net/http/httptest)):** Part of the standard library, useful for testing HTTP clients and servers.
- **`go-fuzz` ([github.com/dvyukov/go-fuzz](https://github.com/dvyukov/go-fuzz)):** Fuzz testing tool. Go also has native fuzzing support since Go 1.18.

Effective testing is a cornerstone of robust Go development. The built-in tools provide a solid foundation, and the community offers further enhancements for more specialized testing needs.
