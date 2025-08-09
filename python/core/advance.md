# Python Advanced Cheatsheet

## Metaclasses and Descriptors

### Understanding Metaclasses
```python
# Everything in Python is an object, including classes
class MyClass:
    pass

print(type(MyClass))  # <class 'type'>
print(type(type))     # <class 'type'>

# Basic metaclass
class SingletonMeta(type):
    _instances = {}
    
    def __call__(cls, *args, **kwargs):
        if cls not in cls._instances:
            cls._instances[cls] = super().__call__(*args, **kwargs)
        return cls._instances[cls]

class Singleton(metaclass=SingletonMeta):
    def __init__(self, value):
        self.value = value

# Usage
s1 = Singleton("first")
s2 = Singleton("second")
print(s1 is s2)  # True
print(s1.value)  # "first" (not changed by second instantiation)

# Metaclass for automatic registration
class RegisteredMeta(type):
    registry = {}
    
    def __new__(mcs, name, bases, namespace):
        cls = super().__new__(mcs, name, bases, namespace)
        if name != 'RegisteredBase':  # Don't register the base class
            mcs.registry[name] = cls
        return cls

class RegisteredBase(metaclass=RegisteredMeta):
    pass

class DatabaseModel(RegisteredBase):
    pass

class APIEndpoint(RegisteredBase):
    pass

print(RegisteredMeta.registry)  # {'DatabaseModel': <class '__main__.DatabaseModel'>, 'APIEndpoint': <class '__main__.APIEndpoint'>}

# Dynamic class creation
def create_class_with_methods(class_name, method_names):
    def make_method(name):
        def method(self):
            return f"Called {name}"
        method.__name__ = name
        return method
    
    namespace = {name: make_method(name) for name in method_names}
    return type(class_name, (object,), namespace)

DynamicClass = create_class_with_methods('DynamicClass', ['method1', 'method2'])
obj = DynamicClass()
print(obj.method1())  # "Called method1"
```

### Descriptors
```python
class Descriptor:
    def __init__(self, name=None):
        self.name = name
    
    def __set_name__(self, owner, name):
        self.name = name
    
    def __get__(self, obj, objtype=None):
        if obj is None:
            return self
        return obj.__dict__.get(self.name)
    
    def __set__(self, obj, value):
        obj.__dict__[self.name] = value
    
    def __delete__(self, obj):
        del obj.__dict__[self.name]

# Typed property descriptor
class TypedProperty:
    def __init__(self, expected_type, default=None):
        self.expected_type = expected_type
        self.default = default
        self.name = None
    
    def __set_name__(self, owner, name):
        self.name = f'_{name}'
    
    def __get__(self, obj, objtype=None):
        if obj is None:
            return self
        return getattr(obj, self.name, self.default)
    
    def __set__(self, obj, value):
        if not isinstance(value, self.expected_type):
            raise TypeError(f"Expected {self.expected_type.__name__}, got {type(value).__name__}")
        setattr(obj, self.name, value)

class Person:
    name = TypedProperty(str, "Unknown")
    age = TypedProperty(int, 0)
    
    def __init__(self, name, age):
        self.name = name
        self.age = age

person = Person("Alice", 30)
print(person.name)  # Alice
# person.name = 123  # TypeError: Expected str, got int

# Cached property descriptor
class CachedProperty:
    def __init__(self, func):
        self.func = func
        self.name = func.__name__
        self.__doc__ = func.__doc__
    
    def __get__(self, obj, objtype=None):
        if obj is None:
            return self
        
        cache_attr = f'_cached_{self.name}'
        if not hasattr(obj, cache_attr):
            setattr(obj, cache_attr, self.func(obj))
        return getattr(obj, cache_attr)

class DataProcessor:
    def __init__(self, data):
        self.data = data
    
    @CachedProperty
    def expensive_calculation(self):
        print("Performing expensive calculation...")
        return sum(x**2 for x in self.data)

processor = DataProcessor([1, 2, 3, 4, 5])
print(processor.expensive_calculation)  # Calculates and caches
print(processor.expensive_calculation)  # Returns cached value
```

## Advanced Object-Oriented Programming

### Multiple Inheritance and MRO
```python
class A:
    def method(self):
        print("A.method")
        return "A"

class B(A):
    def method(self):
        print("B.method")
        result = super().method()
        return f"B->{result}"

class C(A):
    def method(self):
        print("C.method") 
        result = super().method()
        return f"C->{result}"

class D(B, C):
    def method(self):
        print("D.method")
        result = super().method()
        return f"D->{result}"

# Method Resolution Order (MRO)
print(D.__mro__)  # (<class '__main__.D'>, <class '__main__.B'>, <class '__main__.C'>, <class '__main__.A'>, <class 'object'>)

d = D()
print(d.method())  # D->B->C->A

# Mixin classes
class JSONMixin:
    def to_json(self):
        import json
        return json.dumps(self.__dict__)
    
    def from_json(self, json_str):
        import json
        data = json.loads(json_str)
        for key, value in data.items():
            setattr(self, key, value)

class ValidatorMixin:
    def validate(self):
        required_fields = getattr(self, 'required_fields', [])
        for field in required_fields:
            if not hasattr(self, field) or getattr(self, field) is None:
                raise ValueError(f"Missing required field: {field}")

class User(JSONMixin, ValidatorMixin):
    required_fields = ['username', 'email']
    
    def __init__(self, username=None, email=None):
        self.username = username
        self.email = email

user = User("alice", "alice@example.com")
user.validate()  # OK
json_data = user.to_json()
print(json_data)  # {"username": "alice", "email": "alice@example.com"}
```

