# Python Intermediate Cheatsheet

## Advanced Data Structures

### Collections Module
```python
from collections import Counter, defaultdict, deque, namedtuple, OrderedDict

# Counter - count hashable objects
text = "hello world"
counter = Counter(text)
print(counter)  # Counter({'l': 3, 'o': 2, 'h': 1, 'e': 1, ' ': 1, 'w': 1, 'r': 1, 'd': 1})

# Most common elements
print(counter.most_common(3))  # [('l', 3), ('o', 2), ('h', 1)]

# defaultdict - dictionary with default values
dd = defaultdict(list)
dd['fruits'].append('apple')
dd['fruits'].append('banana')
print(dd)  # defaultdict(<class 'list'>, {'fruits': ['apple', 'banana']})

# deque - double-ended queue
dq = deque([1, 2, 3])
dq.appendleft(0)    # Add to left
dq.append(4)        # Add to right
print(dq)           # deque([0, 1, 2, 3, 4])
dq.popleft()        # Remove from left
dq.pop()            # Remove from right

# namedtuple - tuple with named fields
Person = namedtuple('Person', ['name', 'age', 'city'])
person = Person('Alice', 30, 'New York')
print(person.name)  # Alice
print(person.age)   # 30

# OrderedDict - dictionary that remembers insertion order
od = OrderedDict()
od['first'] = 1
od['second'] = 2
od['third'] = 3
print(list(od.keys()))  # ['first', 'second', 'third']
```

### Advanced Dictionary Operations
```python
# Dictionary comprehensions
squares = {x: x**2 for x in range(5)}
print(squares)  # {0: 0, 1: 1, 2: 4, 3: 9, 4: 16}

# Conditional dictionary comprehension
even_squares = {x: x**2 for x in range(10) if x % 2 == 0}
print(even_squares)  # {0: 0, 2: 4, 4: 16, 6: 36, 8: 64}

# Dictionary merging (Python 3.9+)
dict1 = {'a': 1, 'b': 2}
dict2 = {'c': 3, 'd': 4}
merged = dict1 | dict2  # Union operator
print(merged)  # {'a': 1, 'b': 2, 'c': 3, 'd': 4}

# Dictionary unpacking
dict1.update(dict2)  # In-place merge
merged2 = {**dict1, **dict2}  # Unpacking merge

# get() with default and setdefault()
data = {}
count = data.get('visits', 0) + 1
data.setdefault('visits', 0)
data['visits'] += 1
```

### Set Operations
```python
set1 = {1, 2, 3, 4, 5}
set2 = {4, 5, 6, 7, 8}

# Set operations
union = set1 | set2  # or set1.union(set2)
intersection = set1 & set2  # or set1.intersection(set2)
difference = set1 - set2  # or set1.difference(set2)
symmetric_diff = set1 ^ set2  # or set1.symmetric_difference(set2)

print(f"Union: {union}")  # {1, 2, 3, 4, 5, 6, 7, 8}
print(f"Intersection: {intersection}")  # {4, 5}
print(f"Difference: {difference}")  # {1, 2, 3}
print(f"Symmetric difference: {symmetric_diff}")  # {1, 2, 3, 6, 7, 8}

# Set comprehensions
even_squares = {x**2 for x in range(10) if x % 2 == 0}
print(even_squares)  # {0, 4, 16, 36, 64}
```

## List Comprehensions and Generators

### List Comprehensions
```python
# Basic list comprehension
squares = [x**2 for x in range(10)]
print(squares)  # [0, 1, 4, 9, 16, 25, 36, 49, 64, 81]

# Conditional list comprehension
even_squares = [x**2 for x in range(10) if x % 2 == 0]
print(even_squares)  # [0, 4, 16, 36, 64]

# Nested list comprehension
matrix = [[1, 2, 3], [4, 5, 6], [7, 8, 9]]
flattened = [item for row in matrix for item in row]
print(flattened)  # [1, 2, 3, 4, 5, 6, 7, 8, 9]

# Multiple conditions
filtered = [x for x in range(100) if x % 2 == 0 if x % 3 == 0]
print(filtered[:5])  # [0, 6, 12, 18, 24]

# if-else in list comprehension
result = [x if x % 2 == 0 else -x for x in range(10)]
print(result)  # [0, -1, 2, -3, 4, -5, 6, -7, 8, -9]
```

