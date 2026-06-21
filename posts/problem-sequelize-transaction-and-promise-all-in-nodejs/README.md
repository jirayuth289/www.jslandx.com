# Problem: Sequelize Transaction and Promise.all in Node.js

I found a strange bug about database transaction rollback on my Node.js project in production. It is a critical problem because it leaves data in the database inconsistent.

I use a Sequelize transaction that automatically passes transactions to all queries with the `Promise.all` method.

---

## Problem

**Transaction has been rejected and rollback completed, but some queries still commit partially.**

**Promise.all fail-fast behavior**

`Promise.all` is rejected if any of the elements are rejected. For example, if we pass in ten promises and one rejects, `Promise.all` will reject at the first failure and the transaction will be rolled back.

```
const tenItems = Array(10).fill(null).map((v, i) => ({
    id: i,
    itemName: 'Item' + i
}));

await sequelize.transaction(async () => {
    await Promise.all(tenItems.map(update)); // all queries pass a transaction via CLS
});

async function update(data) {
    ...
    await sequelize.query('UPDATE', data);
}
```

In the above example, assume the completion order of each promise is:

- 1–5/10 fulfilled — first query tasks to complete
- 6/10 rejected — transaction rollback
- 7–10/10 pending — and after fulfilled will be committed to the database

In this scenario, `Promise.all` is rejected at 6/10 and 1–5 rollback, so the transaction is released. By default, Sequelize clears the transaction after it finishes (commit or rollback).

But promise tasks 7–10 (delayed) still query the database. These tasks execute without a transaction, and when they fulfill, they modify the database inconsistently.

---

## Solution

Two approaches to solve the problem:

### 1. Sequential query execution

```
await sequelize.transaction(async (t) => {
    for (const tx of tenDbTxns) {
        const res = await tx;
    }
});
```

Wait for each promise to return before the next one starts.

### 2. Concurrent execution with Promise.allSettled()

```
await sequelize.transaction(async () => {
    const settled = await Promise.allSettled(tenDbTxns);

    const errors = settled.filter(res => res.status === 'rejected');
    if (errors.length) throw errors.map(res => res.reason);
});
```

This approach uses promise concurrency like `Promise.all()`, but waits for a response from every promise. We can handle rejection and throw the error manually to rollback the transaction.

It is faster than the first approach.

Finally, both approaches help ensure that passing a transaction in Sequelize automatically makes rollback work correctly.
