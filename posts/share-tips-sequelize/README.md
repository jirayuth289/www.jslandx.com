# Share Tips Sequelize

## 1. Retry
Retrying when errors, there are many types of support e.g.
- ConnectionError
- ValidationError
- ConnectionTimeOutError

Also, support retry
Add the retry option there are 2 ways

#### 1. Passed at main class Sequelize as Global Config

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

#### 2. Passed at query as specific as you want
```
const result = await sequelize.query(sql, { retry: {...} });
```
API Reference
https://sequelize.org/api/v6/class/src/sequelize.js~sequelize#instance-method-query

ดูค่า parameter ที่สามารถกำหนด retry
https://github.com/mickhansen/retry-as-promised


## 2. Use Replication
Config to support host read and write
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

## 3 useMaster
Sometimes Replica Lag to solve with passed useMaster a specific query

```
const result = await sequelize.query(sql, { useMaster: true });
```

ช่วยให้เมื่อ query library จะเลือก host ที่เป็น write ซึ่งเป็น master database

## 4 Set automatically pass the transaction to queries
ทำให้เราไม่ต้องส่ง transaction ไปทุก ๆ query ช่วยลด โค้ดซ้ำซ้อน

ซึ่ง Sequelize จะใช้ cls-hooked

```
const Sequelize = require('sequelize');
const cls = require('cls-hooked');
const namespace = cls.createNamespace('my-very-own-namespace');

Sequelize.useCLS(namespace);

new Sequelize(....);
```