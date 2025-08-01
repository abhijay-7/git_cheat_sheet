# Python Beginner Cheatsheet

## Installation and Setup

### Installing Python
```bash
# Download from python.org (recommended)
# Or using package managers:

# Windows (using Chocolatey)
choco install python

# macOS (using Homebrew)
brew install python

# Ubuntu/Debian
sudo apt-get install python3 python3-pip

# CentOS/RHEL
sudo yum install python3 python3-pip
```

### Checking Installation
```bash
python --version        # or python3 --version
pip --version          # or pip3 --version
```

### Running Python
```bash
python                 # Interactive interpreter
python script.py       # Run a script file
python -c "print('Hello')"  # Run command directly
```

## Basic Syntax and Variables

### Hello World
```python
print("Hello, World!")
print('Hello, World!')  # Single quotes also work

# Comments
# This is a single-line comment

"""
This is a
multi-line comment
"""

'''
This is also a
multi-line comment
'''
```

### Variables and Assignment
```python
# Variables (no declaration needed)
name = "Alice"          # String
age = 25               # Integer
height = 5.6           # Float
is_student = True      # Boolean

# Multiple assignment
x, y, z = 1, 2, 3
a = b = c = 100

# Variable naming rules
valid_name = "OK"
_private = "OK"
name123 = "OK"
# 123name = "Invalid"  # Cannot start with number
# my-name = "Invalid"  # Cannot use hyphens
```

### Getting User Input
```python
name = input("Enter your name: ")
age = int(input("Enter your age: "))  # Convert to integer
height = float(input("Enter your height: "))  # Convert to float

print(f"Hello {name}, you are {age} years old")
```

## Data Types and Structures

### Basic Data Types
```python
# Numbers
integer = 42
floating = 3.14159
complex_num = 3 + 4j

# Strings
single_quote = 'Hello'
double_quote = "World"
multi_line = """This is a
multi-line string"""

# Boolean
is_true = True
is_false = False

# Check type
print(type(integer))    # <class 'int'>
print(type(floating))   # <class 'float'>
```

### String Operations
```python
text = "Python Programming"

# String methods
print(text.upper())         # PYTHON PROGRAMMING
print(text.lower())         # python programming
print(text.title())         # Python Programming
print(text.replace("Python", "Java"))  # Java Programming
print(text.split())         # ['Python', 'Programming']
print(len(text))           # 18

# String slicing
print(text[0])             # P (first character)
print(text[-1])            # g (last character)
print(text[0:6])           # Python (characters 0-5)
print(text[7:])            # Programming (from index 7 to end)
print(text[:6])            # Python (from start to index 5)

# String formatting
name = "Alice"
age = 25
print(f"My name is {name} and I am {age} years old")  # f-strings (Python 3.6+)
print("My name is {} and I am {} years old".format(name, age))  # .format()
print("My name is %s and I am %d years old" % (name, age))  # % formatting
```

### Lists (Ordered, Mutable)
```python
# Creating lists
fruits = ["apple", "banana", "orange"]
numbers = [1, 2, 3, 4, 5]
mixed = [1, "hello", 3.14, True]
empty_list = []

# Accessing elements
print(fruits[0])           # apple
print(fruits[-1])          # orange (last element)

# List methods
fruits.append("grape")     # Add to end
fruits.insert(1, "mango")  # Insert at specific position
fruits.remove("banana")    # Remove specific item
popped = fruits.pop()      # Remove and return last item
fruits.extend(["kiwi", "pear"])  # Add multiple items

# List operations
print(len(fruits))         # Length of list
print("apple" in fruits)   # Check if item exists
fruits.sort()             # Sort in place
reversed_fruits = fruits[::-1]  # Reverse (creates new list)

# List slicing
print(numbers[1:4])        # [2, 3, 4]
print(numbers[::2])        # [1, 3, 5] (every second element)
```

### Tuples (Ordered, Immutable)
```python
# Creating tuples
coordinates = (10, 20)
colors = ("red", "green", "blue")
single_item = (42,)        # Note the comma for single item
empty_tuple = ()

# Accessing elements (same as lists)
print(coordinates[0])      # 10
print(colors[-1])          # blue

# Tuple unpacking
x, y = coordinates
print(f"x={x}, y={y}")     # x=10, y=20

# Tuples are immutable
# coordinates[0] = 15      # This would cause an error
```

