# Dart Intermediate Programming Cheat Sheet

## Advanced Generics

### Generic Classes
```dart
class Repository<T> {
  final List<T> _items = [];
  
  void add(T item) => _items.add(item);
  
  T? findById(String id) {
    // Complex logic here
    return _items.isNotEmpty ? _items.first : null;
  }
  
  List<T> where(bool Function(T) predicate) {
    return _items.where(predicate).toList();
  }
}

// Usage
var userRepo = Repository<User>();
var productRepo = Repository<Product>();
```

### Generic Constraints
```dart
abstract class Identifiable {
  String get id;
}

class EntityRepository<T extends Identifiable> {
  final Map<String, T> _entities = {};
  
  void store(T entity) {
    _entities[entity.id] = entity;
  }
  
  T? findById(String id) => _entities[id];
}

class User implements Identifiable {
  @override
  final String id;
  final String name;
  
  User(this.id, this.name);
}
```

### Generic Functions
```dart
T? firstWhere<T>(List<T> items, bool Function(T) predicate) {
  for (T item in items) {
    if (predicate(item)) return item;
  }
  return null;
}

// Multiple type parameters
Map<K, V> combine<K, V>(Map<K, V> map1, Map<K, V> map2) {
  return {...map1, ...map2};
}
```

## Mixins & Advanced Inheritance

### Mixins
```dart
mixin Flyable {
  void fly() => print('Flying!');
  
  // Mixin can have state
  bool _isFlying = false;
  bool get isFlying => _isFlying;
  
  void startFlying() {
    _isFlying = true;
    fly();
  }
}

mixin Swimmable {
  void swim() => print('Swimming!');
}

class Duck extends Animal with Flyable, Swimmable {
  @override
  void makeSound() => print('Quack!');
}

// Mixin constraints
mixin Walkable on Animal {
  void walk() {
    print('${this.runtimeType} is walking');
  }
}

class Dog extends Animal with Walkable {
  @override
  void makeSound() => print('Woof!');
}
```

### Interface Implementation
```dart
abstract class Drawable {
  void draw();
  double get area;
}

abstract class Colorable {
  String get color;
  void setColor(String color);
}

class Circle implements Drawable, Colorable {
  double radius;
  String _color = 'white';
  
  Circle(this.radius);
  
  @override
  void draw() => print('Drawing a $_color circle');
  
  @override
  double get area => 3.14 * radius * radius;
  
  @override
  String get color => _color;
  
  @override
  void setColor(String color) => _color = color;
}
```

## Advanced Async Programming

### Stream Transformations
```dart
import 'dart:async';

// Custom stream transformer
class ThrottleTransformer<T> extends StreamTransformerBase<T, T> {
  final Duration duration;
  
  ThrottleTransformer(this.duration);
  
  @override
  Stream<T> bind(Stream<T> stream) {
    return Stream.fromIterable([]).transform(
      StreamTransformer.fromHandlers(
        handleData: (data, sink) {
          // Throttling logic
          sink.add(data);
        },
      ),
    );
  }
}

// Stream manipulation
Stream<int> numbers = Stream.fromIterable([1, 2, 3, 4, 5]);

// Transform streams
Stream<String> stringNumbers = numbers
    .where((n) => n % 2 == 0)
    .map((n) => 'Number: $n')
    .take(3);

// Broadcast streams
StreamController<int> controller = StreamController<int>.broadcast();

// Multiple listeners
controller.stream.listen((data) => print('Listener 1: $data'));
controller.stream.listen((data) => print('Listener 2: $data'));
```

### Async Generators
```dart
// Async generator function
Stream<int> countStream(int max) async* {
  for (int i = 1; i <= max; i++) {
    await Future.delayed(Duration(seconds: 1));
    yield i;
  }
}

// Sync generator function
Iterable<int> getNumbers(int max) sync* {
  for (int i = 1; i <= max; i++) {
    yield i * i;
  }
}

// Usage
await for (int number in countStream(5)) {
  print(number); // Prints 1, 2, 3, 4, 5 with 1 second delay
}
```

