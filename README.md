# DuckStreams

<img src="logo.jpg" width="250">

### Description
DuckStreams acts as a virtual database on top of your Kafka and Redpanda
clusters, effectively providing an ephemeral SQL interface for querying
streaming data.

## Scope 
DuckStreams turns Kafka and Redpanda topics into SQL-accessible virtual tables,
allowing users to query streaming data in real-time. The project interfaces with
the schema registry to map topics to tables and supports deserializing data in
formats like JSON, Protobuf, and Thrift. Using DuckDB, DuckStreams creates
ephemeral tables, runs queries on them, and returns results without persisting
any data. It’s designed to be lightweight, fully in-memory, and ideal for
querying dynamic stream data without caching or long-term storage.

## How it works

First, the service is served as a python DBAPI-compatible driver. If asked for its list of tables,
queried through `INFORMATION_SCHEMA.TABLES`, it interfaces with your streaming cluster's
schema registry to get a list of topics. This table is made ephemeral through duckdb so that
you can apply predicates, grouping and run any SQL against it.

Similarly, when running ANY SQL statement against the database, we parse-out the virtual table
name, which should match an existing topic (or `INFORMATION_SCHEMA.TABLES`), then it will simply:

1. figure out the topic
1. fire up a client + consumer, and apply the time and partition predicate
1. deserialize the data into memory, load it into an ephemeral, in-memory duckdb table
1. run the SQL you ran against this ephemeral table in duck db, retrieve the result set
1. return it through a DBAPI-compatible interface

## Configuration

* clusters: define your clusters into a yaml file

* policies:
  * levels inheritance: top-level, cluster-level or table-level 
  * parameters
    * row_limit: limit the number of rows the consumer will read, it just stops once reached
    * time_range_limit: define a max time range that can be queries, can be anything from seconds to years
    * bytes (?)
    * cells (?)

## Thoughts & questions

* nesting: it's pretty common to have deeply/oddly nested schemas on the transport layer,
  how good are duckdb's support for complex schema? arbitrary json? Should we auto-comlumnize
  things as we deserialize? automagically? based on configs?
