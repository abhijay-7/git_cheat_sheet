# Kotlin Advanced Operations Cheat Sheet

## Networking & HTTP

### Basic HTTP with Ktor Client
```kotlin
import io.ktor.client.*
import io.ktor.client.engine.cio.*
import io.ktor.client.request.*
import io.ktor.client.response.*
import io.ktor.client.statement.*
import io.ktor.http.*
import io.ktor.client.plugins.contentnegotiation.*
import io.ktor.serialization.kotlinx.json.*
import kotlinx.serialization.Serializable

@Serializable
data class User(val id: Int, val name: String, val email: String)

// Create HTTP client
val client = HttpClient(CIO) {
    install(ContentNegotiation) {
        json()
    }
}

// GET request
suspend fun fetchUser(id: Int): User {
    return client.get("https://api.example.com/users/$id")
}

// POST request
suspend fun createUser(user: User): User {
    return client.post("https://api.example.com/users") {
        contentType(ContentType.Application.Json)
        setBody(user)
    }
}

// PUT request
suspend fun updateUser(id: Int, user: User): User {
    return client.put("https://api.example.com/users/$id") {
        contentType(ContentType.Application.Json)
        setBody(user)
    }
}

// DELETE request
suspend fun deleteUser(id: Int): HttpResponse {
    return client.delete("https://api.example.com/users/$id")
}

// Download file
suspend fun downloadFile(url: String, outputFile: String) {
    client.get(url).bodyAsChannel().copyAndClose(
        File(outputFile).writeChannel()
    )
}

// Custom headers and parameters
suspend fun fetchWithHeaders(): String {
    return client.get("https://api.example.com/data") {
        headers {
            append(HttpHeaders.Authorization, "Bearer token")
            append(HttpHeaders.UserAgent, "MyApp/1.0")
        }
        parameter("page", 1)
        parameter("limit", 10)
    }.bodyAsText()
}
```

### Using Java's Built-in HTTP Client
```kotlin
import java.net.http.HttpClient
import java.net.http.HttpRequest
import java.net.http.HttpResponse
import java.net.URI
import java.time.Duration

val client = HttpClient.newBuilder()
    .connectTimeout(Duration.ofSeconds(10))
    .build()

// GET request
fun fetchData(url: String): String {
    val request = HttpRequest.newBuilder()
        .uri(URI.create(url))
        .timeout(Duration.ofSeconds(30))
        .header("Accept", "application/json")
        .GET()
        .build()
    
    val response = client.send(request, HttpResponse.BodyHandlers.ofString())
    return response.body()
}

// POST request
fun postData(url: String, json: String): String {
    val request = HttpRequest.newBuilder()
        .uri(URI.create(url))
        .timeout(Duration.ofSeconds(30))
        .header("Content-Type", "application/json")
        .POST(HttpRequest.BodyPublishers.ofString(json))
        .build()
    
    val response = client.send(request, HttpResponse.BodyHandlers.ofString())
    return response.body()
}
```

## Sockets

### TCP Socket Server
```kotlin
import java.net.ServerSocket
import java.net.Socket
import java.io.*
import kotlinx.coroutines.*

class TcpServer(private val port: Int) {
    private var serverSocket: ServerSocket? = null
    private val scope = CoroutineScope(Dispatchers.IO)
    
    fun start() {
        serverSocket = ServerSocket(port)
        println("Server started on port $port")
        
        scope.launch {
            while (true) {
                try {
                    val clientSocket = serverSocket?.accept()
                    clientSocket?.let { socket ->
                        launch { handleClient(socket) }
                    }
                } catch (e: Exception) {
                    println("Error accepting client: ${e.message}")
                    break
                }
            }
        }
    }
    
    private suspend fun handleClient(socket: Socket) {
        withContext(Dispatchers.IO) {
            try {
                val input = BufferedReader(InputStreamReader(socket.getInputStream()))
                val output = PrintWriter(socket.getOutputStream(), true)
                
                var inputLine: String?
                while (input.readLine().also { inputLine = it } != null) {
                    println("Received: $inputLine")
                    output.println("Echo: $inputLine")
                    
                    if (inputLine == "bye") break
                }
                
                socket.close()
            } catch (e: Exception) {
                println("Error handling client: ${e.message}")
            }
        }
    }
    
    fun stop() {
        serverSocket?.close()
        scope.cancel()
    }
}
```