### Abstract Base Classes
```python
from abc import ABC, abstractmethod, abstractproperty
from typing import Any, List

class Shape(ABC):
    @abstractmethod
    def area(self) -> float:
        pass
    
    @abstractmethod
    def perimeter(self) -> float:
        pass
    
    @property
    @abstractmethod
    def name(self) -> str:
        pass
    
    def describe(self):
        return f"{self.name}: area={self.area():.2f}, perimeter={self.perimeter():.2f}"

class Rectangle(Shape):
    def __init__(self, width: float, height: float):
        self.width = width
        self.height = height
    
    def area(self) -> float:
        return self.width * self.height
    
    def perimeter(self) -> float:
        return 2 * (self.width + self.height)
    
    @property
    def name(self) -> str:
        return "Rectangle"

class Circle(Shape):
    def __init__(self, radius: float):
        self.radius = radius
    
    def area(self) -> float:
        import math
        return math.pi * self.radius ** 2
    
    def perimeter(self) -> float:
        import math
        return 2 * math.pi * self.radius
    
    @property
    def name(self) -> str:
        return "Circle"

# Usage
shapes: List[Shape] = [
    Rectangle(5, 3),
    Circle(2)
]

for shape in shapes:
    print(shape.describe())

# Protocol (Python 3.8+)
from typing import Protocol

class Drawable(Protocol):
    def draw(self) -> None: ...
    def get_area(self) -> float: ...

class Square:
    def __init__(self, side: float):
        self.side = side
    
    def draw(self) -> None:
        print(f"Drawing square with side {self.side}")
    
    def get_area(self) -> float:
        return self.side ** 2

def render(drawable: Drawable) -> None:
    drawable.draw()
    print(f"Area: {drawable.get_area()}")

square = Square(5)
render(square)  # Works without explicit inheritance
```

### Design Patterns
```python
# Singleton Pattern
class Singleton:
    _instance = None
    _initialized = False
    
    def __new__(cls):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
        return cls._instance
    
    def __init__(self):
        if not self._initialized:
            self.data = {}
            self._initialized = True

# Factory Pattern
class Animal:
    def speak(self):
        raise NotImplementedError

class Dog(Animal):
    def speak(self):
        return "Woof!"

class Cat(Animal):
    def speak(self):
        return "Meow!"

class AnimalFactory:
    @staticmethod
    def create_animal(animal_type: str) -> Animal:
        animals = {
            'dog': Dog,
            'cat': Cat
        }
        
        animal_class = animals.get(animal_type.lower())
        if animal_class:
            return animal_class()
        raise ValueError(f"Unknown animal type: {animal_type}")

# Observer Pattern
class Subject:
    def __init__(self):
        self._observers = []
    
    def attach(self, observer):
        self._observers.append(observer)
    
    def detach(self, observer):
        self._observers.remove(observer)
    
    def notify(self, *args, **kwargs):
        for observer in self._observers:
            observer.update(self, *args, **kwargs)

class Observer:
    def update(self, subject, *args, **kwargs):
        raise NotImplementedError

class NewsPublisher(Subject):
    def __init__(self):
        super().__init__()
        self._news = None
    
    def add_news(self, news):
        self._news = news
        self.notify(news)

class NewsSubscriber(Observer):
    def __init__(self, name):
        self.name = name
    
    def update(self, subject, news):
        print(f"{self.name} received news: {news}")

# Usage
publisher = NewsPublisher()
subscriber1 = NewsSubscriber("Alice")
subscriber2 = NewsSubscriber("Bob")

publisher.attach(subscriber1)
publisher.attach(subscriber2)
publisher.add_news("Python 4.0 released!")

# Command Pattern
class Command:
    def execute(self):
        raise NotImplementedError
    
    def undo(self):
        raise NotImplementedError

class Light:
    def __init__(self):
        self.is_on = False
    
    def turn_on(self):
        self.is_on = True
        print("Light is ON")
    
    def turn_off(self):
        self.is_on = False
        print("Light is OFF")

class LightOnCommand(Command):
    def __init__(self, light):
        self.light = light
    
    def execute(self):
        self.light.turn_on()
    
    def undo(self):
        self.light.turn_off()

class LightOffCommand(Command):
    def __init__(self, light):
        self.light = light
    
    def execute(self):
        self.light.turn_off()
    
    def undo(self):
        self.light.turn_on()

class RemoteControl:
    def __init__(self):
        self.commands = []
        self.current = -1
    
    def execute_command(self, command):
        # Remove any commands after current position
        self.commands = self.commands[:self.current + 1]
        self.commands.append(command)
        self.current += 1
        command.execute()
    
    def undo(self):
        if self.current >= 0:
            command = self.commands[self.current]
            command.undo()
            self.current -= 1

# Usage
light = Light()
remote = RemoteControl()

remote.execute_command(LightOnCommand(light))
remote.execute_command(LightOffCommand(light))
remote.undo()  # Light turns on
remote.undo()  # Light turns off
```

