# coroutines

## asynchronous programming

```kotlin
fun requestToken(): Token { ... }
fun createPost(token: Token, item: Item): Post { ... }
fun processPost(post: Post) { ... }

fun postItem(item: Item) {
    // all work can be done in threads
    val token = requestToken()
    val post = createPost(token, item)
    processPost()
}
```

## problem of thread

```kotlin
fun requestToken(): Token {
    // making request
    
    // waiting for result(blocking) <- what's wrong?
    
    return token
}
fun createPost(token: Token, item: Item): Post { ... }
fun processPost(post: Post) { ... }

fun postItem(item: Item) {
    // all work can be done in threads
    val token = requestToken()
    val post = createPost(token, item)
    processPost()
}
```

max thread: 100 or 1000 OK. but 10,000? 

- thread needs some memory


## callbacks to the rescue

```kotlin
fun requestTokenAsync(cb: (Token) -> Unit): Token { 
    // requests token
    // invokes callback when done
    // returns immediately
}

fun createPostAsync(token: Token, item: Item, cb: (Post) -> Unit) { ... }
fun processPost(post: Post) { ... }

fun postItem(item: Item) {
    requestTokenAsync { token ->  // how to handle exception???
        createPostAsync(token, item) { post ->
            processPost(post)
        }
    }  // too many callback. "callback hell"
}
```

- real code can be much more complex

## Futures/Promises/Rx

```kotlin
fun requestTokenAsync(): Promise<Token> {
    // requests token
    // invokes callback when done
    // returns poomise immediately
}

fun createPostAsync(token: Token, item: Item): Promise<Post> { ... }

fun processPost(post: Post) { ... }

fun postItem(item: Item) {
    requestTokenAsync()
        .thenCompose { token -> createPostAsync(token, item) }
        .thenAccept { post -> processPost(post) }
}
```

- composable
- propagates exceptions
- nearly complete solution?

- problem: combinators too complex
- exception or loop or etc -> more combinators

(약간 문제가 있는 지적 아닌지? combinator 개수 자체는 얼마 되지 않음)

## Coroutines


```kotlin
suspend fun requestToken(): Token {
    // requests token & suspends
    return token  // return result when received
}

suspend fun createPost(token: Token, item: Item): Post { ... }
fun processPost(post: Post) { ... }

suspend fun postItem(item: Item) {
    // all work can be done in threads
    val token = requestToken()          // suspension point
    val post = createPost(token, item)  // suspension point
    processPost()
}
```

- `suspend`: this function will suspend and return result when possible later
- the `postItem` code is same as regular code
- code looks natural, but the suspension poitns are not obvious(if you do not watch the actual function signature)

### bonus

- regular loop

```kotlin
for((token,item) in list) {
    createPost(token, item)
}
```

- regular exception handling

```kotlin
try {
    createPost(token, item)
} catch(e:BadTokenException) {

}
```

- regular HOF

```kotlin
file.readLines().forEach { line ->
    createPost(token, line.toItem())
}
```

- forEach, let, apply, repeat, filter, map, use, etc
- everything like in blocking code

## suspending functions

- whene the suspending functions come from?

### retrofit async

- `Call<Post>` is just another future

```
interface Service {
    fun createPost(token: Token, item: Item): Call<Post>
}
```

- kotlin wrapper

```
suspend fun createPost(token: Token, item: Item): Post = 
    serviceInstance.createPost(token, item).await()
```

- `await()` : suspending extension function from integration library

## composition

```kotlin
val post = createPost(token, item)
```

- retry logic? how to implement

```kotln

// retryIO do the retry
val post = retryIO {
    createPost(token, itme)
}
```

- how to implement `retryIO`?

```kotlin
suspend fun <T> retryIO(block: -`suspend () -> T): T {
    val curDelay = 1000L  // start with 1 sec
    while(true) {
        try {
            return block()
        } catch(e: IOException) {
            e.printStackTrace() // log error
        }
        delay(curDelay) // wait some time without blocking
        curDelay = (curDelay*2).coerceAtMost(60000L)
    }
}
```

- `suspend () -> T` makes the lambda suspending
- every other code is just like blocking code


## koroutin builder

- what happens when you forget the `suspend` before `fun postItem`

```kotlin
suspend fun requestToken(): Token { ... }
suspend fun createPost(token: Token, item: Item): Post { ... }
fun processPost(post: Post) { ... }