### TCP Socket Client
```kotlin
import java.net.Socket
import java.io.*

class TcpClient(private val host: String, private val port: Int) {
    private var socket: Socket? = null
    private var output: PrintWriter? = null
    private var input: BufferedReader? = null
    
    fun connect(): Boolean {
        return try {
            socket = Socket(host, port)
            output = PrintWriter(socket!!.getOutputStream(), true)
            input = BufferedReader(InputStreamReader(socket!!.getInputStream()))
            true
        } catch (e: Exception) {
            println("Connection failed: ${e.message}")
            false
        }
    }
    
    fun sendMessage(message: String): String? {
        return try {
            output?.println(message)
            input?.readLine()
        } catch (e: Exception) {
            println("Error sending message: ${e.message}")
            null
        }
    }
    
    fun disconnect() {
        try {
            input?.close()
            output?.close()
            socket?.close()
        } catch (e: Exception) {
            println("Error closing connection: ${e.message}")
        }
    }
}

// Usage
fun main() {
    val client = TcpClient("localhost", 8080)
    if (client.connect()) {
        val response = client.sendMessage("Hello Server!")
        println("Server response: $response")
        client.disconnect()
    }
}
```

### UDP Socket
```kotlin
import java.net.*

class UdpServer(private val port: Int) {
    private var socket: DatagramSocket? = null
    
    fun start() {
        socket = DatagramSocket(port)
        println("UDP Server started on port $port")
        
        val buffer = ByteArray(1024)
        
        while (true) {
            try {
                val packet = DatagramPacket(buffer, buffer.size)
                socket?.receive(packet)
                
                val message = String(packet.data, 0, packet.length)
                println("Received: $message from ${packet.address}")
                
                // Echo back
                val response = "Echo: $message".toByteArray()
                val responsePacket = DatagramPacket(
                    response, response.size, packet.address, packet.port
                )
                socket?.send(responsePacket)
                
            } catch (e: Exception) {
                println("Error: ${e.message}")
                break
            }
        }
    }
    
    fun stop() {
        socket?.close()
    }
}

class UdpClient(private val host: String, private val port: Int) {
    private var socket: DatagramSocket? = null
    
    fun sendMessage(message: String): String? {
        return try {
            socket = DatagramSocket()
            val data = message.toByteArray()
            val address = InetAddress.getByName(host)
            
            val packet = DatagramPacket(data, data.size, address, port)
            socket?.send(packet)
            
            val buffer = ByteArray(1024)
            val responsePacket = DatagramPacket(buffer, buffer.size)
            socket?.receive(responsePacket)
            
            String(responsePacket.data, 0, responsePacket.length)
        } catch (e: Exception) {
            println("Error: ${e.message}")
            null
        } finally {
            socket?.close()
        }
    }
}
```

## OS Commands & Process Management

### Execute System Commands
```kotlin
import java.io.*
import kotlin.concurrent.thread

// Simple command execution
fun executeCommand(command: String): String {
    return try {
        val process = ProcessBuilder(*command.split(" ").toTypedArray())
            .redirectErrorStream(true)
            .start()
        
        process.inputStream.bufferedReader().use { it.readText() }
    } catch (e: Exception) {
        "Error: ${e.message}"
    }
}

// Execute with timeout
fun executeCommandWithTimeout(command: String, timeoutSeconds: Long): String? {
    return try {
        val process = ProcessBuilder(*command.split(" ").toTypedArray())
            .redirectErrorStream(true)
            .start()
        
        val completed = process.waitFor(timeoutSeconds, java.util.concurrent.TimeUnit.SECONDS)
        
        if (completed) {
            process.inputStream.bufferedReader().use { it.readText() }
        } else {
            process.destroyForcibly()
            "Command timed out"
        }
    } catch (e: Exception) {
        "Error: ${e.message}"
    }
}

// Execute with real-time output
fun executeCommandRealTime(command: String, onOutput: (String) -> Unit) {
    try {
        val process = ProcessBuilder(*command.split(" ").toTypedArray())
            .redirectErrorStream(true)
            .start()
        
        thread {
            process.inputStream.bufferedReader().useLines { lines ->
                lines.forEach { line ->
                    onOutput(line)
                }
            }
        }
        
        process.waitFor()
    } catch (e: Exception) {
        onOutput("Error: ${e.message}")
    }
}

// Execute with input/output streams
class ProcessExecutor {
    fun executeInteractive(command: String): Process {
        return ProcessBuilder(*command.split(" ").toTypedArray())
            .redirectInput(ProcessBuilder.Redirect.PIPE)
            .redirectOutput(ProcessBuilder.Redirect.PIPE)
            .redirectError(ProcessBuilder.Redirect.PIPE)
            .start()
    }
    
    fun sendInput(process: Process, input: String) {
        process.outputStream.bufferedWriter().use { writer ->
            writer.write(input)
            writer.newLine()
            writer.flush()
        }
    }
    
    fun readOutput(process: Process): String {
        return process.inputStream.bufferedReader().use { it.readText() }
    }
}

// Environment variables and working directory
fun executeWithEnvironment(command: String, workingDir: String, env: Map<String, String>): String {
    return try {
        val processBuilder = ProcessBuilder(*command.split(" ").toTypedArray())
            .directory(File(workingDir))
            .redirectErrorStream(true)
        
        // Add environment variables
        processBuilder.environment().putAll(env)
        
        val process = processBuilder.start()
        process.inputStream.bufferedReader().use { it.readText() }
    } catch (e: Exception) {
        "Error: ${e.message}"
    }
}
```

