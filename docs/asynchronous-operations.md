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