### Dictionaries (Key-Value Pairs)
```python
# Creating dictionaries
person = {
    "name": "Alice",
    "age": 25,
    "city": "New York"
}

# Alternative creation
person2 = dict(name="Bob", age=30, city="London")

# Accessing values
print(person["name"])      # Alice
print(person.get("age"))   # 25
print(person.get("country", "Unknown"))  # Unknown (default value)

# Dictionary methods
person["email"] = "alice@email.com"  # Add new key-value pair
person.update({"phone": "123-456-7890", "age": 26})  # Update multiple

print(person.keys())       # dict_keys(['name', 'age', 'city', 'email'])
print(person.values())     # dict_values(['Alice', 26, 'New York', 'alice@email.com'])
print(person.items())      # dict_items([('name', 'Alice'), ...])

# Remove items
del person["email"]        # Remove specific key
age = person.pop("age")    # Remove and return value
```

### Sets (Unordered, Unique Elements)
```python
# Creating sets
fruits = {"apple", "banana", "orange"}
numbers = set([1, 2, 3, 4, 5])
empty_set = set()  # Note: {} creates an empty dict, not set

# Set operations
fruits.add("grape")        # Add single item
fruits.update(["kiwi", "pear"])  # Add multiple items
fruits.remove("banana")    # Remove item (raises error if not found)
fruits.discard("mango")    # Remove item (no error if not found)

# Set mathematics
set1 = {1, 2, 3, 4}
set2 = {3, 4, 5, 6}
print(set1.union(set2))         # {1, 2, 3, 4, 5, 6}
print(set1.intersection(set2))  # {3, 4}
print(set1.difference(set2))    # {1, 2}
```

## Control Flow

### Conditional Statements
```python
# Basic if statement
age = 18
if age >= 18:
    print("You are an adult")

# if-else
temperature = 25
if temperature > 30:
    print("It's hot!")
else:
    print("It's not too hot")

# if-elif-else
score = 85
if score >= 90:
    grade = "A"
elif score >= 80:
    grade = "B"
elif score >= 70:
    grade = "C"
elif score >= 60:
    grade = "D"
else:
    grade = "F"
print(f"Your grade is: {grade}")

# Logical operators
age = 25
has_license = True
if age >= 18 and has_license:
    print("You can drive")

# Ternary operator (conditional expression)
message = "Adult" if age >= 18 else "Minor"
```

### Loops

#### For Loops
```python
# Loop through a list
fruits = ["apple", "banana", "orange"]
for fruit in fruits:
    print(fruit)

# Loop through a range
for i in range(5):          # 0, 1, 2, 3, 4
    print(i)

for i in range(1, 6):       # 1, 2, 3, 4, 5
    print(i)

for i in range(0, 10, 2):   # 0, 2, 4, 6, 8
    print(i)

# Loop with enumerate (get index and value)
for index, fruit in enumerate(fruits):
    print(f"{index}: {fruit}")

# Loop through dictionary
person = {"name": "Alice", "age": 25, "city": "New York"}
for key in person:
    print(f"{key}: {person[key]}")

for key, value in person.items():
    print(f"{key}: {value}")
```

#### While Loops
```python
# Basic while loop
count = 0
while count < 5:
    print(f"Count: {count}")
    count += 1

# While loop with condition
number = 1
while number <= 100:
    if number % 2 == 0:
        print(f"{number} is even")
    number += 1

# Infinite loop with break
while True:
    user_input = input("Enter 'quit' to exit: ")
    if user_input.lower() == 'quit':
        break
    print(f"You entered: {user_input}")
```

#### Loop Control
```python
# Break and continue
for i in range(10):
    if i == 3:
        continue  # Skip this iteration
    if i == 7:
        break     # Exit the loop
    print(i)

# Nested loops
for i in range(3):
    for j in range(3):
        print(f"i={i}, j={j}")

# Loop with else (executes if loop completes normally)
for i in range(5):
    print(i)
else:
    print("Loop completed successfully")
```

## Functions

