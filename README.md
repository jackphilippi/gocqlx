# gocqlx [![GoDoc](http://img.shields.io/badge/go-documentation-blue.svg?style=flat-square)](http://godoc.org/github.com/scylladb/gocqlx) [![Go Report Card](https://goreportcard.com/badge/github.com/scylladb/gocqlx)](https://goreportcard.com/report/github.com/scylladb/gocqlx) [![Build Status](https://travis-ci.org/scylladb/gocqlx.svg?branch=master)](https://travis-ci.org/scylladb/gocqlx)

Package `gocqlx` is a Scylla / Cassandra productivity toolkit for `gocql`, it's 
similar to what `sqlx` is to `database/sql`.

It contains wrappers over `gocql` types that provide convenience methods which
are useful in the development of database driven applications.  Under the
hood it uses `sqlx/reflectx` package so `sqlx` models will also work with `gocqlx`.

## Installation

    go get github.com/scylladb/gocqlx

## Features

Fast, boilerplate free and flexible `SELECTS`, `INSERTS`, `UPDATES` and `DELETES`.

```go
type Person struct {
	FirstName string  // no need to add `db:"first_name"` etc.
	LastName  string
	Email     []string
}

p := &Person{
	"Patricia",
	"Citizen",
	[]string{"patricia.citzen@gocqlx_test.com"},
}

// Insert
{
	q := Query(qb.Insert("person").Columns("first_name", "last_name", "email").ToCql())
	if err := q.BindStruct(p); err != nil {
		t.Fatal("bind:", err)
	}
	mustExec(q.Query)
}

// Update
{
	p.Email = append(p.Email, "patricia1.citzen@gocqlx_test.com")

	q := Query(qb.Update("person").Set("email").Where(qb.Eq("first_name"), qb.Eq("last_name")).ToCql())
	if err := q.BindStruct(p); err != nil {
		t.Fatal("bind:", err)
	}
	mustExec(q.Query)
}

// Select
{
	q := Query(qb.Select("person").Where(qb.In("first_name")).ToCql())
	m := map[string]interface{}{
		"first_name": []string{"Patricia", "John"},
	}
	if err := q.BindMap(m); err != nil {
		t.Fatal("bind:", err)
	}

	var people []Person
	if err := gocqlx.Select(&people, q.Query); err != nil {
		t.Fatal(err)
	}
	t.Log(people)

	// [{Patricia Citizen [patricia.citzen@gocqlx_test.com patricia1.citzen@gocqlx_test.com]} {John Doe [johndoeDNE@gmail.net]}]
}
```

For more details see [example test](https://github.com/scylladb/gocqlx/blob/master/example_test.go).

## Performance

Gocqlx is fast, below is a benchmark result comparing `gocqlx` to raw `gocql` on
my machine, see the benchmark [here](https://github.com/scylladb/gocqlx/blob/master/benchmark_test.go).

For query binding gocqlx is faster as it does not require parameter rewriting 
while binding. For get and insert the performance is comparable.

```
BenchmarkE2EGocqlInsert-4           1000           1580420 ns/op            2624 B/op         59 allocs/op
BenchmarkE2EGocqlxInsert-4          2000            648769 ns/op            1557 B/op         34 allocs/op
BenchmarkE2EGocqlGet-4              3000            664618 ns/op            1086 B/op         29 allocs/op
BenchmarkE2EGocqlxGet-4             3000            631415 ns/op            1440 B/op         32 allocs/op
BenchmarkE2EGocqlSelect-4             50          35646283 ns/op           34072 B/op        922 allocs/op
BenchmarkE2EGocqlxSelect-4            50          37128897 ns/op           28304 B/op        933 allocs/op
```

Gocqlx comes with automatic snake case support for field names and does not 
require manual tagging. This is also fast, below is a comparison to 
`strings.ToLower` function (`sqlx` default).

```
BenchmarkSnakeCase-4            10000000               124 ns/op              32 B/op          2 allocs/op
BenchmarkToLower-4              100000000               57.9 ns/op             0 B/op          0 allocs/op
```

Building queries is fast and low on allocations too.

```
BenchmarkCmp-4                   3000000               464 ns/op             112 B/op          3 allocs/op
BenchmarkDeleteBuilder-4        10000000               214 ns/op             112 B/op          2 allocs/op
BenchmarkInsertBuilder-4        20000000               103 ns/op              64 B/op          1 allocs/op
BenchmarkSelectBuilder-4        10000000               214 ns/op             112 B/op          2 allocs/op
BenchmarkUpdateBuilder-4        10000000               212 ns/op             112 B/op          2 allocs/op
```

Enyoy!