/*suspend*/ fun postItem(item: Item) {
    val token = requestToken()
    val post = createPost(token, item)
    processPost()
}
```

- `Error: Suspend function `requestToken` should be called only from a coroutine or another suspend function`

- how to introduce the suspend feature to regular function/code

## `launch`

- fire and forget coroutine

```kotlin
fun postItem(item: Item) {
    launch { // run in another thread
        val token = requestToken()
        val post = createPost(token, item)
        processPost() // <- this cannot update UI
    }
}
```

- run in some context

```kotlin
fun postItem(item: Item) {
    launch(UI) { // run in UI thread
        val token = requestToken()
        val post = createPost(token, item)
        processPost() // this can update UI
    }
}
```

- what is the magic of `launch` 

- `launch` signature - regular function but it gets some suspending lambda

```kotlin
fun launch(
    context: CoroutineContext = DefaultDispatcher,
    block: suspend() -> Unit
): Job { ... }
```

- `Job`: using this we can handle the couroutine task

## `async`/`await`

- C#, typescript, python : `async`/`await`

- kotlin way


```kotlin
suspend fun requestToken(): Token { ... }
suspend fun createPost(token: Token, item: Item): Post { ... }
fun processPost(post: Post) { ... }

suspend fun postItem(item: Item) {
    val token = requestToken()
    val post = createPost(token, item)
    processPost()
}
```


- C# classical example

```kotlin
async fun requestToken(): Token { ... }
async fun createPost(token: Token, item: Item): Post { ... }
fun processPost(post: Post) { ... }

async Task postItem(item: Item) {
    val token = await requestToken()
    val post = await createPost(token, item)
    processPost()
}
```

- C# is the pioneer

- difference:
-- marking: `async` vs `suspend`
-- no `await` : we cannot distinguish where is the waiting point(in Kotlin)
-- `Task` : some Future

## Why no `await` keyword in Kotlin

- problem with async
- two invocation possible
    -- C# `requestToken()` returns valid `Task<Token>`  : concurrent behivior
    -- C# `await requestToken()` returns `Token` (wait till get the token) : sequential behavior

- philosophy
    -- concurrency is the default
    -- concurrency has to be explicit

- **Kotlin suspending functions are design to imitate *sequential* behavior by default**


## Kotlin approach

- usecase for async

```C#
async Task<Image> loadImageAsync(String anme) { ... }

var promise1 = loadImageAsync(name1);
var promise2 = loadImageAsync(name1);

var image1 = await promise1;
var image2 = await promise2;

var result = combineImages(image1, image2);
```

- kotlin async function

```kotlin
// loadImageAsync is a regular function
// Deferred<Image> : kotlin future type
// sync: coroutine builder (take block of code and exe)
fun loadImageAsync(name: String): Deferred<Image> = 
    async { ... }
    
val deferred1 = loadImageAsync(name1)
val deferred2 = loadImageAsync(name2)

val image1 = deferred1.await()
val image2 = deferred2.await()

val result = combineImages(image1, image2)
```

- using async function when needed

```kotlin
// loadImage is defined as suspending function
suspend fun loadImage(name:String): Image { ... }

suspend fun loadAndCombine(name1:String, name2:String): Image {
    // do concurrently load image explicitly
    val deferred1 = async { loadImage(name1) } 
    val deferred2 = async { loadImage(name2) }
    return combineImages(deferred1.await(), deferred2.await())
}
```

- kotlin approach

= `requestToken()`      => produces `Token`  

 (sequential, default) behavior


= `async { requestToken() }    => produces `deferred<Token>`

  (concurrent behavior)


## coroutines

- what is coroutine conceptually?
- D. Knuth invented

- like very ligh-weight thread 


```kotlin
fun main(args: Array<String>) = runBlocking<Unit> {
    val jobs = List(100_000) {
        launch {
            delay(1000L)
            print(".")
        }
    }
    jobs.forEach { it.join() }
}
```

- `runBlocking` runs coroutine in the context of invoker thread
- works like a charm

### with threads

```kotlin
fun main(args: Array<String>) {
    val jobs = List(100_000) {
        thread {
            Thread.sleep(1000L)
            print(".")
        }
    }
    jobs.forEach { it.join() }
}
```

- OutOfMemoryError : unable to create new native thread


## java interop







