### Generator Expressions and Functions
```python
# Generator expression
squares_gen = (x**2 for x in range(10))
print(type(squares_gen))  # <class 'generator'>
print(list(squares_gen))  # [0, 1, 4, 9, 16, 25, 36, 49, 64, 81]

# Generator function
def fibonacci(n):
    a, b = 0, 1
    for _ in range(n):
        yield a
        a, b = b, a + b

fib_gen = fibonacci(10)
print(list(fib_gen))  # [0, 1, 1, 2, 3, 5, 8, 13, 21, 34]

# Generator with send() method
def counter():
    count = 0
    while True:
        increment = yield count
        if increment is not None:
            count += increment
        else:
            count += 1

gen = counter()
print(next(gen))  # 0
print(gen.send(5))  # 5
print(next(gen))  # 6

# Generator pipeline
def read_numbers(filename):
    with open(filename) as f:
        for line in f:
            yield int(line.strip())

def square_numbers(numbers):
    for num in numbers:
        yield num ** 2

def sum_numbers(numbers):
    return sum(numbers)

# Pipeline usage
# result = sum_numbers(square_numbers(read_numbers('numbers.txt')))
```

## Decorators and Context Managers

### Function Decorators
```python
import functools
import time

# Basic decorator
def my_decorator(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        print(f"Calling {func.__name__}")
        result = func(*args, **kwargs)
        print(f"Finished {func.__name__}")
        return result
    return wrapper

@my_decorator
def greet(name):
    return f"Hello, {name}!"

print(greet("Alice"))

# Decorator with arguments
def repeat(times):
    def decorator(func):
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            for _ in range(times):
                result = func(*args, **kwargs)
            return result
        return wrapper
    return decorator

@repeat(3)
def say_hello():
    print("Hello!")

say_hello()  # Prints "Hello!" 3 times

# Timing decorator
def timer(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        start = time.time()
        result = func(*args, **kwargs)
        end = time.time()
        print(f"{func.__name__} took {end - start:.4f} seconds")
        return result
    return wrapper

@timer
def slow_function():
    time.sleep(1)
    return "Done"

# Class-based decorator
class CountCalls:
    def __init__(self, func):
        self.func = func
        self.count = 0
        functools.update_wrapper(self, func)
    
    def __call__(self, *args, **kwargs):
        self.count += 1
        print(f"{self.func.__name__} has been called {self.count} times")
        return self.func(*args, **kwargs)

@CountCalls
def test_function():
    pass

test_function()  # test_function has been called 1 times
test_function()  # test_function has been called 2 times
```

### Built-in Decorators
```python
# @property, @staticmethod, @classmethod
class Person:
    def __init__(self, first_name, last_name):
        self.first_name = first_name
        self.last_name = last_name
        self._age = 0
    
    @property
    def full_name(self):
        return f"{self.first_name} {self.last_name}"
    
    @property
    def age(self):
        return self._age
    
    @age.setter
    def age(self, value):
        if value < 0:
            raise ValueError("Age cannot be negative")
        self._age = value
    
    @staticmethod
    def is_adult(age):
        return age >= 18
    
    @classmethod
    def from_string(cls, name_string):
        first, last = name_string.split(' ', 1)
        return cls(first, last)

person = Person("John", "Doe")
print(person.full_name)  # John Doe
person.age = 25
print(person.age)  # 25
print(Person.is_adult(20))  # True
person2 = Person.from_string("Jane Smith")
```

### Context Managers
```python
# Basic context manager with class
class FileManager:
    def __init__(self, filename, mode):
        self.filename = filename
        self.mode = mode
        self.file = None
    
    def __enter__(self):
        print(f"Opening {self.filename}")
        self.file = open(self.filename, self.mode)
        return self.file
    
    def __exit__(self, exc_type, exc_value, traceback):
        print(f"Closing {self.filename}")
        if self.file:
            self.file.close()
        if exc_type:
            print(f"Exception occurred: {exc_value}")
        return False  # Don't suppress exceptions

# Using the context manager
with FileManager('test.txt', 'w') as f:
    f.write('Hello, World!')

# Context manager with contextlib
from contextlib import contextmanager

@contextmanager
def timer_context():
    start = time.time()
    print("Timer started")
    try:
        yield
    finally:
        end = time.time()
        print(f"Timer finished: {end - start:.4f} seconds")

with timer_context():
    time.sleep(1)
    print("Doing some work...")

# Multiple context managers
with open('file1.txt', 'r') as f1, open('file2.txt', 'w') as f2:
    data = f1.read()
    f2.write(data.upper())
```

## Lambda Functions and Functional Programming

