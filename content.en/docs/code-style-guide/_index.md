---
title: "Code Style Guide"
weight: 10
bookFlatSection: true
---

# Code Style Guide

The Go community is large and diverse, there are many codebases adopt several of style guide. It's also fine continue to use your own style consistently. Adhering to a common set of guidelines ensures that your codebase is readable by everyone involved. 

The [Go Style Guide](https://google.github.io/styleguide/go/guide) of google is a good starting place or reference. It privide a few overarching principles that summarize how to think about writing readable Go code. 

Ideally, you should write Go code that adheres a well-known standard.

- [Read about Go Style Best Practices](https://google.github.io/styleguide/go/best-practices).

- [Read about Uber Go Style Guide
](https://github.com/uber-go/guide/blob/master/style.md)

Still doubt, learn more about it.

- [Read about Go Style Decisions](https://google.github.io/styleguide/go/decisions)

- [Read about Go Code Review Comments](https://go.dev/wiki/CodeReviewComments)

- [Read about FAQ](https://groups.google.com/g/golang-nuts/c/8_X8O3k62pI)

- [Read about Clean Go Code](https://github.com/Pungyeon/clean-go-article)

You can fix the majority of code style issues by using the following tools:

- All-in-one [golangci-lint](https://github.com/golangci/golangci-lint), Fast linters runners for Go. Bundle of `gofmt`, `govet`, `errcheck`, `staticcheck`, `revive` and many other linters.
    ```
    golangci-lint run file.go
    ```

- The standard library [gofmt](https://pkg.go.dev/cmd/gofmt) which almost all Go code in the wild used.

- An alternative is to use [goimports](https://pkg.go.dev/golang.org/x/tools/cmd/goimports), a superset of gofmt which additionally adds (and removes) import lines as necessary.

Finally, a good supplementary resource for writing clean Go code is [Clean Go Code](https://github.com/Pungyeon/clean-go-article).


Refer to [Go Wiki: CodeTools
](https://go.dev/wiki/CodeTools)