## Memory Management and Optimization

### Memory Profiling
```python
import tracemalloc
import sys
from memory_profiler import profile
import gc

# Track memory usage
def memory_usage_demo():
    # Start tracing
    tracemalloc.start()
    
    # Create some objects
    data = []
    for i in range(100000):
        data.append(f"item_{i}")
    
    # Get current memory usage
    current, peak = tracemalloc.get_traced_memory()
    print(f"Current memory usage: {current / 1024 / 1024:.2f} MB")
    print(f"Peak memory usage: {peak / 1024 / 1024:.2f} MB")
    
    # Stop tracing
    tracemalloc.stop()
    
    return data

# Memory profiler decorator
@profile
def memory_intensive_function():
    # Create large list
    big_list = [i for i in range(1000000)]
    
    # Create dictionary
    big_dict = {i: f"value_{i}" for i in range(100000)}
    
    # Some calculations
    result = sum(big_list)
    
    return result, big_dict

# Memory-efficient generator vs list
def memory_comparison():
    # Memory-hungry list comprehension
    def create_list():
        return [x**2 for x in range(1000000)]
    
    # Memory-efficient generator
    def create_generator():
        return (x**2 for x in range(1000000))
    
    # Compare memory usage
    import sys
    
    my_list = create_list()
    print(f"List size: {sys.getsizeof(my_list) / 1024 / 1024:.2f} MB")
    
    my_gen = create_generator()
    print(f"Generator size: {sys.getsizeof(my_gen)} bytes")

# Weak references to avoid circular references
import weakref

class Parent:
    def __init__(self, name):
        self.name = name
        self.children = []
    
    def add_child(self, child):
        self.children.append(child)
        child.parent = weakref.ref(self)  # Weak reference

class Child:
    def __init__(self, name):
        self.name = name
        self.parent = None
    
    def get_parent(self):
        if self.parent:
            return self.parent()  # Call weak reference
        return None

# Object pooling for expensive objects
class ObjectPool:
    def __init__(self, create_func, reset_func=None, max_size=10):
        self.create_func = create_func
        self.reset_func = reset_func
        self.max_size = max_size
        self.pool = []
    
    def acquire(self):
        if self.pool:
            obj = self.pool.pop()
            if self.reset_func:
                self.reset_func(obj)
            return obj
        return self.create_func()
    
    def release(self, obj):
        if len(self.pool) < self.max_size:
            self.pool.append(obj)

class ExpensiveObject:
    def __init__(self):
        self.data = [0] * 1000000  # Simulate expensive creation
        print("ExpensiveObject created")
    
    def reset(self):
        self.data = [0] * 1000000
        print("ExpensiveObject reset")

# Usage
pool = ObjectPool(
    create_func=ExpensiveObject,
    reset_func=lambda obj: obj.reset(),
    max_size=5
)

obj1 = pool.acquire()  # Creates new object
pool.release(obj1)
obj2 = pool.acquire()  # Reuses obj1
```