### Lambda Functions
```python
# Basic lambda functions
square = lambda x: x ** 2
add = lambda x, y: x + y
is_even = lambda x: x % 2 == 0

print(square(5))  # 25
print(add(3, 4))  # 7
print(is_even(6))  # True

# Lambda with conditional
max_of_two = lambda x, y: x if x > y else y
print(max_of_two(10, 5))  # 10

# Lambda in data structures
operations = {
    'add': lambda x, y: x + y,
    'subtract': lambda x, y: x - y,
    'multiply': lambda x, y: x * y,
    'divide': lambda x, y: x / y if y != 0 else None
}

print(operations['add'](5, 3))  # 8
```

### map(), filter(), and reduce()
```python
from functools import reduce

numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

# map() - apply function to all items
squares = list(map(lambda x: x**2, numbers))
print(squares)  # [1, 4, 9, 16, 25, 36, 49, 64, 81, 100]

# map with multiple iterables
nums1 = [1, 2, 3, 4]
nums2 = [10, 20, 30, 40]
sums = list(map(lambda x, y: x + y, nums1, nums2))
print(sums)  # [11, 22, 33, 44]

# filter() - filter items based on condition
evens = list(filter(lambda x: x % 2 == 0, numbers))
print(evens)  # [2, 4, 6, 8, 10]

# reduce() - reduce sequence to single value
total = reduce(lambda x, y: x + y, numbers)
print(total)  # 55

factorial = reduce(lambda x, y: x * y, range(1, 6))
print(factorial)  # 120

# Complex example: word frequency
text = "hello world hello python world"
words = text.split()
word_freq = reduce(
    lambda freq, word: {**freq, word: freq.get(word, 0) + 1},
    words,
    {}
)
print(word_freq)  # {'hello': 2, 'world': 2, 'python': 1}
```

### Higher-Order Functions
```python
def apply_operation(operation, x, y):
    return operation(x, y)

def create_multiplier(n):
    return lambda x: x * n

def compose(*functions):
    return reduce(lambda f, g: lambda x: f(g(x)), functions, lambda x: x)

# Using higher-order functions
result = apply_operation(lambda x, y: x ** y, 2, 3)
print(result)  # 8

double = create_multiplier(2)
triple = create_multiplier(3)
print(double(5))  # 10
print(triple(4))  # 12

# Function composition
add_one = lambda x: x + 1
multiply_by_two = lambda x: x * 2
square = lambda x: x ** 2

# Compose functions: square(multiply_by_two(add_one(x)))
composed = compose(square, multiply_by_two, add_one)
print(composed(3))  # ((3 + 1) * 2)^2 = 64
```

## Regular Expressions

### Basic Patterns
```python
import re

text = "The quick brown fox jumps over 123 lazy dogs. Email: test@example.com"

# Basic matching
pattern = r"fox"
match = re.search(pattern, text)
if match:
    print(f"Found '{match.group()}' at position {match.start()}-{match.end()}")

# Find all matches
numbers = re.findall(r'\d+', text)
print(numbers)  # ['123']

# Split by pattern
words = re.split(r'\s+', text)
print(len(words))  # Number of words

# Replace patterns
cleaned = re.sub(r'\d+', 'XXX', text)
print(cleaned)  # Numbers replaced with XXX
```

### Advanced Patterns
```python
# Groups and capturing
email_pattern = r'([a-zA-Z0-9._%+-]+)@([a-zA-Z0-9.-]+\.[a-zA-Z]{2,})'
email_text = "Contact us at: support@company.com or admin@test.org"

matches = re.finditer(email_pattern, email_text)
for match in matches:
    print(f"Full email: {match.group(0)}")
    print(f"Username: {match.group(1)}")
    print(f"Domain: {match.group(2)}")

# Named groups
phone_pattern = r'(?P<area>\d{3})-(?P<exchange>\d{3})-(?P<number>\d{4})'
phone_text = "Call me at 555-123-4567"
match = re.search(phone_pattern, phone_text)
if match:
    print(f"Area code: {match.group('area')}")
    print(f"Exchange: {match.group('exchange')}")
    print(f"Number: {match.group('number')}")

# Lookahead and lookbehind
# Positive lookahead: match 'foo' only if followed by 'bar'
text = "foobar foobaz"
matches = re.findall(r'foo(?=bar)', text)
print(matches)  # ['foo']

# Negative lookahead: match 'foo' only if NOT followed by 'bar'
matches = re.findall(r'foo(?!bar)', text)
print(matches)  # ['foo']
```

### Compiled Patterns and Flags
```python
# Compile patterns for better performance
email_pattern = re.compile(r'\b[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Z|a-z]{2,}\b')
text = "Email addresses: john@example.com, jane@test.org"
emails = email_pattern.findall(text)
print(emails)

# Flags
pattern = re.compile(r'hello', re.IGNORECASE | re.MULTILINE)
text = "HELLO\nworld\nHello again"
matches = pattern.findall(text)
print(matches)  # ['HELLO', 'Hello']

# Verbose mode for complex patterns
phone_pattern = re.compile(r'''
    (?P<area>\d{3})     # Area code
    -                   # Separator
    (?P<exchange>\d{3}) # Exchange
    -                   # Separator
    (?P<number>\d{4})   # Number
''', re.VERBOSE)
```

