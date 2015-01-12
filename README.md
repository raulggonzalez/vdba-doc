# VDBA

`VDBA` (Valencia Database API) is an asynchronous API for the JavaScript language
that programmers can use to access data such as databases.
The VDBA philosophy is similar to the `Node.js` API's.

## VDBA features

When designing the API, we have beared in mind the following features:

  - Two specifications: one for DQL and DML commands and the other for DQL, DML and DDL.
  - Improve the development of applications independent from the database engine. An application
    developed using the VDBA must work with a engine or another without changes.
  - Improve the unit testing of databases, without another testing frameworks specific from the
    database engine.

## Importing the driver package

The VDBA specification has two types:

  - **Type 1**. The drivers implementing this specification can run DQL and DML commands.
  - **Type 2**. The drivers implementing this specification can run DQL, DML and DDL commands.

Generally, the users use the *Type 1* specification. The `Type 1` is lighter and this must be used
whenever possible.

By convention, the drivers must follow the following pattern: `dbms-typeX-vdba-driver`.
Some examples:

  - C*: `cassandra-type1-vdba-driver` and `cassandra-type2-vdba-driver`.
  - PostgreSQL: `postgresql-type1-vdba-driver` and `postgresql-type2-vdba-driver`.
  - Redis: `redis-type1-vdba-driver` and `redis-type2-vdba-driver`.
  - SQLite: `sqlite-type1-vdba-driver` and `sqlite-type2-vdba-driver`.

The application must install the package to fulfill its need.

## Including the driver package

Once imported the driver package, next we must include the driver in our application.
By convention, we associate the `vdba` name to the import. For example:

  ```
  var vdba = require("sqlite-type1-vdba-driver");
  ```

### Including the SQLite driver package

SQLite has two implementations:

  - `sqlite-type1-vdba-driver` that complies with the Type 1 specification.
  - `sqlite-type2-vdba-driver` that complies with the Type 2 specification.

Next, some include examples:

  ```
  var vdba = require("sqlite-type1-vdba-driver");
  var vdba = require("sqlite-type2-vdba-driver");
  ```

## Getting the driver

First of all, we have to get the driver, using the `getDriver()` function of the
`Driver` class. The driver names are:

  - `C*` or `Cassandra` for Apache Cassandra.
  - `PostgreSQL` for PostgreSQL.
  - `SQLite` for SQLite.
  - `Redis` for Redis.

### Getting SQLite driver

Example:

  ```
  var drv = vdba.Driver.getDriver("SQLite");
  ```

## Getting the connection

Once we have the driver, the next thing is getting the connection. Each driver has
its own options, being the following standard:

  - `database` (String). The database name. If SQLite, the database file. If C*, the database is the keyspace.
  - `host` (String). The host to connect. Default: `localhost`. If C*, we can indicate several hosts using a string array.
  - `port` (Integet). The port to connect.
  - `username` (String). The username.
  - `password` (String). The user password.
  - `mode` (String). Connection mode: `readonly` or `readwrite`. Default: `readwrite`. If `readonly`, the driver can't run DML commands.

We have to use the `Driver.createConnection()` method for a closed connection or
`Driver.openConnection()` for opened connection. The `openConnection()` is similar to:

  ```
  var drv = vdba.Driver.getDriver("SQLite");
  var cx = drv.createConnection({database: "data/mydb.db", mode: "readwrite"});
  cx.open(function(error) {
    ...
  });
  ```

### Getting a SQLite connection

The SQLite connection options are: `database`, the database file path; `mode`, the connection mode.

Example:

  ```
  var drv = vdba.Driver.getDriver("SQLite");
  var cx = drv.createConnection({database: "data/mydb.db", mode: "readwrite"});
  ```

## Opening the connection

If we have used the `Driver.createConnection()`, we have to open the connection using
the `Connection.open()` method. For example:

  ```
  cx.open();
  cx.open(function(error) { ... });
  ```

## Closing the connection

We have to call the method `Connection.close()`:

  ```
  cx.close();
  cx.close(function(error) { ... });
  ```

## Getting the current database

When we have to query the data, we need a database object. For getting the current database,
we must use the `Connection.database` property:

  ```
  var db = cx.database;
  ```

## Schemas

Most database engines support the schema concept. When the engine doesn't it,
the driver has to support the schemas.

### Creating schemas (only *Type 2 drivers*)

For creating a schema, we have to use the `Database.createSchema()` method:

  ```
  db.createSchema("schema");
  db.createSchema("schema", function(error) { ... });
  ```

### Getting schemas

For getting a schema, we use the `Database.findSchema()` method:

  ```
  db.findSchema("schema", function(error, schema) { ... });
  ```

### Checking whether a schema exists

For checking whether a schema exists, we'll use `Database.hasSchema()`:

  ```
  db.hasSchema("schema", function(error, exists) { ... });
  ```

### Dropping schemas (only *Type 2 drivers*)

For dropping a schema, to use `Database.dropSchema()`:

  ```
  db.dropSchema("schema");
  db.dropSchema("schema", function(error) { ... });
  ```

### Working with schemas in SQLite