### Performance Optimization
```python
import time
import cProfile
import pstats
from functools import lru_cache, wraps
from typing import Any, Callable

# Timing decorator
def timer(func: Callable) -> Callable:
    @wraps(func)
    def wrapper(*args, **kwargs):
        start = time.perf_counter()
        result = func(*args, **kwargs)
        end = time.perf_counter()
        print(f"{func.__name__} took {end - start:.6f} seconds")
        return result
    return wrapper

# Memoization for expensive calculations
@lru_cache(maxsize=128)
def fibonacci(n: int) -> int:
    if n < 2:
        return n
    return fibonacci(n - 1) + fibonacci(n - 2)

# Custom memoization decorator
def memoize(func: Callable) -> Callable:
    cache = {}
    
    @wraps(func)
    def wrapper(*args, **kwargs):
        key = str(args) + str(sorted(kwargs.items()))
        if key not in cache:
            cache[key] = func(*args, **kwargs)
        return cache[key]
    
    wrapper.cache = cache
    wrapper.cache_clear = lambda: cache.clear()
    return wrapper

@memoize
def expensive_calculation(x: int, y: int) -> int:
    time.sleep(0.1)  # Simulate expensive operation
    return x ** y

# Profiling code
def profile_function(func: Callable) -> Callable:
    @wraps(func)
    def wrapper(*args, **kwargs):
        profiler = cProfile.Profile()
        profiler.enable()
        
        result = func(*args, **kwargs)
        
        profiler.disable()
        stats = pstats.Stats(profiler)
        stats.sort_stats('cumulative')
        stats.print_stats(10)  # Top 10 functions
        
        return result
    return wrapper

@profile_function
def slow_function():
    data = []
    for i in range(100000):
        data.append(i ** 2)
    return sum(data)

# Optimized data structures
class OptimizedCounter:
    """Memory-efficient counter using __slots__"""
    __slots__ = ['_counts']
    
    def __init__(self):
        self._counts = {}
    
    def increment(self, key):
        self._counts[key] = self._counts.get(key, 0) + 1
    
    def get_count(self, key):
        return self._counts.get(key, 0)

# Efficient string operations
def efficient_string_join(items):
    # Efficient: O(n)
    return ''.join(str(item) for item in items)

def inefficient_string_concat(items):
    # Inefficient: O(n²)
    result = ""
    for item in items:
        result += str(item)
    return result

# Efficient list operations
def efficient_list_ops():
    # Use list comprehensions instead of loops
    squares = [x**2 for x in range(1000)]
    
    # Use built-in functions
    total = sum(squares)
    maximum = max(squares)
    
    # Use collections.deque for frequent insertions/deletions at both ends
    from collections import deque
    dq = deque(squares)
    dq.appendleft(0)  # O(1)
    dq.popleft()      # O(1)

# Efficient dictionary operations
def efficient_dict_ops():
    # Use dict.get() instead of checking membership
    counts = {}
    for item in ['a', 'b', 'a', 'c', 'b', 'a']:
        counts[item] = counts.get(item, 0) + 1
    
    # Use collections.defaultdict for cleaner code
    from collections import defaultdict
    groups = defaultdict(list)
    for i, item in enumerate(['a', 'b', 'a', 'c']):
        groups[item].append(i)
    
    return counts, dict(groups)
```

## Concurrency and Parallelism

### Advanced Threading
```python
import threading
import queue
import time
from concurrent.futures import ThreadPoolExecutor, as_completed
from threading import Event, Condition, Barrier, Semaphore
import weakref

# Thread-safe singleton with double-checked locking
class ThreadSafeSingleton:
    _instance = None
    _lock = threading.Lock()
    
    def __new__(cls):
        if cls._instance is None:
            with cls._lock:
                if cls._instance is None:
                    cls._instance = super().__new__(cls)
        return cls._instance

# Producer-Consumer with multiple consumers
class ProducerConsumer:
    def __init__(self, max_queue_size=10):
        self.queue = queue.Queue(maxsize=max_queue_size)
        self.shutdown_event = Event()
    
    def producer(self, items):
        for item in items:
            if self.shutdown_event.is_set():
                break
            self.queue.put(item)
            print(f"Produced: {item}")
            time.sleep(0.1)
        
        # Signal completion
        self.queue.put(None)
    
    def consumer(self, consumer_id):
        while not self.shutdown_event.is_set():
            try:
                item = self.queue.get(timeout=1)
                if item is None:
                    self.queue.put(None)  # Re-queue sentinel for other consumers
                    break
                
                print(f"Consumer {consumer_id} processing: {item}")
                time.sleep(0.2)  # Simulate work
                self.queue.task_done()
                
            except queue.Empty:
                continue
    
    def shutdown(self):
        self.shutdown_event.set()

# Thread coordination with Barrier
def worker_with_barrier(barrier, worker_id):
    print(f"Worker {worker_id} starting")
    time.sleep(1)  # Simulate work
    
    print(f"Worker {worker_id} waiting at barrier")
    barrier.wait()  # Wait for all workers
    
    print(f"Worker {worker_id} continuing after barrier")

def demonstrate_barrier():
    num_workers = 3
    barrier = Barrier(num_workers)
    
    threads = []
    for i in range(num_workers):
        t = threading.Thread(target=worker_with_barrier, args=(barrier, i))
        threads.append(t)
        t.start()
    
    for t in threads:
        t.join()

# Resource pool with semaphore
class ResourcePool:
    def __init__(self, resources):
        self.resources = queue.Queue()
        self.semaphore = Semaphore(len(resources))
        
        for resource in resources:
            self.resources.put(resource)
    
    def acquire(self, timeout=None):
        if self.semaphore.acquire(timeout=timeout):
            return self.resources.get()
        return None
    
    def release(self, resource):
        self.resources.put(resource)
        self.semaphore.release()

# Thread-local data
thread_local_data = threading.local()

def use_thread_local():
    # Each thread gets its own copy
    thread_local_data.value = threading.current_thread().name
    time.sleep(1)
    print(f"Thread {threading.current_thread().name}: {thread_local_data.value}")

# Advanced ThreadPoolExecutor usage
class AdvancedThreadPool:
    def __init__(self, max_workers=4):
        self.executor = ThreadPoolExecutor(max_workers=max_workers)
        self.futures = []
    
    def submit_task(self, func, *args, **kwargs):
        future = self.executor.submit(func, *args, **kwargs)
        self.futures.append(future)
        return future
    
    def wait_for_completion(self, timeout=None):
        results = []
        for future in as_completed(self.futures, timeout=timeout):
            try:
                result = future.result()
                results.append(result)
            except Exception as e:
                print(f"Task failed: {e}")
                results.append(None)
        return results
    
    def shutdown(self):
        self.executor.shutdown(wait=True)

def expensive_task(n):
    time.sleep(1)
    return n ** 2

# Usage
pool = AdvancedThreadPool(max_workers=3)
for i in range(5):
    pool.submit_task(expensive_task, i)

results = pool.wait_for_completion(timeout=10)
print(f"Results: {results}")
pool.shutdown()
```

