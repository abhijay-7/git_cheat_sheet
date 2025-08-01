# Kotlin Async Programming Cheat Sheet

## Coroutine Fundamentals

### Basic Coroutine Concepts
```kotlin
import kotlinx.coroutines.*
import kotlin.time.Duration.Companion.seconds

// Suspend functions - can be paused and resumed
suspend fun fetchUserData(userId: String): String {
    delay(1000) // Non-blocking delay
    return "User data for $userId"
}

suspend fun fetchUserPosts(userId: String): List<String> {
    delay(1500)
    return listOf("Post 1", "Post 2", "Post 3")
}

// Basic coroutine launch
fun basicCoroutineExample() {
    runBlocking {
        println("Starting...")
        
        launch {
            delay(1000)
            println("Coroutine 1 completed")
        }
        
        launch {
            delay(500)
            println("Coroutine 2 completed")
        }
        
        println("Both coroutines launched")
        delay(2000) // Wait for coroutines to complete
    }
}
```

### Coroutine Builders
```kotlin
// runBlocking - blocks current thread until completion
fun runBlockingExample() {
    runBlocking {
        delay(1000)
        println("runBlocking completed")
    }
}

// launch - fire and forget
fun launchExample() {
    runBlocking {
        val job = launch {
            repeat(5) { i ->
                println("Launch $i")
                delay(500)
            }
        }
        
        delay(2000)
        job.cancel() // Cancel the job
    }
}

// async - returns Deferred (like Future)
suspend fun asyncExample(): String {
    return coroutineScope {
        val deferred1 = async { fetchUserData("user1") }
        val deferred2 = async { fetchUserPosts("user1") }
        
        val userData = deferred1.await()
        val userPosts = deferred2.await()
        
        "$userData with ${userPosts.size} posts"
    }
}

// produce - creates channel-based producer
fun produceExample() {
    runBlocking {
        val numbers = produce {
            for (i in 1..5) {
                delay(100)
                send(i * i)
            }
        }
        
        numbers.consumeEach { println("Received: $it") }
    }
}
```

## Coroutine Scopes

### Built-in Scopes
```kotlin
import kotlinx.coroutines.*

// GlobalScope - application lifetime (use sparingly)
fun globalScopeExample() {
    GlobalScope.launch {
        delay(1000)
        println("GlobalScope task completed")
    }
    // Main thread continues immediately
}

// CoroutineScope - structured concurrency
class MyService {
    private val scope = CoroutineScope(Dispatchers.IO + SupervisorJob())
    
    fun doWork() {
        scope.launch {
            // Background work
            delay(1000)
            println("Work completed")
        }
    }
    
    fun cleanup() {
        scope.cancel() // Cancel all child coroutines
    }
}

// coroutineScope - scoped parallel execution
suspend fun scopedParallelWork(): String {
    return coroutineScope {
        val result1 = async { heavyComputation1() }
        val result2 = async { heavyComputation2() }
        
        "${result1.await()} + ${result2.await()}"
    }
}

suspend fun heavyComputation1(): String {
    delay(1000)
    return "Result1"
}

suspend fun heavyComputation2(): String {
    delay(800)
    return "Result2"
}

// supervisorScope - independent child failure handling
suspend fun supervisorScopeExample() {
    supervisorScope {
        val job1 = launch {
            delay(1000)
            throw RuntimeException("Job 1 failed")
        }
        
        val job2 = launch {
            delay(2000)
            println("Job 2 completed successfully")
        }
        
        // Job 2 continues even if Job 1 fails
    }
}
```

### Custom Scopes
```kotlin
// Creating custom scope with specific context
class DataProcessor {
    private val processingScope = CoroutineScope(
        Dispatchers.Default + 
        SupervisorJob() + 
        CoroutineName("DataProcessor")
    )
    
    fun processDataAsync(data: List<String>) {
        processingScope.launch {
            data.forEach { item ->
                launch { processItem(item) }
            }
        }
    }
    
    private suspend fun processItem(item: String) {
        delay(100)
        println("Processed: $item")
    }
    
    fun shutdown() {
        processingScope.cancel()
    }
}

// Scope with exception handler
class ServiceWithErrorHandling {
    private val exceptionHandler = CoroutineExceptionHandler { _, exception ->
        println("Caught exception: ${exception.message}")
    }
    
    private val scope = CoroutineScope(
        Dispatchers.IO + SupervisorJob() + exceptionHandler
    )
    
    fun riskyOperation() {
        scope.launch {
            throw RuntimeException("Something went wrong")
        }
    }
}
```

