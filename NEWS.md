All major and minor releases are briefly explained below.

For richer information consult the commit log on github with referenced pull requests.

We do not include break-fix version release in this file.

### v3.0.0

#### Breaking changes
- [Parse the DATE PostgreSQL type as local time](https://github.com/brianc/node-postgres/pull/514)

After [some discussion](https://github.com/brianc/node-postgres/issues/510) it was decided node-postgres was non-compliant in how it was handling DATE results.  They were being converted to UTC, but the PostgreSQL documentation specifies they should be returned in the client timezone.  This is a breaking change, and if you use the `date` type you might want to examine your code and make sure nothing is impacted.

- [Fix possible numeric precision loss on numeric & int8 arrays](https://github.com/brianc/node-postgres/pull/501)

pg@v2.0 included changes to not convert large integers into their JavaScript number representation because of possibility for numeric precision loss. The same types in arrays were not taken into account.  This fix applies the same type of type-coercion rules to arrays of those types, so there will be no more possible numeric loss on an array of very large int8s for example. This is a breaking change because now a return type from a query of `int8[]` will contain _string_ representations
of the integers.  Use your favorite JavaScript bignum module to represent them without precision loss, or punch over the type converter to return the old style arrays again.

- [Fix to input array of dates being improperly converted to utc](https://github.com/benesch/node-postgres/commit/c41eedc3e01e5527a3d5c242fa1896f02ef0b261#diff-7172adb1fec2457a2700ed29008a8e0aR108)

Single `date` parameters were properly sent to the PostgreSQL server properly in local time, but an input array of dates was being changed into utc dates.  This is a violation of what PostgreSQL expects.  Small breaking change, but none-the-less something you should check out if you are inserting an array of dates.

- [Query no longer emits `end` event if it ends due to an error](https://github.com/brianc/node-postgres/commit/357b64d70431ec5ca721eb45a63b082c18e6ffa3)

This is a small change to bring the semantics of query more in line with other EventEmitters. The tests all passed after this change, but I suppose it could still be a breaking change in certain use cases.  If you are doing clever things with the `end` and `error` events of a query object you might want to check to make sure its still behaving normally, though it is most likely not an issue.

#### New features
- [Supercharge `prepareValue`](https://github.com/brianc/node-postgres/pull/555)

The long & short of it is now any object you supply in the list of query values will be inspected for a `.toPostgres` method.  If the method is present it will be called and its result used as the raw text value sent to PostgreSQL for that value.  This allows the same type of custom type coercion on query parameters as was previously afforded to query result values.

- [Domain aware connection pool](https://github.com/brianc/node-postgres/pull/531)

If domains are active node-postgres will honor them and do everything it can to ensure all callbacks are properly fired in the active domain. If you have tried to use domains with node-postgres (or many other modules which pool long lived event emitters) you may have run into an issue where the active domain changes before and after a callback. This has been a longstanding footgun within node-postgres and I am happy to get it fixed.

- [Disconnected clients now removed from pool](https://github.com/brianc/node-postgres/pull/543)

Avoids a scenario where your pool could fill up with disconnected & unusable clients.

- [Break type parsing code into separate module](https://github.com/brianc/node-postgres/pull/541)

To provide better documentation and a clearer explanation of how to override the query result parsing system we broke the type converters [into their own module](https://github.com/brianc/node-pg-types). There is still work around removing the 'global-ness' of the type converters so each query or connection can return types differently, but this is a good first step and allow a lot more obvious way to return int8 results as JavaScript numbers, for example

### v2.11.0
- Add support for [application_name](https://github.com/brianc/node-postgres/pull/497)

### v2.10.0
- Add support for [the password file](http://www.postgresql.org/docs/9.3/static/libpq-pgpass.html)

### v2.9.0
- Add better support for [unix domain socket](https://github.com/brianc/node-postgres/pull/487) connections

### v2.8.0
- Add support for parsing JSON[] and UUID[] result types

### v2.7.0
- Use single row mode in native bindings when available [@rpedela]
  - reduces memory consumption when handling row values in 'row' event
- Automatically bind buffer type parameters as binary [@eugeneware]

### v2.6.0
- Respect PGSSLMODE environment variable

### v2.5.0
- Ability to opt-in to int8 parsing via `pg.defaults.parseInt8 = true`

### v2.4.0
- Use eval in the result set parser to increase performance

### v2.3.0
- Remove built-in support for binary Int64 parsing.
_Due to the low usage & required compiled dependency this will be pushed into a 3rd party add-on_

### v2.2.0
- [Add support for excapeLiteral and escapeIdentifier in both JavaScript and the native bindings](https://github.com/brianc/node-postgres/pull/396)

### v2.1.0
- Add support for SSL connections in JavaScript driver
 - this means you can connect to heroku postgres from your local machine without the native bindings!
- [Add field metadata to result object](https://github.com/brianc/node-postgres/blob/master/test/integration/client/row-description-on-results-tests.js)
- [Add ability for rows to be returned as arrays instead of objects](https://github.com/brianc/node-postgres/blob/master/test/integration/client/results-as-array-tests.js)

### v2.0.0

- Properly handle various PostgreSQL to JavaScript type conversions to avoid data loss:

```
PostgreSQL | pg@v2.0 JavaScript | pg@v1.0 JavaScript
--------------------------------|----------------
float4     | number (float)     | string
float8     | number (float)     | string
int8       | string             | number (int)
numeric    | string             | number (float)
decimal    | string             | number (float)
```

For more information see https://github.com/brianc/node-postgres/pull/353
If you are unhappy with these changes you can always [override the built in type parsing fairly easily](https://github.com/brianc/node-pg-parse-float). 

### v1.3.0

- Make client_encoding configurable and optional

### v1.2.0

- return field metadata on result object: access via result.fields[i].name/dataTypeID

### v1.1.0

- built in support for `JSON` data type for PostgreSQL Server @ v9.2.0 or greater

### v1.0.0

- remove deprecated functionality
  - Callback function passed to `pg.connect` now __requires__ 3 arguments
  - Client#pauseDrain() / Client#resumeDrain removed
  - numeric, decimal, and float data types no longer parsed into float before being returned. Will be returned from query results as `String`

### v0.15.0

- client now emits `end` when disconnected from back-end server
- if client is disconnected in the middle of a query, query receives an error

### v0.14.0

- add deprecation warnings in prep for v1.0
- fix read/write failures in native module under node v0.9.x