### Multiprocessing
```python
import multiprocessing as mp
from multiprocessing import Pool, Process, Queue, Value, Array, Lock
from concurrent.futures import ProcessPoolExecutor
import os
import time

# Basic multiprocessing
def worker_function(name, shared_value, lock):
    with lock:
        temp = shared_value.value
        time.sleep(0.01)  # Simulate work
        shared_value.value = temp + 1
        print(f"Worker {name}: {shared_value.value}")

def basic_multiprocessing():
    # Shared memory
    shared_value = Value('i', 0)  # 'i' for integer
    lock = Lock()
    
    processes = []
    for i in range(4):
        p = Process(target=worker_function, args=(f"Process-{i}", shared_value, lock))
        processes.append(p)
        p.start()
    
    for p in processes:
        p.join()
    
    print(f"Final value: {shared_value.value}")

# Process pool for CPU-intensive tasks
def cpu_intensive_task(n):
    """Simulate CPU-intensive work"""
    result = 0
    for i in range(n * 1000000):
        result += i ** 0.5
    return result

def process_pool_example():
    numbers = [10, 20, 30, 40, 50]
    
    # Sequential processing
    start_time = time.time()
    sequential_results = [cpu_intensive_task(n) for n in numbers]
    sequential_time = time.time() - start_time
    
    # Parallel processing
    start_time = time.time()
    with Pool(processes=mp.cpu_count()) as pool:
        parallel_results = pool.map(cpu_intensive_task, numbers)
    parallel_time = time.time() - start_time
    
    print(f"Sequential time: {sequential_time:.2f}s")
    print(f"Parallel time: {parallel_time:.2f}s")
    print(f"Speedup: {sequential_time/parallel_time:.2f}x")

# Producer-Consumer with multiprocessing
def producer(queue, items):
    for item in items:
        queue.put(item)
        print(f"Produced: {item}")
        time.sleep(0.1)
    queue.put(None)  # Sentinel

def consumer(queue, consumer_id):
    while True:
        item = queue.get()
        if item is None:
            break
        print(f"Consumer {consumer_id} processing: {item}")
        time.sleep(0.2)

def multiprocess_producer_consumer():
    queue = Queue()
    items = list(range(10))
    
    # Start producer
    producer_process = Process(target=producer, args=(queue, items))
    producer_process.start()
    
    # Start consumers
    consumer_processes = []
    for i in range(2):
        p = Process(target=consumer, args=(queue, i))
        consumer_processes.append(p)
        p.start()
    
    # Wait for producer to finish
    producer_process.join()
    
    # Send sentinel to each consumer
    for _ in consumer_processes:
        queue.put(None)
    
    # Wait for consumers to finish
    for p in consumer_processes:
        p.join()

# Advanced: Custom process pool with error handling
class RobustProcessPool:
    def __init__(self, max_workers=None):
        self.max_workers = max_workers or mp.cpu_count()
        self.executor = ProcessPoolExecutor(max_workers=self.max_workers)
    
    def map_with_timeout(self, func, iterable, timeout=60):
        futures = {self.executor.submit(func, item): item for item in iterable}
        results = {}
        errors = {}
        
        for future in futures:
            item = futures[future]
            try:
                result = future.result(timeout=timeout)
                results[item] = result
            except Exception as e:
                errors[item] = str(e)
        
        return results, errors
    
    def shutdown(self):
        self.executor.shutdown(wait=True)

# Memory-mapped files for large data sharing
def create_shared_array(size):
    return Array('d', [0.0] * size)  # 'd' for double

def worker_with_shared_array(shared_array, start_idx, end_idx):
    # Process a portion of the shared array
    for i in range(start_idx, end_idx):
        shared_array[i] = i ** 2

def parallel_array_processing():
    size = 1000000
    shared_array = create_shared_array(size)
    
    # Split work among processes
    num_processes = mp.cpu_count()
    chunk_size = size // num_processes
    
    processes = []
    for i in range(num_processes):
        start_idx = i * chunk_size
        end_idx = start_idx + chunk_size if i < num_processes - 1 else size
        
        p = Process(target=worker_with_shared_array, 
                   args=(shared_array, start_idx, end_idx))
        processes.append(p)
        p.start()
    
    for p in processes:
        p.join()
    
    # Verify results
    print(f"First 10 values: {shared_array[:10]}")
    print(f"Last 10 values: {shared_array[-10:]}")
```