## Dispatchers

### Built-in Dispatchers
```kotlin
// Dispatchers.Main - UI thread (Android)
fun mainDispatcherExample() {
    runBlocking {
        launch(Dispatchers.Main) {
            // Update UI elements
            println("Running on main thread: ${Thread.currentThread().name}")
        }
    }
}

// Dispatchers.IO - I/O operations
suspend fun ioOperations() {
    withContext(Dispatchers.IO) {
        // File operations, network calls, database queries
        println("I/O operation on: ${Thread.currentThread().name}")
        delay(1000) // Simulate I/O operation
    }
}

// Dispatchers.Default - CPU intensive work
suspend fun cpuIntensiveWork() {
    withContext(Dispatchers.Default) {
        // Computations, data processing
        println("CPU work on: ${Thread.currentThread().name}")
        repeat(1000000) { /* simulate heavy computation */ }
    }
}

// Dispatchers.Unconfined - not confined to any specific thread
suspend fun unconfinedExample() {
    withContext(Dispatchers.Unconfined) {
        println("Before delay: ${Thread.currentThread().name}")
        delay(100)
        println("After delay: ${Thread.currentThread().name}")
    }
}
```

### Custom Dispatchers
```kotlin
import java.util.concurrent.Executors

// Single thread dispatcher
val singleThreadDispatcher = Executors.newSingleThreadExecutor().asCoroutineDispatcher()

// Fixed thread pool dispatcher
val fixedThreadPoolDispatcher = Executors.newFixedThreadPool(4).asCoroutineDispatcher()

// Custom dispatcher usage
suspend fun customDispatcherExample() {
    withContext(singleThreadDispatcher) {
        println("Running on custom single thread: ${Thread.currentThread().name}")
    }
    
    withContext(fixedThreadPoolDispatcher) {
        println("Running on custom thread pool: ${Thread.currentThread().name}")
    }
}

// Remember to close custom dispatchers
fun cleanup() {
    singleThreadDispatcher.close()
    fixedThreadPoolDispatcher.close()
}

// Limited parallelism dispatcher
val limitedDispatcher = Dispatchers.IO.limitedParallelism(2)

suspend fun limitedParallelismExample() {
    repeat(10) { i ->
        launch(limitedDispatcher) {
            println("Task $i on ${Thread.currentThread().name}")
            delay(1000)
        }
    }
}
```

## Exception Handling

### Basic Exception Handling
```kotlin
// Try-catch in coroutines
suspend fun basicExceptionHandling() {
    try {
        val result = withContext(Dispatchers.IO) {
            if (kotlin.random.Random.nextBoolean()) {
                throw IOException("Network error")
            }
            "Success"
        }
        println("Result: $result")
    } catch (e: IOException) {
        println("IO Exception: ${e.message}")
    } catch (e: Exception) {
        println("General exception: ${e.message}")
    }
}

// Exception handling with async
suspend fun asyncExceptionHandling() {
    coroutineScope {
        try {
            val deferred = async {
                delay(1000)
                throw RuntimeException("Async error")
            }
            
            val result = deferred.await() // Exception thrown here
            println("Result: $result")
        } catch (e: RuntimeException) {
            println("Caught async exception: ${e.message}")
        }
    }
}
```

### CoroutineExceptionHandler
```kotlin
// Global exception handler
val exceptionHandler = CoroutineExceptionHandler { context, exception ->
    println("Coroutine exception in $context: ${exception.message}")
    // Log to crash reporting service
}

fun exceptionHandlerExample() {
    runBlocking {
        val job = launch(exceptionHandler) {
            throw RuntimeException("Unhandled exception")
        }
        
        job.join()
    }
}

// Exception handler with SupervisorJob
fun supervisorJobExample() {
    runBlocking {
        val supervisorJob = SupervisorJob()
        val scope = CoroutineScope(Dispatchers.Default + supervisorJob + exceptionHandler)
        
        scope.launch {
            throw RuntimeException("Child 1 failed")
        }
        
        scope.launch {
            delay(2000)
            println("Child 2 completed successfully")
        }
        
        delay(3000)
        supervisorJob.cancel()
    }
}
```

