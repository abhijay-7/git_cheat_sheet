## Destructuring Declarations

```kotlin
data class Person(val name: String, val age: Int)

val person = Person("Alice", 30)
val (name, age) = person

// With collections
val list = listOf(1, 2, 3)
val (first, second, third) = list

// In loops
val map = mapOf("a" to 1, "b" to 2)
for ((key, value) in map) {
    println("$key -> $value")
}
```

## Advanced Features

### Inline Functions
```kotlin
inline fun <T> measureTime(block: () -> T): T {
    val start = System.currentTimeMillis()
    val result = block()
    println("Time taken: ${System.currentTimeMillis() - start}ms")
    return result
}

// Reified type parameters
inline fun <reified T> isInstanceOf(value: Any): Boolean {
    return value is T
}
```

### Delegation
```kotlin
// Property delegation
class Example {
    var property: String by Delegates.observable("initial") { _, old, new ->
        println("$old -> $new")
    }
    
    val lazyValue: String by lazy {
        println("Computed!")
        "Hello"
    }
}

// Class delegation
interface Base {
    fun print()
}

class BaseImpl(val x: Int) : Base {
    override fun print() = println(x)
}

class Derived(b: Base) : Base by b
```

### Sealed Classes & Interfaces
```kotlin
sealed class Result<out T>
data class Success<T>(val data: T) : Result<T>()
data class Error(val exception: Throwable) : Result<Nothing>()
object Loading : Result<Nothing>()

fun handleResult(result: Result<String>) = when (result) {
    is Success -> println("Data: ${result.data}")
    is Error -> println("Error: ${result.exception}")
    Loading -> println("Loading...")
}
```

### Value Classes
```kotlin
@JvmInline
value class UserId(val value: String)

@JvmInline
value class Password(val value: String)

fun authenticate(userId: UserId, password: Password) {
    // Type-safe parameters
}
```# Kotlin Cheat Sheet

## Variables & Constants

```kotlin
// Mutable variables
var name = "John"
var age: Int = 25

// Immutable variables (preferred)
val pi = 3.14
val message: String = "Hello"

// Nullable types
var nullableString: String? = null
```

## Basic Data Types

```kotlin
val byte: Byte = 127
val short: Short = 32767
val int: Int = 2147483647
val long: Long = 9223372036854775807L
val float: Float = 3.14f
val double: Double = 3.14159
val char: Char = 'A'
val boolean: Boolean = true
val string: String = "Hello World"
```

## String Templates

```kotlin
val name = "Alice"
val age = 30
val message = "My name is $name and I'm $age years old"
val calculation = "Sum: ${2 + 3}"
```

## Functions

```kotlin
// Basic function
fun greet(name: String): String {
    return "Hello, $name!"
}

// Single expression function
fun add(a: Int, b: Int) = a + b

// Function with default parameters
fun greet(name: String = "World") = "Hello, $name!"

// Function with varargs
fun sum(vararg numbers: Int): Int {
    return numbers.sum()
}

// Extension function
fun String.removeSpaces() = this.replace(" ", "")
```

## Classes & Objects

```kotlin
// Basic class
class Person(val name: String, var age: Int) {
    fun introduce() = "I'm $name, $age years old"
}

// Data class
data class User(val id: Int, val name: String)

// Class with constructor
class Car(brand: String) {
    val brand: String = brand.uppercase()
    
    constructor(brand: String, model: String) : this(brand) {
        println("Model: $model")
    }
}

// Object (singleton)
object DatabaseManager {
    fun connect() = println("Connected to database")
}
```

## Inheritance

```kotlin
open class Animal(val name: String) {
    open fun makeSound() = "Some sound"
}

class Dog(name: String) : Animal(name) {
    override fun makeSound() = "Woof!"
}

// Abstract class
abstract class Shape {
    abstract fun area(): Double
}

class Circle(val radius: Double) : Shape() {
    override fun area() = Math.PI * radius * radius
}
```

## Interfaces

```kotlin
interface Drawable {
    fun draw()
    fun getArea(): Double = 0.0 // Default implementation
}

class Rectangle(val width: Double, val height: Double) : Drawable {
    override fun draw() = println("Drawing rectangle")
    override fun getArea() = width * height
}
```

## Collections

```kotlin
// Lists
val immutableList = listOf(1, 2, 3)
val mutableList = mutableListOf(1, 2, 3)
mutableList.add(4)

// Sets
val immutableSet = setOf(1, 2, 3)
val mutableSet = mutableSetOf(1, 2, 3)

// Maps
val immutableMap = mapOf("key1" to "value1", "key2" to "value2")
val mutableMap = mutableMapOf("key1" to "value1")
mutableMap["key3"] = "value3"

// Arrays
val array = arrayOf(1, 2, 3, 4, 5)
val intArray = intArrayOf(1, 2, 3, 4, 5)
```

## Control Flow