### Asyncio Advanced Patterns
```python
import asyncio
import aiohttp
import aiofiles
from asyncio import Queue, Semaphore, Event, Condition
import time
from typing import List, Any

# Advanced async patterns
class AsyncProducerConsumer:
    def __init__(self, max_queue_size=10):
        self.queue = Queue(maxsize=max_queue_size)
        self.shutdown_event = Event()
    
    async def producer(self, items):
        for item in items:
            if self.shutdown_event.is_set():
                break
            await self.queue.put(item)
            print(f"Produced: {item}")
            await asyncio.sleep(0.1)
        
        await self.queue.put(None)  # Sentinel
    
    async def consumer(self, consumer_id):
        while not self.shutdown_event.is_set():
            try:
                item = await asyncio.wait_for(self.queue.get(), timeout=1.0)
                if item is None:
                    await self.queue.put(None)  # Re-queue for other consumers
                    break
                
                print(f"Consumer {consumer_id} processing: {item}")
                await asyncio.sleep(0.2)  # Simulate async work
                
            except asyncio.TimeoutError:
                continue
    
    def shutdown(self):
        self.shutdown_event.set()

# Rate limiting with semaphore
class RateLimiter:
    def __init__(self, max_concurrent=5, rate_per_second=10):
        self.semaphore = Semaphore(max_concurrent)
        self.rate_per_second = rate_per_second
        self.last_request_time = time.time()
        self.request_times = []
    
    async def __aenter__(self):
        await self.semaphore.acquire()
        
        # Rate limiting
        now = time.time()
        self.request_times = [t for t in self.request_times if now - t < 1.0]
        
        if len(self.request_times) >= self.rate_per_second:
            sleep_time = 1.0 - (now - self.request_times[0])
            if sleep_time > 0:
                await asyncio.sleep(sleep_time)
        
        self.request_times.append(now)
        return self
    
    async def __aexit__(self, exc_type, exc_val, exc_tb):
        self.semaphore.release()

# Async HTTP client with retry and circuit breaker
class AsyncHTTPClient:
    def __init__(self, max_retries=3, timeout=10):
        self.max_retries = max_retries
        self.timeout = timeout
        self.session = None
        self.rate_limiter = RateLimiter(max_concurrent=10, rate_per_second=20)
    
    async def __aenter__(self):
        self.session = aiohttp.ClientSession(timeout=aiohttp.ClientTimeout(total=self.timeout))
        return self
    
    async def __aexit__(self, exc_type, exc_val, exc_tb):
        if self.session:
            await self.session.close()
    
    async def fetch_with_retry(self, url, **kwargs):
        last_exception = None
        
        for attempt in range(self.max_retries + 1):
            try:
                async with self.rate_limiter:
                    async with self.session.get(url, **kwargs) as response:
                        if response.status == 200:
                            return await response.json()
                        else:
                            raise aiohttp.ClientResponseError(
                                request_info=response.request_info,
                                history=response.history,
                                status=response.status
                            )
            except Exception as e:
                last_exception = e
                if attempt < self.max_retries:
                    wait_time = (2 ** attempt) * 0.1  # Exponential backoff
                    await asyncio.sleep(wait_time)
                    print(f"Retry {attempt + 1} for {url}")
        
        raise last_exception

# Async file processing
async def process_large_file(filename):
    async with aiofiles.open(filename, 'r') as file:
        line_count = 0
        word_count = 0
        
        async for line in file:
            line_count += 1
            word_count += len(line.split())
            
            # Yield control periodically
            if line_count % 1000 == 0:
                await asyncio.sleep(0)
        
        return line_count, word_count

# Async task management
class AsyncTaskManager:
    def __init__(self, max_concurrent_tasks=10):
        self.semaphore = Semaphore(max_concurrent_tasks)
        self.tasks = []
        self.results = []
        self.errors = []
    
    async def add_task(self, coro):
        async def wrapped_task():
            async with self.semaphore:
                try:
                    result = await coro
                    self.results.append(result)
                    return result
                except Exception as e:
                    self.errors.append(e)
                    raise
        
        task = asyncio.create_task(wrapped_task())
        self.tasks.append(task)
        return task
    
    async def wait_all(self, return_when=asyncio.ALL_COMPLETED):
        if not self.tasks:
            return [], []
        
        done, pending = await asyncio.wait(self.tasks, return_when=return_when)
        
        results = []
        exceptions = []
        
        for task in done:
            try:
                result = await task
                results.append(result)
            except Exception as e:
                exceptions.append(e)
        
        return results, exceptions

# Usage example
async def main_async_example():
    # Producer-Consumer example
    pc = AsyncProducerConsumer(max_queue_size=5)
    
    producer_task = asyncio.create_task(pc.producer(range(10)))
    consumer_tasks = [
        asyncio.create_task(pc.consumer(i))
        for i in range(3)
    ]
    
    # Wait for producer to finish
    await producer_task
    
    # Wait for consumers to finish
    await asyncio.gather(*consumer_tasks)
    
    # HTTP client example
    urls = [
        'https://httpbin.org/delay/1',
        'https://httpbin.org/delay/2',
        'https://httpbin.org/status/200'
    ]
    
    async with AsyncHTTPClient(max_retries=2) as client:
        tasks = [client.fetch_with_retry(url) for url in urls]
        results = await asyncio.gather(*tasks, return_exceptions=True)
        
        for url, result in zip(urls, results):
            if isinstance(result, Exception):
                print(f"Error fetching {url}: {result}")
            else:
                print(f"Success fetching {url}")

# Run async examples
# asyncio.run(main_async_example())
```

