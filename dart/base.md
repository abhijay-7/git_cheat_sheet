# Dart Programming Cheat Sheet

## Variables & Data Types

### Variable Declaration
```dart
// Explicit type
int age = 25;
String name = 'John';
double height = 5.9;
bool isActive = true;

// Type inference
var city = 'New York';  // String
var count = 10;         // int
var price = 99.99;      // double

// Constants
const pi = 3.14159;           // Compile-time constant
final timestamp = DateTime.now(); // Runtime constant

// Nullable types (Null Safety)
String? nullableName;    // Can be null
int? nullableAge;        // Can be null
```

### Basic Data Types
```dart
// Numbers
int whole = 42;
double decimal = 3.14;
num anyNumber = 100; // Can be int or double

// Strings
String single = 'Hello';
String double = "World";
String multiline = '''
  This is a
  multiline string
''';

// String interpolation
String greeting = 'Hello, $name!';
String info = 'Age: ${age + 1}';

// Booleans
bool isTrue = true;
bool isFalse = false;
```

## Collections

### Lists
```dart
// List declaration
List<int> numbers = [1, 2, 3, 4, 5];
var fruits = ['apple', 'banana', 'orange'];
List<String> emptyList = [];

// List operations
numbers.add(6);              // Add element
numbers.addAll([7, 8]);      // Add multiple
numbers.remove(3);           // Remove element
numbers.removeAt(0);         // Remove by index
numbers.insert(0, 10);       // Insert at index
print(numbers.length);       // Get length
print(numbers.isEmpty);      // Check if empty
```

### Sets
```dart
// Set declaration
Set<String> colors = {'red', 'green', 'blue'};
var uniqueNumbers = <int>{1, 2, 3, 4, 5};

// Set operations
colors.add('yellow');
colors.remove('red');
print(colors.contains('blue')); // true
```

### Maps
```dart
// Map declaration
Map<String, int> scores = {'Alice': 95, 'Bob': 87};
var person = {
  'name': 'John',
  'age': 30,
  'city': 'NYC'
};

// Map operations
scores['Charlie'] = 92;      // Add/update
scores.remove('Bob');        // Remove
print(scores.keys);          // Get keys
print(scores.values);        // Get values
print(scores.containsKey('Alice')); // Check key
```

## Control Flow

### Conditionals
```dart
// If-else
if (age >= 18) {
  print('Adult');
} else if (age >= 13) {
  print('Teenager');
} else {
  print('Child');
}

// Ternary operator
String status = age >= 18 ? 'Adult' : 'Minor';

// Switch statement
switch (grade) {
  case 'A':
    print('Excellent');
    break;
  case 'B':
    print('Good');
    break;
  default:
    print('Keep trying');
}
```

### Loops
```dart
// For loop
for (int i = 0; i < 5; i++) {
  print(i);
}

// For-in loop
for (String fruit in fruits) {
  print(fruit);
}

// While loop
int count = 0;
while (count < 3) {
  print(count);
  count++;
}

// Do-while loop
do {
  print('At least once');
} while (false);
```

## Functions

### Function Declaration
```dart
// Basic function
int add(int a, int b) {
  return a + b;
}

// Arrow function
int multiply(int a, int b) => a * b;

// Optional parameters
String greet(String name, [String title = 'Mr.']) {
  return '$title $name';
}

// Named parameters
void createUser({required String name, int age = 0}) {
  print('Name: $name, Age: $age');
}

// Function as parameter
void executeFunction(Function callback) {
  callback();
}
```

## Classes & Objects

### Basic Class
```dart
class Person {
  // Properties
  String name;
  int age;
  
  // Constructor
  Person(this.name, this.age);
  
  // Named constructor
  Person.baby(this.name) : age = 0;
  
  // Methods
  void introduce() {
    print('Hi, I\'m $name and I\'m $age years old');
  }
  
  // Getter
  String get info => '$name ($age)';
  
  // Setter
  set updateAge(int newAge) {
    if (newAge >= 0) age = newAge;
  }
}

// Usage
var person = Person('Alice', 25);
person.introduce();
```

### Inheritance
```dart
class Student extends Person {
  String school;
  
  Student(String name, int age, this.school) : super(name, age);
  
  @override
  void introduce() {
    super.introduce();
    print('I study at $school');
  }
}
```

