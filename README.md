# InfluxDB Foreign Data Wrapper for PostgreSQL
This PostgreSQL extension is a Foreign Data Wrapper (FDW) for InfluxDB.

The current version can work with PostgreSQL 9.6, 10 and 11.

## Installation
Install InfluxDB Go client library
<pre>
go get github.com/influxdata/influxdb1-client/v2
</pre>

Add a directory of pg_config to PATH and build and install influxdb_fdw.
<pre>
make USE_PGXS=1 with_llvm=no
make install USE_PGXS=1 with_llvm=no
</pre>
with_llvm=no is necessary to disable llvm bit code generation when PostgreSQL is configured with --with-llvm because influxdb_fdw use go code and cannot be compiled to llvm bit code.

If you want to build influxdb_fdw in a source tree of PostgreSQL instead, use
<pre>
make with_llvm=no
make install  with_llvm=no
</pre>

## Usage
### Load extension
<pre>
CREATE EXTENSION influxdb_fdw;
</pre>

### Create server
<pre>
CREATE SERVER influxdb_server FOREIGN DATA WRAPPER influxdb_fdw OPTIONS
(dbname 'mydb', host 'http://localhost', port '8086') ;
</pre>

### Create user mapping
<pre>
CREATE USER MAPPING FOR CURRENT_USER SERVER influxdb_server OPTIONS(user 'user', password 'pass');
</pre>

### Create foreign table
You need to declare a column named "time" to access InfluxDB time column.
<pre>
CREATE FOREIGN TABLE t1(time timestamp with time zone , tag1 text, field1 integer) SERVER influxdb_server OPTIONS (table 'measurement1');
</pre>

### Import foreign schema
<pre>
IMPORT FOREIGN SCHEMA public FROM SERVER influxdb_server INTO public;
</pre>

### Access foregin table
<pre>
SELECT * FROM t1;
</pre>

## Features
- WHERE clauses including timestamp, interval and `now()` functions are pushed down
- Some of aggregation are pushed down

## Limitations
- INSERT, UPDATE and DELETE are not supported.

Following limitations originate from data model and query language of InfluxDB.
- Result sets have different number of rows depending on specified target list.
For example, `SELECT field1 FROM t1` and `SELECT field2 FROM t1` returns different number of rows if
the number of points with field1 and field2 are different in InfluxDB database. 
- Currently `GROUP BY` works for only tag keys, not for field keys([#3](/../../issues/3))

When a query to foreing tables fails, you can find why it fails by seeing a query executed in InfluxDB with `EXPLAIN (VERBOSE)`.

## Contributing
Opening issues and pull requests on GitHub are welcome.

## License
Copyright (c) 2018 - 2019, TOSHIBA Corporation 
Copyright (c) 2011 - 2016, EnterpriseDB Corporation

Permission to use, copy, modify, and distribute this software and its documentation for any purpose, without fee, and without a written agreement is hereby granted, provided that the above copyright notice and this paragraph and the following two paragraphs appear in all copies.

See the [`LICENSE`][4] file for full details.

[4]: LICENSE