## Working with APIs and JSON

### JSON Processing
```python
import json

# Python to JSON
data = {
    'name': 'Alice',
    'age': 30,
    'city': 'New York',
    'hobbies': ['reading', 'swimming'],
    'married': True
}

# Serialize to JSON string
json_string = json.dumps(data, indent=2)
print(json_string)

# Serialize to file
with open('data.json', 'w') as f:
    json.dump(data, f, indent=2)

# Deserialize from JSON string
parsed_data = json.loads(json_string)
print(parsed_data['name'])  # Alice

# Deserialize from file
with open('data.json', 'r') as f:
    loaded_data = json.load(f)

# Custom JSON encoder
class DateTimeEncoder(json.JSONEncoder):
    def default(self, obj):
        if isinstance(obj, datetime):
            return obj.isoformat()
        return super().default(obj)

from datetime import datetime
data_with_date = {'timestamp': datetime.now()}
json_with_date = json.dumps(data_with_date, cls=DateTimeEncoder)
```

### HTTP Requests
```python
import requests
import json

# GET request
response = requests.get('https://api.github.com/users/octocat')
if response.status_code == 200:
    user_data = response.json()
    print(f"User: {user_data['name']}")
    print(f"Followers: {user_data['followers']}")

# POST request with JSON data
api_url = 'https://httpbin.org/post'
post_data = {'name': 'Alice', 'age': 30}
response = requests.post(api_url, json=post_data)
print(response.json())

# Custom headers and authentication
headers = {
    'Authorization': 'Bearer your-token-here',
    'User-Agent': 'My Python App 1.0'
}
response = requests.get(api_url, headers=headers)

# Session for persistent connections
session = requests.Session()
session.headers.update({'Authorization': 'Bearer token'})
response1 = session.get('https://api.example.com/endpoint1')
response2 = session.get('https://api.example.com/endpoint2')

# Error handling
try:
    response = requests.get('https://api.example.com/data', timeout=5)
    response.raise_for_status()  # Raises HTTPError for bad responses
    data = response.json()
except requests.exceptions.Timeout:
    print("Request timed out")
except requests.exceptions.HTTPError as e:
    print(f"HTTP error: {e}")
except requests.exceptions.RequestException as e:
    print(f"Request error: {e}")
```

### API Client Class
```python
class APIClient:
    def __init__(self, base_url, api_key=None):
        self.base_url = base_url.rstrip('/')
        self.session = requests.Session()
        if api_key:
            self.session.headers.update({'Authorization': f'Bearer {api_key}'})
    
    def get(self, endpoint, params=None):
        url = f"{self.base_url}/{endpoint.lstrip('/')}"
        response = self.session.get(url, params=params)
        response.raise_for_status()
        return response.json()
    
    def post(self, endpoint, data=None, json_data=None):
        url = f"{self.base_url}/{endpoint.lstrip('/')}"
        response = self.session.post(url, data=data, json=json_data)
        response.raise_for_status()
        return response.json()
    
    def put(self, endpoint, data=None, json_data=None):
        url = f"{self.base_url}/{endpoint.lstrip('/')}"
        response = self.session.put(url, data=data, json=json_data)
        response.raise_for_status()
        return response.json()
    
    def delete(self, endpoint):
        url = f"{self.base_url}/{endpoint.lstrip('/')}"
        response = self.session.delete(url)
        response.raise_for_status()
        return response.status_code == 204

# Usage
# client = APIClient('https://api.example.com', 'your-api-key')
# users = client.get('/users')
# new_user = client.post('/users', json_data={'name': 'John', 'email': 'john@example.com'})
```

## Database Connectivity

