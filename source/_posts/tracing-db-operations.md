---
title: Trace MySQL DB Operations in Opentracing System
date: 2018-11-28 22:17:00
tags: [golang, database, mysql-driver, opentracing]
---

# Prerequisite

* go >= 1.8
* mysql driver >= 1.4.0 (with Context support)
* OpenTracing System (e.g. Zipkin/Jaeger)

# Examples

## Code Examples

```golang
import (
    ...
    "github.com/go-sql-driver/mysql"
    "github.com/luna-duclos/instrumentedsql"
    "github.com/luna-duclos/instrumentedsql/opentracing"
    ...
)

    sql.Register("instrumented-mysql",
		instrumentedsql.WrapDriver(mysql.MySQLDriver{},
			instrumentedsql.WithTracer(opentracing.NewTracer(false)),
			instrumentedsql.WithOmitArgs(),
		),
	)
    db, err := sql.Open("instrumented-mysql", dsn)
    // db, err := sql.Open("mysql", dsn)
```

## Jaeger Example

{% asset_img "opentracing-db.png" "Jaeger Example" %}

# Performance Benchmark

Wrapped MySQL driver does not have obvious performance impact.

```golang
goos: darwin
goarch: amd64
pkg: demo/dbtracing
 
BenchmarkDriverSelect1-8             1000000          1098 ns/op
BenchmarkWrappedDriverSelect1-8      1000000          1108 ns/op
BenchmarkDriverPing-8                1000000          1091 ns/op
BenchmarkWrappedDriverPing-8         1000000          1097 ns/op
 
PASS
ok      demo/dbtracing  4.485s
```

# Reference

* https://github.com/golang/go/issues/18080
* https://github.com/go-sql-driver/mysql/pull/445
* https://github.com/luna-duclos/instrumentedsql
* https://opencensus.io/