SQLite doesn't support the schema concept, but their drivers do.

In SQLite, the **default schema** exists: the schema when we don't want to
use the schema feature. This schema has the name `default`.

The VDBA team recommends to use schemas.

## Tables

### Creating a table (only *Type 2 drivers*)

If we have to create a table, we have to use a *Type 2 driver* and the `Database.createTable()`
method. The VDBA spec defines the types and then these are mapped to the DBMS types. These
types are:

  - `blob`.
  - `boolean`.
  - `date`, only date.
  - `datetime`, date and time.
  - `integer`.
  - `real`.
  - `sequence`, a sequence or serial integer.
  - `set<integer>`, set of integers.
  - `set<text>`, set of texts.
  - `text`.
  - `time`, only time.

When the DB engine doesn't support some type, its driver is responsible for supporting it.
So, for example, SQLite doesn't support the date, datetime, time, set types,
but the user can work with columns of these types with no problem.

In the near future, the following types will be supported too:

  - `list<integer>` or `array<integer>`.
  - `list<text>` or `array<text>`.
  - `json`.

For creating a table, we can use `Database.createTable()` or `Schema.createTable()`. Some examples:

  ```
  var schema = "sec";
  var table = "User";
  var columns = {
    userId: {type: "sequence", pk: true},
    username: {type: "text", uq: true, nullable: false},
    password: {type: "text", nullable: false},
    tags: "set<text>"
  };
  var options = {};

  db.createTable(schema, table, columns, options);
  sch.createTable(table, columns, options);

  db.createTable(schema, table, columns, options, function(error) { ... });
  sch.createTable(table, columns, options, function(error) { ... });
  ```

The columns are specified with an object, where each property is the column name and
its value contains the column info:

  - `type` (String). The data type: `blob`, `boolean`, `date`, `datime`...
  - `pk` or `primaryKey` (Boolean). Whether the column is the PK.
  - `uq` or `unique` (Boolean). Whether the column has a unique index.
  - `nullable` (Boolean). Whether the column is nullable.
  - `default` (String). The default expression.
  - `check` (String). A check constraint.
  - `ref` (String). For defining foreign keys, we use the `ref` option: `[schema.]table.column`.
    If no schema specified, the schema of the table to create. Example: `ref: "auth.user.userid`.

The create options are:

  - `temporary` (Boolean) for creating temporary tables.
  - `ifNotExists` (Boolean) for creating the table if it doesn't exist.

### Getting tables

For getting a table object, we can use the `Database.findTable()` or `Schema.findTable()`methods:

  ```
  db.findTable("schema", "table", function(error, table) { ... });
  sch.findTable("table", function(error, table) { ... });
  ```

### Checking whether a table exists

For checking whether a table exists, we can use `Database.hasTable()` or `Schema.hasTable()`:

  ```
  db.hasTable("schema", "table", function(error, exists) { ... });
  sch.hasTable("table", function(error, exists) { ... });
  ```

### Dropping tables (only *Type 2 drivers*)

For dropping a table:

  ```
  db.dropTable("schema", "table", function(error) { ... });
  sch.dropTable("table", function(error) { ... });
  ```

## Inserting data

For inserting new rows into the tables, we will use the `Table.insert()` method. This method
can insert one row or several rows.

Examples for inserting one row:

  ```
  table.insert({userId: 1, username: "user01", password: "pwd01"});
  table.insert({userId: 1, username: "user01", password: "pwd01"}, function(error) { ... });
  ```

Examples for inserting several rows:

  ```
  table.insert([
    {userId: 1, username: "user01", password: "pwd01"},
    {userId: 2, username: "user02", password: "pwd02"}
  ]);

  table.insert([
    {userId: 1, username: "user01", password: "pwd01"},
    {userId: 2, username: "user02", password: "pwd02"}
  ], function(error) { ... });
  ```

## Deleting data

For deleting rows, we can use the `Table.remove()` method.

The following examples illustrate how to delete the row with username equal to user01:

  ```
  table.remove({username: "user01"});
  table.remove({username: "user01"}, function(error) { ... });
  ```

For truncating a table:

  ```
  table.remove();
  table.remove(function(error) { ... });

  table.truncate();
  table.truncate(function(error) { ... });
  ```

## Updating data

For updating data, we use the `Table.update()` method.

Next, how to update the state of a row to locked:

  ```
  table.update({userId: 1}, {state: "locked"});
  table.update({userId: 1}, {state: "locked"}, function(error) { ... });
  ```

Now, all rows:

  ```
  table.update({state: "locked"});
  table.update({state: "locked"}, function(error) { ... });
  ```

The update operators are:

  - `{column: value}`: column = value.
  - `{column: {$set: value}}`: column = value.
  - `{column: {$inc: value}}`: column = column + value.
  - `{column: {$dec: value}}`: column = column - value.
  - `{column: {$add: value}}`: set = set + value; text = text || value; number = number + value.
  - `{column: {$del: value}}`: Deletes the value from the set.
  - `{column: {$mul: value}}`: column = column * value.

## Conditional expressions

The conditional expressions are specified using objects. Each property is a column name and its
value indicates the operation. For example: `{age: {$ge: 18}}`.