### Exception Propagation
```kotlin
// launch vs async exception behavior
fun exceptionPropagationExample() {
    runBlocking {
        // launch - exceptions are not propagated to parent until joined
        val launchJob = launch {
            throw RuntimeException("Launch exception")
        }
        
        // async - exceptions are propagated when awaited
        val asyncJob = async {
            throw RuntimeException("Async exception")
        }
        
        try {
            launchJob.join() // Exception propagated here
        } catch (e: Exception) {
            println("Caught launch exception: ${e.message}")
        }
        
        try {
            asyncJob.await() // Exception propagated here
        } catch (e: Exception) {
            println("Caught async exception: ${e.message}")
        }
    }
}
```

## Cancellation & Timeouts

### Job Cancellation
```kotlin
// Basic cancellation
fun basicCancellation() {
    runBlocking {
        val job = launch {
            repeat(1000) { i ->
                if (!isActive) return@launch // Check cancellation
                println("Working... $i")
                delay(100)
            }
        }
        
        delay(500)
        job.cancel() // Cancel the job
        job.join()   // Wait for cancellation to complete
        println("Job cancelled")
    }
}

// Cancellation with reason
fun cancellationWithReason() {
    runBlocking {
        val job = launch {
            try {
                repeat(1000) { i ->
                    ensureActive() // Throws if cancelled
                    println("Working... $i")
                    delay(100)
                }
            } catch (e: CancellationException) {
                println("Cancelled: ${e.message}")
                throw e // Re-throw to properly cancel
            }
        }
        
        delay(500)
        job.cancel("User requested cancellation")
        job.join()
    }
}

// Non-cancellable operations
suspend fun nonCancellableCleanup() {
    try {
        // Cancellable work
        repeat(100) { i ->
            delay(100)
            println("Working $i")
        }
    } finally {
        withContext(NonCancellable) {
            // This block cannot be cancelled
            println("Performing cleanup...")
            delay(1000) // This delay won't be cancelled
            println("Cleanup completed")
        }
    }
}
```

### Timeouts
```kotlin
// withTimeout - throws TimeoutCancellationException
suspend fun withTimeoutExample() {
    try {
        withTimeout(2000) {
            repeat(5) { i ->
                delay(500)
                println("Step $i")
            }
        }
    } catch (e: TimeoutCancellationException) {
        println("Operation timed out")
    }
}

// withTimeoutOrNull - returns null on timeout
suspend fun withTimeoutOrNullExample(): String? {
    return withTimeoutOrNull(1000) {
        delay(2000) // This will timeout
        "Success"
    }
}

// Custom timeout handling
suspend fun customTimeoutHandling() {
    val result = try {
        withTimeout(1500) {
            val data = fetchDataWithRetry()
            "Success: $data"
        }
    } catch (e: TimeoutCancellationException) {
        "Timeout: Operation took too long"
    }
    
    println(result)
}

suspend fun fetchDataWithRetry(): String {
    repeat(3) { attempt ->
        try {
            delay(600) // Simulate network call
            if (kotlin.random.Random.nextBoolean()) {
                return "Data from attempt ${attempt + 1}"
            }
        } catch (e: Exception) {
            if (attempt == 2) throw e
        }
    }
    throw RuntimeException("Failed after retries")
}
```

## Flows

### Basic Flow Operations
```kotlin
import kotlinx.coroutines.flow.*

// Simple flow creation
fun simpleFlow(): Flow<Int> = flow {
    for (i in 1..5) {
        delay(1000)
        emit(i)
    }
}

// Flow from collections
fun flowFromCollection(): Flow<String> {
    return listOf("A", "B", "C", "D").asFlow()
}

// Flow consumption
suspend fun consumeFlow() {
    simpleFlow().collect { value ->
        println("Received: $value")
    }
}

// Flow with exception handling
fun flowWithExceptionHandling(): Flow<String> = flow {
    emit("Start")
    delay(1000)
    emit("Middle")
    throw RuntimeException("Flow error")
    emit("End") // Never reached
}.catch { e ->
    emit("Error: ${e.message}")
}
```

### Flow Transformation Operators
```kotlin
suspend fun flowTransformations() {
    val numbers = (1..10).asFlow()
    
    // map - transform each element
    numbers
        .map { it * it }
        .collect { println("Square: $it") }
    
    // filter - keep elements matching predicate
    numbers
        .filter { it % 2 == 0 }
        .collect { println("Even: $it") }
    
    // transform - more flexible than map
    numbers
        .transform { value ->
            emit("Value: $value")
            emit("Square: ${value * value}")
        }
        .collect { println(it) }
    
    // take - limit number of elements
    numbers
        .take(3)
        .collect { println("First 3: $it") }
    
    // drop - skip first n elements
    numbers
        .drop(3)
        .collect { println("After dropping 3: $it") }
}

// Flow operators with delay
fun delayedFlow(): Flow<String> = flow {
    emit("First")
    delay(1000)
    emit("Second")
    delay(1000)
    emit("Third")
}

suspend fun flowWithDelay() {
    delayedFlow()
        .onEach { println("Emitted: $it") }
        .collect()
}
```