### Completer & Future.wait
```dart
// Completer for custom futures
Completer<String> completer = Completer<String>();

void startAsyncOperation() {
  Timer(Duration(seconds: 2), () {
    completer.complete('Operation completed');
  });
}

Future<String> getResult() => completer.future;

// Parallel execution
Future<void> parallelOperations() async {
  List<Future<String>> futures = [
    fetchUserData(),
    fetchUserPreferences(),
    fetchUserSettings(),
  ];
  
  List<String> results = await Future.wait(futures);
  print('All operations completed: $results');
}

// Handle partial failures
Future<void> parallelWithErrorHandling() async {
  List<Future<String?>> futures = [
    fetchUserData().catchError((e) => null),
    fetchUserPreferences().catchError((e) => null),
    fetchUserSettings().catchError((e) => null),
  ];
  
  List<String?> results = await Future.wait(futures);
  List<String> successfulResults = results.whereType<String>().toList();
}
```

## Advanced Collections & Iterables

### Custom Collections
```dart
class ObservableList<T> implements List<T> {
  final List<T> _items = [];
  final List<void Function(T)> _listeners = [];
  
  void addListener(void Function(T) listener) {
    _listeners.add(listener);
  }
  
  @override
  void add(T value) {
    _items.add(value);
    for (var listener in _listeners) {
      listener(value);
    }
  }
  
  @override
  int get length => _items.length;
  
  @override
  T operator [](int index) => _items[index];
  
  @override
  void operator []=(int index, T value) {
    _items[index] = value;
  }
  
  // Implement other List methods...
  @override
  bool remove(Object? value) => _items.remove(value);
  
  @override
  T removeAt(int index) => _items.removeAt(index);
  
  // ... (implement remaining List interface methods)
}
```

### Advanced Iterable Operations
```dart
extension IterableExtensions<T> on Iterable<T> {
  // Chunk into smaller lists
  Iterable<List<T>> chunk(int size) sync* {
    var iterator = this.iterator;
    while (iterator.moveNext()) {
      var chunk = <T>[iterator.current];
      for (int i = 1; i < size && iterator.moveNext(); i++) {
        chunk.add(iterator.current);
      }
      yield chunk;
    }
  }
  
  // Group by key
  Map<K, List<T>> groupBy<K>(K Function(T) keySelector) {
    var map = <K, List<T>>{};
    for (var item in this) {
      var key = keySelector(item);
      map.putIfAbsent(key, () => <T>[]).add(item);
    }
    return map;
  }
  
  // Distinct by property
  Iterable<T> distinctBy<K>(K Function(T) keySelector) {
    var seen = <K>{};
    return where((item) => seen.add(keySelector(item)));
  }
}

// Usage
var users = [User('1', 'Alice'), User('2', 'Bob'), User('1', 'Alice')];
var uniqueUsers = users.distinctBy((u) => u.id);
var usersByFirstLetter = users.groupBy((u) => u.name[0]);
```

## Functional Programming Concepts

### Higher-Order Functions
```dart
typedef Predicate<T> = bool Function(T);
typedef Mapper<T, R> = R Function(T);

class FunctionalList<T> {
  final List<T> _items;
  
  FunctionalList(this._items);
  
  FunctionalList<T> filter(Predicate<T> predicate) {
    return FunctionalList(_items.where(predicate).toList());
  }
  
  FunctionalList<R> map<R>(Mapper<T, R> mapper) {
    return FunctionalList(_items.map(mapper).toList());
  }
  
  T? reduce(T Function(T, T) combiner) {
    return _items.isEmpty ? null : _items.reduce(combiner);
  }
  
  FunctionalList<T> take(int count) {
    return FunctionalList(_items.take(count).toList());
  }
}
```

### Currying & Partial Application
```dart
// Currying
typedef CurriedFunction<A, B, C> = C Function(B) Function(A);

CurriedFunction<int, int, int> add = (a) => (b) => a + b;
var add5 = add(5); // Returns function that adds 5
print(add5(3)); // 8

// Partial application
Function partial(Function fn, List args) {
  return (List additionalArgs) {
    return Function.apply(fn, [...args, ...additionalArgs]);
  };
}
```