```kotlin
// If-else
val max = if (a > b) a else b

// When (switch equivalent)
when (x) {
    1 -> println("One")
    2, 3 -> println("Two or Three")
    in 4..10 -> println("Between 4 and 10")
    is String -> println("It's a string")
    else -> println("Something else")
}

// When as expression
val result = when (grade) {
    'A' -> "Excellent"
    'B' -> "Good"
    'C' -> "Average"
    else -> "Need improvement"
}
```

## Loops

```kotlin
// For loop
for (i in 1..5) println(i)
for (i in 1 until 5) println(i) // 1 to 4
for (i in 5 downTo 1) println(i)
for (i in 1..10 step 2) println(i)

// For each
val list = listOf(1, 2, 3)
for (item in list) println(item)
list.forEach { println(it) }

// While loop
var i = 0
while (i < 5) {
    println(i)
    i++
}

// Do-while loop
do {
    println("At least once")
} while (false)
```

## Null Safety

```kotlin
var nullable: String? = null

// Safe call operator
val length = nullable?.length

// Elvis operator
val length = nullable?.length ?: 0

// Safe cast
val str: String? = obj as? String

// Not-null assertion (use carefully!)
val length = nullable!!.length
```

## Lambda Expressions & Higher-Order Functions

```kotlin
// Lambda expressions
val square = { x: Int -> x * x }
val sum = { x: Int, y: Int -> x + y }

// Higher-order functions
fun processNumber(x: Int, operation: (Int) -> Int): Int {
    return operation(x)
}

val result = processNumber(5) { it * 2 }

// Common collection functions
val numbers = listOf(1, 2, 3, 4, 5)
val doubled = numbers.map { it * 2 }
val evens = numbers.filter { it % 2 == 0 }
val sum = numbers.reduce { acc, n -> acc + n }
val total = numbers.fold(0) { acc, n -> acc + n }
```

## Scope Functions

```kotlin
// let - transform object and return result
val result = "Hello".let { it.uppercase() }

// run - execute code block and return result
val result = run {
    val x = 10
    val y = 20
    x + y
}

// with - execute code block on object
val result = with(StringBuilder()) {
    append("Hello")
    append(" World")
    toString()
}

// apply - configure object and return it
val person = Person().apply {
    name = "John"
    age = 30
}

// also - perform additional operations and return object
val numbers = mutableListOf(1, 2, 3).also {
    println("Original list: $it")
}
```

## Exception Handling

```kotlin
try {
    val result = 10 / 0
} catch (e: ArithmeticException) {
    println("Division by zero!")
} catch (e: Exception) {
    println("General exception: ${e.message}")
} finally {
    println("Cleanup code")
}

// Try as expression
val result = try {
    parseInt(input)
} catch (e: NumberFormatException) {
    null
}
```

## Coroutines (Advanced)

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.flow.*
import kotlinx.coroutines.channels.*

// Basic coroutine launch
fun basicCoroutines() {
    runBlocking {
        launch {
            delay(1000L)
            println("World!")
        }
        println("Hello")
    }
}

// Async and await
suspend fun fetchUserData(): String {
    delay(1000L)
    return "User data"
}

suspend fun fetchUserPosts(): String {
    delay(1500L)
    return "User posts"
}

fun parallelExecution() {
    runBlocking {
        val userData = async { fetchUserData() }
        val userPosts = async { fetchUserPosts() }
        
        println("${userData.await()} and ${userPosts.await()}")
    }
}

// Coroutine Scopes
class MyRepository {
    private val scope = CoroutineScope(Dispatchers.IO + SupervisorJob())
    
    fun fetchData() {
        scope.launch {
            // Network call
        }
    }
    
    fun cleanup() {
        scope.cancel()
    }
}

// Dispatchers
runBlocking {
    launch(Dispatchers.Main) { /* UI updates */ }
    launch(Dispatchers.IO) { /* File/Network operations */ }
    launch(Dispatchers.Default) { /* CPU intensive work */ }
    launch(Dispatchers.Unconfined) { /* Not confined to any thread */ }
}

// Exception handling in coroutines
fun coroutineExceptionHandling() {
    runBlocking {
        val handler = CoroutineExceptionHandler { _, exception ->
            println("Caught $exception")
        }
        
        launch(handler) {
            throw AssertionError()
        }
        
        // Try-catch in coroutines
        try {
            val result = async { 
                delay(100)
                throw RuntimeException("Something went wrong")
            }
            result.await()
        } catch (e: Exception) {
            println("Caught: ${e.message}")
        }
    }
}

// Flows - Cold streams
fun flowExample() {
    runBlocking {
        val flow = flow {
            for (i in 1..5) {
                delay(1000)
                emit(i)
            }
        }
        
        flow.collect { value ->
            println("Received: $value")
        }
    }
}

// Flow operators
fun flowOperators() {
    runBlocking {
        (1..10).asFlow()
            .filter { it % 2 == 0 }
            .map { it * it }
            .take(3)
            .collect { println(it) }
    }
}

// StateFlow and SharedFlow
class ViewModel {
    private val _uiState = MutableStateFlow(UiState())
    val uiState: StateFlow<UiState> = _uiState.asStateFlow()
    
    private val _events = MutableSharedFlow<Event>()
    val events: SharedFlow<Event> = _events.asSharedFlow()
    
