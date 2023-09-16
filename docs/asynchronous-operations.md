## Error handling: don’t mix rejections and exceptions
- Don’t mix (asynchronous) rejections and (synchronous) exceptions. This makes our synchronous and asynchronous code more predictable and simpler because we can always focus on a single error-handling mechanism.
- For Promise-based functions and methods, the rule means that they should never throw exceptions.

```
// Don’t do this
function asyncFunc() {
  doSomethingSync(); // (A)
  return doSomethingAsync()
    .then(result => {
      //
    });
}
```

The problem is that if an exception is thrown in line A, then asyncFunc() will throw an exception. Callers of that function only expect rejections and are not prepared for an exception. There are three ways in which we can fix this issue.

- We can wrap the whole body of the function in a try-catch statement and return a rejected Promise if an exception is thrown:
```
// Solution 1
function asyncFunc() {
  try {
    doSomethingSync();
    return doSomethingAsync()
      .then(result => {
        // ···
      });
  } catch (err) {
    return Promise.reject(err);
  }
}
```

- Given that .then() converts exceptions to rejections, we can execute doSomethingSync() inside a .then() callback. To do so, we start a Promise chain via Promise.resolve(). We ignore the fulfillment value undefined of that initial Promise.
```
// Solution 2
function asyncFunc() {
  return Promise.resolve()
    .then(() => {
      doSomethingSync();
      return doSomethingAsync();
    })
    .then(result => {
      // ···
    });
}
```

- Lastly, new Promise() also converts exceptions to rejections. Using this constructor is therefore similar to the previous solution:
```
// Solution 3
function asyncFunc() {
  return new Promise((resolve, reject) => {
      doSomethingSync();
      resolve(doSomethingAsync());
    })
    .then(result => {
      // ···
    });
}
```

## Promise-based functions start synchronously, settle asynchronously 
Most Promise-based functions are executed as follows:

- Their execution starts right away, synchronously (in the current task).
- But the Promise they return is guaranteed to be settled asynchronously (in a later task) – if ever.

```
function asyncFunc() {
  console.log('asyncFunc');
  return new Promise(
    (resolve, _reject) => {
      console.log('new Promise()');
      resolve();
    });
}
console.log('START');
asyncFunc()
  .then(() => {
    console.log('.then()'); // (A)
  });
console.log('END');

// Output:
// 'START'
// 'asyncFunc'
// 'new Promise()'
// 'END'
// '.then()'
```

## Short-circuiting (advanced)

For a Promise combinator, short-circuiting means that the output Promise is settled early – before all input Promises are settled. The following combinators short-circuit:

- Promise.all(): The output Promise is rejected as soon as one input Promise is rejected.
- Promise.race(): The output Promise is settled as soon as one input Promise is settled.
- Promise.any(): The output Promise is fulfilled as soon as one input Promise is fulfilled.
Once again, settling early does not mean that the operations behind the ignored Promises are stopped. It just means that their settlements are ignored.


## Tips for chaining Promises 

- Chaining mistake: losing the tail 
Problem:
```
// Don’t do this
function foo() {
  const promise = asyncFunc();
  promise.then(result => {
    // ···
  });

  return promise;
}
```

Computation starts with the Promise returned by asyncFunc(). But afterward, computation continues and another Promise is created via .then(). foo() returns the former Promise, but should return the latter. This is how to fix it:

```
function foo() {
  const promise = asyncFunc();
  return promise.then(result => {
    //···
  });
}
```

- Chaining mistake: nesting 
Problem:
```
// Don’t do this
asyncFunc1()
  .then(result1 => {
    return asyncFunc2()
    .then(result2 => { // (A)
      // ···
    });
  });
```
The .then() in line A is nested. A flat structure would be better:
```
asyncFunc1()
  .then(result1 => {
    return asyncFunc2();
  })
  .then(result2 => {
    // ···
  });
```
- Chaining mistake: more nesting than necessary 
This is another example of avoidable nesting:
```
// Don’t do this
asyncFunc1()
  .then(result1 => {
    if (result1 < 0) {
      return asyncFuncA()
      .then(resultA => 'Result: ' + resultA);
    } else {
      return asyncFuncB()
      .then(resultB => 'Result: ' + resultB);
    }
  });
```
We can once again get a flat structure:
```
asyncFunc1()
  .then(result1 => {
    return result1 < 0 ? asyncFuncA() : asyncFuncB();
  })
  .then(resultAB => {
    return 'Result: ' + resultAB;
  });
```
