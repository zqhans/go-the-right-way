# Security in Go

Security is a critical aspect of any application. Go provides tools and features that can help you build secure software, but secure development practices are paramount.

## Web Application Security

When building web applications in Go, consider the OWASP Top 10 and other common web vulnerabilities:

- **SQL Injection:** Prevent by using prepared statements with `database/sql` or ORMs that properly sanitize inputs. Avoid concatenating user input directly into SQL queries.
- **Cross-Site Scripting (XSS):**
    - Use Go's `html/template` package, which provides context-aware automatic escaping for HTML content.
    - Sanitize user input before rendering it if you're not using `html/template` or if you're injecting data into JavaScript contexts. Libraries like [BLUEMonday](https://github.com/microcosm-cc/bluemonday) can help.
- **Cross-Site Request Forgery (CSRF):** Implement CSRF tokens. Libraries like [Gorilla CSRF](https://github.com/gorilla/csrf) or framework-specific middleware can assist.
- **Authentication and Authorization:**
    - Securely store passwords (see Password Hashing below).
    - Implement robust session management.
    - Enforce proper authorization checks for all sensitive operations.
- **Secure Headers:** Use HTTP headers like `Strict-Transport-Security`, `Content-Security-Policy`, `X-Frame-Options`, and `X-Content-Type-Options` to enhance security. Middleware can help set these.
- **Input Validation:** Always validate and sanitize input from users, APIs, and other sources.
- **Dependency Management:** Keep your Go version and third-party libraries up-to-date to patch known vulnerabilities. Use `go list -m -u all` to check for updates and tools like [Nancy](https://github.com/sonatype-nexus-community/nancy) or Snyk to scan for vulnerabilities in dependencies.

- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
- [Go Web Application Security (Presentation Slides)](https://speakerdeck.com/golangnews/go-web-application-security)

## Password Hashing

Never store passwords in plain text. Always hash them using a strong, slow hashing algorithm.

- **`golang.org/x/crypto/bcrypt`:** This package provides functions for hashing passwords with bcrypt, which is a widely accepted and secure algorithm.

   ```go
   import "golang.org/x/crypto/bcrypt"

   // Hash a password
   hashedPassword, err := bcrypt.GenerateFromPassword([]byte("mysecretpassword"), bcrypt.DefaultCost)
   if err != nil {
       // Handle error
   }

   // Store hashedPassword in the database

   // Verify a password
   err = bcrypt.CompareHashAndPassword(hashedPassword, []byte("mysecretpassword"))
   if err == nil {
       // Password matches
   } else if err == bcrypt.ErrMismatchedHashAndPassword {
       // Password does not match
   } else {
       // Other error
   }
   ```
- **Cost Factor:** The `bcrypt.DefaultCost` is usually sufficient. You can adjust the cost factor (higher is slower and more secure, but uses more CPU).

- [Package `golang.org/x/crypto/bcrypt` Documentation](https://pkg.go.dev/golang.org/x/crypto/bcrypt)
- [How to Safely Store a Password](https://codahale.com/how-to-safely-store-a-password/)

## Input Validation and Data Filtering/Sanitization

Treat all external input as untrusted.

- **Validation:** Check if the data conforms to expected formats, types, ranges, and lengths.
    - Use Go's type system to your advantage.
    - Libraries like [go-playground/validator](https://github.com/go-playground/validator) are popular for struct-based validation.
- **Sanitization:** Remove or escape potentially harmful characters from input before using it in different contexts (HTML, SQL, JavaScript, command line).
    - For HTML: `html/template` for automatic escaping, or libraries like BLUEMonday for more complex sanitization.
    - For SQL: Use prepared statements or ORMs that handle sanitization.
    - For JavaScript: Be very careful when injecting Go data into JavaScript. Consider using `json.Marshal` and then carefully handling it on the client-side.
    - For OS Commands: Use `os/exec` and ensure arguments are passed separately, not as part of a single command string. If you must construct a command string, use functions like `exec.Command` with separate arguments or ensure meticulous sanitization.

## Secure Configuration Management

Managing configuration data securely is crucial.

- **Avoid Hardcoding Secrets:** Don't store database credentials, API keys, or other secrets directly in your source code.
- **Environment Variables:** A common practice is to store secrets in environment variables. Libraries like [Viper](https://github.com/spf13/viper) or [godotenv](https://github.com/joho/godotenv) (for local development) can help manage these.
- **Configuration Files:** If using configuration files, ensure they are not checked into version control if they contain secrets. Keep them in a secure location with restricted permissions.
- **Secrets Management Systems:** For more robust solutions, especially in production, use dedicated secrets management tools like [HashiCorp Vault](https://www.vaultproject.io/) or cloud provider solutions (e.g., AWS Secrets Manager, Google Secret Manager).

## Error Handling and Logging

- **Avoid Leaking Sensitive Information:** Be careful not to expose detailed internal error messages or stack traces to users in production. Log them securely for developers.
- **Structured Logging:** Use structured logging (e.g., Go's `slog` package or libraries like `logrus`, `zap`) to include context, making it easier to audit and analyze security-related events.

This is not an exhaustive list, but it covers some of the fundamental security practices for Go development. Always stay updated on the latest security best practices and potential vulnerabilities.
