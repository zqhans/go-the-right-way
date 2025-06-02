---
title: "Language Highlights"
weight: 20
bookFlatSection: true
---

# Language Highlights

> Go is a general-purpose language designed with systems programming in mind. It is strongly typed and garbage-collected and has explicit support for concurrent programming. Programs are constructed from packages, whose properties allow efficient management of dependencies.

## Programming Paradigms

### Concurrency Programming

Go has unique design with build-in concurrency mechanism. which could reduce to a famous slogan:

> Do not communicate by sharing memory; instead, share memory by communicating.

**Goroutines**: It is a function executing concurrently with other goroutines in the same address space. Goroutines are managed by the Go runtime, not the operating system like traditional OS threads. This make them lightweight, and cheap to create or switch between. Prefix a function or method call with the `go` keyword to run the call in a new goroutine. When the call completes, the goroutine exits, silently.

**Channels**: Channels adheres to the Communicating Sequential Processes (CSP) model. This approach inherently helps avoid common concurrency issues like race conditions. Goroutines communicate and synchronize their executation through channels. Channels are allocated with the build-in function `make`.

**GMP Model(Groutine-Machine-Processor)**: Go's scheduler efficiently maps many goroutines (G) onto a smaller number of OS threads (M) using logical processors (P). This intelligent scheduling, including work-stealing, maximizes CPU utilization and minimizes context-switching overhead.

- [Read About Scalable Go Scheduler Design Doc](https://docs.google.com/document/d/1TTj4T2JO42uD5ID9e89oa0sLKhJYD0Y_kqxDv3I3XMw/edit?tab=t.0#heading=h.mmq8lm48qfcw)
- [Read About Source file src/runtime/proc.go](https://go.dev/src/runtime/proc.go)
- [Read About Go Wiki Learn Concurrency](https://go.dev/wiki/LearnConcurrency)

Refer to [Go Concurrency](https://go.dev/doc/effective_go#concurrency)

### Functional Programming

> Go supports first class functions, higher-order functions, user-defined function types, function literals, closures, and multiple return values.

Function is [first-class citizen](https://en.wikipedia.org/wiki/First-class_citizen) in Go, meaning that a function can be passed as an argument, returned from a function, and assigned to a variable.

- [Read about First-class Functions in Go](https://go.dev/doc/codewalk/functions/)
- [Read about Function literals](https://go.dev/doc/go1.17_spec#Function_literals)

### Object-oriented Programming

Go is not strictly object-oriented in the traditional sense. Go has types and methods and allows an object-oriented style of programming, there is no type inheritance or "implement" declarations.

**No classes, but Structs and Methods**: Go uses structs to group data types and methods that can operate on those structs.
**Interfaces for Polymorphism**: An interface defines a set of methods, and a type automatically satisfies any interface that specifies a subset of its methods.

- [Read about Struct types](https://go.dev/ref/spec#Struct_types)
- [Read about Methods](https://go.dev/doc/effective_go#methods)
- [Read about Interface](https://go.dev/ref/spec#Interface_types)
- [FAQ: Is Go an object-oriented language?](https://go.dev/doc/faq#Is_Go_an_object-oriented_language)

### Meta Programming

Go supports meta-programming through reflection mechanism, and does not have Magic Methods like PHP.
Package `reflect` implements run-time reflection, allowing a program to manipulate objects with arbitary types. Calling `TypeOf` returns the reflection Type that represents the dynamic type of a static type interface{}. Calling `ValueOf` returns the run-time data Value stored in the interface{}.

- [Read abuut The Laws of Reflection](https://go.dev/blog/laws-of-reflection)

## Packages

Go organizes code into packages, which serve as namespaces to prevent naming conflicts.

A package is a collection of related Go files which located at same folder. Every Go source file must belong to a package. 
The first statement in a Go source file must be package name. Constants, types, variables, and functions defined in one source file are accessible to others within the same package. Exported identifiers (which begin with an uppercase letter) can be used by other packages that import the defining package.

Executable commands should always use package main.

- [Read about Packages](https://go.dev/ref/spec#Packages)

## Standard Library

Go provides a rich standard library out of the box. Covering areas such as input/output, networking, concurrency, data structures, algorithms, and etc. Familiar with the standard library is essential for efficient Go development.

- [Standard library](https://pkg.go.dev/std)

## Command Line Interface (CLI)

Go is well-suited for building CLI applications. The standard library provides packages such as `flag` and `os` to facilitate parsing command-line parameters and interacting with the operating system. This makes it straightforward to create powerful and portable command-line tools.

- [Read about Command-line Interfaces](https://go.dev/solutions/clis)

## Debugging

Most Go IDEs, such as VS Code and GoLand, offer integrated debugging capabilities, allowing you to step through code, set breakpoints, inspect variables, and analyze call stacks. The `delve` debugger is a popular command-line debugger for Go. 

- [Read about Debugging](https://go.dev/doc/diagnostics#debugging)