### Flow Combining & Merging
```kotlin
// Combine flows
suspend fun combineFlows() {
    val flow1 = (1..5).asFlow().onEach { delay(1000) }
    val flow2 = listOf("A", "B", "C", "D", "E").asFlow().onEach { delay(1500) }
    
    // combine - combines latest values from both flows
    flow1.combine(flow2) { num, letter ->
        "$num$letter"
    }.collect { println("Combined: $it") }
    
    // zip - pairs corresponding elements
    flow1.zip(flow2) { num, letter ->
        "$num-$letter"
    }.collect { println("Zipped: $it") }
}

// Merge multiple flows
suspend fun mergeFlows() {
    val flow1 = (1..3).asFlow().onEach { delay(1000) }
    val flow2 = (4..6).asFlow().onEach { delay(1500) }
    val flow3 = (7..9).asFlow().onEach { delay(800) }
    
    merge(flow1, flow2, flow3)
        .collect { println("Merged: $it") }
}

// flatMap operations
suspend fun flatMapOperations() {
    val requests = (1..3).asFlow()
    
    // flatMapConcat - sequential processing
    requests
        .flatMapConcat { request ->
            flow {
                emit("Response for request $request - Part 1")
                delay(1000)
                emit("Response for request $request - Part 2")
            }
        }
        .collect { println("Concat: $it") }
    
    // flatMapMerge - concurrent processing
    requests
        .flatMapMerge { request ->
            flow {
                emit("Response for request $request - Part 1")
                delay(1000)
                emit("Response for request $request - Part 2")
            }
        }
        .collect { println("Merge: $it") }
}
```

### StateFlow & SharedFlow
```kotlin
import kotlinx.coroutines.flow.*

// StateFlow - hot flow that holds state
class CounterViewModel {
    private val _counter = MutableStateFlow(0)
    val counter: StateFlow<Int> = _counter.asStateFlow()
    
    fun increment() {
        _counter.value += 1
    }
    
    fun decrement() {
        _counter.value -= 1
    }
}

// SharedFlow - hot flow for events
class EventBus {
    private val _events = MutableSharedFlow<String>()
    val events: SharedFlow<String> = _events.asSharedFlow()
    
    fun sendEvent(event: String) {
        _events.tryEmit(event)
    }
}

// Usage example
suspend fun stateFlowExample() {
    val viewModel = CounterViewModel()
    val eventBus = EventBus()
    
    // Collect StateFlow
    launch {
        viewModel.counter.collect { count ->
            println("Counter: $count")
        }
    }
    
    // Collect SharedFlow
    launch {
        eventBus.events.collect { event ->
            println("Event: $event")
        }
    }
    
    // Update values
    delay(1000)
    viewModel.increment()
    eventBus.sendEvent("Button clicked")
    
    delay(1000)
    viewModel.increment()
    eventBus.sendEvent("Data loaded")
}

// StateFlow vs SharedFlow configuration
class ConfiguredFlows {
    // StateFlow - always has a value, conflated
    private val _state = MutableStateFlow("initial")
    val state: StateFlow<String> = _state
    
    // SharedFlow - configurable replay and buffer
    private val _events = MutableSharedFlow<String>(
        replay = 3,        // Keep last 3 events for new subscribers
        extraBufferCapacity = 10  // Additional buffer capacity
    )
    val events: SharedFlow<String> = _events
    
    // SharedFlow with onBufferOverflow strategy
    private val _criticalEvents = MutableSharedFlow<String>(
        extraBufferCapacity = 5,
        onBufferOverflow = BufferOverflow.DROP_OLDEST
    )
    val criticalEvents: SharedFlow<String> = _criticalEvents
}
```

## Channels