## File I/O Operations

### Basic File Operations
```kotlin
import java.io.*
import java.nio.file.*
import java.nio.charset.StandardCharsets
import kotlin.io.path.*

// Read entire file
fun readFileAsString(filePath: String): String {
    return File(filePath).readText()
}

// Read file lines
fun readFileLines(filePath: String): List<String> {
    return File(filePath).readLines()
}

// Write to file
fun writeToFile(filePath: String, content: String) {
    File(filePath).writeText(content)
}

// Append to file
fun appendToFile(filePath: String, content: String) {
    File(filePath).appendText(content)
}

// Read file with specific encoding
fun readFileWithEncoding(filePath: String, charset: java.nio.charset.Charset = StandardCharsets.UTF_8): String {
    return Files.readString(Paths.get(filePath), charset)
}

// Copy file
fun copyFile(source: String, destination: String) {
    Files.copy(Paths.get(source), Paths.get(destination), StandardCopyOption.REPLACE_EXISTING)
}

// Move/rename file
fun moveFile(source: String, destination: String) {
    Files.move(Paths.get(source), Paths.get(destination), StandardCopyOption.REPLACE_EXISTING)
}

// Delete file
fun deleteFile(filePath: String): Boolean {
    return File(filePath).delete()
}

// Check file exists
fun fileExists(filePath: String): Boolean {
    return File(filePath).exists()
}

// Get file info
fun getFileInfo(filePath: String): FileInfo? {
    val file = File(filePath)
    return if (file.exists()) {
        FileInfo(
            name = file.name,
            size = file.length(),
            lastModified = file.lastModified(),
            isDirectory = file.isDirectory,
            isReadable = file.canRead(),
            isWritable = file.canWrite(),
            isExecutable = file.canExecute()
        )
    } else null
}

data class FileInfo(
    val name: String,
    val size: Long,
    val lastModified: Long,
    val isDirectory: Boolean,
    val isReadable: Boolean,
    val isWritable: Boolean,
    val isExecutable: Boolean
)
```

### Advanced File Operations
```kotlin
// Read large files efficiently
fun readLargeFile(filePath: String, onLine: (String) -> Unit) {
    File(filePath).bufferedReader().use { reader ->
        reader.lineSequence().forEach { line ->
            onLine(line)
        }
    }
}

// Write large files efficiently
fun writeLargeFile(filePath: String, lines: Sequence<String>) {
    File(filePath).bufferedWriter().use { writer ->
        lines.forEach { line ->
            writer.write(line)
            writer.newLine()
        }
    }
}

// Directory operations
fun listDirectory(dirPath: String): List<File> {
    return File(dirPath).listFiles()?.toList() ?: emptyList()
}

fun createDirectory(dirPath: String): Boolean {
    return File(dirPath).mkdirs()
}

fun deleteDirectoryRecursively(dirPath: String): Boolean {
    return File(dirPath).deleteRecursively()
}

// Walk directory tree
fun walkDirectory(dirPath: String, onFile: (File) -> Unit) {
    File(dirPath).walk().forEach { file ->
        if (file.isFile) {
            onFile(file)
        }
    }
}

// File watching
import java.nio.file.StandardWatchEventKinds.*
import java.nio.file.WatchService

fun watchDirectory(dirPath: String, onChange: (Path, String) -> Unit) {
    val watchService = FileSystems.getDefault().newWatchService()
    val path = Paths.get(dirPath)
    
    path.register(watchService, ENTRY_CREATE, ENTRY_DELETE, ENTRY_MODIFY)
    
    while (true) {
        val key = watchService.take()
        
        for (event in key.pollEvents()) {
            val kind = event.kind()
            val fileName = event.context() as Path
            
            onChange(fileName, kind.name())
        }
        
        val valid = key.reset()
        if (!valid) break
    }
}

// Temporary files
fun createTempFile(prefix: String, suffix: String): File {
    return kotlin.io.createTempFile(prefix, suffix)
}

fun createTempDirectory(prefix: String): File {
    return kotlin.io.createTempDir(prefix)
}
```

