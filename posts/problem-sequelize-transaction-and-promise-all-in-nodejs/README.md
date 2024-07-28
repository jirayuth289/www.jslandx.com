# problem-sequelize-transaction-and-promise-all-in-nodejs
I found a strange bug about the database transaction rollback on my nodejs project in the production, It is critical problem becuase it change data in database inconsistent.
<br/><br/>
I use a sequelize transaction that automatically passes transactions to all queries with Promise.all method.
<br/>
# Problem
**Transaction has been rejected and rollback complete, but some query still commit partially**
<br/>
**Promise.all fail-fast behavior**
<br/><br/>
Promise.all is rejected if any of the elements are rejected. For example following, we pass in ten promises that there is one promise that rejects, then Promise.all will reject at first failure then  transactions rollbacked.

```
const tenItems = Array(10).fill(null).map((v, i) => ({
    id: i,
    itemName: 'Item' + i
}));

await sequelize.transaction(async () => {
    await Promise.all(tenItems.map(update)); //all query has been paased a transaction by use CLS
});

async function update(date) {
    ...
    await sequelize.query('UPDATE', data);
}
```

In the above example, assume the completion order of each promise following

* 1-5/10 fulfilled &nbsp;&nbsp; : first query task to complete
* 6/10 rejected &nbsp;&nbsp;&nbsp;&nbsp; : transaction rollback
* 7-10/10 pending&nbsp;: and after fulfilled will be committed to database

<br/>
In the scenario, Promise.all is rejected at 6/10 and 1-5 rollback so transactions released, because by default sequelize will clear after the transaction finishes (commit or rollback).
<br/><br/>
but the promise task 7-10 (delay) still queries the database, these promise task execute without transaction and when it fulfilled, it modify database inconsistent
<br/>
# Solution

The approach to solve the problem following
<br/>
1. sequential query execution
<br/><br/>
```
await sequelize.transaction(async (t) => {
    for (const tx of tenDbTxns) {
        const res = await tx;
    }
});
```
we have to wait on each one of these Promises to return before the next one can start.
<br/><br/>
2. concurent execution with Promise.allSettled()
<br/><br/>
```
await sequelize.transaction(async () => {
    const settled = await Promise.allSettled(tenDbTxns);

    const errors = settled.filter(res => res.status === 'rejected)
    if (errors.length) throw errors.map(res => res.reason);
});
```
This approach use the promise concurrency method like Promise.all(), but it waits a response of every promise and we can handle rejection and throw the error manual to rollback the transaction
<br/><br/>
It take speed up better the first approach.
<br/><br/>
finalllay, two approach help passing a transaction in sequelize automatically can make rollback correctly.