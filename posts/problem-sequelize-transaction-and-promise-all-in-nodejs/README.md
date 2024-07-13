# problem-sequelize-transaction-and-promise-all-in-nodejs

# Problem

I found the error transaction rollback on my project in the production.

I use sequelize transaction that automatically passes transactions to all queries with Promise.all() node typescript.

I searched, and I think the cause will be.

**Transaction has been rejected and rollback complete, but some delay query still commit partially**

Because sequelize concurrent transactions with Promise.all can work incorrect because of Promise.all() behavior (doesn't wait all promise results).

```
await Promise.all(tenDbTxns);

// 5/10 fulfilled first query task to complete
// 6 rejected ---> sequelize rollback 6/10
// 4/10 pending and after fulfilled will be committed to database (slow query)
```

Commit will happen if you use a pool connection. In the example, you rollback the changes in the connection and then release the connection back to the pool(6/10). (thinking this connection is now clean) BUT, your delay promises will still use it.

So the result of execution will occur commit partiallly at 5/10 fulfilled, 4/10 and other your delay promises will still use it

And Promise.all() execute as concurrent if there is any reject, it catches the first reject. It won't roll back anything probably cuase next task query still execute without stop other promises

 #### Other cuase
By default, Sequelize might call CLS.clear after the transaction finishes (commit or rollback).

# Solution

1. sequential query execution:

```
const data = [];
for (const tx of tenDbTxns) {
    const res = await tx;

    data.push(tx);
}
```

2. concurent execution but with Promise.allSettled (waits all promise results):

```
const settled = await Promise.allSettled(tenDbTxns);
const data = [];
for (const res of settled) {
    if (res.status === 'rejected') {
        throw res.reason;
    }

    data.push(res.value);
}
```

---------Transactions and promises in Node.js---------

We have a database which need to be consistently updated with some batch of data and Node.JS application which executes queries in database.

These actions executes by following code:

try {
    await connection.transaction();
    const promises = [];
    for (const elem of data) {
        promises.push(connection.query('updateElem', elem));
    }
    await Promise.all(promises) 
    await connection.commit();
} catch(error) {
    await connection.rollback();
}
Problem:
Using this code we expect the following behavior: when one query fails, transaction rolls back and data should stay consistent. But this code will lead you to one strange bug: when one query fails, other queries roll back partially (e.g. 6 of 10 queries roll back, but other ones modifiy data so database becomes inconsistent)

Reason:
To understand how we got this problem we need to read MDN docs about Promise.all(). It says to us:

It rejects with the reason of the first promise that rejects

And no words about stopping of other promises execution.

So, this bug can be illustrated by the following code:

const p1 = new Promise((resolve, reject) => {
    setTimeout(() => resolve(console.log('1')), 1000);
});
const p2 = new Promise((resolve, reject) => {
    setTimeout(() => resolve(console.log('2')), 2000, 'two');
});
const p3 = new Promise((resolve, reject) => {
    setTimeout(() => resolve(console.log('3')), 3000, 'three');
});
const p4 = new Promise((resolve, reject) => {
    reject('reject');
});

Promise.all([p1, p2, p3, p4]).then(values => {
    console.log(values);
}).catch(reason => {
    console.log(reason);
});
And result will be:

reject
1
2
3
Something like that happened when we used Promise.all() inside transaction:

Transaction started;
Queries queued to execution;
One query failed;
Transaction rolled back;
Some slow queries executed after transaction rollback.
Solutions:
Thanks to the Chad Elliott for it: As of NodeJS@12.10.0 you can use Promise.allSettled(). This will not only wait for all promises to complete, but will give you an array with the result of each promise. Just iterate through the results and verify that none have a status of ‘rejected’.
Not the best solution but it works: do not use Promise.all() inside transctions, use sequential query execution.

Comment

As of NodeJS@12.10.0 you can use Promise.allSettled. This will not only wait for all promises to complete, but will give you an array with the result of each promise. Just iterate through the results and verify that none have a status of 'rejected'.

Or write small tool that allows to run all in once and returns array of results/errors (but all resolved or rejected).



Had the same issue and resolve with catching the errors for each promise in the Promis.all function.
For Jehy, Commit will happen if you use a pool connection. In the example, you rollback the changes in the connection and then release the connection back to the pool. (thinking this connection is now clean) BUT, your delay promises will still use it.
You right no one will commit those changes in this point, but guess what will happen if the next transaction will take the same connection and then will commit all the changes…. (Partial commit from the failed transaction).

----------------------------------------
https://stackoverflow.com/questions/67988319/is-there-any-reason-to-use-promise-all-in-a-typeorm-transaction

Cause

Typeorm concurrent transactions with Promise.all can work incorrect because of Promise.all behavior (doesn't wait all promise results).
```
await Promise.all(tenDbTxns);

// 5/10 fulfilled
// 6 rejected ---> typeorm rollback 6/10
// 4/10 pending and after fulfilled will be committed to database
```

So as solutions you can use:

sequential query execution:
const data = [];
for (const tx of tenDbTxns) {
    const res = await tx;

    data.push(tx);
}

concurent execution but with Promise.allSettled (waits all promise results):

const settled = await Promise.allSettled(tenDbTxns);
const data = [];
for (const res of settled) {
    if (res.status === 'rejected') {
        throw res.reason;
    }

    data.push(res.value);
}

-----------------------------------------
https://stackoverflow.com/questions/61416687/sequelize-transaction-with-promise-all-fails-to-rollback-when-one-promise-reject

Use Promise.allSettled method. It waits for all promises to complete and returns the result of all promises in an array.
Make an array of all db writes and pass the same transaction to each. Then await Promise.all settled. Then check if all promises are resolved. If they are commit transaction else rollback.

--------------

https://github.com/typeorm/typeorm/issues/1014

I can reproduce this issue.

However, its how promises are working and its because of how Promise.all works.
The problem is that when promise.all executes it does not wait for all promises to be resolved and throws error once any of promise rejects but other promises are executed at this moment, after rejection has made. ORM listens to rejects and executed rollback. But since there is a promise in execution at the same moment, DELETE is executed anyway.

Solution is to combine for loop and await, or simply run promises in order without using Promise.all. You can also use PromiseUtils.runInSequence.

You need something like bluebird's settle function. I just added it into PromiseUtils class internally used in orm. Example:

    /**
     * Returns a promise that is fulfilled with an array of promise state snapshots,
     * but only after all the original promises have settled, i.e. become either fulfilled or rejected.
     */
    function settle(promises: Promise<any>[]) {
        return Promise.all(promises.map(p => Promise.resolve(p).then(v => ({
            state: "fulfilled",
            value: v,
        }), r => ({
            state: "rejected",
            reason: r,
        })))).then((results: any[]): any => {
            const rejected = results.find(result => result.state === "rejected");
            if (rejected)
                return Promise.reject(rejected.reason);

            return results.map(result => result.value);
        });
    }

    // --------------------

     await connection.transaction(manager => {
                await settle([
                    manager.remove(TestEntity, { id: 1 }),
                    new Promise((resolve, reject) => reject(new Error())),
                ]);
      });