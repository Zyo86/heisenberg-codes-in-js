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

## Asynchronous functions vs. async functions

The difference between the terms asynchronous function and async function is subtle, but important:

-  An asynchronous function is any function that delivers its result asynchronously – for example, a callback-based function or a Promise-based function.

-  An async function is defined via special syntax, involving the keywords async and await. It is also called async/await due to these two keywords. Async functions are based on Promises and therefore also asynchronous functions (which is somewhat confusing).

## The following code demonstrates that an async function is started synchronously (line A), then the current task finishes (line C), then the result Promise is settled – asynchronously (line B).
```
async function asyncFunc() {
  console.log('asyncFunc() starts'); // (A)
  return 'abc';
}
asyncFunc().
then(x => { // (B)
  console.log(`Resolved: ${x}`);
});
console.log('Task ends'); // (C)

// Output:
// 'asyncFunc() starts'
// 'Task ends'
// 'Resolved: abc'
```

await: working with Promises 

The await operator can only be used inside async functions and async generators (which are explained in §42.2 “Asynchronous generators”). Its operand is usually a Promise and leads to the following steps being performed:

The current async function is paused and returned from. This step is similar to how yield works in sync generators.
Eventually, the current task is finished and processing of the task queue continues.
When and if the Promise is settled, the async function is resumed in a new task:
If the Promise is fulfilled, await returns the fulfillment value.
If the Promise is rejected, await throws the rejection value.
Read on to find out more about how await handles Promises in various states.

## await and fulfilled Promises 

> If its operand ends up being a fulfilled Promise, await returns its fulfillment value:
`assert.equal(await Promise.resolve('yes!'), 'yes!');`

> Non-Promise values are allowed, too, and simply passed on (synchronously, without pausing the async function):
`assert.equal(await 'yes!', 'yes!');`

## await and rejected Promises 
If its operand is a rejected Promise, then await throws the rejection value:
```
try {
  await Promise.reject(new Error());
  assert.fail(); // we never get here
} catch (e) {
  assert.equal(e instanceof Error, true);
}
```

## IMPORTANT EXAMPLES of await
> await is shallow (we can’t use it in callbacks) 
If we are inside an async function and want to pause it via await, we must do so directly within that function; we can’t use it inside a nested function, such as a callback. That is, pausing is shallow.

For example, the following code can’t be executed:
```
async function downloadContent(urls) {
  return urls.map((url) => {
    return await httpGet(url); // SyntaxError!
  });
}
```
The reason is that normal arrow functions don’t allow await inside their bodies.

OK, let’s try an async arrow function then:
```
async function downloadContent(urls) {
  return urls.map(async (url) => {
    return await httpGet(url);
  });
}
```
Alas, this doesn’t work either: Now .map() (and therefore downloadContent()) returns an Array with Promises, not an Array with (unwrapped) values.

One possible solution is to use Promise.all() to unwrap all Promises:
```
async function downloadContent(urls) {
  const promiseArray = urls.map(async (url) => {
    return await httpGet(url); // (A)
  });
  return await Promise.all(promiseArray);
}
```
Can this code be improved? Yes it can: in line A, we are unwrapping a Promise via await, only to re-wrap it immediately via return. If we omit await, we don’t even need an async arrow function:
```
async function downloadContent(urls) {
  const promiseArray = urls.map(
    url => httpGet(url));
  return await Promise.all(promiseArray); // (B)
}
```
**For the same reason, we can also omit await in line B.**


## await: running asynchronous functions sequentially 
If we prefix the invocations of multiple asynchronous functions with await, then those functions are executed sequentially:
```
async function sequentialAwait() {
  const result1 = await paused('first');
  assert.equal(result1, 'first');
  
  const result2 = await paused('second');
  assert.equal(result2, 'second');
}

// Output:
// 'START first'
// 'END first'
// 'START second'
// 'END second'
```
That is, paused('second') is only started after paused('first') is completely finished.

## await: running asynchronous functions concurrently #
If we want to run multiple functions concurrently, we can use the tool method Promise.all():
```
async function concurrentPromiseAll() {
  const result = await Promise.all([
    paused('first'), paused('second')
  ]);
  assert.deepEqual(result, ['first', 'second']);
}
```
// Output:
// 'START first'
// 'START second'
// 'END first'
// 'END second'
Here, both asynchronous functions are started at the same time. Once both are settled, await gives us either an Array of fulfillment values or – if at least one Promise is rejected – an exception.

What counts is when we start a Promise-based computation; not how we process its result. Therefore, the following code is as “concurrent” as the previous one:
```
async function concurrentAwait() {
  const resultPromise1 = paused('first');
  const resultPromise2 = paused('second');
  
  assert.equal(await resultPromise1, 'first');
  assert.equal(await resultPromise2, 'second');
}
```
// Output:
// 'START first'
// 'START second'
// 'END first'
// 'END second'