### SQLite Database Operations
```python
import sqlite3
from contextlib import contextmanager

# Basic database operations
conn = sqlite3.connect('example.db')
cursor = conn.cursor()

# Create table
cursor.execute('''
    CREATE TABLE IF NOT EXISTS users (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        name TEXT NOT NULL,
        email TEXT UNIQUE NOT NULL,
        age INTEGER
    )
''')

# Insert data
cursor.execute("INSERT INTO users (name, email, age) VALUES (?, ?, ?)", 
               ("Alice", "alice@example.com", 30))

# Insert multiple records
users_data = [
    ("Bob", "bob@example.com", 25),
    ("Charlie", "charlie@example.com", 35),
    ("Diana", "diana@example.com", 28)
]
cursor.executemany("INSERT INTO users (name, email, age) VALUES (?, ?, ?)", users_data)

# Commit changes
conn.commit()

# Query data
cursor.execute("SELECT * FROM users WHERE age > ?", (25,))
results = cursor.fetchall()
for row in results:
    print(row)

# Query with column names
cursor.execute("SELECT name, email FROM users")
columns = [description[0] for description in cursor.description]
rows = cursor.fetchall()
for row in rows:
    user_dict = dict(zip(columns, row))
    print(user_dict)

conn.close()

# Context manager for database operations
@contextmanager
def get_db_connection(db_path):
    conn = sqlite3.connect(db_path)
    try:
        yield conn
    finally:
        conn.close()

# Usage with context manager
with get_db_connection('example.db') as conn:
    cursor = conn.cursor()
    cursor.execute("SELECT COUNT(*) FROM users")
    count = cursor.fetchone()[0]
    print(f"Total users: {count}")
```

### Database Class with ORM-like Features
```python
class SimpleORM:
    def __init__(self, db_path):
        self.db_path = db_path
        self.connection = None
    
    def connect(self):
        self.connection = sqlite3.connect(self.db_path)
        self.connection.row_factory = sqlite3.Row  # Dict-like access
        return self.connection
    
    def disconnect(self):
        if self.connection:
            self.connection.close()
    
    def __enter__(self):
        self.connect()
        return self
    
    def __exit__(self, exc_type, exc_val, exc_tb):
        self.disconnect()
    
    def execute(self, query, params=None):
        cursor = self.connection.cursor()
        if params:
            cursor.execute(query, params)
        else:
            cursor.execute(query)
        return cursor
    
    def fetchall(self, query, params=None):
        cursor = self.execute(query, params)
        return [dict(row) for row in cursor.fetchall()]
    
    def fetchone(self, query, params=None):
        cursor = self.execute(query, params)
        row = cursor.fetchone()
        return dict(row) if row else None
    
    def insert(self, table, data):
        columns = ', '.join(data.keys())
        placeholders = ', '.join(['?' for _ in data])
        query = f"INSERT INTO {table} ({columns}) VALUES ({placeholders})"
        cursor = self.execute(query, list(data.values()))
        self.connection.commit()
        return cursor.lastrowid
    
    def update(self, table, data, where_clause, where_params):
        set_clause = ', '.join([f"{k} = ?" for k in data.keys()])
        query = f"UPDATE {table} SET {set_clause} WHERE {where_clause}"
        self.execute(query, list(data.values()) + where_params)
        self.connection.commit()
        return self.connection.total_changes

# Usage
with SimpleORM('example.db') as db:
    # Insert user
    user_id = db.insert('users', {
        'name': 'John Doe',
        'email': 'john@example.com',
        'age': 32
    })
    
    # Fetch users
    users = db.fetchall("SELECT * FROM users WHERE age > ?", [25])
    for user in users:
        print(f"{user['name']} ({user['age']})")
    
    # Update user
    db.update('users', {'age': 33}, 'id = ?', [user_id])
```

## Threading and Async Programming

### Basic Threading
```python
import threading
import time
import queue

# Basic thread example
def worker(name, duration):
    print(f"Worker {name} starting")
    time.sleep(duration)
    print(f"Worker {name} finished")

# Create and start threads
threads = []
for i in range(3):
    t = threading.Thread(target=worker, args=(f"Thread-{i}", 2))
    threads.append(t)
    t.start()

# Wait for all threads to complete
for t in threads:
    t.join()

print("All threads completed")

# Thread with shared data and locks
class Counter:
    def __init__(self):
        self.value = 0
        self.lock = threading.Lock()
    
    def increment(self):
        with self.lock:
            current = self.value
            time.sleep(0.001)  # Simulate some work
            self.value = current + 1
    
    def get_value(self):
        with self.lock:
            return self.value

counter = Counter()

def increment_counter(counter, count):
    for _ in range(count):
        counter.increment()

# Create multiple threads that increment the counter
threads = []
for i in range(5):
    t = threading.Thread(target=increment_counter, args=(counter, 100))
    threads.append(t)
    t.start()

for t in threads:
    t.join()

print(f"Final counter value: {counter.get_value()}")
```

