# Kotlin Advanced Concepts Cheat Sheet

## Singleton Patterns

### Object Declaration (Kotlin's Built-in Singleton)
```kotlin
// Thread-safe singleton by default
object DatabaseManager {
    private var connection: String? = null
    
    fun connect(url: String) {
        connection = url
        println("Connected to $url")
    }
    
    fun disconnect() {
        connection = null
        println("Disconnected")
    }
    
    fun isConnected(): Boolean = connection != null
}

// Usage
DatabaseManager.connect("jdbc:mysql://localhost:3306/mydb")
println(DatabaseManager.isConnected()) // true
```

### Class-based Singleton Patterns
```kotlin
// Thread-safe lazy initialization
class Logger private constructor() {
    companion object {
        @Volatile
        private var INSTANCE: Logger? = null
        
        fun getInstance(): Logger {
            return INSTANCE ?: synchronized(this) {
                INSTANCE ?: Logger().also { INSTANCE = it }
            }
        }
    }
    
    fun log(message: String) {
        println("[LOG] $message")
    }
}

// Kotlin idiomatic singleton with lazy delegate
class ConfigManager private constructor() {
    companion object {
        val instance: ConfigManager by lazy { ConfigManager() }
    }
    
    private val properties = mutableMapOf<String, String>()
    
    fun setProperty(key: String, value: String) {
        properties[key] = value
    }
    
    fun getProperty(key: String): String? = properties[key]
}

// Enum singleton (most efficient)
enum class NetworkManager {
    INSTANCE;
    
    fun makeRequest(url: String): String {
        return "Response from $url"
    }
}

// Usage
val response = NetworkManager.INSTANCE.makeRequest("https://api.example.com")
```

### Parameterized Singleton
```kotlin
class DatabaseConnection private constructor(private val url: String) {
    companion object {
        @Volatile
        private var INSTANCE: DatabaseConnection? = null
        
        fun getInstance(url: String): DatabaseConnection {
            return INSTANCE ?: synchronized(this) {
                INSTANCE ?: DatabaseConnection(url).also { INSTANCE = it }
            }
        }
    }
    
    fun query(sql: String): List<String> {
        return listOf("Result from $url: $sql")
    }
}
```

## Companion Objects

### Basic Companion Object
```kotlin
class User(val name: String, val email: String) {
    companion object {
        const val MIN_AGE = 18
        private var userCount = 0
        
        fun createAdmin(name: String): User {
            userCount++
            return User(name, "${name.lowercase()}@admin.com")
        }
        
        fun getUserCount(): Int = userCount
        
        // Factory methods
        fun fromEmail(email: String): User {
            val name = email.substringBefore("@")
            return User(name, email)
        }
    }
    
    init {
        User.userCount++
    }
}

// Usage
val admin = User.createAdmin("John")
val user = User.fromEmail("jane@example.com")
println("Total users: ${User.getUserCount()}")
```

### Named Companion Objects
```kotlin
class MathUtils {
    companion object Calculator {
        fun add(a: Int, b: Int) = a + b
        fun multiply(a: Int, b: Int) = a * b
    }
}

// Usage - both ways work
val sum1 = MathUtils.add(2, 3)
val sum2 = MathUtils.Calculator.add(2, 3)
```

### Companion Object with Interface Implementation
```kotlin
interface JsonSerializer {
    fun toJson(): String
}

class Product(val name: String, val price: Double) {
    companion object : JsonSerializer {
        override fun toJson(): String = """{"type": "Product"}"""
        
        fun create(name: String, price: Double): Product {
            require(price >= 0) { "Price must be non-negative" }
            return Product(name, price)
        }
    }
}
```

### Companion Object Extensions
```kotlin
class Rectangle(val width: Double, val height: Double) {
    companion object
}

// Extension functions on companion object
fun Rectangle.Companion.square(side: Double) = Rectangle(side, side)
fun Rectangle.Companion.unit() = Rectangle(1.0, 1.0)

// Usage
val square = Rectangle.square(5.0)
val unit = Rectangle.unit()
```

## Object Expressions & Anonymous Objects

