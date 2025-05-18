---
title: "Project Structure"
weight: 6
---

# Project Structure


While Go offers flexibility in how you structure your projects, adhering to a common layout can improve organization and maintainability. A widely adopted project structure includes:

- cmd: Contains the main applications for your project. Each subdirectory typically represents a single executable.

- internal: Contains private code that is not intended for use outside of your project.

- pkg: Contains reusable libraries that can be used by other projects.

Other common directories might include build for build scripts, examples for example code, etc. 

Keep in mind there is no official standard Go project layout. You can keep it simpler.

- [Read about Organizing a Go module
](https://go.dev/doc/modules/layout)
- [Read about Organizing Go code](https://go.dev/blog/organizing-go-code)