### Thread Pool and Queue
```python
from concurrent.futures import ThreadPoolExecutor, as_completed
import requests

# Thread pool example
def fetch_url(url):
    try:
        response = requests.get(url, timeout=5)
        return f"{url}: {response.status_code}"
    except Exception as e:
        return f"{url}: Error - {str(e)}"

urls = [
    'https://httpbin.org/delay/1',
    'https://httpbin.org/delay/2',
    'https://httpbin.org/delay/3',
    'https://httpbin.org/status/200',
    'https://httpbin.org/status/404'
]

# Using ThreadPoolExecutor
with ThreadPoolExecutor(max_workers=3) as executor:
    # Submit all tasks
    future_to_url = {executor.submit(fetch_url, url): url for url in urls}
    
    # Process completed tasks as they finish
    for future in as_completed(future_to_url):
        url = future_to_url[future]
        try:
            result = future.result()
            print(result)
        except Exception as e:
            print(f"{url}: Exception - {e}")

# Producer-Consumer with Queue
def producer(q, items):
    for item in items:
        print(f"Producing {item}")
        q.put(item)
        time.sleep(0.5)
    q.put(None)  # Sentinel to indicate completion

def consumer(q, name):
    while True:
        item = q.get()
        if item is None:
            q.task_done()
            break
        print(f"Consumer {name} processing {item}")
        time.sleep(1)
        q.task_done()

# Create queue and threads
q = queue.Queue()
items = list(range(5))

producer_thread = threading.Thread(target=producer, args=(q, items))
consumer_threads = [
    threading.Thread(target=consumer, args=(q, f"Consumer-{i}"))
    for i in range(2)
]

# Start threads
producer_thread.start()
for t in consumer_threads:
    t.start()

# Wait for completion
producer_thread.join()
q.join()  # Wait for all tasks to be processed

# Stop consumer threads
for _ in consumer_threads:
    q.put(None)
for t in consumer_threads:
    t.join()
```

### Async Programming with asyncio
```python
import asyncio
import aiohttp
import time

# Basic async function
async def say_hello(name, delay):
    print(f"Hello {name}, starting...")
    await asyncio.sleep(delay)
    print(f"Hello {name}, finished!")
    return f"Greeting for {name}"

# Run async functions
async def main():
    # Sequential execution
    start = time.time()
    result1 = await say_hello("Alice", 1)
    result2 = await say_hello("Bob", 2)
    print(f"Sequential time: {time.time() - start:.2f}s")
    
    # Concurrent execution
    start = time.time()
    task1 = asyncio.create_task(say_hello("Charlie", 1))
    task2 = asyncio.create_task(say_hello("Diana", 2))
    
    results = await asyncio.gather(task1, task2)
    print(f"Concurrent time: {time.time() - start:.2f}s")
    print(f"Results: {results}")

# asyncio.run(main())

# Async HTTP requests
async def fetch_url_async(session, url):
    try:
        async with session.get(url) as response:
            return f"{url}: {response.status}"
    except Exception as e:
        return f"{url}: Error - {str(e)}"

async def fetch_multiple_urls():
    urls = [
        'https://httpbin.org/delay/1',
        'https://httpbin.org/delay/2',
        'https://httpbin.org/status/200',
        'https://httpbin.org/status/404'
    ]
    
    async with aiohttp.ClientSession() as session:
        tasks = [fetch_url_async(session, url) for url in urls]
        results = await asyncio.gather(*tasks)
        for result in results:
            print(result)

# asyncio.run(fetch_multiple_urls())

# Async generator
async def async_range(count):
    for i in range(count):
        await asyncio.sleep(0.1)
        yield i

async def consume_async_generator():
    async for value in async_range(5):
        print(f"Received: {value}")

# asyncio.run(consume_async_generator())
```

## Package Management and Virtual Environments

### Virtual Environments
```bash
# Create virtual environment
python -m venv myenv

# Activate virtual environment
# Windows
myenv\Scripts\activate
# macOS/Linux
source myenv/bin/activate

# Deactivate
deactivate

# Create requirements.txt
pip freeze > requirements.txt

# Install from requirements.txt
pip install -r requirements.txt

# Create virtual environment with specific Python version
python3.9 -m venv myenv39

# Using virtualenv (alternative)
pip install virtualenv
virtualenv myenv
```