### Object Expressions
```kotlin
// Anonymous object implementing interface
interface ClickListener {
    fun onClick()
    fun onLongClick()
}

val button = object : ClickListener {
    override fun onClick() {
        println("Button clicked")
    }
    
    override fun onLongClick() {
        println("Button long clicked")
    }
}

// Anonymous object with custom properties/methods
val calculator = object {
    fun add(a: Int, b: Int) = a + b
    fun subtract(a: Int, b: Int) = a - b
    val version = "1.0"
}

println(calculator.add(5, 3))
println("Calculator version: ${calculator.version}")
```

### Local Objects
```kotlin
fun processData(data: List<String>) {
    // Local object for processing logic
    val processor = object {
        fun validate(item: String): Boolean = item.isNotBlank()
        fun transform(item: String): String = item.trim().uppercase()
        fun filter(items: List<String>): List<String> = 
            items.filter { validate(it) }.map { transform(it) }
    }
    
    val processed = processor.filter(data)
    println("Processed: $processed")
}
```

## Delegation Patterns

### Class Delegation
```kotlin
interface Repository {
    fun save(data: String)
    fun load(): String
}

class DatabaseRepository : Repository {
    override fun save(data: String) {
        println("Saving to database: $data")
    }
    
    override fun load(): String {
        return "Data from database"
    }
}

class CachedRepository(
    private val delegate: Repository
) : Repository by delegate {
    private var cache: String? = null
    
    override fun load(): String {
        return cache ?: delegate.load().also { cache = it }
    }
    
    fun clearCache() {
        cache = null
    }
}

// Usage
val dbRepo = DatabaseRepository()
val cachedRepo = CachedRepository(dbRepo)
cachedRepo.save("Important data")  // Delegated to DatabaseRepository
val data = cachedRepo.load()       // Cached version
```

### Property Delegation
```kotlin
import kotlin.properties.Delegates

class PropertyDelegationExample {
    // Lazy property
    val expensiveValue: String by lazy {
        println("Computing expensive value...")
        "Expensive Result"
    }
    
    // Observable property
    var name: String by Delegates.observable("Initial") { prop, old, new ->
        println("${prop.name} changed from $old to $new")
    }
    
    // Vetoable property (can reject changes)
    var age: Int by Delegates.vetoable(0) { prop, old, new ->
        new >= 0 // Only allow non-negative ages
    }
    
    // NotNull property (must be initialized before first access)
    var id: String by Delegates.notNull()
    
    // Map delegation
    private val map = mutableMapOf<String, Any?>()
    var email: String by map
    var isActive: Boolean by map
}

// Custom property delegate
class LoggingDelegate<T>(private var value: T) {
    operator fun getValue(thisRef: Any?, property: kotlin.reflect.KProperty<*>): T {
        println("Getting ${property.name} = $value")
        return value
    }
    
    operator fun setValue(thisRef: Any?, property: kotlin.reflect.KProperty<*>, value: T) {
        println("Setting ${property.name} = $value")
        this.value = value
    }
}

class ExampleWithCustomDelegate {
    var data: String by LoggingDelegate("initial")
}
```

## Advanced Generics

### Generic Constraints
```kotlin
// Upper bound constraint
fun <T : Number> sum(items: List<T>): Double {
    return items.sumOf { it.toDouble() }
}

// Multiple constraints
interface Comparable<T> {
    fun compareTo(other: T): Int
}

fun <T> max(items: List<T>): T? where T : Number, T : Comparable<T> {
    return items.maxOrNull()
}

// Generic functions with reified types
inline fun <reified T> createList(size: Int, factory: () -> T): List<T> {
    return List(size) { factory() }
}

inline fun <reified T> filterByType(items: List<Any>): List<T> {
    return items.filterIsInstance<T>()
}

// Usage
val numbers = createList(5) { kotlin.random.Random.nextInt(100) }
val strings = filterByType<String>(listOf(1, "hello", 2, "world", 3))
```

### Variance (Covariance and Contravariance)
```kotlin
// Covariance (out) - Producer
interface Producer<out T> {
    fun produce(): T
}

class StringProducer : Producer<String> {
    override fun produce(): String = "Hello"
}

// Can assign Producer<String> to Producer<Any>
val stringProducer: Producer<String> = StringProducer()
val anyProducer: Producer<Any> = stringProducer

// Contravariance (in) - Consumer
interface Consumer<in T> {
    fun consume(item: T)
}

class AnyConsumer : Consumer<Any> {
    override fun consume(item: Any) {
        println("Consuming: $item")
    }
}

// Can assign Consumer<Any> to Consumer<String>
val anyConsumer: Consumer<Any> = AnyConsumer()
val stringConsumer: Consumer<String> = anyConsumer

// Invariance - both producer and consumer
interface ProcessorInvariant<T> {
    fun process(input: T): T
}

// PECS (Producer Extends, Consumer Super) example
class DataProcessor {
    fun <T> process(
        source: Producer<out T>,  // Covariant - can read T or its subtypes
        sink: Consumer<in T>      // Contravariant - can write T or its supertypes
    ) {
        val data = source.produce()
        sink.consume(data)
    }
}
```