    fun updateState(newState: UiState) {
        _uiState.value = newState
    }
    
    fun sendEvent(event: Event) {
        _events.tryEmit(event)
    }
}

// Channels - Hot streams
fun channelExample() {
    runBlocking {
        val channel = Channel<Int>()
        
        launch {
            for (x in 1..5) {
                channel.send(x * x)
            }
            channel.close()
        }
        
        for (y in channel) {
            println(y)
        }
    }
}

// Select expression - waiting for multiple coroutines
fun selectExample() {
    runBlocking {
        val deferred1 = async { delay(1000); "Result 1" }
        val deferred2 = async { delay(2000); "Result 2" }
        
        val result = select<String> {
            deferred1.onAwait { it }
            deferred2.onAwait { it }
        }
        println("First result: $result")
    }
}

// Cancellation and timeouts
fun cancellationExample() {
    runBlocking {
        val job = launch {
            try {
                repeat(1000) { i ->
                    println("Working... $i")
                    delay(500L)
                }
            } finally {
                println("Cleanup")
            }
        }
        
        delay(1300L)
        job.cancelAndJoin()
        
        // With timeout
        withTimeout(1000L) {
            delay(1500L) // Will throw TimeoutCancellationException
        }
    }
}

// Structured concurrency
class DataRepository {
    suspend fun fetchData() = coroutineScope {
        val data1 = async { fetchFromApi1() }
        val data2 = async { fetchFromApi2() }
        
        CombinedData(data1.await(), data2.await())
    }
    
    private suspend fun fetchFromApi1(): String {
        delay(1000)
        return "API 1 data"
    }
    
    private suspend fun fetchFromApi2(): String {
        delay(1500)
        return "API 2 data"
    }
}
```

## Useful Built-in Functions

```kotlin
// String functions
"hello".capitalize() // "Hello"
"Hello World".split(" ") // ["Hello", "World"]
"  trim me  ".trim() // "trim me"

// Collection functions
listOf(1, 2, 3).joinToString(", ") // "1, 2, 3"
listOf(1, 2, 3).any { it > 2 } // true
listOf(1, 2, 3).all { it > 0 } // true
listOf(1, 2, 3).count { it % 2 == 0 } // 1

// Range functions
(1..10).contains(5) // true
(1..10).random() // random number between 1 and 10
```

## Type Aliases & Generics

```kotlin
// Type alias
typealias UserMap = Map<String, User>

// Generic function
fun <T> List<T>.second(): T = this[1]

// Generic class
class Box<T>(val value: T)

// Bounded generics
fun <T : Number> List<T>.sumAsDouble(): Double {
    return this.sumOf { it.toDouble() }
}
```

## Important Imports by Functionality

### Core Kotlin
```kotlin
import kotlin.collections.*
import kotlin.text.*
import kotlin.math.*
import kotlin.random.Random
import kotlin.time.*
```

### Coroutines
```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.flow.*
import kotlinx.coroutines.channels.*
import kotlinx.coroutines.sync.*
```

### Collections & Sequences
```kotlin
import kotlin.collections.List
import kotlin.collections.Map
import kotlin.collections.Set
import kotlin.sequences.*
```

### JSON Serialization (kotlinx.serialization)
```kotlin
import kotlinx.serialization.*
import kotlinx.serialization.json.*
import kotlinx.serialization.descriptors.*
import kotlinx.serialization.encoding.*
```

### Date & Time
```kotlin
import kotlin.time.Duration
import kotlin.time.TimeSource
import kotlinx.datetime.*
```

### Reflection
```kotlin
import kotlin.reflect.*
import kotlin.reflect.full.*
import kotlin.reflect.jvm.*
```

### Android Development
```kotlin
// Activity & Fragments
import androidx.activity.ComponentActivity
import androidx.fragment.app.Fragment
import androidx.appcompat.app.AppCompatActivity

// Lifecycle
import androidx.lifecycle.*
import androidx.lifecycle.viewmodel.compose.viewModel

// Compose
import androidx.compose.runtime.*
import androidx.compose.ui.*
import androidx.compose.foundation.*
import androidx.compose.material3.*
import androidx.compose.foundation.layout.*

// Navigation
import androidx.navigation.compose.*
import androidx.navigation.fragment.*
```

### Testing
```kotlin
// JUnit
import org.junit.Test
import org.junit.Assert.*
import org.junit.Before
import org.junit.After

// Kotlin Test
import kotlin.test.*

// Mockk (Kotlin mocking)
import io.mockk.*

// Coroutines Testing
import kotlinx.coroutines.test.*
```

### Functional Programming
```kotlin
import arrow.core.*
import arrow.fx.coroutines.*
```

### Networking (Ktor)
```kotlin
import io.ktor.client.*
import io.ktor.client.request.*
import io.ktor.client.response.*
import io.ktor.http.*
```

### Database (Exposed)
```kotlin
import org.jetbrains.exposed.sql.*
import org.jetbrains.exposed.dao.*
import org.jetbrains.exposed.dao.id.*
```