## Package Building and Distribution

### Creating a Package Structure
```python
# Directory structure:
"""
my_package/
├── setup.py
├── pyproject.toml
├── README.md
├── LICENSE
├── requirements.txt
├── tests/
│   ├── __init__.py
│   ├── test_core.py
│   └── test_utils.py
├── docs/
│   └── index.md
└── my_package/
    ├── __init__.py
    ├── core.py
    ├── utils.py
    └── cli.py
"""

# setup.py (traditional approach)
from setuptools import setup, find_packages
import os

# Read long description from README
def read_readme():
    with open("README.md", "r", encoding="utf-8") as fh:
        return fh.read()

# Read requirements from file
def read_requirements():
    with open("requirements.txt", "r", encoding="utf-8") as fh:
        return [line.strip() for line in fh if line.strip() and not line.startswith("#")]

setup(
    name="my-awesome-package",
    version="0.1.0",
    author="Your Name",
    author_email="your.email@example.com",
    description="A short description of your package",
    long_description=read_readme(),
    long_description_content_type="text/markdown",
    url="https://github.com/yourusername/my-awesome-package",
    project_urls={
        "Bug Reports": "https://github.com/yourusername/my-awesome-package/issues",
        "Source": "https://github.com/yourusername/my-awesome-package",
        "Documentation": "https://my-awesome-package.readthedocs.io/",
    },
    packages=find_packages(exclude=["tests*"]),
    classifiers=[
        "Development Status :: 3 - Alpha",
        "Intended Audience :: Developers",
        "License :: OSI Approved :: MIT License",
        "Operating System :: OS Independent",
        "Programming Language :: Python :: 3",
        "Programming Language :: Python :: 3.8",
        "Programming Language :: Python :: 3.9",
        "Programming Language :: Python :: 3.10",
        "Programming Language :: Python :: 3.11",
    ],
    python_requires=">=3.8",
    install_requires=read_requirements(),
    extras_require={
        "dev": ["pytest>=6.0", "black", "flake8", "mypy"],
        "docs": ["sphinx", "sphinx-rtd-theme"],
    },
    entry_points={
        "console_scripts": [
            "my-cli=my_package.cli:main",
        ],
    },
    include_package_data=True,
    package_data={
        "my_package": ["data/*.json", "templates/*.html"],
    },
)

# pyproject.toml (modern approach)
pyproject_content = """
[build-system]
requires = ["setuptools>=45", "wheel"]
build-backend = "setuptools.build_meta"

[project]
name = "my-awesome-package"
version = "0.1.0"
description = "A short description of your package"
readme = "README.md"
requires-python = ">=3.8"
license = {text = "MIT"}
authors = [
    {name = "Your Name", email = "your.email@example.com"},
]
maintainers = [
    {name = "Your Name", email = "your.email@example.com"},
]
keywords = ["python", "package", "example"]
classifiers = [
    "Development Status :: 3 - Alpha",
    "Intended Audience :: Developers",
    "License :: OSI Approved :: MIT License",
    "Operating System :: OS Independent",
    "Programming Language :: Python :: 3",
    "Programming Language :: Python :: 3.8",
    "Programming Language :: Python :: 3.9",
    "Programming Language :: Python :: 3.10",
    "Programming Language :: Python :: 3.11",
]
dependencies = [
    "requests>=2.25.0",
    "click>=8.0.0",
]

[project.optional-dependencies]
dev = ["pytest>=6.0", "black", "flake8", "mypy"]
docs = ["sphinx", "sphinx-rtd-theme"]

[project.urls]
Homepage = "https://github.com/yourusername/my-awesome-package"
Documentation = "https://my-awesome-package.readthedocs.io/"
Repository = "https://github.com/yourusername/my-awesome-package"
"Bug Reports" = "https://github.com/yourusername/my-awesome-package/issues"

[project.scripts]
my-cli = "my_package.cli:main"

[tool.setuptools.packages.find]
exclude = ["tests*"]

[tool.setuptools.package-data]
my_package = ["data/*.json", "templates/*.html"]

[tool.black]
line-length = 88
target-version = ['py38']
include = '\\.pyi?$'
extend-exclude = '''
/(
  # directories
  \\.eggs
  | \\.git
  | \\.hg
  | \\.mypy_cache
  | \\.tox
  | \\.venv
  | build
  | dist
)/
'''

[tool.mypy]
python_version = "3.8"
warn_return_any = true
warn_unused_configs = true
disallow_untyped_defs = true

[tool.pytest.ini_options]
testpaths = ["tests"]
addopts = "-v --tb=short"
"""

# __init__.py with version management
__init_content = '''
"""My Awesome Package

A comprehensive Python package for awesome things.
"""

import sys
from typing import Tuple

# Version info
__version__ = "0.1.0"
__author__ = "Your Name"
__email__ = "your.email@example.com"

# Minimum Python version check
MIN_PYTHON = (3, 8)
if sys.version_info < MIN_PYTHON:
    sys.exit(f"Python {MIN_PYTHON[0]}.{MIN_PYTHON[1]} or later is required.")

# Import main components
from .core import MainClass, main_function
from .utils import helper_function, UtilityClass

# Define what gets imported with "from my_package import *"
__all__ = [
    "MainClass",
    "main_function", 
    "helper_function",
    "UtilityClass",
    "__version__",
]

def get_version() -> str:
    """Return the package version."""
    return __version__

def get_version_info() -> Tuple[int, ...]:
    """Return version as a tuple of integers."""
    return tuple(map(int, __version__.split(".")))
'''

# CLI module with Click
cli_content = '''
"""Command-line interface for my-awesome-package."""

import click
from . import __version__
from .core import main_function

@click.group()
@click.version_option(version=__version__)
@click.option('--verbose', '-v', is_flag=True, help='Verbose output')
@click.pass_context
def cli(ctx, verbose):
    """My Awesome Package CLI."""
    ctx.ensure_object(dict)
    ctx.obj['verbose'] = verbose

@cli.command()
@click.argument('input_file', type=click.Path(exists=True))
@click.option('--output', '-o', type=click.Path(), help='Output file')
@click.option('--format', type=click.Choice(['json', 'csv', 'xml']), default='json')
@click.pass_context
def process(ctx, input_file, output, format):
    """Process the input file."""
    verbose = ctx.obj['verbose']
    
    if verbose:
        click.echo(f"Processing {input_file} with format {format}")
    
    try:
        result = main_function(input_file, output_format=format)
        
        if output:
            with open(output, 'w') as f:
                f.write(result)
            click.echo(f"Output written to {output}")
        else:
            click.echo(result)
    
    except Exception as e:
        click.echo(f"Error: {e}", err=True)
        raise click.ClickException(str(e))

@cli.command()
@click.option('--count', default=1, help='Number of greetings')
@click.argument('name')
def hello(count, name):
    """Simple program that greets NAME for a total of COUNT times."""
    for _ in range(count):
        click.echo(f'Hello, {name}!')

def main():
    """Entry point for the CLI."""
    cli()

if __name__ == '__main__':
    main()
'''
```