### Abstract Classes & Interfaces
```dart
abstract class Animal {
  void makeSound(); // Abstract method
  
  void sleep() {    // Concrete method
    print('Sleeping...');
  }
}

class Dog extends Animal {
  @override
  void makeSound() {
    print('Woof!');
  }
}
```

## Error Handling

```dart
// Try-catch
try {
  int result = 10 ~/ 0; // Integer division
} catch (e) {
  print('Error: $e');
} finally {
  print('Cleanup code');
}

// Specific exception handling
try {
  // Some code
} on FormatException catch (e) {
  print('Format error: $e');
} on Exception catch (e) {
  print('Other exception: $e');
}

// Throwing exceptions
void validateAge(int age) {
  if (age < 0) {
    throw ArgumentError('Age cannot be negative');
  }
}
```

## Null Safety

```dart
// Null-aware operators
String? nullableName;

// Null check
if (nullableName != null) {
  print(nullableName.length);
}

// Null-aware access
print(nullableName?.length); // Returns null if nullableName is null

// Null coalescing
String displayName = nullableName ?? 'Unknown';

// Null assertion (use carefully!)
print(nullableName!.length); // Throws if null

// Null-aware assignment
nullableName ??= 'Default Name';
```

## Async Programming

### Futures
```dart
// Future function
Future<String> fetchData() async {
  await Future.delayed(Duration(seconds: 2));
  return 'Data loaded';
}

// Using async/await
void loadData() async {
  try {
    String data = await fetchData();
    print(data);
  } catch (e) {
    print('Error: $e');
  }
}

// Future.then()
fetchData().then((data) {
  print(data);
}).catchError((error) {
  print('Error: $error');
});
```

### Streams
```dart
// Stream controller
import 'dart:async';

StreamController<int> controller = StreamController<int>();

// Listen to stream
controller.stream.listen((data) {
  print('Received: $data');
});

// Add data to stream
controller.add(42);
controller.close();
```

## Common Operators

```dart
// Arithmetic
+ - * / ~/ % // Basic math, ~/ is integer division

// Comparison
== != < > <= >=

// Logical
&& || !

// Assignment
= += -= *= /= ~/= %=

// Type test
is is! as

// Cascade notation
person
  ..name = 'John'
  ..age = 30
  ..introduce();
```

## Useful Built-in Methods

### String Methods
```dart
String text = 'Hello World';
text.toLowerCase();        // 'hello world'
text.toUpperCase();        // 'HELLO WORLD'
text.contains('World');    // true
text.startsWith('Hello');  // true
text.split(' ');          // ['Hello', 'World']
text.substring(0, 5);     // 'Hello'
text.replaceAll('o', '0'); // 'Hell0 W0rld'
```

### List Methods
```dart
List<int> numbers = [1, 2, 3, 4, 5];
numbers.where((n) => n > 3);     // Filter
numbers.map((n) => n * 2);       // Transform
numbers.reduce((a, b) => a + b); // Reduce to single value
numbers.forEach((n) => print(n)); // Iterate
numbers.any((n) => n > 3);       // Check if any match
numbers.every((n) => n > 0);     // Check if all match
```

## Import & Libraries

```dart
// Core libraries
import 'dart:math';
import 'dart:convert';
import 'dart:io';

// Package imports
import 'package:http/http.dart' as http;

// Local imports
import 'models/user.dart';
import '../utils/helpers.dart';

// Show/hide specific items
import 'dart:math' show Random, pi;
import 'dart:math' hide cos, sin;
```

## Common Patterns

### Factory Constructor
```dart
class Logger {
  static Logger? _instance;
  
  Logger._internal();
  
  factory Logger() {
    _instance ??= Logger._internal();
    return _instance!;
  }
}
```

### Extension Methods
```dart
extension StringExtension on String {
  String capitalize() {
    if (isEmpty) return this;
    return this[0].toUpperCase() + substring(1);
  }
}

// Usage
String name = 'john';
print(name.capitalize()); // 'John'
```

## Tips & Best Practices

- Use `final` for variables that won't be reassigned
- Enable null safety in your `pubspec.yaml`
- Use meaningful variable and function names
- Follow Dart naming conventions (camelCase for variables, PascalCase for classes)
- Use `const` constructors when possible for better performance
- Handle exceptions appropriately
- Use `async`/`await` for asynchronous operations
- Leverage Dart's type inference but be explicit when it improves readability