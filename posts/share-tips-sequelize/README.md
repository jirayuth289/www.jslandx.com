# Share Tips Sequelize

## 1. Retry

Retrying when Database Errors such as
- ConnectionError
- ValidationError
- ConnectionTimeOutError

Add the retry option there are 2 ways

1. Passed at main class Sequelize as Global Config

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

2. Passed at query as specific as you want

const result = await sequelize.query(sql, { retry: { ... } });

https://sequelize.org/api/v6/class/src/sequelize.js~sequelize#instance-method-query
https://github.com/mickhansen/retry-as-promised


## 2. Use Replication

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

https://sequelize.org/api/v6/class/src/sequelize.js~sequelize#instance-constructor-constructor

## 3 useMaster
Sometimes Replica Lag to solve with passed useMaster a specific query

const result = await sequelize.query(sql, { useMaster: true });