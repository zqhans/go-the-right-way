---
title: "Templating"
weight: 70
bookFlatSection: true
# bookToc: true
# bookHidden: false
# bookCollapseSection: false
# bookComments: false
# bookSearchExclude: false
---

# Templating

**Notice:** generated with AI (google-labs-jules)

Go provides two primary packages for templating: `text/template` for generating any kind of textual output, and `html/template` for generating HTML output that is safe against code injection. The `html/template` package is generally preferred for web applications due to its security features.

## `html/template` Package

The `html/template` package is designed to produce HTML output that is safe against common vulnerabilities like Cross-Site Scripting (XSS). It achieves this through context-aware escaping, meaning it understands the HTML structure and applies appropriate escaping rules based on where the data is being injected.

**Key Features:**
- **Context-Aware Escaping:** Automatically escapes data based on the HTML context (e.g., within HTML text, an HTML attribute, a URL, or JavaScript).
- **Actions:** Control structures like `{{if .Condition}}...{{end}}`, `{{range .Items}}...{{end}}`, `{{with .Data}}...{{end}}`.
- **Pipelines:** Data can be piped through functions: `{{.Data | printf "%.2f"}}`.
- **Variables:** Define and use variables within templates: `{{$name := .User.Name}}`.
- **Functions:** Call built-in functions or register custom functions.
- **Template Composition:** Define and use sub-templates (e.g., for headers, footers).

**Basic Example:**

```go
package main

import (
	"html/template"
	"os"
)

type User struct {
	Name string
	Age  int
    IsAdmin bool
    Bio string
}

func main() {
	tpl := `<!DOCTYPE html>
<html>
<head>
    <title>User Profile</title>
</head>
<body>
    <h1>Hello, {{.Name}}!</h1>
    <p>Age: {{.Age}}</p>
    {{if .IsAdmin}}
        <p>You are an administrator.</p>
    {{else}}
        <p>You are a regular user.</p>
    {{end}}
    <p>Bio: {{.Bio}}</p> <!-- .Bio will be HTML escaped -->
</body>
</html>`

	// Parse the template
	t, err := template.New("userProfile").Parse(tpl)
	if err != nil {
		panic(err)
	}

	user := User{
		Name:    "Alice <script>alert('XSS')</script>", // Malicious input
		Age:     30,
        IsAdmin: true,
        Bio:     "Loves <b>Go</b> programming!",
	}

	// Execute the template
	err = t.Execute(os.Stdout, user)
	if err != nil {
		panic(err)
	}
}
```
In the output, `<script>alert('XSS')</script>` in `user.Name` will be escaped to `Alice &lt;script&gt;alert(&#39;XSS&#39;)&lt;/script&gt;`, preventing the XSS attack. The `<b>Go</b>` in `user.Bio` will also be escaped.

**Parsing Templates from Files:**

```go
// Single file
t, err := template.ParseFiles("template.html")

// Multiple files (e.g., for base templates and specific pages)
// The first file passed to ParseFiles is the one Execute will be called on.
// Other files are parsed to be available as associated templates.
t, err := template.ParseFiles("layout.html", "content.html", "header.html")
if err != nil { /* ... */ }
err = t.ExecuteTemplate(os.Stdout, "layout.html", data) // Execute a specific named template
```

**Custom Functions:**

```go
func toUpper(s string) string {
    return strings.ToUpper(s)
}

// ...
t, err := template.New("test").Funcs(template.FuncMap{
    "upper": toUpper,
}).Parse("Hello, {{.Name | upper}}!")
// ...
```

- [Package `html/template` Documentation](https://pkg.go.dev/html/template)

## `text/template` Package

The `text/template` package implements data-driven templates for generating textual output. Its API is very similar to `html/template`, but it **does not perform any automatic escaping**. Therefore, `text/template` should **not** be used for generating HTML that will be rendered in a browser if the data comes from untrusted sources.

It's suitable for:
- Generating plain text emails.
- Configuration files.
- Source code generation.
- Any other text-based format where HTML escaping is not needed or is handled differently.

**Example:**

```go
package main

import (
	"os"
	"text/template"
)

type Config struct {
	Host string
	Port int
}

func main() {
	tpl := `server {
    listen {{.Port}};
    server_name {{.Host}};
}`

	t, err := template.New("nginxConf").Parse(tpl)
	if err != nil {
		panic(err)
	}

	conf := Config{Host: "example.com", Port: 8080}
	err = t.Execute(os.Stdout, conf)
	if err != nil {
		panic(err)
	}
}
```

- [Package `text/template` Documentation](https://pkg.go.dev/text/template)

## Template Composition and Inheritance

A common pattern is to define a base "layout" template and then have specific page templates "embed" themselves into this layout.

**`layout.html`:**
```html
<!DOCTYPE html>
<html>
<head><title>{{template "title" .}}</title></head>
<body>
    <header>{{template "header" .}}</header>
    <main>
        {{template "content" .}}
    </main>
    <footer>{{template "footer" .}}</footer>
</body>
</html>

{{define "title"}}Default Title{{end}}
{{define "header"}}Default Header{{end}}
{{define "footer"}}Default Footer{{end}}
```

**`page.html`:**
```html
{{template "layout.html" .}} {{/* Use layout.html as the base */}}

{{define "title"}}{{.PageTitle}}{{end}}

{{define "header"}}
    <h1>My Page Header</h1>
    <p>Welcome, {{.User.Name}}</p>
{{end}}

{{define "content"}}
    <p>This is the main content for the page.</p>
    <p>Data: {{.SomeData}}</p>
{{end}}
```

**Go Code:**
```go
templates, err := template.ParseFiles("layout.html", "page.html")
if err != nil { /* ... */ }

data := struct {
    PageTitle string
    User      struct{ Name string }
    SomeData  string
}{
    PageTitle: "My Awesome Page",
    User:      struct{ Name string }{"Alice"},
    SomeData:  "Important info here.",
}

// When executing, "layout.html" is the entry point,
// but it will look for defined templates like "title", "content"
// which are provided by page.html (and layout.html itself for defaults).
err = templates.ExecuteTemplate(os.Stdout, "layout.html", data)
if err != nil { /* ... */ }
```
This technique allows you to reuse common HTML structures across multiple pages. The `{{define "name"}}...{{end}}` action creates a named template, and `{{template "name" .pipeline}}` invokes it.

Choosing between `html/template` and `text/template` depends entirely on the type of output you are generating. For web applications, `html/template` is almost always the correct choice for security reasons.
