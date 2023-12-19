# Understanding Kotlin Coroutines & Suspend Functions

In the world of software development, understanding how to make asynchronous tasks more readable and maintaining order among them is crucial. Kotlin coroutines and suspend functions are essential elements in achieving this. By understanding the differences between concurrency and parallelism and in what situations to apply "runBlocking", "launch", "async", and even non-suspending functions, we save ourselves from future headaches pertaining to race conditions and inefficient software design. Let's delve deeper into these topics.

## Concurrency & Parallelism in Coroutines

An important aspect of programming, especially in a multi-threaded environment, is understanding the nuances between concurrency and parallelism. At times, even the best of us can confuse the two concepts. So, let's explore this with a particular focus on Kotlin's coroutines, and more specifically, Kotlin's suspend functions.

### Defining Concurrency and Parallelism
Concurrency implies dealing with multiple tasks at the same time but not necessarily executing them simultaneously. 

Imagine you're organising a party and there are multiple tasks to handle: cooking, decorating, and creating a playlist. As the sole organiser, you switch between these tasks, say, setting the oven to preheat (cooking), then while it's heating, you pick out some party decorations (decorating), and in the meantime while you’re waiting for the over the heap up, you search for some songs to play (creating playlist). You're working on all tasks but not truly at the same moment - you are context-switching between different tasks which are part of a single larger task (organising a party). This is an example of _concurrency_.

Now, say you invite three friends over and assign them each a task: one cooks, one decorates, and one creates the playlist. Now all tasks are being done at the same time by different people. This is _parallelism_.

### Practical Example
In the world of programming, these concepts apply in the same way. So when working with threads and coroutines in Kotlin, it's important to remember the distinction.

Coroutines help handle many tasks at once like a single party organiser. However, they don’t automatically do all things at exactly the same time. For that to happen, these coroutines (tasks) must be distributed across multiple threads (helpers).

Imagine you're working with a back-end server written in Kotlin that handles HTTP requests from clients. These could be requests to fetch data from a database, perform some calculations, or store new data.

In a typical synchronous server, each client request is handled sequentially as follows:
```kotlin
fun handleRequest() {
    val requestData = receiveRequest() // This is a network request
    val processedData = processData(requestData)
    sendResponse(processedData)
}
```

In this case, the `receiveRequest()` function waits for a client request and during this time, the server is blocked and can't handle any other incoming requests. This is inefficient and makes poor use of server resources, especially when dealing with I/O-bound tasks where waiting is often inevitable.

Fortunately, Kotlin's support for coroutines can offer a better alternative. By using a suspend function for handling the requests, you can easily prevent blocking while waiting for a request:

```kotlin
suspend fun handleRequest() {
    val requestData = receiveRequest() // This is a network request
    val processedData = processData(requestData)
    sendResponse(processedData)
}

suspend fun receiveRequest() {
    // receive network request and suspend
    // resume when client's request is received
}
```

With the help of coroutines, the `receiveRequest()`  function suspends while waiting, allowing the server to handle other incoming requests. This won't block the entire server, and it can keep accepting and processing other requests concurrently.

However, it's critical to note that suspend doesn't automatically mean parallel execution. Even with coroutines, the `processData(requestData)` function still only runs after `receiveRequest()` has completed its execution. To achieve parallel execution of these tasks, more steps would be required, like launching the coroutines on different threads.

In summary, understanding the principles of suspending functions and the difference between concurrency and parallelism is important for efficient and safe backend development with Kotlin. Coroutines allow for improved resource utilisation, especially for I/O-bound tasks, without compromising the server's ability to handle multiple requests concurrently.

## runBlocking

`runBlocking` is a method used in Kotlin to block the current thread, meaning no other operations can take place in the thread until the action within runBlocking is complete. runBlocking bridges the regular blocking code and the non-blocking style of coroutines, but does so at great cost, since suspending is no longer possible.

While this function comes in handy in specific cases, it should generally not be used extensively in application code, especially for large-scale applications where performance is important. It fits well as the top-level coroutine for a specific use case, like running blocking operations in unit tests or when developing a proof of concept.

## Coroutine Builders: launch and async

In the Kotlin Coroutines API, we've got two primary coroutine builders, launch and async.
- `launch`: This function returns a `Job` that can be used to cancel the running coroutine. The principle application is for fire-and-forget tasks, meaning it doesn’t return anything after the operation has executed, except for the Job that can indicate if the execution resulted in an exception or completed as expected
- `async`: This function returns a `Deferred<T>`, a future promise of a value. async is ideal when there is a need to fetch a result from the coroutine that will be needed later

When dealing with launch and async, developers must take extra caution so as not to introduce race conditions in their code. It's also essential to ensure appropriate exception handling and coroutine cancelation for launch and async respectively.

## Non-Suspending Asynchronous Functions

Lastly, an essential thing to note is that in Kotlin (and also in Java), a function can still be asynchronous without being a suspending function. A good example is using Java's Future. While futures facilitate producing values asynchronously, they do not suspend their execution which contributes to blocking threads and resulting in potential race conditions. Making something not suspend does not keep you safe from race condition - it still depends on how you handle the asynchronous execution.

## Summary
- A function being a suspend function does not imply parallelism, but it does enable concurrency
- If `runBlocking` is used, concurrency is not possible, not even in suspend functions
- A function can be asynchronous, or execute parts of its requests asynchronously, regardless of it being a suspend function or not
- Usage of `launch` and `async` requires extra care, in order to not introduce the risk of race conditions 

In conclusion, asynchronous programming in Kotlin through the use of coroutines, suspending functions and the other mechanisms discussed, while a powerful tool, needs to be used carefully and correctly to avoid potential race conditions and other common pitfalls. Ensuring a clear understanding of concurrency and parallelism is crucial.

Remember, performance is good, but not at the cost of correctness.

## Additional Resources

- [Coroutines basics](https://kotlinlang.org/docs/coroutines-basics.html) 
- [KotlinConf 2017 - Introduction to Coroutines by Roman Elizarov](https://www.youtube.com/watch?v=_hfBv0a09Jc&t=1094s)
- [KotlinConf 2018 - Kotlin Coroutines in Practice by Roman Elizarov](https://www.youtube.com/watch?v=a3agLJQ6vt8)
- [Blocking threads, suspending coroutines](https://elizarov.medium.com/blocking-threads-suspending-coroutines-d33e11bf4761)
- [Explicit concurrency](https://elizarov.medium.com/explicit-concurrency-67a8e8fd9b25)
