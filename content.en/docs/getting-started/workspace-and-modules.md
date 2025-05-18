---
title: "Workspace and Modules"
weight: 4
---

# Workspace and Modules

Modern Go development relies on the module system, which was introduced in Go 1.11. Modules replace the traditional GOPATH-based workspace and provide better dependency management.

A module is a collection of Go packages stored in a file tree with a go.mod file at its root, the go.mod file declares the module path, which serves as the import path prefix for all packages within the module. This file also tracks your project's dependencies on other modules.

- [Read about Using Go Mudules](https://go.dev/blog/using-go-modules)
- [Read about Go Modules Reference](https://go.dev/ref/mod)
- [Read about Managing Dependencies](https://go.dev/doc/modules/managing-dependencies)