### Building and Publishing
```bash
# Build commands
python -m pip install --upgrade build twine

# Build the package
python -m build

# Check the built package
twine check dist/*

# Upload to TestPyPI (for testing)
twine upload --repository testpypi dist/*

# Upload to PyPI (production)
twine upload dist/*

# Install from TestPyPI
pip install --index-url https://test.pypi.org/simple/ my-awesome-package

# Development installation
pip install -e .
pip install -e .[dev]  # With development dependencies
```

### Continuous Integration Configuration
```yaml
# .github/workflows/ci.yml
name: CI

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        python-version: [3.8, 3.9, "3.10", "3.11"]

    steps:
    - uses: actions/checkout@v3
    
    - name: Set up Python ${{ matrix.python-version }}
      uses: actions/setup-python@v4
      with:
        python-version: ${{ matrix.python-version }}
    
    - name: Install dependencies
      run: |
        python -m pip install --upgrade pip
        pip install -e .[dev]
    
    - name: Lint with flake8
      run: |
        flake8 my_package tests
    
    - name: Format check with black
      run: |
        black --check my_package tests
    
    - name: Type check with mypy
      run: |
        mypy my_package
    
    - name: Test with pytest
      run: |
        pytest --cov=my_package --cov-report=xml
    
    - name: Upload coverage to Codecov
      uses: codecov/codecov-action@v3
      with:
        file: ./coverage.xml

  build:
    runs-on: ubuntu-latest
    needs: test
    
    steps:
    - uses: actions/checkout@v3
    
    - name: Set up Python
      uses: actions/setup-python@v4
      with:
        python-version: "3.10"
    
    - name: Install build dependencies
      run: |
        python -m pip install --upgrade pip build
    
    - name: Build package
      run: |
        python -m build
    
    - name: Check package
      run: |
        python -m pip install --upgrade twine
        twine check dist/*
```