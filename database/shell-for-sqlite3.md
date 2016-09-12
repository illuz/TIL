# Shell for Sqlite3

tag: sqlite, database

---

## Support SQL Commands

DDL:

- CREATE
- ALTER
- DROP

DML:

- INSERT
- UPDATE
- DELETE

DQL:

- SELECT

## CLI

- help: `.help`
- sql
- `.schema`
- `.dump`
- backup `.backup`
- output `.output`
- exit `.exit`
- explain `.explain`
- time `.timer ON|OFF`
- load file `.load FILE`

## Lines


```sql
sqlite3 test.sqlite3
SQLite version 3.8.10.2 2015-05-20 18:17:19
Enter ".help" for usage hints.

sqlite> .show
        echo: off
         eqp: off
  explain: off
     headers: off
        mode: list
   nullvalue: ""
      output: stdout
colseparator: "|"
rowseparator: "\n"
       stats: off
       width:

sqlite> .databases
seq  name             file
---  ---------------  ------
0    main             ./sqlit3/test.sqlite3
sqlite> create table tbl1 (
   ...> f1 varchar(30)
   ...> );

sqlite> .tables
tbl1

sqlite> .schema
CREATE TABLE tbl1 (
f1 varchar(30)
);

sqlite> select * from tbl1;

sqlite> .dump
PRAGMA foreign_keys=OFF;
BEGIN TRANSACTION;
CREATE TABLE tbl1 (
f1 varchar(30)
);
INSERT INTO "tbl1" VALUES('111');
COMMIT;

```


---


https://www.sqlite.org/cli.html
http://www.tutorialspoint.com/sqlite/