### Package Management
```python
# setup.py for creating packages
from setuptools import setup, find_packages

setup(
    name="my_package",
    version="0.1.0",
    author="Your Name",
    author_email="your.email@example.com",
    description="A short description of your package",
    long_description=open("README.md").read(),
    long_description_content_type="text/markdown",
    url="https://github.com/yourusername/my_package",
    packages=find_packages(),
    classifiers=[
        "Programming Language :: Python :: 3",
        "License :: OSI Approved :: MIT License",
        "Operating System :: OS Independent",
    ],
    python_requires='>=3.6',
    install_requires=[
        "requests>=2.25.0",
        "click>=7.0",
    ],
    entry_points={
        'console_scripts': [
            'my-command=my_package.cli:main',
        ],
    },
)

# pyproject.toml (modern approach)
"""
[build-system]
requires = ["setuptools>=45", "wheel"]
build-backend = "setuptools.build_meta"

[project]
name = "my_package"
version = "0.1.0"
description = "A short description"
authors = [{name = "Your Name", email = "your.email@example.com"}]
dependencies = [
    "requests>=2.25.0",
    "click>=7.0",
]
requires-python = ">=3.6"

[project.scripts]
my-command = "my_package.cli:main"
"""
```

## Testing and Debugging

### unittest Framework
```python
import unittest
from unittest.mock import Mock, patch, MagicMock

class Calculator:
    def add(self, a, b):
        return a + b
    
    def divide(self, a, b):
        if b == 0:
            raise ValueError("Cannot divide by zero")
        return a / b
    
    def get_data_from_api(self):
        # Simulate API call
        import requests
        response = requests.get('https://api.example.com/data')
        return response.json()

class TestCalculator(unittest.TestCase):
    def setUp(self):
        self.calc = Calculator()
    
    def test_add(self):
        result = self.calc.add(2, 3)
        self.assertEqual(result, 5)
    
    def test_add_negative(self):
        result = self.calc.add(-1, 1)
        self.assertEqual(result, 0)
    
    def test_divide(self):
        result = self.calc.divide(10, 2)
        self.assertEqual(result, 5.0)
    
    def test_divide_by_zero(self):
        with self.assertRaises(ValueError):
            self.calc.divide(10, 0)
    
    @patch('requests.get')
    def test_get_data_from_api(self, mock_get):
        # Mock the API response
        mock_response = Mock()
        mock_response.json.return_value = {'data': 'test'}
        mock_get.return_value = mock_response
        
        result = self.calc.get_data_from_api()
        self.assertEqual(result, {'data': 'test'})
        mock_get.assert_called_once_with('https://api.example.com/data')

if __name__ == '__main__':
    unittest.main()
```

### pytest Framework
```python
import pytest
from unittest.mock import patch

# Test functions (not classes)
def test_add():
    calc = Calculator()
    assert calc.add(2, 3) == 5

def test_divide_by_zero():
    calc = Calculator()
    with pytest.raises(ValueError, match="Cannot divide by zero"):
        calc.divide(10, 0)

# Fixtures
@pytest.fixture
def calculator():
    return Calculator()

def test_add_with_fixture(calculator):
    assert calculator.add(2, 3) == 5

# Parametrized tests
@pytest.mark.parametrize("a,b,expected", [
    (2, 3, 5),
    (-1, 1, 0),
    (0, 0, 0),
    (10, -5, 5)
])
def test_add_parametrized(calculator, a, b, expected):
    assert calculator.add(a, b) == expected

# Mock with pytest
@patch('requests.get')
def test_api_call(mock_get, calculator):
    mock_get.return_value.json.return_value = {'data': 'test'}
    result = calculator.get_data_from_api()
    assert result == {'data': 'test'}

# Fixtures with scope
@pytest.fixture(scope="module")
def database_connection():
    # Setup
    conn = create_test_database()
    yield conn
    # Teardown
    conn.close()

# Running tests
# pytest test_file.py
# pytest -v (verbose)
# pytest -k "test_add" (run tests matching pattern)
# pytest --cov=my_module (with coverage)
```

### Debugging Techniques
```python
import pdb
import logging
from pprint import pprint

# Basic debugging with print
def debug_function(data):
    print(f"Input data: {data}")
    result = process_data(data)
    print(f"Processed result: {result}")
    return result

# Using pdb debugger
def problematic_function(data):
    processed = []
    for item in data:
        pdb.set_trace()  # Debugger will stop here
        if item > 0:
            processed.append(item * 2)
    return processed

# Logging for debugging
logging.basicConfig(level=logging.DEBUG, 
                   format='%(asctime)s - %(levelname)s - %(message)s')

def logged_function(data):
    logging.info(f"Processing {len(data)} items")
    result = []
    for i, item in enumerate(data):
        logging.debug(f"Processing item {i}: {item}")
        if item > 0:
            result.append(item * 2)
            logging.debug(f"Added {item * 2} to result")
    logging.info(f"Finished processing, result has {len(result)} items")
    return result

# Pretty printing for complex data structures
complex_data = {
    'users': [
        {'name': 'Alice', 'age': 30, 'skills': ['Python', 'JavaScript']},
        {'name': 'Bob', 'age': 25, 'skills': ['Java', 'C++']}
    ],
    'settings': {'debug': True, 'version': '1.0'}
}

pprint(complex_data, width=50, depth=2)

# Assert statements for debugging
def validate_data(data):
    assert isinstance(data, list), f"Expected list, got {type(data)}"
    assert len(data) > 0, "Data list cannot be empty"
    assert all(isinstance(x, (int, float)) for x in data), "All items must be numbers"
    return True

# Exception handling with traceback
import traceback

try:
    risky_operation()
except Exception as e:
    logging.error(f"Error occurred: {e}")
    logging.error(traceback.format_exc())
```

