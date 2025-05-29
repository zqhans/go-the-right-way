---
title: "Documenting Your Code"
weight: 130
bookFlatSection: true
# bookToc: true
# bookHidden: false
# bookCollapseSection: false
# bookComments: false
# bookSearchExclude: false
---

# Documenting Your Code

**Notice:** generated with AI (google-labs-jules)

Go has excellent built-in support for documentation through a convention known as "Go Doc" comments and the `go doc` command-line tool (and the historical `godoc` web server). Writing good documentation is crucial for making your code understandable, maintainable, and usable by others (and your future self).

## Go Doc Comments

Go Doc comments are regular Go comments written directly before the declaration of a package, function, type, constant, or variable. They are used to generate human-readable documentation.

**Key Conventions:**

1.  **Package Comments:**
    A comment preceding a `package` clause is a package comment. It should provide a general overview of the package's purpose and functionality. It's good practice to have a package comment in one of the Go files for that package (often in a file named `doc.go` or the main file of the package).

    ```go
    // Package strconv implements conversions to and from string representations
    // of basic data types.
    package strconv
    ```

2.  **Function Comments:**
    Comments for exported functions should start with the name of the function.

    ```go
    // Println formats using the default formats for its operands and writes to standard output.
    // Spaces are always added between operands and a newline is appended.
    // It returns the number of bytes written and any write error encountered.
    func Println(a ...interface{}) (n int, err error) {
        // ...
    }
    ```
    For unexported functions, comments are still useful but not processed by `godoc` for public documentation.

3.  **Type Comments:**
    Comments for exported types should start with the name of the type.

    ```go
    // Reader is the interface that wraps the basic Read method.
    type Reader interface {
        Read(p []byte) (n int, err error)
    }
    ```

4.  **Constant and Variable Comments:**
    Comments for exported constants or variables should start with the name of the const/var. For groups of constants/variables, a single comment before the `const` or `var` keyword can document the group, or individual comments can be used for each.

    ```go
    // ErrNotFound is returned when a requested entity is not found.
    var ErrNotFound = errors.New("entity not found")

    const (
        // Sunday represents the first day of the week.
        Sunday Weekday = iota
        Monday // Monday represents the second day of the week.
        // ...
    )
    ```

5.  **Formatting:**
    - The first sentence of a doc comment is used as a summary. It should be concise.
    - Subsequent paragraphs provide more detail.
    - Code examples can be included and are often very helpful. Indent example code so `godoc` can format it properly.
    - `godoc` will format comments, so you don't need to worry excessively about manual line wrapping for plain text. It respects pre-formatted blocks (like indented code).

6.  **Sections:**
    You can create sections in your documentation using headings (lines starting with `#`). This is less common in `godoc` but can be seen in some larger package docs.

7.  **Deprecation Notices:**
    To mark an API as deprecated, start its doc comment with "Deprecated: " followed by advice on what to use instead or why it's deprecated.

    ```go
    // Deprecated: Use NewShinyFunction instead. OldFunction has known issues.
    func OldFunction() {
        // ...
    }
    ```

## Viewing Documentation

1.  **`go doc` Command-Line Tool:**
    The `go doc` command is the primary way to view documentation in your terminal.

    - `go doc <pkg>`: Show documentation for a package.
      ```bash
      go doc fmt
      go doc github.com/gin-gonic/gin
      ```
    - `go doc <pkg>.<symbol>`: Show documentation for a specific symbol (function, type, var) in a package.
      ```bash
      go doc fmt.Println
      go doc time.Time
      ```
    - `go doc -all <pkg>`: Show all documentation for the package, including unexported symbols (useful for development).
    - `go doc -src <pkg>.<symbol>`: Show the source code along with the documentation.

2.  **`godoc` Web Server (Legacy and Local Use):**
    The `godoc` command (if installed separately or via `go install golang.org/x/tools/cmd/godoc@latest`) can run a local web server that provides documentation for all packages in your `GOPATH`/`GOMODCACHE` and your modules.
    ```bash
    godoc -http=:6060
    ```
    Then open `http://localhost:6060` in your browser. This is very useful for browsing your own private code or code not available on public servers.

3.  **[pkg.go.dev](https://pkg.go.dev/):**
    The official Go package discovery and documentation site hosted by Google. It serves documentation for public Go packages. When you publish a module, its documentation will eventually appear here.

## Best Practices for Writing Documentation

- **Be Clear and Concise:** Write for an audience that may not be familiar with your code.
- **Document Exported APIs:** All exported functions, types, constants, and variables should have doc comments.
- **Provide Examples:** Code examples are often the best way to show how to use an API. The `go test` tool can even run [Example tests](https://blog.golang.org/examples) that are also used as documentation.
- **Keep Documentation Up-to-Date:** Outdated documentation can be worse than no documentation. Update comments when you update code.
- **Explain "Why," Not Just "What":** Good documentation explains the purpose and reasoning behind the code, not just a literal description of what it does.
- **Document Behavior, Especially Edge Cases:** Explain how functions handle errors, boundary conditions, or specific inputs.

Writing good documentation is an investment that pays off significantly in the long run by making your Go code more accessible and maintainable.