### Monads & Maybe Pattern
```dart
abstract class Maybe<T> {
  const Maybe();
  
  factory Maybe.some(T value) = Some<T>;
  factory Maybe.none() = None<T>;
  
  Maybe<R> map<R>(R Function(T) transform);
  Maybe<R> flatMap<R>(Maybe<R> Function(T) transform);
  T getOrElse(T defaultValue);
  bool get isSome;
  bool get isNone;
}

class Some<T> extends Maybe<T> {
  final T value;
  const Some(this.value);
  
  @override
  Maybe<R> map<R>(R Function(T) transform) => Some(transform(value));
  
  @override
  Maybe<R> flatMap<R>(Maybe<R> Function(T) transform) => transform(value);
  
  @override
  T getOrElse(T defaultValue) => value;
  
  @override
  bool get isSome => true;
  
  @override
  bool get isNone => false;
}

class None<T> extends Maybe<T> {
  const None();
  
  @override
  Maybe<R> map<R>(R Function(T) transform) => None<R>();
  
  @override
  Maybe<R> flatMap<R>(Maybe<R> Function(T) transform) => None<R>();
  
  @override
  T getOrElse(T defaultValue) => defaultValue;
  
  @override
  bool get isSome => false;
  
  @override
  bool get isNone => true;
}
```

## Advanced Error Handling

### Custom Exceptions
```dart
class ValidationException implements Exception {
  final String message;
  final Map<String, List<String>> errors;
  
  ValidationException(this.message, this.errors);
  
  @override
  String toString() => 'ValidationException: $message';
}

class NetworkException implements Exception {
  final int statusCode;
  final String message;
  
  NetworkException(this.statusCode, this.message);
  
  @override
  String toString() => 'NetworkException($statusCode): $message';
}
```

### Result Pattern
```dart
abstract class Result<T, E> {
  const Result();
  
  factory Result.success(T value) = Success<T, E>;
  factory Result.failure(E error) = Failure<T, E>;
  
  Result<R, E> map<R>(R Function(T) transform);
  Result<T, R> mapError<R>(R Function(E) transform);
  Result<R, E> flatMap<R>(Result<R, E> Function(T) transform);
  
  bool get isSuccess;
  bool get isFailure;
  T? get value;
  E? get error;
}

class Success<T, E> extends Result<T, E> {
  final T _value;
  const Success(this._value);
  
  @override
  Result<R, E> map<R>(R Function(T) transform) {
    try {
      return Success(transform(_value));
    } catch (e) {
      return Failure(e as E);
    }
  }
  
  @override
  Result<T, R> mapError<R>(R Function(E) transform) => Success(_value);
  
  @override
  Result<R, E> flatMap<R>(Result<R, E> Function(T) transform) {
    return transform(_value);
  }
  
  @override
  bool get isSuccess => true;
  @override
  bool get isFailure => false;
  @override
  T get value => _value;
  @override
  E? get error => null;
}

class Failure<T, E> extends Result<T, E> {
  final E _error;
  const Failure(this._error);
  
  @override
  Result<R, E> map<R>(R Function(T) transform) => Failure(_error);
  
  @override
  Result<T, R> mapError<R>(R Function(E) transform) {
    return Failure(transform(_error));
  }
  
  @override
  Result<R, E> flatMap<R>(Result<R, E> Function(T) transform) {
    return Failure(_error);
  }
  
  @override
  bool get isSuccess => false;
  @override
  bool get isFailure => true;
  @override
  T? get value => null;
  @override
  E get error => _error;
}

// Usage
Result<User, String> parseUser(String json) {
  try {
    // Parse JSON logic
    return Result.success(User.fromJson(json));
  } catch (e) {
    return Result.failure('Invalid JSON: $e');
  }
}
```

## Design Patterns

### Singleton Pattern
```dart
class DatabaseConnection {
  static DatabaseConnection? _instance;
  static final Lock _lock = Lock();
  
  DatabaseConnection._internal();
  
  static Future<DatabaseConnection> getInstance() async {
    if (_instance == null) {
      await _lock.synchronized(() async {
        if (_instance == null) {
          _instance = DatabaseConnection._internal();
          await _instance!._initialize();
        }
      });
    }
    return _instance!;
  }
  
  Future<void> _initialize() async {
    // Initialize database connection
  }
}
```

### Factory Pattern
```dart
abstract class Animal {
  void makeSound();
  
  factory Animal.create(String type) {
    switch (type.toLowerCase()) {
      case 'dog':
        return Dog();
      case 'cat':
        return Cat();
      default:
        throw ArgumentError('Unknown animal type: $type');
    }
  }
}

class Dog implements Animal {
  @override
  void makeSound() => print('Woof!');
}

class Cat implements Animal {
  @override
  void makeSound() => print('Meow!');
}
```