### Basic Channel Operations
```kotlin
import kotlinx.coroutines.channels.*

// Basic channel usage
suspend fun basicChannelExample() {
    val channel = Channel<String>()
    
    launch {
        // Producer
        channel.send("Hello")
        channel.send("World")
        channel.close()
    }
    
    launch {
        // Consumer
        for (message in channel) {
            println("Received: $message")
        }
    }
}

// Buffered channels
suspend fun bufferedChannelExample() {
    val bufferedChannel = Channel<Int>(capacity = 5)
    
    launch {
        repeat(10) { i ->
            println("Sending $i")
            bufferedChannel.send(i)
            println("Sent $i")
        }
        bufferedChannel.close()
    }
    
    launch {
        delay(2000) // Delay consumer to see buffering effect
        for (value in bufferedChannel) {
            println("Received: $value")
            delay(500)
        }
    }
}

// Channel types
suspend fun channelTypes() {
    // Rendezvous channel (capacity = 0)
    val rendezvousChannel = Channel<String>()
    
    // Unlimited channel
    val unlimitedChannel = Channel<String>(Channel.UNLIMITED)
    
    // Conflated channel (keeps only latest)
    val conflatedChannel = Channel<String>(Channel.CONFLATED)
    
    // Buffered channel with specific capacity
    val bufferedChannel = Channel<String>(10)
}
```

### Channel Producers & Consumers
```kotlin
// produce builder - creates channel with producer
fun numberProducer() = produce<Int> {
    for (i in 1..5) {
        delay(1000)
        send(i * i)
    }
}

suspend fun consumeNumbers() {
    val numbers = numberProducer()
    numbers.consumeEach { number ->
        println("Square: $number")
    }
}

// Multiple producers, single consumer
suspend fun multipleProducers() {
    val channel = Channel<String>()
    
    // Producer 1
    launch {
        repeat(3) { i ->
            delay(1000)
            channel.send("Producer1-$i")
        }
    }
    
    // Producer 2
    launch {
        repeat(3) { i ->
            delay(1500)
            channel.send("Producer2-$i")
        }
    }
    
    // Consumer
    launch {
        repeat(6) {
            val message = channel.receive()
            println("Consumed: $message")
        }
        channel.close()
    }
}

// Fan-out pattern (single producer, multiple consumers)
suspend fun fanOutPattern() {
    val producer = produce<Int> {
        var x = 1
        while (true) {
            send(x++)
            delay(100)
        }
    }
    
    repeat(3) { consumerId ->
        launch {
            for (value in producer) {
                println("Consumer $consumerId received: $value")
                delay(1000) // Different processing speeds
            }
        }
    }
    
    delay(5000)
    producer.cancel()
}
```

### Advanced Channel Patterns
```kotlin
// Pipeline pattern
fun pipeline(): ReceiveChannel<String> = produce {
    val numbers = produce {
        var x = 1
        while (true) send(x++)
    }
    
    val squares = produce {
        for (x in numbers) send(x * x)
    }
    
    for (square in squares) {
        send("Square: $square")
    }
}

// Select with channels
suspend fun selectChannelExample() {
    val channel1 = produce {
        while (true) {
            delay(1000)
            send("Channel 1")
        }
    }
    
    val channel2 = produce {
        while (true) {
            delay(1500)
            send("Channel 2")
        }
    }
    
    repeat(10) {
        select<Unit> {
            channel1.onReceive { value ->
                println("From channel1: $value")
            }
            channel2.onReceive { value ->
                println("From channel2: $value")
            }
        }
    }
    
    channel1.cancel()
    channel2.cancel()
}

// Ticker channel
suspend fun tickerExample() {
    val ticker = ticker(delayMillis = 1000, initialDelayMillis = 0)
    
    repeat(5) {
        ticker.receive()
        println("Tick ${it + 1}")
    }
    
    ticker.cancel()
}
```

## Advanced Async Patterns

### Async Sequences
```kotlin
// sequence vs asynchronous sequence
fun regularSequence() = sequence {
    for (i in 1..5) {
        Thread.sleep(1000) // Blocking
        yield(i)
    }
}

fun asyncSequence() = flow {
    for (i in 1..5) {
        delay(1000) // Non-blocking
        emit(i)
    }
}

// Async sequence processing
suspend fun processAsyncSequence() {
    asyncSequence()
        .map { it * 2 }
        .filter { it > 4 }
        .collect { println("Processed: $it") }
}
```

