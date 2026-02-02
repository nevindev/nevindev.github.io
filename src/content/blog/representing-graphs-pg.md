---
title: 'Representing graphs in PostgreSQL with SQL/PGQ'
description: ''
pubDate: 'Feb 18 2025'
heroImage: '../../assets/6n-graf.png'
---
*[Originally posted](https://www.enterprisedb.com/blog/representing-graphs-postgresql-sqlpgq) on the [EDB Technical Blog](https://www.enterprisedb.com/blog), a fantastic source of Postgres knowledge!*

--- 

As a huge fan of Tolkien and graph theory, [this excellent blog post](https://www.richard-towers.com/2025/02/16/representing-graphs-in-postgres.html) shared by my colleagues at EDB piqued my interest.

The author describes how a graph can be modeled in PostgreSQL and recursive CTEs can be used to execute graph-traversal queries and concludes: 

This is one way of representing graph-like data in Postgresql, and querying it in a flexible way.

I’d be very interested in hearing about other techniques or improvements.

And indeed there is a new technique under development for natively working with graphs in Postgres!

### SQL Property Graph Queries

SQL Property Graph Queries (SQL/PGQ) are now part of the [SQL:2023](https://peter.eisentraut.org/blog/2023/04/04/sql-2023-is-finished-here-is-whats-new) ISO standard and provide the ability to efficiently represent and query existing relational data as a graph without requiring a separate graph database management system (such as Neo4j).

Representing relationships between nodes through connected edges is a very natural and useful ability which is particularly well-suited for representing networks— social networks of [friends of friends](https://en.wikipedia.org/wiki/Six_degrees_of_separation), transportation networks, computer networks and more.

### Adding this Power to Postgres!

While there are third-party extensions which add graph database functionality to Postgres, such as [Apache AGE™](https://age.apache.org/overview/), discussion and work to implement SQL/PGQ into Postgres has been ongoing by contributors in the [pgsql-hackers](https://www.postgresql.org/message-id/flat/CAEG8a3L3uZZRT5Ra5%3D9G-SOCEYULejw5eqQE99VL0YfTeX3-BA%40mail.gmail.com#f477d33034dd8df626c0cfdad4e9e085) mailing list.

While still very much a work-in-progress with no release date, it is functional and can be patched into Postgres for you to explore!

### Patching in three easy steps

Applying these patches requires that Postgres is built and installed from the source code. The [Postgres documentation](https://www.postgresql.org/docs/current/installation.html) provides a comprehensive overview on the topic.

1. Download all `.patch` files from [this psql-hackers thread](https://www.postgresql.org/message-id/flat/CAEG8a3L3uZZRT5Ra5%3D9G-SOCEYULejw5eqQE99VL0YfTeX3-BA%40mail.gmail.com#f477d33034dd8df626c0cfdad4e9e085)
2. Get the Postgres source code (see https://www.postgresql.org/docs/current/install-getsource.html), then change into the directory that contains the Postgres source code for the rest of the installation procedure
3. For each `.patch` file downloaded (mine downloaded under the Downloads directory on Debian 12), run

```bash
patch -p1 < ~/Downloads/v10-00NN-x-y-z.patch
```
Replacing `NN-x-y-z` with the appropriate patch file name.

Once all the patches are applied, we are ready to build and install SQL/PGQ-patched Postgres!

### Building and Installing

* NOTE: Before running `./configure`, ensure the required tools are installed (on a Debian 12 OS, I had to install `libicu-dev`, `bison` and `flex`).

The short version should suffice for building and installing Postgres, but build it according to your needs.

```bash
./configure
make
su
make install
adduser postgres
mkdir -p /usr/local/pgsql/data
chown postgres /usr/local/pgsql/data
su - postgres
/usr/local/pgsql/bin/initdb -D /usr/local/pgsql/data
/usr/local/pgsql/bin/pg_ctl -D /usr/local/pgsql/data -l logfile start
/usr/local/pgsql/bin/createdb test
/usr/local/pgsql/bin/psql test
```

### Graphs in Action!

The blog post which inspired this post uses Tolkien characters as example data; we will create the same data and demonstrate query parity using **SQL/PGQ** methods.

##### Creating tables

First, fire up `psql` and generate two tables; one for characters (nodes) and one for relationships (edges):

```sql
CREATE TABLE nodes (
 id SERIAL PRIMARY KEY,
 name TEXT,
 details JSONB
);
CREATE TABLE edges (
 id SERIAL PRIMARY KEY,
 type TEXT,
 from_id INTEGER REFERENCES nodes(id),
 to_id INTEGER REFERENCES nodes(id),
 details JSONB
);
```

##### Creating data

Next, populate these tables with data:

```sql
INSERT INTO nodes (name, details) VALUES
 ('Frodo Baggins', '{"species": "Hobbit"}'), -- 1
 ('Bilbo Baggins', '{"species": "Hobbit"}'), -- 2
 ('Samwise Gamgee', '{"species": "Hobbit"}'), -- 3
 ('Hamfast Gamgee', '{"species": "Hobbit"}'), -- 4
 ('Gandalf', '{"species": "Wizard"}'), -- 5
 ('Aragorn', '{"species": "Human"}'), -- 6
 ('Arathorn', '{"species": "Human"}'), -- 7
 ('Legolas', '{"species": "Elf"}'), -- 8
 ('Thranduil', '{"species": "Elf"}'), -- 9
 ('Gimli', '{"species": "Dwarf"}'), -- 10
 ('Gloin', '{"species": "Dwarf"}'); -- 11
-- Parents
INSERT INTO edges (type, from_id, to_id, details) VALUES
 ('parent', 2, 1, '{}'),
 ('parent', 4, 3, '{}'),
 ('parent', 7, 6, '{}'),
 ('parent', 9, 8, '{}'),
 ('parent', 11, 10, '{}');
```

##### Creating a Property Graph

SQL/PGQ allows the user to create "Property Graphs" on top of one or more existing relational tables, and query these graphs natively using a powerful new operator, called `GRAPH_TABLE`, which provides a graph pattern matching language fully integrated into SQL.

* Note: Because SQL/PGQ is still under development in Postgres, there is sparse documentation.

Below, we reference the documentation for OracleDB's Property Graph Query Language (`PGQL`), which has incorporated the `SQL:2023` standard and provides an explanation of the features, even though it is for a different database management system.

By defining `VERTEX TABLES` to encapsulate our nodes and `EDGE TABLES` to encapsulate the relationships between these nodes, a `PROPERTY GRAPH` represents our data as a graph.

We have only one `VERTEX TABLE` corresponding to our nodes table for characters and a single `EDGE TABLE` corresponding to our edges table for relationships. We give this edge a `LABEL` named `relationship` which has the `type` property ('parent' or 'friend'). Pay special attention to the keys (`SOURCE KEY` and `DESTINATION KEY`) of the edge, as direction matters when traversing a graph (which will be seen querying data).

```sql
CREATE PROPERTY GRAPH characters 
  VERTEX TABLES ( 
    nodes LABEL node PROPERTIES ( id, name, details )
  ) 
  EDGE TABLES ( 
   edges 
    SOURCE KEY ( from_id ) REFERENCES nodes ( id ) 
    DESTINATION KEY ( to_id ) REFERENCES nodes ( id ) 
    LABEL relationship PROPERTIES (type) 
  );
```

### Simple queries - finding parents and children

We can find someone’s parents by traversing our characters property graph along edges with the relationship label of type parent.

Let's find Samwise Gamgee's parent:

*Using a Recursive CTE*
```sql
WITH child AS (SELECT id FROM nodes WHERE name = 'Samwise Gamgee')
SELECT parent.name FROM child
JOIN edges ON edges.to_id = child.id
JOIN nodes parent ON edges.from_id = parent.id;
```

*Using SQL/PGQ*
```sql
SELECT name FROM GRAPH_TABLE (characters
MATCH (a IS node WHERE a.name='Samwise Gamgee') <-[e IS relationship WHERE e.type='parent']- (b IS node)
COLUMNS (b.name AS name)
);
```

As expected, this returns Hamfast Gamgee!

When creating a property graph, the `SOURCE KEY` and `DESTINATION KEY` direction was mentioned. As is seen here, the syntax of PG/PGQ uses an arrow `<-[e]-` when querying the relationship between two nodes sharing an edge. The head of the arrow points to the `DESTINATION KEY` from the `SOURCE KEY`. In this case, the query looks to match nodes where the `SOURCE KEY` has a "parent" relationship to the `DESTINATION KEY`.

Because the query is from the child (destination) looking for parents (source), it is looking "up" the graph (a tree) from the child to the parents, so it makes sense the arrow is left ("upwards").
```
     a                        e                    b
DESTINATION KEY <-[relationship.type = parent]- SOURCE KEY
   to_id                                        from_id
Samwise Gamgee                               Hamfast Gamgee
```
If the head of the arrow is changed to point right, the query flips: it is now the parent looking for children! This will return zero results, which makes sense since Sam's children were not added to the database.
```
   a                     e                       b
SOURCE KEY -[relationship.type = parent]-> DESTINATION KEY
 from_id                                       to_id
Samwise Gamgee                                 NO MATCH
```
It is easy to keep the arrow direction correct by saying:

`SOURCE` is `RELATIONSHIP` to `DESTINATION`

### More complex queries - friends of friends

Not all of our characters knew each other directly. The friendship graph is a bit more complex.

Run the following to add "friend" relationships into the edges table:
```sql
INSERT INTO edges (type, from_id, to_id, details) VALUES
 -- Everyone in the fellowship is friends with everyone else
 ('friend', 1, 3, '{}'), -- Frodo and Sam
 ('friend', 1, 5, '{}'), -- Frodo and Gandalf
 ('friend', 1, 6, '{}'), -- Frodo and Aragorn
 ('friend', 1, 8, '{}'), -- Frodo and Legolas
 ('friend', 1, 10, '{}'), -- Frodo and Gimli
 ('friend', 3, 1, '{}'), -- Sam and Frodo
 ('friend', 3, 5, '{}'), -- Sam and Gandalf
 ('friend', 3, 6, '{}'), -- Sam and Aragorn
 ('friend', 3, 8, '{}'), -- Sam and Legolas
 ('friend', 3, 10, '{}'), -- Sam and Gimli
 ('friend', 5, 1, '{}'), -- Gandalf and Frodo
 ('friend', 5, 3, '{}'), -- Gandalf and Sam
 ('friend', 5, 6, '{}'), -- Gandalf and Aragorn
 ('friend', 5, 8, '{}'), -- Gandalf and Legolas
 ('friend', 5, 10, '{}'), -- Gandalf and Gimli
 ('friend', 6, 1, '{}'), -- Aragorn and Frodo
 ('friend', 6, 3, '{}'), -- Aragorn and Sam
 ('friend', 6, 5, '{}'), -- Aragorn and Gandalf
 ('friend', 6, 8, '{}'), -- Aragorn and Legolas
 ('friend', 6, 10, '{}'), -- Aragorn and Gimli
 ('friend', 8, 1, '{}'), -- Legolas and Frodo
 ('friend', 8, 3, '{}'), -- Legolas and Sam
 ('friend', 8, 5, '{}'), -- Legolas and Gandalf
 ('friend', 8, 6, '{}'), -- Legolas and Aragorn
 ('friend', 8, 10, '{}'), -- Legolas and Gimli
 ('friend', 10, 1, '{}'), -- Gimli and Frodo
 ('friend', 10, 3, '{}'), -- Gimli and Sam
 ('friend', 10, 5, '{}'), -- Gimli and Gandalf
 ('friend', 10, 6, '{}'), -- Gimli and Aragorn
 ('friend', 10, 8, '{}'), -- Gimli and Legolas
 -- Bilbo was friends with Hamfast and Gandalf
 ('friend', 2, 4, '{}'), -- Bilbo and Hamfast
 ('friend', 2, 5, '{}'), -- Bilbo and Gandalf
 -- And vice versa
 ('friend', 4, 2, '{}'), -- Hamfast and Bilbo
 ('friend', 5, 2, '{}'), -- Gandalf and Bilbo
 -- Gandalf was friends with Bilbo, Hamfast and Thranduil, but for the sake of
 -- argument let's say he didn't know Gloin or Arathorn
 ('friend', 5, 2, '{}'), -- Gandalf and Bilbo
 ('friend', 5, 4, '{}'), -- Gandalf and Hamfast
 ('friend', 5, 9, '{}'), -- Gandalf and Thranduil
 -- And vice versa
 ('friend', 2, 5, '{}'), -- Bilbo and Gandalf
 ('friend', 4, 5, '{}'), -- Hamfast and Gandalf
 ('friend', 9, 5, '{}'); -- Thranduil and Gandalf
 ```
Using SQL/PGQ dramatically simplifies the syntax for finding the parents of friends-of-friends when compared to the recursive CTE:

*Using a Recursive CTE*
```sql
WITH RECURSIVE 
root(id) AS (SELECT id FROM nodes WHERE name = 'Samwise Gamgee'),
paths(path) AS (VALUES ('{friend,friend,parent}'::text[])),
results(id) AS (
 SELECT root.id, 1 as path_index from root
UNION
 SELECT edges.from_id, path_index + 1 AS path_index FROM results
 JOIN edges ON edges.to_id = results.id
 JOIN paths ON edges.type = paths.path[path_index]
)
SELECT * FROM results
JOIN nodes ON nodes.id = results.id
JOIN paths ON cardinality(paths.path) + 1 = results.path_index;
```

*Using SQL/PGQ*
```sql
SELECT DISTINCT id, fof_parents, details FROM GRAPH_TABLE(characters
 MATCH (a IS node WHERE a.name='Samwise Gamgee')-[x IS relationship WHERE x.type='friend']->(b IS node)-[y IS relationship WHERE y.type='friend']->(c IS node)<-[z IS relationship WHERE z.type='parent']-(d IS node)  
 COLUMNS (d.id, d.name as fof_parents, d.details as details)
);
```
The shape of the arrow in the MATCH statement neatly describes the graph traversal:

`(Samwise Gamgee) -[friendship]-> (friend) -[friendship]-> (friend-of-friend) <-[parent]-(parent-of-friend-of-friend)`

We traverse across all 'friend' relationship edges connected to Samwise Gamgee two levels away to find all friend-of-friends nodes and (like our first example, note the leftward arrow) look for the parent of each friend-of-friend node.

The query results are the same as the original blog post
```sql
id |  fof_parents   |        details        
----+----------------+-----------------------
 2 | Bilbo Baggins  | {"species": "Hobbit"}
 4 | Hamfast Gamgee | {"species": "Hobbit"}
 7 | Arathorn       | {"species": "Human"}
 9 | Thranduil      | {"species": "Elf"}
11 | Gloin          | {"species": "Dwarf"}
(5 rows)
```
### Conclusion

SQL/PGQ provides a neat interface for working with graph-like data directly in PostgreSQL that is flexible, powerful and intuitive to read! These examples barely scratch the surface, but do show parity with the results of the reference blog post and present another technique for working with graphs in Postgres. It does not yet have a release date and this post demonstrates work-in-progress functionality, but I look forward to this exciting feature gaining full support in a future release of Postgres!

Thank you to all the pgsql-hackers whose hard work and cooperation makes this possible.