## List Methods & Operations

### List Creation & Basic Operations
```kotlin
// Creation
val list = listOf(1, 2, 3, 4, 5)
val mutableList = mutableListOf(1, 2, 3)
val emptyList = emptyList<Int>()
val listOf100 = List(100) { it }

// Access
val first = list.first()
val last = list.last()
val element = list[2]
val elementOrNull = list.getOrNull(10)
val elementOrDefault = list.getOrElse(10) { -1 }

// Modification (mutable lists)
mutableList.add(4)
mutableList.addAll(listOf(5, 6, 7))
mutableList.remove(2)
mutableList.removeAt(0)
mutableList.clear()

// Size and emptiness
val size = list.size
val isEmpty = list.isEmpty()
val isNotEmpty = list.isNotEmpty()
```

### List Transformation Methods
```kotlin
val numbers = listOf(1, 2, 3, 4, 5, 6, 7, 8, 9, 10)

// map - transform each element
val doubled = numbers.map { it * 2 }
val strings = numbers.map { "Number: $it" }

// mapIndexed - transform with index
val withIndex = numbers.mapIndexed { index, value -> "$index: $value" }

// mapNotNull - transform and filter nulls
val evenDoubled = numbers.mapNotNull { if (it % 2 == 0) it * 2 else null }

// flatMap - flatten nested collections
val nestedLists = listOf(listOf(1, 2), listOf(3, 4), listOf(5, 6))
val flattened = nestedLists.flatMap { it }

// flatten - flatten directly
val alsoFlattened = nestedLists.flatten()

// associate - create map from list
val numberMap = numbers.associate { it to it * it }
val groupedByParity = numbers.associateBy { it % 2 }
val valueMapped = numbers.associateWith { it * it }
```

### List Filtering Methods
```kotlin
val numbers = listOf(1, 2, 3, 4, 5, 6, 7, 8, 9, 10)

// filter - keep elements matching predicate
val evens = numbers.filter { it % 2 == 0 }
val odds = numbers.filterNot { it % 2 == 0 }

// filterIndexed - filter with index
val evenIndexed = numbers.filterIndexed { index, _ -> index % 2 == 0 }

// filterIsInstance - filter by type
val mixed = listOf(1, "two", 3, "four", 5)
val ints = mixed.filterIsInstance<Int>()
val strings = mixed.filterIsInstance<String>()

// filterNotNull - remove nulls
val withNulls = listOf(1, null, 2, null, 3)
val noNulls = withNulls.filterNotNull()

// take/drop operations
val firstThree = numbers.take(3)
val lastThree = numbers.takeLast(3)
val dropFirstThree = numbers.drop(3)
val dropLastThree = numbers.dropLast(3)

// takeWhile/dropWhile
val takeWhileLessThan5 = numbers.takeWhile { it < 5 }
val dropWhileLessThan5 = numbers.dropWhile { it < 5 }

// distinct operations
val duplicates = listOf(1, 2, 2, 3, 3, 3, 4)
val unique = duplicates.distinct()
val uniqueBy = duplicates.distinctBy { it % 2 }
```

### List Aggregation Methods
```kotlin
val numbers = listOf(1, 2, 3, 4, 5)

// reduce operations
val sum = numbers.reduce { acc, n -> acc + n }
val product = numbers.reduce { acc, n -> acc * n }

// fold operations
val sumWithInitial = numbers.fold(0) { acc, n -> acc + n }
val stringConcat = numbers.fold("") { acc, n -> acc + n }

// Built-in aggregations
val sum2 = numbers.sum()
val sumOf = numbers.sumOf { it * 2 }
val average = numbers.average()
val max = numbers.maxOrNull()
val min = numbers.minOrNull()
val maxBy = numbers.maxByOrNull { it % 3 }
val minBy = numbers.minByOrNull { it % 3 }

// count operations
val count = numbers.count()
val countEvens = numbers.count { it % 2 == 0 }

// any/all/none
val hasEvens = numbers.any { it % 2 == 0 }
val allPositive = numbers.all { it > 0 }
val noneNegative = numbers.none { it < 0 }
```