### Observer Pattern
```dart
abstract class Observer<T> {
  void update(T data);
}

class Subject<T> {
  final List<Observer<T>> _observers = [];
  
  void addObserver(Observer<T> observer) {
    _observers.add(observer);
  }
  
  void removeObserver(Observer<T> observer) {
    _observers.remove(observer);
  }
  
  void notifyObservers(T data) {
    for (var observer in _observers) {
      observer.update(data);
    }
  }
}

class WeatherStation extends Subject<WeatherData> {
  WeatherData? _currentWeather;
  
  void setWeatherData(WeatherData weather) {
    _currentWeather = weather;
    notifyObservers(weather);
  }
}

class WeatherDisplay implements Observer<WeatherData> {
  @override
  void update(WeatherData data) {
    print('Weather updated: ${data.temperature}°C');
  }
}
```

### Command Pattern
```dart
abstract class Command {
  void execute();
  void undo();
}

class Light {
  bool _isOn = false;
  
  void turnOn() {
    _isOn = true;
    print('Light is ON');
  }
  
  void turnOff() {
    _isOn = false;
    print('Light is OFF');
  }
}

class LightOnCommand implements Command {
  final Light light;
  
  LightOnCommand(this.light);
  
  @override
  void execute() => light.turnOn();
  
  @override
  void undo() => light.turnOff();
}

class RemoteControl {
  final List<Command?> _commands = List.filled(7, null);
  final List<Command> _undoStack = [];
  
  void setCommand(int slot, Command command) {
    _commands[slot] = command;
  }
  
  void pressButton(int slot) {
    var command = _commands[slot];
    if (command != null) {
      command.execute();
      _undoStack.add(command);
    }
  }
  
  void pressUndo() {
    if (_undoStack.isNotEmpty) {
      _undoStack.removeLast().undo();
    }
  }
}
```

## Meta-programming & Reflection

### Annotations
```dart
class Deprecated {
  final String message;
  const Deprecated(this.message);
}

class ApiEndpoint {
  final String path;
  final String method;
  const ApiEndpoint(this.path, {this.method = 'GET'});
}

class UserController {
  @ApiEndpoint('/users', method: 'GET')
  @Deprecated('Use getUsersV2 instead')
  List<User> getUsers() {
    return [];
  }
  
  @ApiEndpoint('/users', method: 'POST')
  User createUser(User user) {
    return user;
  }
}
```

### Extension Methods (Advanced)
```dart
extension FutureExtensions<T> on Future<T> {
  Future<T> timeout(Duration duration, {T? fallback}) {
    return Future.any([
      this,
      Future.delayed(duration).then((_) {
        if (fallback != null) return fallback;
        throw TimeoutException('Operation timed out', duration);
      }),
    ]);
  }
  
  Future<Result<T, E>> toResult<E>() async {
    try {
      final value = await this;
      return Result.success(value);
    } catch (e) {
      return Result.failure(e as E);
    }
  }
}

extension StreamExtensions<T> on Stream<T> {
  Stream<T> debounce(Duration duration) {
    return transform(StreamTransformer.fromHandlers(
      handleData: (data, sink) {
        Timer(duration, () => sink.add(data));
      },
    ));
  }
  
  Stream<List<T>> buffer(Duration duration) {
    return transform(StreamTransformer.fromHandlers(
      handleData: (data, sink) {
        // Buffer implementation
      },
    ));
  }
}
```

## Advanced Type System

### Typedef & Function Types
```dart
// Function type definitions
typedef StringProcessor = String Function(String);
typedef AsyncProcessor<T> = Future<T> Function(T);
typedef EventHandler<T> = void Function(T);

// Generic function typedef
typedef Converter<TInput, TOutput> = TOutput Function(TInput);

class DataPipeline<TInput, TOutput> {
  final List<Converter> _converters = [];
  
  void addConverter<TIntermediate>(
    Converter<TInput, TIntermediate> converter,
  ) {
    _converters.add(converter);
  }
  
  TOutput process(TInput input) {
    dynamic result = input;
    for (var converter in _converters) {
      result = converter(result);
    }
    return result as TOutput;
  }
}
```

### Covariant & Contravariant
```dart
abstract class Animal {
  void makeSound();
}

class Dog extends Animal {
  @override
  void makeSound() => print('Woof');
}

class AnimalShelter<T extends Animal> {
  final List<T> _animals = [];
  
  void add(T animal) => _animals.add(animal);
  
  // Covariant return type
  T? getFirst() => _animals.isNotEmpty ? _animals.first : null;
  
  // Method with covariant parameter
  void processAnimals(void Function(covariant T) processor) {
    for (var animal in _animals) {
      processor(animal);
    }
  }
}
```

