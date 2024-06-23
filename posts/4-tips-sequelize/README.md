# 4 Tips Sequelize

# Retry
Retrying when errors, there are many types of support e.g.
* ConnectionError
* ValidationError
* ConnectionTimeOutError

Also, support retry
Add the retry option there are 2 ways

### 1. Passed at main class Sequelize as Global Config

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

### 2. Passed at query as specific as you want
```
const result = await sequelize.query(sql, { retry: {...} });
```
API Reference
https://sequelize.org/api/v6/class/src/sequelize.js~sequelize#instance-method-query

Look for parameters supported retry
https://github.com/mickhansen/retry-as-promised


# Use Replication
Config sequelize to support pool host read and write
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
API Reference
https://sequelize.org/api/v6/class/src/sequelize.js~sequelize#instance-constructor-constructor

# useMaster
Sometimes Replica Lag to solve with passed useMaster a specific query

```
const result = await sequelize.query(sql, { useMaster: true });
```

Will help our query choose pool host write to execute

# Set automatically pass the transaction to queries
Sequelize use package named **cls-hooked**, it help our must not pass a transaction every query

```
const Sequelize = require('sequelize');
const cls = require('cls-hooked');
const namespace = cls.createNamespace('my-very-own-namespace');

Sequelize.useCLS(namespace);

new Sequelize(....);
```