## Complete Example: Web Scraper with Database

```python
import requests
from bs4 import BeautifulSoup
import sqlite3
import time
import logging
from contextlib import contextmanager
from dataclasses import dataclass
from typing import List, Optional
import json

# Configure logging
logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

@dataclass
class Article:
    title: str
    url: str
    summary: str
    published_date: Optional[str] = None

class NewsDatabase:
    def __init__(self, db_path: str):
        self.db_path = db_path
        self._create_tables()
    
    @contextmanager
    def get_connection(self):
        conn = sqlite3.connect(self.db_path)
        try:
            yield conn
        finally:
            conn.close()
    
    def _create_tables(self):
        with self.get_connection() as conn:
            conn.execute('''
                CREATE TABLE IF NOT EXISTS articles (
                    id INTEGER PRIMARY KEY AUTOINCREMENT,
                    title TEXT NOT NULL,
                    url TEXT UNIQUE NOT NULL,
                    summary TEXT,
                    published_date TEXT,
                    scraped_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
                )
            ''')
            conn.commit()
    
    def save_article(self, article: Article) -> bool:
        with self.get_connection() as conn:
            try:
                conn.execute('''
                    INSERT INTO articles (title, url, summary, published_date)
                    VALUES (?, ?, ?, ?)
                ''', (article.title, article.url, article.summary, article.published_date))
                conn.commit()
                return True
            except sqlite3.IntegrityError:
                logger.warning(f"Article already exists: {article.url}")
                return False
    
    def get_articles(self, limit: int = 10) -> List[Article]:
        with self.get_connection() as conn:
            cursor = conn.execute('''
                SELECT title, url, summary, published_date
                FROM articles
                ORDER BY scraped_at DESC
                LIMIT ?
            ''', (limit,))
            
            return [Article(*row) for row in cursor.fetchall()]

class NewsScraper:
    def __init__(self, db: NewsDatabase):
        self.db = db
        self.session = requests.Session()
        self.session.headers.update({
            'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36'
        })
    
    def scrape_site(self, url: str, max_articles: int = 10) -> List[Article]:
        """Scrape articles from a news site."""
        try:
            response = self.session.get(url, timeout=10)
            response.raise_for_status()
            
            soup = BeautifulSoup(response.content, 'html.parser')
            articles = []
            
            # This is a generic example - you'd need to adapt for specific sites
            article_links = soup.find_all('a', href=True)[:max_articles]
            
            for link in article_links:
                href = link.get('href')
                if href and href.startswith('http'):
                    article_title = link.get_text(strip=True)
                    if article_title and len(article_title) > 10:
                        article = Article(
                            title=article_title,
                            url=href,
                            summary=article_title[:100] + "..."
                        )
                        articles.append(article)
                        
                        # Rate limiting
                        time.sleep(0.5)
            
            return articles
            
        except Exception as e:
            logger.error(f"Error scraping {url}: {e}")
            return []
    
    def scrape_and_save(self, urls: List[str], max_articles_per_site: int = 5):
        """Scrape multiple sites and save to database."""
        total_saved = 0
        
        for url in urls:
            logger.info(f"Scraping {url}")
            articles = self.scrape_site(url, max_articles_per_site)
            
            for article in articles:
                if self.db.save_article(article):
                    total_saved += 1
                    logger.info(f"Saved: {article.title}")
        
        logger.info(f"Total articles saved: {total_saved}")
        return total_saved

# Usage example
def main():
    # Initialize database
    db = NewsDatabase('news.db')
    scraper = NewsScraper(db)
    
    # URLs to scrape (example - replace with real news sites)
    urls = [
        'https://example-news-site.com',
        'https://another-news-site.com'
    ]
    
    # Scrape and save articles
    scraper.scrape_and_save(urls, max_articles_per_site=5)
    
    # Display recent articles
    recent_articles = db.get_articles(limit=5)
    print("\nRecent Articles:")
    for article in recent_articles:
        print(f"- {article.title}")
        print(f"  URL: {article.url}")
        print()

if __name__ == '__main__':
    main()
```