The possible operations are:

  - `{column: value}`: column = value.
  - `{column: {$eq: value}}`: column = value.
  - `{column: {$ne: value}}`: column <> value or column != value.
  - `{column: {$lt: value}}`: column < value.
  - `{column: {$le: value}}`: column <= value.
  - `{column: {$gt: value}}`: column > value.
  - `{column: {$ge: value}}`: column >= value.
  - `{column: {$like: value}}`: column like value. Wildcards: `%` and `_`.
  - `{column: {$nlike: value}}`: column not like value.
  - `{column: {$notLike: value}}`: column not like value.
  - `{column: {$in: value}}`: column in value.
  - `{column: {$nin: value}}`: column not in value.
  - `{column: {$notIn: value}}`: column not in value.
  - `{column: {$contains: value}}`: column contains value.
  - `{column: {$ncontains: value}}`: column not contains value.
  - `{column: {$notContains: value}}`: column not contains value.

## Finding data

For finding data, we need a query object; this object is created using a table object:

  ```
  var q = table.query();
  ```

### Simple queries

A simple query is one that works with an only table. We can:

  - Limit the maximum number of rows (LIMIT-OFFSET).
  - Sort the result (ORDER BY).
  - Restrict the rows of the result (WHERE).

Once we have the query object, we can use the following methods:

  - `limit()` to limit the maximum number of rows (LIMIT-OFFSET).
  - `sort()` to order the result (ORDER BY).
  - `filter()` to filter the rows (WHERE).
  - `find()` to execute the query.

The query can indicate several operations. Example:

  ```
  q.limit(10).sort("userId").find(function(error, result) { ... });
  ```

The query is issued when the `find()` method is called. We can call implicitly
this method passing a callback in limit(), sort() or filter(). The previous example
can write as follows:

  ```
  q.limit(10).sort("userId", function(error, result) { ... });
  ```

The `find()` can indicate a conditional expression, that is, the filter expression.
Next, two equivalent examples:

  ```
  q.filter({username: {$like: "user%"}}).find(function(error, result) { ... });
  q.find({username: {$like: "user%"}}, function(error, result) { ... });
  ```

The methods return the same query for chaining if needed.

### Multi table queries

The Valencia DB API supports the JOINs. If the DB engine doesn't support the JOIN
concept, the driver must support it. So, for example, the Cassandra driver must implement
JOINs although Cassandra doesn't support internally.

For joining two tables, we can use the `Query.join()`.

Example of natural join (sec.user natural join sec.session):

  ```
  var q = user.query();
  q.join("sec.session", "userId").find(function(error, result) { ... });
  ```

Example of inner join (sec.user inner join sec.session on sec.user.userId = sec.session.user):

  ```
  q.join("sec.session", "userId", "user").find(function(error, result) { ... });
  ```

The table that creates the query is known as **source table**. The other table as **target table**.

Sometimes, the target info is wanted as an object. For these ocassions, we can use the
`Query.joinoo()` method; joinoo is equal to *join one-to-one*:

  ```
  q.joinoo("sec.session", "userId").find(function(error, result) { ... });
  ```

The query will return an object that will contain the target info in a property with the
target table. For example, something like:

  ```
  {
    userId: 1,
    username: "user01",
    password: "pwd01",
    session: {
      sessionId: 123,
      userId: 1,
      login: new Date("2015-01-13")
    }
  }
  ```

The `join()` method returns something like:

  ```
  {userId: 1, username: "user01, password: "pwd01", sessionId: 123, login: new Date("2015-01-13")}
  ```

### findOne() and findAll()

The queries have two special methods:

  - `Query.findOne()`. Similar to `find()` but the callback will receive a row instead of a result.
  - `Query.findAll()`. Similar to `find()` with no filter.

### Table aliases

The table can create queries using:

  - `Table.limit(count[, offset][, callback])`
  - `Table.sort(column[, callback])`
  - `Table.sort([column1, column2], [, callback])`
  - `Table.sort({column1: "ASC|DESC", column2: "ASC|DESC"}[, callback])`
  - `Table.join(table, column[, targetColumn][, callback])`
  - `Table.joinoo(table, column[, targetColumn][, callback])`
  - `Table.find(callback)`
  - `Table.find(filter, callback)`
  - `Table.findOne(callback)`
  - `Table.findOne(filter, callback)`
  - `Table.findAll(callback)`

The previous methods always return a `Query` object, using the creator table as source table.

Next, illustrative:

  ```
  Table.limit(10, function(error, result) { ... });
  Table.limit(10).find(error, result) { ... });
  Table.limit(10).sort("userId").find(error, result) { ... });
  ```

## Result

`find()` and `findAll()` invoke their callbacks passing a result object. This object contains
the following properties:

  - `length` (Number). The number of rows.
  - `rows` (Object[]). The rows.

Example:

  ```
  user.find({password: {$ne: null}}, function(error, result) {
    if (error) {
      console.log("Error:", error);
    } else {
      for (var i = 0; i < result.length; ++i) {
        console.log("Row #", i, ":", result.rows[i]);
      }
    }
  });
  ```