### List Ordering Methods
```kotlin
val numbers = listOf(3, 1, 4, 1, 5, 9, 2, 6)
val words = listOf("banana", "apple", "cherry", "date")

// Basic sorting
val sorted = numbers.sorted()
val sortedDesc = numbers.sortedDescending()

// Sort by property
val sortedByLength = words.sortedBy { it.length }
val sortedByLengthDesc = words.sortedByDescending { it.length }

// Custom sorting
val customSorted = numbers.sortedWith { a, b -> b.compareTo(a) }

// Reverse
val reversed = numbers.reversed()

// Shuffle
val shuffled = numbers.shuffled()

// In-place sorting (mutable lists)
val mutableNumbers = numbers.toMutableList()
mutableNumbers.sort()
mutableNumbers.sortBy { it % 3 }
mutableNumbers.reverse()
mutableNumbers.shuffle()
```

### List Grouping & Partitioning
```kotlin
val numbers = listOf(1, 2, 3, 4, 5, 6, 7, 8, 9, 10)
val words = listOf("apple", "banana", "cherry", "date", "elderberry")

// groupBy - group into map
val groupedByParity = numbers.groupBy { it % 2 }
val groupedByLength = words.groupBy { it.length }

// partition - split into two lists
val (evens, odds) = numbers.partition { it % 2 == 0 }

// chunked - split into chunks
val chunks = numbers.chunked(3)
val chunkedWithTransform = numbers.chunked(3) { it.sum() }

// windowed - sliding window
val windowed = numbers.windowed(3)
val windowedWithStep = numbers.windowed(3, 2)
val windowedWithTransform = numbers.windowed(3) { it.sum() }

// zip operations
val letters = listOf("a", "b", "c", "d")
val zipped = numbers.zip(letters)
val zippedWithTransform = numbers.zip(letters) { n, l -> "$l$n" }

// unzip
val pairs = listOf(1 to "a", 2 to "b", 3 to "c")
val (nums, chars) = pairs.unzip()
```

## Map Methods & Operations

### Map Creation & Basic Operations
```kotlin
// Creation
val map = mapOf("a" to 1, "b" to 2, "c" to 3)
val mutableMap = mutableMapOf("a" to 1, "b" to 2)
val emptyMap = emptyMap<String, Int>()

// Access
val value = map["a"]
val valueOrNull = map.get("a")
val valueOrDefault = map.getOrDefault("d", 0)
val valueOrElse = map.getOrElse("d") { 0 }

// Modification (mutable maps)
mutableMap["d"] = 4
mutableMap.put("e", 5)
mutableMap.putAll(mapOf("f" to 6, "g" to 7))
mutableMap.remove("b")
mutableMap.clear()

// Properties
val size = map.size
val isEmpty = map.isEmpty()
val keys = map.keys
val values = map.values
val entries = map.entries
```

### Map Transformation Methods
```kotlin
val map = mapOf("a" to 1, "b" to 2, "c" to 3)

// map transformations
val doubledValues = map.mapValues { (_, value) -> value * 2 }
val uppercaseKeys = map.mapKeys { (key, _) -> key.uppercase() }
val stringEntries = map.map { (key, value) -> "$key=$value" }

// filterKeys/filterValues
val filteredByKey = map.filterKeys { it > "a" }
val filteredByValue = map.filterValues { it > 1 }
val filteredEntries = map.filter { (key, value) -> key > "a" && value > 1 }

// toList operations
val entryList = map.toList()
val keyList = map.keys.toList()
val valueList = map.values.toList()
```

### Map Merge & Combine Operations
```kotlin
val map1 = mapOf("a" to 1, "b" to 2)
val map2 = mapOf("b" to 3, "c" to 4)

// plus operator (right side wins on conflicts)
val combined = map1 + map2

// merge with custom logic
val merged = map1.toMutableMap().apply {
    map2.forEach { (key, value) ->
        merge(key, value) { old, new -> old + new }
    }
}

// Group multiple maps
val maps = listOf(map1, map2)
val allEntries = maps.flatMap { it.entries }
val groupedByKey = allEntries.groupBy({ it.key }, { it.value })
```

## Type Conversion Methods

