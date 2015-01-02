# VDBA

`VDBA` (Valencia Database API) is an asynchronous API for the JavaScript language
that programmers can use to access data such as databases.
The VDBA philosophy is similar to the `Node.js` API's.

## vdba namespace

When the driver is included, a `vdba` object is created automatically.
This is the API start point.

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
  var vdba = require("redis-type1-vdba-driver");
  ```

## Getting the driver

First of all, we have to get the driver, using the `getDriver()` function of the
`Driver` class. The driver names are:

  - `C*` or `Cassandra` for Apache Cassandra.
  - `PostgreSQL` for PostgreSQL.
  - `SQLite` for SQLite.
  - `Redis` for Redis.

Example:

  ```
  var drv = vdba.Driver.getDriver("Redis");
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

Here, an example:

  ```
  var drv = vdba.Driver.getDriver("Redis");
  var cx = drv.createConnection({host: "localhost", port: 6379, username: "user01", password: "pwd01"});
  ```

The connection returned by the `createConnection()` method is closed. If we want an opened connection,
we can use the `openConnection()` method:

  ```
  var drv = vdba.Driver.getDriver("Redis");
  drv.openConnection({host: "localhost", port: 6379, username: "user01", password: "pwd01"}, function(error, cx) {
    ...
  });
  ```

The `openConnection()` is similar to:

  ```
  var drv = vdba.Driver.getDriver("Redis");
  var cx = drv.createConnection({host: "localhost", port: 6379, username: "user01", password: "pwd01"});
  cx.open(function(error) {
    ...
  });
  ```