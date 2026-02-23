---
title: 'Writing Postgres extensions in Rust with pgrx'
description: 'The early stages of developing pgautograph'
pubDate: 'Feb 23 2026'
heroImage: '../../assets/pgrx-logo-color.png'
---

I first became familiar with `pgrx` after discovering [this insightful blog post](https://tuttlem.github.io/2025/11/23/creating-extensions-in-rust-for-postgresql.html) by [tuttlem](https://github.com/tuttlem) aptly titled 'Creating extensions in Rust for PostgreSQL'.

Extensibility has been at the core of Postgres' design since it's early inception; without a doubt this feature plays a key role in it's enduring legacy *and* recent explosion in adoption. For as long as I have used Postgres -- whether for work, school or fun -- extensions have always empowered my database to do the job. Looking at the excellent [State of PostgreSQL 2024](https://www.tigerdata.com/state-of-postgres/2024#ecosystem-and-tools) provided by [Tiger Data](https://www.tigerdata.com) (makers of the phenomenal [timescaledb](https://github.com/timescale/timescaledb) extension), a gigantic **EXTENSIBILITY** is front-and-center in the wordcloud! I am happy to see `postgis` featured too, it is one of my personal favourite extensions (I really love geospatially-enabled apps.)

I am immensely grateful to the very dedicated and talented individuals, teams and companies who work hard to build these incredible extensions, many of which are generously open source and permissively licensed, for the benefit of all users.

C has always been the traditional choice for writing extensions (and indeed a large portion of the PostgreSQL source code), but requires careful thought to ensure memory is managed safely. Rust enforces memory safety during compilation thanks to the borrow checker. Using it to write extensions provides a great developer experience and some level of guarantee that compiled code doesn't violate the rules (`unsafe` blocks aside).

The [pgrx repository](https://github.com/pgcentralfoundation/pgrx) README provides a list of key features, but I think one of the coolest is having safe access to the Server Programming Interface (SPI), which provides the ability to run SQL commands inside Rust functions. In the aforementioned blog post, the author provides example number 6, **'Accessing Catalog Metadata (Optional but Fun)'** and concludes the section with *This begins to show how easy it is to build introspection tools — or even something more adventurous, like treating your relational schema as a graph.*

This brings the conversation full-circle back to [Representing graphs in PostgreSQL with SQL/PGQ](https://nevin.dev/blog/representing-graphs-pg/) and my ultimate goal for [pgautograph](https://github.com/nevindev/pgautograph) as an easy tool for introspecting schemas to automatically create Property Graph queries!

This is still under early development, but this is what the main function generates

```
pgautograph=# SELECT * FROM generate_property_graph();
INFO:  
        CREATE PROPERTY GRAPH graph
            VERTEX TABLES (
                Persons LABEL Person NO PROPERTIES
                Accounts LABEL Account NO PROPERTIES
                Transactions LABEL Transaction NO PROPERTIES
            )

            EDGE TABLES (
                Persons AS CompaniesPerson
                        SOURCE KEY ( company_id ) REFERENCES persons ( company_id )
                        DESTINATION Companies NO PROPERTIES

                Accounts AS PersonsAccount
                        SOURCE KEY ( person_id ) REFERENCES accounts ( person_id )
                        DESTINATION Persons NO PROPERTIES

                Accounts AS CompaniesAccount
                        SOURCE KEY ( company_id ) REFERENCES accounts ( company_id )
                        DESTINATION Companies NO PROPERTIES

                Transactions AS AccountsTransaction
                        SOURCE KEY ( id ) REFERENCES accounts ( number )
                        DESTINATION KEY ( id ) REFERENCES accounts ( number )
                        LABEL between NO PROPERTIES

            );
        
 source_table | source_pk | source_columns | target_table | target_column 
--------------+-----------+----------------+--------------+---------------
 accounts     | number    | company_id     | companies    | id
 accounts     | number    | person_id      | persons      | id
 persons      | id        | company_id     | companies    | id
 transactions | id        | from_account   | accounts     | number
 transactions | id        | to_account     | accounts     | number
(5 rows)
```

The example schema replicates the [basic example](https://pgql-lang.org/#a-basic-example) used in the Property Graph Query Language documentation. I am combining this example with the patching of PostgreSQL 18 to have a baseline comparison of feature parity that expands upon the initial demonstration by [Richard Towers](https://www.richard-towers.com/2025/02/16/representing-graphs-in-postgres.html) and my own follow up with [SQL/PGQ](https://www.enterprisedb.com/blog/representing-graphs-postgresql-sqlpgq). 

Stay tuned for more to come about Rust, Postgres and graphs!