### Primitive Type Conversions
```kotlin
// Number conversions
val intValue = 42
val byteValue = intValue.toByte()
val shortValue = intValue.toShort()
val longValue = intValue.toLong()
val floatValue = intValue.toFloat()
val doubleValue = intValue.toDouble()

// String conversions
val stringValue = intValue.toString()
val binaryString = intValue.toString(2)
val hexString = intValue.toString(16)

// From string
val parsedInt = "42".toInt()
val parsedIntOrNull = "invalid".toIntOrNull()
val parsedDouble = "3.14".toDouble()
val parsedBoolean = "true".toBoolean()

// Character conversions
val charValue = 'A'
val charToInt = charValue.code  // ASCII/Unicode value
val intToChar = 65.toChar()     // 'A'
```

### Collection Type Conversions
```kotlin
val list = listOf(1, 2, 3, 4, 5)

// List conversions
val mutableList = list.toMutableList()
val set = list.toSet()
val mutableSet = list.toMutableSet()
val array = list.toTypedArray()
val intArray = list.toIntArray()

// Array conversions
val array2 = arrayOf(1, 2, 3, 4, 5)
val listFromArray = array2.toList()
val setFromArray = array2.toSet()

// Map conversions
val pairs = listOf("a" to 1, "b" to 2, "c" to 3)
val mapFromPairs = pairs.toMap()
val mutableMapFromPairs = pairs.toMutableMap()

val map = mapOf("a" to 1, "b" to 2)
val listFromMap = map.toList()
val entriesSet = map.entries.toSet()

// Sequence conversions
val sequence = list.asSequence()
val listFromSequence = sequence.toList()

// Type-specific conversions
val strings = listOf("1", "2", "3")
val ints = strings.map { it.toInt() }
val intsOrNulls = strings.map { it.toIntOrNull() }
```

### Advanced Type Conversions
```kotlin
// Safe casting
val any: Any = "Hello"
val string = any as? String
val int = any as? Int  // null

// Type checking
val isString = any is String
val isNotString = any !is String

// Generic type conversions
inline fun <reified T> Any?.safeCast(): T? = this as? T

val result = "Hello".safeCast<String>()  // "Hello"
val nullResult = "Hello".safeCast<Int>() // null

// Collection element type conversion
val mixedList = listOf<Any>(1, "2", 3.0, "4")
val strings2 = mixedList.filterIsInstance<String>()
val numbers = mixedList.filterIsInstance<Number>()

// Custom conversions with extension functions
fun String.toIntList(): List<Int> {
    return this.split(",").mapNotNull { it.trim().toIntOrNull() }
}

val intList = "1, 2, 3, 4".toIntList()

// Enum conversions
enum class Color { RED, GREEN, BLUE }

fun String.toColor(): Color? = try {
    Color.valueOf(this.uppercase())
} catch (e: IllegalArgumentException) {
    null
}

val color = "red".toColor()  // Color.RED
```

## Regular Expressions & String Processing

### Regex Operations
```kotlin
// Create regex patterns
val pattern = Regex("\\d+")
val emailPattern = Regex("""[\w._%+-]+@[\w.-]+\.[A-Za-z]{2,}""")
val phonePattern = Regex("""(\d{3})-(\d{3})-(\d{4})""")

// Find operations
val text = "Call me at 123-456-7890 or email test@example.com"
val phoneMatch = phonePattern.find(text)
val allMatches = pattern.findAll("Numbers: 123, 456, 789").toList()

// Match operations
val isValidEmail = emailPattern.matches("user@domain.com")
val containsNumber = pattern.containsMatchIn("abc123def")

// Replace operations
val cleanedText = text.replace(Regex("\\d"), "*")
val formattedPhone = text.replace(phonePattern) { matchResult ->
    "(${ matchResult.groupValues[1] }) ${ matchResult.groupValues[2] }-${ matchResult.groupValues[3] }"
}

// Split with regex
val parts = "word1,word2;word3:word4".split(Regex("[,:;]"))

// Regex groups
val match = phonePattern.find("My number is 555-123-4567")
match?.let {
    val (areaCode, exchange, number) = it.destructured
    println("Area: $areaCode, Exchange: $exchange, Number: $number")
}
```

