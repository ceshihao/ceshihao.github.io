---
title: Prepare Statements in Golang MySQL Driver
date: 2018-01-10 16:27:09
tags: [golang, mysql-driver, prepare-statement]
---

[`go-sql-driver/mysql`](https://github.com/go-sql-driver/mysql) has two kinds of functions `Query()` and `Exec()`.
I would like to see how `Query()` works.

# Two Modes

`Query(query string, args ...interface{})` function has two modes according to whether there are `args`.

## Plaintext Mode

If `Query(query)` is called without `args`, I call it 'Plantext Mode'.

In this mode, driver does NOT do anything on the query string, and just send it directly to MySQL server.

## Interpolation Mode

If there are some placeholders in query string (i.e. `?` in MySQL) and some `args` are passed in to interpolate, I call it 'Interpolation Mode'.

In this mode, driver actually does 3 actions

* Prepare a statement.

* Execute the prepared statement using given `args`.

* Close the prepared statement.

That is exactly the slogan of prepared statement `Prepare Once, Execute Many`.

# Difference

## SQL Injection

Assume you have a table named `prepare`

| id | name  |
|----|-------|
| 1  | name1 |
| 2  | name2 |
| 3  | name3 |

A SQL can be run `select id, name from prepare where id = 1;` on this table.

It returns

| id | name  |
|----|-------|
| 1  | name1 |

Everything is fine. Ok, let's have a look at how to implement it in previous two modes.

* Plaintext Mode

```go
func plaintextQuery(db *sql.DB, id string) *sql.Row {
	return db.Query("select id, name from prepare where id = " + id + ";")
}
```

* Interpolation Mode

```go
func interpolationQuery(db *sql.DB, id string) *sql.Row {
	return db.Query("select id, name from prepare where id = ?;", id)
}
```

When you pass "1" as id, everything is expected. However, is it really OK? Let's try a SQL injection case, pass "1 or 1 = 1" as id.

Oops, `interpolationQuery()` still returns the same, but `plaintextQuery()` returns all data in the table which means violated SQL has been injected.

## Performance

I make up a simple insert SQL through the two modes.

```
Inserts Number:  100000
Plaintext Mode
Duration 16.058662357s
Interpolation Mode
Duration 24.076297264s
```

It means that `Plaintext Mode` has a better performance than `Interpolation Mode`.
It is reasonable because `Interpolation Mode` has to do 3 network communications per `Query()` or `Exec()`.

# Conclusion

* `Interpolation Mode` can be used to avoid most of SQL injection, which is an important benifit. Therefore, it is highly recommended to use it especially for user input parameters may cause SQL injection.

* `Plaintext Mode` has a better performance to some extent. However, there still some methods to speed up `Interpolation Mode`, I will talk about it later.

# Reference

* [Using Prepared Statements](http://go-database-sql.org/prepared.html)
* [Golang Mysql笔记（三）--- Prepared剖析](https://www.jianshu.com/p/ee0d2e7bef54)
