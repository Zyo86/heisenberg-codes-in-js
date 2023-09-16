### Error handling: don’t mix rejections and exceptions
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
