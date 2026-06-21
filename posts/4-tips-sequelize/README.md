# 4 Tips Sequelize

## Retry

Retrying when errors occur. Supported error types include:

- ConnectionError
- ValidationError
- ConnectionTimedOutError

Add the `retry` option in two ways:

### 1. Pass at main Sequelize class as global config

```
const sequelize = new Sequelize({
    retry: {
      max: 3,
      backoffBase: 10000,
      backoffExponent: 1,
      match: [
        ConnectionError,
        ConnectionTimedOutError,
        TimeoutError
      ],
    }
  })
```

### 2. Pass at query level for specific cases

```
const result = await sequelize.query(sql, { retry: {...} });
```

API Reference: https://sequelize.org/api/v6/class/src/sequelize.js~sequelize#instance-method-query

Parameters supported by retry: https://github.com/mickhansen/retry-as-promised

---

## Use Replication

Configure Sequelize to support read/write host pools:

```
const sequelize = new Sequelize({
    database,
    username,
    password,
    port,
    replication: {
      read: [
        { host: 'your-read-host' }
      ],
      write: { host: 'your-write-host' }
    }
  })
```

API Reference: https://sequelize.org/api/v6/class/src/sequelize.js~sequelize#instance-constructor-constructor

---

## useMaster

When replica lag is an issue, pass `useMaster` on a specific query:

```
const result = await sequelize.query(sql, { useMaster: true });
```

This helps the query choose the write host pool to execute.

---

## Automatically pass the transaction to queries

Sequelize uses the **cls-hooked** package so you do not have to pass a transaction to every query.

```
const Sequelize = require('sequelize');
const cls = require('cls-hooked');
const namespace = cls.createNamespace('my-very-own-namespace');

Sequelize.useCLS(namespace);

new Sequelize(....);
```
