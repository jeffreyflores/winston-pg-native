# winston-pg-native

A Winston 3.X compatible transport for PostgreSQL. Uses high performance of native bindings via libpq.

# Origin

I forked https://github.com/remjx/winston-pg-native which is a fork of https://github.com/petpano/winston-pg-native so that we can push to an app column in the database.

## Installation

```console
  $ npm install winston
  $ npm install @jeffreyflores/winston-pg-native
```

You must have a table in your PostgreSQL database, for example:

```sql
CREATE TABLE winston_logs
(
  timestamp timestamp without time zone DEFAULT now(),
  level character varying,
  message character varying,
  app character varying,
  meta json
)
```

## Options

- **connectionString:** The PostgreSQL connection string. Required.
- **level:** The winston's log level. Optional, default: info
- **poolConfig:** Pool specific configuration parameters. Optional.
- **tableName:** PostgreSQL table name definition. Optional.

See the default values used:

```js
const options = {
  connectionString: 'postgres://username:password@localhost:5432/database',
  level: 'info',
  app: 'appname',
  poolConfig: {
    // number of milliseconds to wait before timing out when connecting a new client
    // by default this is 0 which means no timeout
    connectionTimeoutMillis: 0,
    // number of milliseconds a client must sit idle in the pool and not be checked out
    // before it is disconnected from the backend and discarded
    // default is 10000 (10 seconds) - set to 0 to disable auto-disconnection of idle clients
    idleTimeoutMillis: 10000,
    // maximum number of clients the pool should contain
    // by default this is set to 10.
    max: 10
  },
  tableName: 'winston_logs'
};
```

## Usage

```js
const { Logger } = require('winston');
const Postgres = require('winston-pg-native');

const logger = new Logger({
  transports: [
    new Postgres({
      connectionString,
      level: 'info',
      app: 'appname',
      poolConfig: {
        connectionTimeoutMillis: 0,
        idleTimeoutMillis: 0,
        max: 10
      },
      tableName: 'winston_logs'
    })
  ]
});

module.exports = logger;
```

## Logging

```js
logger.log('info', 'message', {});
```

## Querying Logs

This transport supports querying of logs with Loggly-like options. [See Loggly Search API](https://www.loggly.com/docs/api-retrieving-data/)

```js
const options = {
  fields: ['message'],
  from: new Date() - 24 * 60 * 60 * 1000,
  until: new Date(),
  start: 0,
  limit: 10,
  order: 'desc'
};

//
// Find items logged between today and yesterday.
//
logger.query(options, (err, results) => {
  if (err) {
    throw err;
  }

  console.log(results);
});
```

## Streaming Logs

Streaming allows you to stream your logs back

```js
//
// Start at the end.
//
logger.stream({ start: -1 }).on('log', log => {
  console.log(log);
});
```

## Run Tests

The tests are written in [vows](http://vowsjs.org), and designed to be run with npm.

```bash
  $ npm test
```

## LICENSE

[MIT License](http://en.wikipedia.org/wiki/MIT_License)