### Advanced String Operations
```kotlin
// String building
val builder = StringBuilder()
builder.append("Hello")
    .append(" ")
    .append("World")
val result = builder.toString()

// String formatting
val formatted = String.format("Name: %s, Age: %d, Score: %.2f", "John", 25, 95.67)
val templated = "Name: $name, Age: $age"

// String manipulation
val text = "  Hello World  "
val trimmed = text.trim()
val padded = "Hi".padStart(10, '0')  // "00000000Hi"
val paddedEnd = "Hi".padEnd(10, '0') // "Hi00000000"

// Case operations
val upper = text.uppercase()
val lower = text.lowercase()
val capitalized = text.replaceFirstChar { it.uppercase() }

// Substring operations
val substring = text.substring(2, 7)
val substringAfter = text.substringAfter("Hello")
val substringBefore = text.substringBefore("World")
val substringAfterLast = "path/to/file.txt".substringAfterLast("/")

// String validation
fun String.isValidEmail(): Boolean = 
    Regex("""[\w._%+-]+@[\w.-]+\.[A-Za-z]{2,}""").matches(this)

fun String.isNumeric(): Boolean = 
    this.all { it.isDigit() }

fun String.isAlphaNumeric(): Boolean = 
    this.all { it.isLetterOrDigit() }
```

## Date & Time Operations

### Modern Date-Time API (kotlinx-datetime)
```kotlin
import kotlinx.datetime.*
import kotlin.time.Duration.Companion.days
import kotlin.time.Duration.Companion.hours

// Current date/time
val now = Clock.System.now()
val today = Clock.System.todayIn(TimeZone.currentSystemDefault())

// Create specific dates
val specificDate = LocalDate(2024, 8, 1)
val specificDateTime = LocalDateTime(2024, 8, 1, 14, 30, 0)
val specificInstant = Instant.parse("2024-08-01T14:30:00Z")

// Date arithmetic
val tomorrow = today.plus(1.days)
val nextWeek = today.plus(7.days)
val lastMonth = today.minus(30.days)

// Time zones
val utcTime = now
val localTime = now.toLocalDateTime(TimeZone.currentSystemDefault())
val nyTime = now.toLocalDateTime(TimeZone.of("America/New_York"))

// Formatting
val formatted = today.toString()  // ISO format: 2024-08-01
val customFormat = "${today.dayOfMonth}/${today.monthNumber}/${today.year}"

// Parsing
val parsedDate = LocalDate.parse("2024-08-01")
val parsedDateTime = LocalDateTime.parse("2024-08-01T14:30:00")

// Duration calculations
val duration = specificInstant - now
val daysBetween = today.daysUntil(specificDate)

// Date ranges
val dateRange = today..tomorrow
val isInRange = specificDate in dateRange
```

### Java Date-Time Interop
```kotlin
import java.time.*
import java.time.format.DateTimeFormatter

// Java LocalDate/LocalDateTime
val javaDate = LocalDate.now()
val javaDateTime = LocalDateTime.now()
val javaInstant = Instant.now()

// Formatting
val formatter = DateTimeFormatter.ofPattern("dd/MM/yyyy HH:mm:ss")
val formattedDate = javaDateTime.format(formatter)

// Parsing with custom format
val customFormatter = DateTimeFormatter.ofPattern("MM-dd-yyyy")
val parsedDate2 = LocalDate.parse("08-01-2024", customFormatter)

// Time calculations
val nextHour = javaDateTime.plusHours(1)
val yesterday = javaDate.minusDays(1)
val duration2 = Duration.between(javaDateTime, nextHour)
val period = Period.between(javaDate, yesterday)
```

## Functional Programming Extensions

### Higher-Order Functions
```kotlin
// Function composition
fun <A, B, C> compose(f: (B) -> C, g: (A) -> B): (A) -> C = { a -> f(g(a)) }

val addOne = { x: Int -> x + 1 }
val multiplyByTwo = { x: Int -> x * 2 }
val addThenMultiply = compose(multiplyByTwo, addOne)

// Currying
fun add(x: Int): (Int) -> Int = { y -> x + y }
val add5 = add(5)
val result = add5(3)  // 8

// Partial application
fun multiply(x: Int, y: Int, z: Int): Int = x * y * z
fun partialMultiply(x: Int, y: Int): (Int) -> Int = { z -> multiply(x, y, z) }

// Memoization
fun <T, R> memoize(fn: (T) -> R): (T) -> R {
    val cache = mutableMapOf<T, R>()
    return { input ->
        cache.getOrPut(input) { fn(input) }
    }
}

val expensiveFunction = { n: Int -> 
    Thread.sleep(1000)  // Simulate expensive operation
    n * n 
}
val memoizedFunction = memoize(expensiveFunction)
```