## Memory Management & Performance

### Weak References
```dart
import 'dart:core';

class EventManager {
  final List<WeakReference<EventListener>> _listeners = [];
  
  void addListener(EventListener listener) {
    _listeners.add(WeakReference(listener));
  }
  
  void fireEvent(Event event) {
    _listeners.removeWhere((ref) => ref.target == null);
    
    for (var ref in _listeners) {
      var listener = ref.target;
      if (listener != null) {
        listener.onEvent(event);
      }
    }
  }
}
```

### Finalizers
```dart
import 'dart:ffi';

class NativeResource {
  final Pointer<Void> _handle;
  static final Finalizer<Pointer<Void>> _finalizer = 
      Finalizer((handle) => _cleanup(handle));
  
  NativeResource(this._handle) {
    _finalizer.attach(this, _handle);
  }
  
  static void _cleanup(Pointer<Void> handle) {
    // Clean up native resource
  }
  
  void dispose() {
    _finalizer.detach(this);
    _cleanup(_handle);
  }
}
```

## Testing Patterns

### Test Utilities
```dart
// Custom matchers
Matcher throwsValidationException = throwsA(isA<ValidationException>());

Matcher hasProperty<T>(String property, T expectedValue) {
  return predicate<Object>((obj) {
    // Use reflection or other means to check property
    return true; // Simplified
  }, 'has property $property with value $expectedValue');
}

// Test data builders
class UserBuilder {
  String _id = '1';
  String _name = 'Test User';
  String? _email;
  
  UserBuilder withId(String id) {
    _id = id;
    return this;
  }
  
  UserBuilder withName(String name) {
    _name = name;
    return this;
  }
  
  UserBuilder withEmail(String email) {
    _email = email;
    return this;
  }
  
  User build() => User(_id, _name, email: _email);
}

// Usage in tests
void main() {
  group('User tests', () {
    test('should create user with email', () {
      final user = UserBuilder()
          .withName('John Doe')
          .withEmail('john@example.com')
          .build();
          
      expect(user.name, equals('John Doe'));
      expect(user.email, equals('john@example.com'));
    });
  });
}
```

## Advanced Best Practices

### Immutable Data Structures
```dart
@immutable
class ImmutableList<T> {
  final List<T> _items;
  
  const ImmutableList(this._items);
  
  ImmutableList<T> add(T item) {
    return ImmutableList([..._items, item]);
  }
  
  ImmutableList<T> remove(T item) {
    final newItems = List<T>.from(_items);
    newItems.remove(item);
    return ImmutableList(newItems);
  }
  
  T operator [](int index) => _items[index];
  int get length => _items.length;
  
  @override
  bool operator ==(Object other) {
    if (identical(this, other)) return true;
    return other is ImmutableList<T> && 
           listEquals(_items, other._items);
  }
  
  @override
  int get hashCode => _items.hashCode;
}
```

### Builder Pattern for Complex Objects
```dart
class ApiClientBuilder {
  String? _baseUrl;
  Duration _timeout = Duration(seconds: 30);
  Map<String, String> _headers = {};
  List<Interceptor> _interceptors = [];
  
  ApiClientBuilder baseUrl(String url) {
    _baseUrl = url;
    return this;
  }
  
  ApiClientBuilder timeout(Duration timeout) {
    _timeout = timeout;
    return this;
  }
  
  ApiClientBuilder addHeader(String key, String value) {
    _headers[key] = value;
    return this;
  }
  
  ApiClientBuilder addInterceptor(Interceptor interceptor) {
    _interceptors.add(interceptor);
    return this;
  }
  
  ApiClient build() {
    if (_baseUrl == null) {
      throw StateError('Base URL is required');
    }
    return ApiClient._(
      baseUrl: _baseUrl!,
      timeout: _timeout,
      headers: Map.unmodifiable(_headers),
      interceptors: List.unmodifiable(_interceptors),
    );
  }
}
```

This intermediate cheat sheet covers advanced Dart concepts that you'll encounter in larger applications and complex codebases. These patterns and techniques will help you write more maintainable, efficient, and robust Dart code.