### Parallel Processing
```kotlin
// Parallel map operation
suspend fun parallelMap(items: List<String>): List<String> {
    return items.map { item ->
        async {
            processItem(item)
        }
    }.awaitAll()
}

suspend fun processItem(item: String): String {
    delay(1000) // Simulate processing
    return "Processed: $item"
}

// Controlled parallelism
suspend fun controlledParallelism(items: List<String>, concurrency: Int): List<String> {
    val semaphore = Semaphore(concurrency)
    
    return items.map { item ->
        async {
            semaphore.withPermit {
                processItem(item)
            }
        }
    }.awaitAll()
}

// Parallel collection processing
suspend fun parallelCollectionProcessing() {
    val items = (1..100).toList()
    
    // Process in chunks
    val results = items
        .chunked(10)
        .map { chunk ->
            async {
                chunk.map { processNumber(it) }
            }
        }
        .awaitAll()
        .flatten()
    
    println("Processed ${results.size} items")
}

suspend fun processNumber(number: Int): Int {
    delay(100)
    return number * number
}
```

### Resource Management
```kotlin
// Resource management with coroutines
class DatabaseConnection : AutoCloseable {
    fun query(sql: String): String {
        return "Result for: $sql"
    }
    
    override fun close() {
        println("Database connection closed")
    }
}

suspend fun useResource() {
    val connection = DatabaseConnection()
    try {
        // Use resource
        val result = connection.query("SELECT * FROM users")
        println(result)
    } finally {
        connection.close()
    }
}

// Resource pooling
class ConnectionPool(private val maxConnections: Int) {
    private val connections = Channel<DatabaseConnection>(maxConnections)
    
    init {
        repeat(maxConnections) {
            connections.trySend(DatabaseConnection())
        }
    }
    
    suspend fun <T> useConnection(block: suspend (DatabaseConnection) -> T): T {
        val connection = connections.receive()
        return try {
            block(connection)
        } finally {
            connections.send(connection)
        }
    }
    
    fun close() {
        connections.close()
        // Close all connections
    }
}

// Rate limiting
class RateLimiter(private val permits: Int, private val timeWindow: Long) {
    private val semaphore = Semaphore(permits)
    
    suspend fun <T> execute(block: suspend () -> T): T {
        return semaphore.withPermit {
            val result = block()
            delay(timeWindow / permits) // Simple rate limiting
            result
        }
    }
}

suspend fun rateLimitedOperations() {
    val rateLimiter = RateLimiter(permits = 5, timeWindow = 1000)
    
    repeat(20) { i ->
        launch {
            rateLimiter.execute {
                println("Operation $i at ${System.currentTimeMillis()}")
                // Simulate API call
                delay(100)
            }
        }
    }
}
```

### Error Recovery Patterns
```kotlin
// Retry with exponential backoff
suspend fun <T> retryWithBackoff(
    maxRetries: Int = 3,
    initialDelay: Long = 1000,
    maxDelay: Long = 10000,
    factor: Double = 2.0,
    block: suspend () -> T
): T {
    repeat(maxRetries) { attempt ->
        try {
            return block()
        } catch (e: Exception) {
            if (attempt == maxRetries - 1) throw e
            
            val delay = (initialDelay * factor.pow(attempt.toDouble())).toLong()
                .coerceAtMost(maxDelay)
            
            println("Attempt ${attempt + 1} failed, retrying in ${delay}ms")
            delay(delay)
        }
    }
    error("Should not reach here")
}

// Circuit breaker pattern
enum class CircuitState {
    CLOSED, OPEN, HALF_OPEN
}

class CircuitBreaker(
    private val failureThreshold: Int = 5,
    private val recoveryTimeout: Long = 60000
) {
    private var state = CircuitState.CLOSED
    private var failureCount = 0
    private var lastFailureTime = 0L
    
    suspend fun <T> execute(block: suspend () -> T): T {
        when (state) {
            CircuitState.CLOSED -> {
                return try {
                    val result = block()
                    onSuccess()
                    result
                } catch (e: Exception) {
                    onFailure()
                    throw e
                }
            }
            CircuitState.OPEN -> {
                if (System.currentTimeMillis() - lastFailureTime > recoveryTimeout) {
                    state = CircuitState.HALF_OPEN
                    return execute(block)
                } else {
                    throw RuntimeException("Circuit breaker is OPEN")
                }
            }
            CircuitState.HALF_OPEN -> {
                return try {
                    val result = block()
                    onSuccess()
                    result
                } catch (e: Exception) {
                    onFailure()
                    throw e
                }
            }
        }
    }
    
    private fun onSuccess() {
        failureCount = 0
        state = CircuitState.CLOSED
    }
    
    private fun onFailure() {
        failureCount++
        lastFailureTime = System.currentTimeMillis()
        if (failureCount >= failureThreshold) {
            state = CircuitState.OPEN
        }
    }
}
```