### Star Projection
```kotlin
// Star projection for unknown generic types
fun handleList(list: List<*>) {
    println("List size: ${list.size}")
    // Can only read elements as Any?
    list.forEach { item ->
        println("Item: $item")
    }
}

// Generic class with star projection
class Container<T>(val item: T)

fun processContainer(container: Container<*>) {
    // Can access the item but only as Any?
    println("Container item: ${container.item}")
}
```

## Sealed Classes & Sealed Interfaces

### Sealed Classes
```kotlin
sealed class Result<out T> {
    data class Success<T>(val data: T) : Result<T>()
    data class Error(val exception: Throwable) : Result<Nothing>()
    object Loading : Result<Nothing>()
    
    // Sealed classes can have abstract members
    abstract fun isComplete(): Boolean
}

// Implementations must override abstract members
data class Success<T>(val data: T) : Result<T>() {
    override fun isComplete(): Boolean = true
}

data class Error(val exception: Throwable) : Result<Nothing>() {
    override fun isComplete(): Boolean = true
}

object Loading : Result<Nothing>() {
    override fun isComplete(): Boolean = false
}

// Pattern matching with when (exhaustive)
fun <T> handleResult(result: Result<T>): String = when (result) {
    is Result.Success -> "Got data: ${result.data}"
    is Result.Error -> "Error: ${result.exception.message}"
    is Result.Loading -> "Loading..."
    // No else needed - compiler knows all cases are covered
}
```

### Sealed Interfaces (Kotlin 1.5+)
```kotlin
sealed interface Shape {
    val area: Double
}

data class Circle(val radius: Double) : Shape {
    override val area: Double get() = Math.PI * radius * radius
}

data class Rectangle(val width: Double, val height: Double) : Shape {
    override val area: Double get() = width * height
}

data class Triangle(val base: Double, val height: Double) : Shape {
    override val area: Double get() = 0.5 * base * height
}

// Multiple inheritance with sealed interfaces
sealed interface Drawable {
    fun draw()
}

sealed interface Resizable {
    fun resize(factor: Double)
}

class Square(private var side: Double) : Shape, Drawable, Resizable {
    override val area: Double get() = side * side
    
    override fun draw() {
        println("Drawing square with side $side")
    }
    
    override fun resize(factor: Double) {
        side *= factor
    }
}
```

### Hierarchical Sealed Classes
```kotlin
sealed class NetworkResponse {
    sealed class Success : NetworkResponse() {
        data class Ok(val data: String) : Success()
        data class Created(val id: String) : Success()
    }
    
    sealed class Error : NetworkResponse() {
        data class ClientError(val code: Int, val message: String) : Error()
        data class ServerError(val code: Int) : Error()
        object NetworkError : Error()
    }
}

fun handleResponse(response: NetworkResponse): String = when (response) {
    is NetworkResponse.Success.Ok -> "Success: ${response.data}"
    is NetworkResponse.Success.Created -> "Created with ID: ${response.id}"
    is NetworkResponse.Error.ClientError -> "Client error ${response.code}: ${response.message}"
    is NetworkResponse.Error.ServerError -> "Server error: ${response.code}"
    is NetworkResponse.Error.NetworkError -> "Network connection failed"
}
```

## Value Classes & Inline Classes

### Value Classes (Kotlin 1.5+)
```kotlin
// Basic value class
@JvmInline
value class UserId(val value: String) {
    init {
        require(value.isNotBlank()) { "UserId cannot be blank" }
    }
}

@JvmInline
value class Email(val value: String) {
    init {
        require(value.contains("@")) { "Invalid email format" }
    }
    
    // Value classes can have methods
    fun domain(): String = value.substringAfter("@")
}

// Type-safe API
fun sendEmail(userId: UserId, email: Email, message: String) {
    println("Sending to user ${userId.value} at ${email.value}: $message")
}

// Usage - prevents mixing up parameters
val userId = UserId("user123")
val email = Email("user@example.com")
sendEmail(userId, email, "Hello!")
// sendEmail(email, userId, "Hello!") // Compile error!
```