### Result & Either Pattern
```kotlin
// Result handling
inline fun <T> runCatching(block: () -> T): Result<T> = 
    try { Result.success(block()) } 
    catch (e: Throwable) { Result.failure(e) }

fun divide(a: Int, b: Int): Result<Double> = 
    if (b == 0) Result.failure(ArithmeticException("Division by zero"))
    else Result.success(a.toDouble() / b)

// Chain operations
val result = divide(10, 2)
    .map { it * 2 }
    .map { "Result: $it" }
    .getOrElse { "Error occurred" }

// Custom Either implementation
sealed class Either<out L, out R> {
    data class Left<out L>(val value: L) : Either<L, Nothing>()
    data class Right<out R>(val value: R) : Either<Nothing, R>()
    
    fun <T> map(transform: (R) -> T): Either<L, T> = when (this) {
        is Left -> this
        is Right -> Right(transform(value))
    }
    
    fun <T> flatMap(transform: (R) -> Either<L, T>): Either<L, T> = when (this) {
        is Left -> this
        is Right -> transform(value)
    }
}
```

## Performance & Memory Operations

### Lazy Evaluation
```kotlin
// Lazy properties
class ExpensiveResource {
    val data: String by lazy {
        println("Computing expensive data...")
        "Expensive Result"
    }
}

// Lazy sequences
val lazySequence = generateSequence(1) { it + 1 }
    .filter { it % 2 == 0 }
    .map { it * it }
    .take(5)

val result = lazySequence.toList()  // Only computed when needed

// Custom lazy evaluation
fun <T> lazyValue(initializer: () -> T): Lazy<T> = lazy(initializer)
```

### Memory Management
```kotlin
// Weak references
import java.lang.ref.WeakReference

class Cache<K, V> {
    private val cache = mutableMapOf<K, WeakReference<V>>()
    
    fun put(key: K, value: V) {
        cache[key] = WeakReference(value)
    }
    
    fun get(key: K): V? {
        return cache[key]?.get()
    }
    
    fun cleanup() {
        cache.entries.removeAll { it.value.get() == null }
    }
}

// Memory monitoring
fun getMemoryInfo(): MemoryInfo {
    val runtime = Runtime.getRuntime()
    return MemoryInfo(
        totalMemory = runtime.totalMemory(),
        freeMemory = runtime.freeMemory(),
        usedMemory = runtime.totalMemory() - runtime.freeMemory(),
        maxMemory = runtime.maxMemory()
    )
}

data class MemoryInfo(
    val totalMemory: Long,
    val freeMemory: Long,
    val usedMemory: Long,
    val maxMemory: Long
)

// Force garbage collection (use sparingly)
fun forceGC() {
    System.gc()
}
```

## Serialization & JSON

### kotlinx.serialization
```kotlin
import kotlinx.serialization.*
import kotlinx.serialization.json.*
import kotlinx.serialization.encodeToString
import kotlinx.serialization.decodeFromString

@Serializable
data class User(
    val id: Int,
    val name: String,
    val email: String,
    val isActive: Boolean = true,
    @SerialName("created_at") val createdAt: String? = null
)

@Serializable
data class ApiResponse<T>(
    val data: T,
    val status: String,
    val message: String? = null
)

// JSON configuration
val json = Json {
    ignoreUnknownKeys = true
    isLenient = true
    prettyPrint = true
    encodeDefaults = false
}

// Serialize to JSON
val user = User(1, "John Doe", "john@example.com")
val jsonString = json.encodeToString(user)

// Deserialize from JSON
val userFromJson = json.decodeFromString<User>(jsonString)

// Handle lists
val users = listOf(user, User(2, "Jane", "jane@example.com"))
val usersJson = json.encodeToString(users)
val usersFromJson = json.decodeFromString<List<User>>(usersJson)

// Generic responses
val response = ApiResponse(user, "success", "User retrieved")
val responseJson = json.encodeToString(response)
```

### Custom Serializers
```kotlin
import kotlinx.serialization.KSerializer
import kotlinx.serialization.descriptors.*
import kotlinx.serialization.encoding.*
import java.time.LocalDate
import java.time.format.DateTimeFormatter

@Serializer(forClass = LocalDate::class)
object LocalDateSerializer : KSerializer<LocalDate> {
    private val formatter = DateTimeFormatter.ISO_LOCAL_DATE
    
    override val descriptor: SerialDescriptor = 
        PrimitiveSerialDescriptor("LocalDate", PrimitiveKind.STRING)
    
    override fun serialize(encoder: Encoder, value: LocalDate) {
        encoder.encodeString(value.format(formatter))
    }
    
    override fun deserialize(decoder: Decoder): LocalDate {
        return LocalDate.parse(decoder.decodeString(), formatter)
    }
}

@Serializable
data class Event(
    val id: Int,
    val title: String,
    @Serializable(with = LocalDateSerializer::class)
    val date: LocalDate
)
```