### Defining Functions
```python
# Basic function
def greet():
    print("Hello, World!")

greet()  # Call the function

# Function with parameters
def greet_person(name):
    print(f"Hello, {name}!")

greet_person("Alice")

# Function with return value
def add_numbers(a, b):
    return a + b

result = add_numbers(5, 3)
print(result)  # 8

# Function with default parameters
def greet_with_title(name, title="Mr./Ms."):
    return f"Hello, {title} {name}!"

print(greet_with_title("Smith"))           # Hello, Mr./Ms. Smith!
print(greet_with_title("Smith", "Dr."))    # Hello, Dr. Smith!
```

### Advanced Function Features
```python
# Variable number of arguments
def sum_all(*args):
    return sum(args)

print(sum_all(1, 2, 3, 4, 5))  # 15

# Keyword arguments
def create_profile(**kwargs):
    for key, value in kwargs.items():
        print(f"{key}: {value}")

create_profile(name="Alice", age=25, city="New York")

# Function with both *args and **kwargs
def flexible_function(*args, **kwargs):
    print("Positional arguments:", args)
    print("Keyword arguments:", kwargs)

flexible_function(1, 2, 3, name="Alice", age=25)

# Lambda functions (anonymous functions)
square = lambda x: x ** 2
print(square(5))  # 25

# Lambda with multiple arguments
add = lambda x, y: x + y
print(add(3, 4))  # 7

# Using lambda with built-in functions
numbers = [1, 2, 3, 4, 5]
squared = list(map(lambda x: x ** 2, numbers))
print(squared)  # [1, 4, 9, 16, 25]

even_numbers = list(filter(lambda x: x % 2 == 0, numbers))
print(even_numbers)  # [2, 4]
```

## File Handling

### Reading Files
```python
# Method 1: Basic file reading
file = open("example.txt", "r")
content = file.read()
print(content)
file.close()

# Method 2: Using with statement (recommended)
with open("example.txt", "r") as file:
    content = file.read()
    print(content)
# File is automatically closed

# Reading line by line
with open("example.txt", "r") as file:
    for line in file:
        print(line.strip())  # strip() removes newline characters

# Reading all lines into a list
with open("example.txt", "r") as file:
    lines = file.readlines()
    print(lines)
```

### Writing Files
```python
# Writing to a file (overwrites existing content)
with open("output.txt", "w") as file:
    file.write("Hello, World!\n")
    file.write("This is a new line.\n")

# Appending to a file
with open("output.txt", "a") as file:
    file.write("This line is appended.\n")

# Writing multiple lines
lines = ["Line 1\n", "Line 2\n", "Line 3\n"]
with open("output.txt", "w") as file:
    file.writelines(lines)
```

### File Operations
```python
import os

# Check if file exists
if os.path.exists("example.txt"):
    print("File exists")
else:
    print("File does not exist")

# Get file information
import os.path
print(os.path.getsize("example.txt"))  # File size in bytes
print(os.path.getctime("example.txt"))  # Creation time
print(os.path.getmtime("example.txt"))  # Modification time

# Working with directories
os.makedirs("new_directory", exist_ok=True)  # Create directory
os.listdir(".")  # List files in current directory
os.chdir("new_directory")  # Change directory
os.getcwd()  # Get current working directory
```

## Error Handling

### Basic Exception Handling
```python
# Basic try-except
try:
    number = int(input("Enter a number: "))
    result = 10 / number
    print(f"Result: {result}")
except ValueError:
    print("Invalid input! Please enter a valid number.")
except ZeroDivisionError:
    print("Cannot divide by zero!")

# Catching multiple exceptions
try:
    # Some risky operation
    pass
except (ValueError, TypeError) as e:
    print(f"Error occurred: {e}")

# Catching all exceptions
try:
    # Some risky operation
    pass
except Exception as e:
    print(f"An error occurred: {e}")
```

### Advanced Exception Handling
```python
# try-except-else-finally
try:
    file = open("data.txt", "r")
    data = file.read()
except FileNotFoundError:
    print("File not found!")
else:
    print("File read successfully!")
    print(data)
finally:
    if 'file' in locals():
        file.close()
    print("Cleanup completed")

# Raising custom exceptions
def divide(a, b):
    if b == 0:
        raise ValueError("Cannot divide by zero")
    return a / b

try:
    result = divide(10, 0)
except ValueError as e:
    print(f"Error: {e}")
```