### Value Classes with Interfaces
```kotlin
interface Identifiable {
    val id: String
}

@JvmInline
value class CustomerId(override val id: String) : Identifiable {
    fun toDisplayString(): String = "Customer #$id"
}

@JvmInline
value class ProductId(override val id: String) : Identifiable {
    fun toBarcode(): String = "PROD-$id"
}

fun processIdentifiable(item: Identifiable) {
    println("Processing item with ID: ${item.id}")
}
```

## Type Aliases

### Basic Type Aliases
```kotlin
// Simple type aliases
typealias UserName = String
typealias UserId = Int
typealias UserMap = Map<UserId, UserName>

// Function type aliases
typealias EventHandler = (String) -> Unit
typealias Predicate<T> = (T) -> Boolean
typealias Factory<T> = () -> T

// Generic type aliases
typealias StringList = List<String>
typealias StringPair = Pair<String, String>
typealias ResultCallback<T> = (Result<T>) -> Unit

// Usage
val users: UserMap = mapOf(1 to "Alice", 2 to "Bob")
val validator: Predicate<String> = { it.isNotBlank() }
val stringFactory: Factory<String> = { "Default" }

fun handleResult(callback: ResultCallback<String>) {
    val result = Result.success("Data")
    callback(result)
}
```

### Advanced Type Aliases
```kotlin
// Complex nested types
typealias DatabaseConnection = Triple<String, Int, Map<String, String>>
typealias ApiResponse<T> = Pair<Int, Result<T>>
typealias EventBus = Map<String, List<EventHandler>>

// Class member type aliases
class ApiClient {
    typealias Headers = Map<String, String>
    typealias QueryParams = Map<String, String>
    
    fun makeRequest(
        url: String,
        headers: Headers = emptyMap(),
        params: QueryParams = emptyMap()
    ): String {
        return "Request to $url with headers $headers and params $params"
    }
}

// Functional programming type aliases
typealias Mapper<T, R> = (T) -> R
typealias Reducer<T, R> = (R, T) -> R
typealias AsyncTask<T> = suspend () -> T

fun <T, R> List<T>.mapWith(mapper: Mapper<T, R>): List<R> = this.map(mapper)
fun <T, R> List<T>.reduceWith(initial: R, reducer: Reducer<T, R>): R = 
    this.fold(initial, reducer)
```

## Contracts & Smart Casts

### Contracts (Experimental)
```kotlin
import kotlin.contracts.*

// Custom require function with contract
@OptIn(ExperimentalContracts::class)
fun requireNotNull(value: Any?, message: String): Unit {
    contract {
        returns() implies (value != null)
    }
    if (value == null) throw IllegalArgumentException(message)
}

// Contract with boolean result
@OptIn(ExperimentalContracts::class)
fun String?.isNotNullOrEmpty(): Boolean {
    contract {
        returns(true) implies (this@isNotNullOrEmpty != null)
    }
    return this != null && this.isNotEmpty()
}

// Usage - smart cast works because of contract
fun processString(input: String?) {
    if (input.isNotNullOrEmpty()) {
        // Smart cast: input is now String (not String?)
        println(input.uppercase())
    }
}
```

### Smart Casts
```kotlin
// Smart cast with type checking
fun processAny(value: Any) {
    if (value is String) {
        // Smart cast to String
        println("String length: ${value.length}")
        println("Uppercase: ${value.uppercase()}")
    } else if (value is List<*>) {
        // Smart cast to List
        println("List size: ${value.size}")
    }
}

// Smart cast with null checking
fun processNullable(value: String?) {
    if (value != null) {
        // Smart cast to String (non-null)
        println("Length: ${value.length}")
    }
    
    // Elvis operator doesn't provide smart cast
    val length = value?.length ?: return
    // value is still String? here
}

// Safe cast with smart cast
fun safeCastExample(value: Any) {
    val stringValue = value as? String
    if (stringValue != null) {
        // Smart cast: stringValue is String (not String?)
        println("String: ${stringValue.uppercase()}")
    }
}
```