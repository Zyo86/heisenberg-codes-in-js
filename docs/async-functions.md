## What are async functions?
Async functions are marked with the keyword async.

Inside the body of an async function, we write Promise-based code as if it were synchronous. We only need to apply the await operator whenever a value is a Promise. That operator pauses the async function and resumes it once the Promise is settled:

If the Promise is fulfilled, await returns the fulfillment value.
If the Promise is rejected, await throws the rejection value.
The result of an async function is always a Promise:

> **Any value that is returned (explicitly or implicitly) is used to fulfill the Promise.
Any exception that is thrown is used to reject the Promise.**

The following codes are equivalent
```
async function fetchJsonAsync(url) {
  try {
    const request = await fetch(url); // async
    const text = await request.text(); // async
    return JSON.parse(text); // sync
  }
  catch (error) {
    assert.fail(error);
  }
}
```
The previous, rather synchronous-looking code is equivalent to the following code that uses Promises directly:
```
function fetchJsonViaPromises(url) {
  return fetch(url) // async
  .then(request => request.text()) // async
  .then(text => JSON.parse(text)) // sync
  .catch(error => {
    assert.fail(error);
  });
}
```