## Basic Object-Oriented Programming

### Classes and Objects
```python
# Defining a class
class Person:
    def __init__(self, name, age):
        self.name = name
        self.age = age
    
    def introduce(self):
        return f"Hi, I'm {self.name} and I'm {self.age} years old."
    
    def have_birthday(self):
        self.age += 1
        print(f"Happy birthday! {self.name} is now {self.age} years old.")

# Creating objects
person1 = Person("Alice", 25)
person2 = Person("Bob", 30)

# Using object methods
print(person1.introduce())
person1.have_birthday()

# Accessing attributes
print(person1.name)
print(person1.age)
```

### Class Variables and Methods
```python
class BankAccount:
    # Class variable
    bank_name = "Python Bank"
    interest_rate = 0.02
    
    def __init__(self, account_holder, initial_balance=0):
        self.account_holder = account_holder
        self.balance = initial_balance
    
    def deposit(self, amount):
        if amount > 0:
            self.balance += amount
            print(f"Deposited ${amount}. New balance: ${self.balance}")
        else:
            print("Deposit amount must be positive")
    
    def withdraw(self, amount):
        if amount > 0 and amount <= self.balance:
            self.balance -= amount
            print(f"Withdrew ${amount}. New balance: ${self.balance}")
        else:
            print("Invalid withdrawal amount")
    
    def get_balance(self):
        return self.balance
    
    @classmethod
    def get_bank_name(cls):
        return cls.bank_name
    
    @staticmethod
    def is_business_day(day):
        return day.lower() not in ['saturday', 'sunday']

# Using the class
account = BankAccount("Alice", 1000)
account.deposit(500)
account.withdraw(200)
print(f"Current balance: ${account.get_balance()}")
print(f"Bank name: {BankAccount.get_bank_name()}")
print(f"Is Monday a business day? {BankAccount.is_business_day('Monday')}")
```

## Modules and Packages

### Importing Modules
```python
# Import entire module
import math
print(math.pi)
print(math.sqrt(16))

# Import specific functions
from math import pi, sqrt
print(pi)
print(sqrt(16))

# Import with alias
import math as m
print(m.pi)

from math import pi as PI
print(PI)

# Import all (not recommended)
from math import *
```

### Built-in Modules
```python
# datetime module
from datetime import datetime, date, timedelta

now = datetime.now()
print(now)
print(now.strftime("%Y-%m-%d %H:%M:%S"))

today = date.today()
tomorrow = today + timedelta(days=1)
print(f"Today: {today}")
print(f"Tomorrow: {tomorrow}")

# random module
import random

print(random.randint(1, 10))        # Random integer between 1 and 10
print(random.random())              # Random float between 0 and 1
print(random.choice(['a', 'b', 'c']))  # Random choice from list

numbers = [1, 2, 3, 4, 5]
random.shuffle(numbers)             # Shuffle list in place
print(numbers)

# os module
import os

print(os.getcwd())                  # Current working directory
print(os.listdir('.'))              # List directory contents
print(os.path.exists('file.txt'))   # Check if file exists

# sys module
import sys

print(sys.version)                  # Python version
print(sys.path)                     # Python path
print(sys.argv)                     # Command line arguments
```

### Creating Your Own Modules
```python
# Create a file called my_utils.py
# my_utils.py
def greet(name):
    return f"Hello, {name}!"

def calculate_area(length, width):
    return length * width

PI = 3.14159

# Then in another file:
# import my_utils

# print(my_utils.greet("Alice"))
# print(my_utils.calculate_area(5, 3))
# print(my_utils.PI)
```

## Common Built-in Functions

### Type Conversion
```python
# String to number
str_num = "123"
int_num = int(str_num)          # 123
float_num = float(str_num)      # 123.0

# Number to string
num = 456
str_num = str(num)              # "456"

# List/Tuple conversion
my_list = [1, 2, 3]
my_tuple = tuple(my_list)       # (1, 2, 3)
back_to_list = list(my_tuple)   # [1, 2, 3]

# Boolean conversion
print(bool(1))          # True
print(bool(0))          # False
print(bool(""))         # False
print(bool("hello"))    # True
```

