---
title: "Databases"
weight: 60
bookFlatSection: true
# bookToc: true
# bookHidden: false
# bookCollapseSection: false
# bookComments: false
# bookSearchExclude: false
---

# Databases

**Notice:** generated with AI (google-labs-jules)

Go provides robust support for interacting with databases through its standard library and a rich ecosystem of third-party drivers and Object-Relational Mappers (ORMs).

## The `database/sql` Package

The primary way to interact with SQL databases in Go is through the `database/sql` package. It provides a generic interface around SQL (or SQL-like) databases.

**Key Features:**
- **Driver-Agnostic:** You use the same API to interact with different database systems (MySQL, PostgreSQL, SQLite, SQL Server, etc.). You just need to import the specific database driver.
- **Connection Pooling:** `database/sql` automatically handles connection pooling, which is crucial for performance and resource management.
- **Prepared Statements:** Supports prepared statements to prevent SQL injection vulnerabilities.
- **Transactions:** Provides support for database transactions.

**Basic Workflow:**
1. **Import `database/sql` and the specific driver:**
   ```go
   import (
       "database/sql"
       _ "github.com/lib/pq" // PostgreSQL driver example
       // _ "github.com/go-sql-driver/mysql" // MySQL driver example
   )
   ```
   The underscore `_` before the driver import is used because you only need to register the driver with `database/sql` (via its `init` function), not use any of its exported names directly.

2. **Open a database connection:**
   ```go
   db, err := sql.Open("postgres", "user=pqgotest dbname=pqgotest sslmode=verify-full")
   if err != nil {
       log.Fatal(err)
   }
   defer db.Close() // Important to close the DB when done

   err = db.Ping() // Verify the connection is alive
   if err != nil {
       log.Fatal(err)
   }
   ```

3. **Execute Queries:**
   - **`db.QueryRow()`:** For queries that return a single row.
     ```go
     var name string
     err = db.QueryRow("SELECT name FROM users WHERE id = $1", 1).Scan(&name)
     if err != nil {
         // Handle error (sql.ErrNoRows is common here)
     }
     ```
   - **`db.Query()`:** For queries that return multiple rows.
     ```go
     rows, err := db.Query("SELECT id, name FROM users WHERE age > $1", 30)
     if err != nil {
         log.Fatal(err)
     }
     defer rows.Close()

     for rows.Next() {
         var id int
         var name string
         if err := rows.Scan(&id, &name); err != nil {
             log.Fatal(err)
         }
         fmt.Printf("ID: %d, Name: %s
", id, name)
     }
     if err := rows.Err(); err != nil { // Check for errors during iteration
         log.Fatal(err)
     }
     ```
   - **`db.Exec()`:** For `INSERT`, `UPDATE`, `DELETE` statements that don't return rows.
     ```go
     result, err := db.Exec("UPDATE users SET age = $1 WHERE id = $2", 35, 1)
     if err != nil {
         log.Fatal(err)
     }
     rowsAffected, err := result.RowsAffected()
     // LastInsertId might also be available depending on the driver
     ```

- [Package `database/sql` Documentation](https://pkg.go.dev/database/sql)
- [Go by Example: SQL](https://gobyexample.com/sql)
- [Tutorial: Using `database/sql`](http://go-database-sql.org/)

## Popular Database Drivers

You'll need a driver for the specific database you are using:
- **PostgreSQL:** [github.com/lib/pq](https://github.com/lib/pq), [github.com/jackc/pgx](https://github.com/jackc/pgx) (pgx is often preferred for its performance and features)
- **MySQL:** [github.com/go-sql-driver/mysql](https://github.com/go-sql-driver/mysql)
- **SQLite:** [github.com/mattn/go-sqlite3](https://github.com/mattn/go-sqlite3)
- **SQL Server:** [github.com/denisenkom/go-mssqldb](https://github.com/denisenkom/go-mssqldb)
- **Oracle:** [github.com/sijms/go-ora](https://github.com/sijms/go-ora) (community driver)

## ORMs and Query Builders

While `database/sql` provides the fundamental interface, some developers prefer higher-level abstractions like Object-Relational Mappers (ORMs) or query builders.

**Benefits:**
- Can reduce boilerplate code for common CRUD (Create, Read, Update, Delete) operations.
- May offer more convenient ways to map Go structs to database tables.
- Query builders provide a programmatic way to construct SQL queries.

**Considerations:**
- Can add a layer of complexity and potentially hide performance issues.
- Might have a steeper learning curve than `database/sql`.
- "Leaky abstractions" can sometimes occur, requiring you to understand the underlying SQL anyway.

**Popular Libraries:**

- **GORM ([github.com/go-gorm/gorm](https://github.com/go-gorm/gorm)):** A full-featured ORM with a lot of community support. Supports associations, transactions, migrations, etc.
- **sqlx ([github.com/jmoiron/sqlx](https://github.com/jmoiron/sqlx)):** A set of extensions on top of `database/sql`. It doesn't try to be a full ORM but provides convenient ways to map query results to structs, work with named parameters, and more. Many find it a good balance between raw `database/sql` and a full ORM.
- **SQLBoiler ([github.com/volatiletech/sqlboiler](https://github.com/volatiletech/sqlboiler)):** A "reverse ORM". It generates Go code from your database schema, providing strongly-typed data models and query methods.
- **sqlc ([github.com/kyleconroy/sqlc](https://github.com/kyleconroy/sqlc)):** Generates type-safe Go code from SQL queries. You write SQL, and sqlc generates Go functions and structs for you. This approach often appeals to those who want to write SQL but get the benefits of Go's type safety.
- **Ent ([entgo.io](https://entgo.io/)):** An entity framework for Go, developed at Facebook. It uses a code-generation approach based on defining your schema in Go code.

Choosing whether to use `database/sql` directly or an abstraction layer depends on the project's needs, team familiarity, and performance requirements.