### Useful Built-in Functions
```python
# len() - length of sequence
print(len("hello"))         # 5
print(len([1, 2, 3, 4]))    # 4

# min() and max()
numbers = [5, 2, 8, 1, 9]
print(min(numbers))         # 1
print(max(numbers))         # 9

# sum()
print(sum(numbers))         # 25

# sorted() - returns new sorted list
print(sorted(numbers))      # [1, 2, 5, 8, 9]
print(sorted(numbers, reverse=True))  # [9, 8, 5, 2, 1]

# zip() - combine multiple iterables
names = ["Alice", "Bob", "Charlie"]
ages = [25, 30, 35]
for name, age in zip(names, ages):
    print(f"{name} is {age} years old")

# enumerate() - get index and value
fruits = ["apple", "banana", "orange"]
for index, fruit in enumerate(fruits):
    print(f"{index}: {fruit}")

# range()
print(list(range(5)))           # [0, 1, 2, 3, 4]
print(list(range(2, 8)))        # [2, 3, 4, 5, 6, 7]
print(list(range(0, 10, 2)))    # [0, 2, 4, 6, 8]
```

## Complete Example: Simple Calculator

```python
def calculator():
    print("Simple Calculator")
    print("Operations: +, -, *, /, quit")
    
    while True:
        try:
            operation = input("\nEnter operation (or 'quit' to exit): ").strip()
            
            if operation.lower() == 'quit':
                print("Thank you for using the calculator!")
                break
            
            if operation not in ['+', '-', '*', '/']:
                print("Invalid operation. Please use +, -, *, or /")
                continue
            
            num1 = float(input("Enter first number: "))
            num2 = float(input("Enter second number: "))
            
            if operation == '+':
                result = num1 + num2
            elif operation == '-':
                result = num1 - num2
            elif operation == '*':
                result = num1 * num2
            elif operation == '/':
                if num2 == 0:
                    print("Error: Cannot divide by zero!")
                    continue
                result = num1 / num2
            
            print(f"Result: {num1} {operation} {num2} = {result}")
            
        except ValueError:
            print("Error: Please enter valid numbers!")
        except Exception as e:
            print(f"An unexpected error occurred: {e}")

# Run the calculator
if __name__ == "__main__":
    calculator()
```

## Tips for Beginners

### Python Style Guidelines (PEP 8)
```python
# Variable and function names: lowercase with underscores
my_variable = 10
def my_function():
    pass

# Constants: uppercase with underscores
MAX_SIZE = 100
PI = 3.14159

# Class names: CamelCase
class MyClass:
    pass

# Indentation: 4 spaces (not tabs)
if True:
    print("Properly indented")

# Line length: max 79 characters
# Use parentheses for line continuation
total = (first_variable + second_variable + 
         third_variable + fourth_variable)
```

### Common Mistakes to Avoid
```python
# 1. Forgetting to convert input to correct type
# age = input("Enter your age: ")  # This is a string!
age = int(input("Enter your age: "))  # Convert to integer

# 2. Using mutable default arguments
def bad_function(my_list=[]):  # Don't do this!
    my_list.append(1)
    return my_list

def good_function(my_list=None):  # Do this instead
    if my_list is None:
        my_list = []
    my_list.append(1)
    return my_list

# 3. Not closing files (use 'with' statement)
# Bad:
# file = open("data.txt", "r")
# data = file.read()
# file.close()  # Easy to forget!

# Good:
with open("data.txt", "r") as file:
    data = file.read()
# File automatically closed

# 4. Comparing with is instead of ==
# Bad:
# if name is "Alice":  # Don't do this!

# Good:
if name == "Alice":  # Use == for value comparison
```

### Debugging Tips
```python
# Use print() for debugging
def calculate_total(prices):
    total = 0
    for price in prices:
        print(f"Adding {price} to {total}")  # Debug output
        total += price
    print(f"Final total: {total}")  # Debug output
    return total

# Use type() to check variable types
variable = "123"
print(f"Type of variable: {type(variable)}")

# Use help() to get information about functions/objects
help(len)
help(str.split)
```