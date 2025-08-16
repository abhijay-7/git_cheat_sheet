# JavaScript Comprehensive Cheatsheet

## Table of Contents
1. [Basic Syntax](#basic-syntax)
2. [Variables & Data Types](#variables--data-types)
3. [Functions](#functions)
4. [Objects & Arrays](#objects--arrays)
5. [Control Flow](#control-flow)
6. [ES6+ Features](#es6-features)
7. [Asynchronous JavaScript](#asynchronous-javascript)
8. [DOM Manipulation](#dom-manipulation)
9. [Error Handling](#error-handling)
10. [Advanced Concepts](#advanced-concepts)
11. [Modern JavaScript Patterns](#modern-javascript-patterns)
12. [Performance & Best Practices](#performance--best-practices)

## Basic Syntax

### Hello World
```javascript
console.log("Hello, World!");
alert("Hello, World!");        // Browser only
document.write("Hello, World!"); // Browser only
```

### Comments
```javascript
// Single line comment
/* Multi-line comment */
/**
 * JSDoc comment
 * @param {string} name - The name parameter
 * @returns {string} The greeting
 */
```

### Semicolons & Automatic Semicolon Insertion
```javascript
// Optional but recommended
let x = 5;
let y = 10;

// ASI can cause issues
let a = 1
let b = 2
[a, b] = [b, a]  // Error! ASI doesn't insert semicolon here

// Better:
let a = 1;
let b = 2;
[a, b] = [b, a];
```

## Variables & Data Types

### Variable Declarations
```javascript
// var - function scoped, can be redeclared
var x = 1;
var x = 2; // OK

// let - block scoped, cannot be redeclared
let y = 1;
// let y = 2; // Error!

// const - block scoped, cannot be reassigned
const z = 1;
// z = 2; // Error!

// But objects/arrays can be mutated
const obj = { a: 1 };
obj.a = 2; // OK
obj.b = 3; // OK
```

### Primitive Data Types
```javascript
// Number
let num = 42;
let float = 3.14;
let scientific = 1e5; // 100000
let binary = 0b1010;  // 10
let octal = 0o744;    // 484
let hex = 0xFF;       // 255

// Special numbers
let infinity = Infinity;
let negInfinity = -Infinity;
let notANumber = NaN;

// String
let str1 = "Hello";
let str2 = 'World';
let template = `Hello ${str2}!`; // Template literal

// Boolean
let isTrue = true;
let isFalse = false;

// Null and Undefined
let empty = null;
let notDefined = undefined;
let declared; // undefined by default

// Symbol (ES6)
let sym1 = Symbol();
let sym2 = Symbol('description');

// BigInt (ES2020)
let bigInt = 123n;
let bigInt2 = BigInt(123);
```

### Type Checking
```javascript
typeof 42;              // "number"
typeof "hello";         // "string"
typeof true;            // "boolean"
typeof undefined;       // "undefined"
typeof null;            // "object" (known bug)
typeof {};              // "object"
typeof [];              // "object"
typeof function(){};    // "function"
typeof Symbol();        // "symbol"
typeof 123n;           // "bigint"

// Better type checking
Array.isArray([]);      // true
Number.isNaN(NaN);      // true
Number.isFinite(42);    // true
```

### Type Coercion
```javascript
// Implicit coercion
"5" + 3;        // "53" (string concatenation)
"5" - 3;        // 2 (numeric subtraction)
"5" * 3;        // 15
"5" / 3;        // 1.666...
5 == "5";       // true (loose equality)
5 === "5";      // false (strict equality)

// Truthy/Falsy values
// Falsy: false, 0, -0, 0n, "", null, undefined, NaN
// Everything else is truthy

if ("") console.log("Won't print");
if ("hello") console.log("Will print");
if (0) console.log("Won't print");
if (42) console.log("Will print");
```

## Functions

### Function Declarations & Expressions
```javascript
// Function declaration (hoisted)
function greet(name) {
    return `Hello, ${name}!`;
}

// Function expression (not hoisted)
const greet2 = function(name) {
    return `Hello, ${name}!`;
};

// Named function expression
const greet3 = function greeting(name) {
    return `Hello, ${name}!`;
};

// Arrow functions (ES6)
const greet4 = (name) => {
    return `Hello, ${name}!`;
};

// Shorter arrow function
const greet5 = name => `Hello, ${name}!`;
const add = (a, b) => a + b;
const square = x => x * x;
```

### Parameters & Arguments
```javascript
// Default parameters (ES6)
function greet(name = "World", punctuation = "!") {
    return `Hello, ${name}${punctuation}`;
}

// Rest parameters (ES6)
function sum(...numbers) {
    return numbers.reduce((acc, num) => acc + num, 0);
}

// Destructuring parameters
function createUser({name, age, email = "not provided"}) {
    return { name, age, email };
}

// Arguments object (legacy, avoid in modern JS)
function oldStyleVarArgs() {
    console.log(arguments); // Array-like object
}
```

### Higher-Order Functions
```javascript
// Function that returns a function
function createMultiplier(factor) {
    return function(number) {
        return number * factor;
    };
}

const double = createMultiplier(2);
console.log(double(5)); // 10

// Function that takes a function as argument
function executeFunction(fn, value) {
    return fn(value);
}

executeFunction(square, 4); // 16
```

### IIFE (Immediately Invoked Function Expression)
```javascript
// Classic IIFE
(function() {
    console.log("IIFE executed!");
})();

// Arrow IIFE
(() => {
    console.log("Arrow IIFE executed!");
})();

// With parameters
((name) => {
    console.log(`Hello, ${name}!`);
})("World");
```

## Objects & Arrays

### Object Literals
```javascript
// Basic object
const person = {
    name: "John",
    age: 30,
    isEmployed: true
};

// Property access
console.log(person.name);        // Dot notation
console.log(person['age']);      // Bracket notation

// Dynamic property names
const propName = "email";
person[propName] = "john@example.com";

// Computed property names (ES6)
const dynamicKey = "hobby";
const person2 = {
    name: "Jane",
    [dynamicKey]: "Reading",
    [`${dynamicKey}Count`]: 5
};
```

### Object Methods
```javascript
const calculator = {
    result: 0,
    
    add(value) {
        this.result += value;
        return this; // Method chaining
    },
    
    subtract(value) {
        this.result -= value;
        return this;
    },
    
    getValue() {
        return this.result;
    }
};

calculator.add(5).subtract(2).getValue(); // 3
```

### Object Destructuring (ES6)
```javascript
const person = { name: "John", age: 30, city: "New York" };

// Basic destructuring
const { name, age } = person;

// Renaming variables
const { name: fullName, age: years } = person;

// Default values
const { name, age, country = "USA" } = person;

// Nested destructuring
const user = {
    info: { name: "John", age: 30 },
    settings: { theme: "dark" }
};
const { info: { name, age }, settings: { theme } } = user;
```

### Arrays
```javascript
// Array creation
const arr1 = [1, 2, 3, 4, 5];
const arr2 = new Array(5); // Creates array with 5 empty slots
const arr3 = Array.from({length: 5}, (_, i) => i); // [0,1,2,3,4]

// Array methods
const numbers = [1, 2, 3, 4, 5];

// Mutating methods
numbers.push(6);           // Add to end
numbers.pop();             // Remove from end
numbers.unshift(0);        // Add to beginning
numbers.shift();           // Remove from beginning
numbers.splice(2, 1, 10);  // Remove 1 element at index 2, insert 10

// Non-mutating methods
const doubled = numbers.map(x => x * 2);
const evens = numbers.filter(x => x % 2 === 0);
const sum = numbers.reduce((acc, curr) => acc + curr, 0);
const found = numbers.find(x => x > 3);
const index = numbers.indexOf(3);
const includes = numbers.includes(3);

// Array destructuring
const [first, second, ...rest] = numbers;
const [a, , c] = numbers; // Skip second element
```

### Spread Operator & Rest (ES6)
```javascript
// Spread in arrays
const arr1 = [1, 2, 3];
const arr2 = [4, 5, 6];
const combined = [...arr1, ...arr2]; // [1,2,3,4,5,6]
const copy = [...arr1];

// Spread in objects
const obj1 = { a: 1, b: 2 };
const obj2 = { c: 3, d: 4 };
const combined = { ...obj1, ...obj2 }; // {a:1, b:2, c:3, d:4}
const copy = { ...obj1 };

// Spread in function calls
const numbers = [1, 2, 3];
Math.max(...numbers); // Same as Math.max(1, 2, 3)

// Rest in destructuring
const [first, ...rest] = [1, 2, 3, 4]; // first: 1, rest: [2,3,4]
const {a, ...others} = {a: 1, b: 2, c: 3}; // a: 1, others: {b:2, c:3}
```

## Control Flow

### Conditional Statements
```javascript
// If statements
if (condition) {
    // code
} else if (anotherCondition) {
    // code
} else {
    // code
}

// Ternary operator
const result = condition ? valueIfTrue : valueIfFalse;

// Switch statement
switch (value) {
    case 'a':
        console.log('A');
        break;
    case 'b':
    case 'c':
        console.log('B or C');
        break;
    default:
        console.log('Other');
}

// Nullish coalescing (ES2020)
const value = null ?? 'default'; // 'default'
const value2 = undefined ?? 'default'; // 'default'
const value3 = 0 ?? 'default'; // 0 (not null/undefined)

// Optional chaining (ES2020)
const user = { profile: { name: 'John' } };
console.log(user.profile?.name); // 'John'
console.log(user.settings?.theme); // undefined (no error)
```

### Loops
```javascript
// For loop
for (let i = 0; i < 5; i++) {
    console.log(i);
}

// For...in (object properties)
const obj = { a: 1, b: 2, c: 3 };
for (const key in obj) {
    console.log(key, obj[key]);
}

// For...of (iterable values)
const arr = [1, 2, 3, 4, 5];
for (const value of arr) {
    console.log(value);
}

// While loop
let i = 0;
while (i < 5) {
    console.log(i);
    i++;
}

// Do...while loop
let j = 0;
do {
    console.log(j);
    j++;
} while (j < 5);

// forEach (array method)
arr.forEach((value, index) => {
    console.log(index, value);
});
```

## ES6+ Features

### Classes (ES6)
```javascript
class Animal {
    constructor(name, species) {
        this.name = name;
        this.species = species;
    }
    
    // Method
    speak() {
        console.log(`${this.name} makes a sound`);
    }
    
    // Getter
    get info() {
        return `${this.name} is a ${this.species}`;
    }
    
    // Setter
    set nickname(value) {
        this._nickname = value;
    }
    
    // Static method
    static compare(animal1, animal2) {
        return animal1.species === animal2.species;
    }
}

// Inheritance
class Dog extends Animal {
    constructor(name, breed) {
        super(name, 'dog'); // Call parent constructor
        this.breed = breed;
    }
    
    speak() {
        console.log(`${this.name} barks`);
    }
    
    // Private fields (ES2022)
    #secretTrait = 'loyal';
    
    getSecret() {
        return this.#secretTrait;
    }
}

const dog = new Dog('Rex', 'German Shepherd');
dog.speak(); // 'Rex barks'
```

### Template Literals (ES6)
```javascript
const name = 'John';
const age = 30;

// Basic template literal
const greeting = `Hello, ${name}!`;

// Multi-line strings
const multiLine = `
    Line 1
    Line 2
    Line 3
`;

// Expression evaluation
const calculation = `2 + 3 = ${2 + 3}`;

// Tagged template literals
function highlight(strings, ...values) {
    return strings.reduce((result, string, i) => {
        const value = values[i] ? `<mark>${values[i]}</mark>` : '';
        return result + string + value;
    }, '');
}

const highlighted = highlight`Hello ${name}, you are ${age} years old!`;
```

### Modules (ES6)
```javascript
// export.js
export const PI = 3.14159;
export function add(a, b) {
    return a + b;
}

export default class Calculator {
    multiply(a, b) {
        return a * b;
    }
}

// import.js
import Calculator, { PI, add } from './export.js';
import * as mathUtils from './export.js';

// Dynamic imports (ES2020)
const module = await import('./export.js');
// or
import('./export.js').then(module => {
    // Use module
});
```

### Symbols (ES6)
```javascript
// Unique identifiers
const sym1 = Symbol('description');
const sym2 = Symbol('description');
console.log(sym1 === sym2); // false

// Well-known symbols
const obj = {
    [Symbol.iterator]: function* () {
        yield 1;
        yield 2;
        yield 3;
    }
};

for (const value of obj) {
    console.log(value); // 1, 2, 3
}
```

### Iterators & Generators (ES6)
```javascript
// Generator function
function* numberGenerator() {
    let i = 0;
    while (true) {
        yield i++;
    }
}

const gen = numberGenerator();
console.log(gen.next().value); // 0
console.log(gen.next().value); // 1
console.log(gen.next().value); // 2

// Finite generator
function* fibonacci() {
    let [a, b] = [0, 1];
    while (true) {
        yield a;
        [a, b] = [b, a + b];
    }
}

// Custom iterator
const range = {
    from: 1,
    to: 5,
    
    [Symbol.iterator]() {
        return {
            current: this.from,
            last: this.to,
            
            next() {
                if (this.current <= this.last) {
                    return { done: false, value: this.current++ };
                } else {
                    return { done: true };
                }
            }
        };
    }
};

for (const num of range) {
    console.log(num); // 1, 2, 3, 4, 5
}
```

## Asynchronous JavaScript

### Callbacks
```javascript
// Basic callback
function fetchData(callback) {
    setTimeout(() => {
        callback('Data received');
    }, 1000);
}

fetchData(data => console.log(data));

// Callback hell
getData(function(a) {
    getMoreData(a, function(b) {
        getMoreData(b, function(c) {
            // Nested callbacks...
        });
    });
});
```

### Promises (ES6)
```javascript
// Creating promises
const promise = new Promise((resolve, reject) => {
    const success = Math.random() > 0.5;
    setTimeout(() => {
        if (success) {
            resolve('Success!');
        } else {
            reject(new Error('Failed!'));
        }
    }, 1000);
});

// Using promises
promise
    .then(result => console.log(result))
    .catch(error => console.error(error))
    .finally(() => console.log('Done'));

// Promise methods
Promise.all([promise1, promise2, promise3])
    .then(results => console.log(results));

Promise.allSettled([promise1, promise2, promise3])
    .then(results => console.log(results));

Promise.race([promise1, promise2, promise3])
    .then(result => console.log(result));

// Promise utility methods
Promise.resolve('value'); // Immediately resolved
Promise.reject(new Error('error')); // Immediately rejected
```

### Async/Await (ES2017)
```javascript
// Async function
async function fetchUserData() {
    try {
        const response = await fetch('/api/user');
        const userData = await response.json();
        return userData;
    } catch (error) {
        console.error('Error:', error);
        throw error;
    }
}

// Using async function
fetchUserData()
    .then(data => console.log(data))
    .catch(error => console.error(error));

// Async IIFE
(async () => {
    try {
        const data = await fetchUserData();
        console.log(data);
    } catch (error) {
        console.error(error);
    }
})();

// Parallel vs Sequential
async function sequential() {
    const result1 = await fetch('/api/1');
    const result2 = await fetch('/api/2'); // Waits for result1
    return [result1, result2];
}

async function parallel() {
    const [result1, result2] = await Promise.all([
        fetch('/api/1'),
        fetch('/api/2') // Executes concurrently
    ]);
    return [result1, result2];
}
```

### Fetch API
```javascript
// Basic GET request
fetch('/api/data')
    .then(response => response.json())
    .then(data => console.log(data));

// POST request
fetch('/api/users', {
    method: 'POST',
    headers: {
        'Content-Type': 'application/json'
    },
    body: JSON.stringify({ name: 'John', age: 30 })
})
.then(response => response.json())
.then(data => console.log(data));

// With error handling
async function apiCall() {
    try {
        const response = await fetch('/api/data');
        
        if (!response.ok) {
            throw new Error(`HTTP error! status: ${response.status}`);
        }
        
        const data = await response.json();
        return data;
    } catch (error) {
        console.error('Fetch error:', error);
        throw error;
    }
}
```

## DOM Manipulation

### Selecting Elements
```javascript
// By ID
const element = document.getElementById('myId');

// By class name
const elements = document.getElementsByClassName('myClass');

// By tag name
const paragraphs = document.getElementsByTagName('p');

// Query selectors (preferred)
const element = document.querySelector('.myClass');
const elements = document.querySelectorAll('.myClass');

// Complex selectors
const element = document.querySelector('div.container > p:first-child');
```

### Modifying Elements
```javascript
const element = document.querySelector('#myElement');

// Content
element.textContent = 'New text';
element.innerHTML = '<strong>Bold text</strong>';

// Attributes
element.setAttribute('class', 'newClass');
element.getAttribute('data-value');
element.removeAttribute('data-value');

// Properties
element.id = 'newId';
element.className = 'newClass';
element.style.color = 'red';
element.style.backgroundColor = 'blue';

// CSS classes
element.classList.add('newClass');
element.classList.remove('oldClass');
element.classList.toggle('active');
element.classList.contains('myClass');

// Data attributes
element.dataset.userId = '123';
console.log(element.dataset.userId); // '123'
```

### Creating and Removing Elements
```javascript
// Create elements
const div = document.createElement('div');
div.textContent = 'Hello World';
div.className = 'greeting';

// Append to DOM
document.body.appendChild(div);
document.body.insertBefore(div, existingElement);

// Modern insertion methods
element.append(div); // Append as last child
element.prepend(div); // Prepend as first child
element.before(div); // Insert before element
element.after(div); // Insert after element

// Remove elements
element.remove(); // Modern way
element.parentNode.removeChild(element); // Legacy way
```

### Event Handling
```javascript
const button = document.querySelector('#myButton');

// Add event listener
button.addEventListener('click', function(event) {
    console.log('Button clicked!', event);
});

// Arrow function event handler
button.addEventListener('click', (e) => {
    e.preventDefault(); // Prevent default behavior
    e.stopPropagation(); // Stop event bubbling
    console.log('Clicked!');
});

// Remove event listener
function clickHandler(e) {
    console.log('Clicked!');
}
button.addEventListener('click', clickHandler);
button.removeEventListener('click', clickHandler);

// Event delegation
document.addEventListener('click', (e) => {
    if (e.target.matches('.button')) {
        console.log('Dynamic button clicked!');
    }
});

// Common events
element.addEventListener('mouseenter', handler);
element.addEventListener('mouseleave', handler);
element.addEventListener('keydown', handler);
element.addEventListener('input', handler);
element.addEventListener('change', handler);
element.addEventListener('submit', handler);
```

## Error Handling

### Try-Catch-Finally
```javascript
try {
    // Code that might throw an error
    const result = riskyOperation();
    console.log(result);
} catch (error) {
    // Handle the error
    console.error('An error occurred:', error.message);
} finally {
    // Always executes
    console.log('Cleanup code');
}

// Specific error types
try {
    JSON.parse('invalid json');
} catch (error) {
    if (error instanceof SyntaxError) {
        console.log('JSON syntax error');
    } else {
        console.log('Other error');
    }
}
```

### Custom Errors
```javascript
class CustomError extends Error {
    constructor(message, code) {
        super(message);
        this.name = 'CustomError';
        this.code = code;
    }
}

function validateUser(user) {
    if (!user.email) {
        throw new CustomError('Email is required', 'MISSING_EMAIL');
    }
}

try {
    validateUser({});
} catch (error) {
    if (error instanceof CustomError) {
        console.log('Custom error:', error.message, error.code);
    }
}
```

### Error Handling with Promises
```javascript
// Promise error handling
promise
    .then(result => {
        // Success handler
        return result;
    })
    .catch(error => {
        // Error handler
        console.error('Promise rejected:', error);
    });

// Async/await error handling
async function handleErrors() {
    try {
        const result = await someAsyncOperation();
        return result;
    } catch (error) {
        console.error('Async error:', error);
        throw error; // Re-throw if needed
    }
}
```

## Advanced Concepts

### Closures
```javascript
function outerFunction(x) {
    // Outer function's variable
    const outerVariable = x;
    
    function innerFunction(y) {
        // Inner function has access to outer variables
        console.log(outerVariable + y);
    }
    
    return innerFunction;
}

const closure = outerFunction(10);
closure(5); // 15

// Practical example: Counter
function createCounter() {
    let count = 0;
    
    return {
        increment: () => ++count,
        decrement: () => --count,
        getValue: () => count
    };
}

const counter = createCounter();
console.log(counter.increment()); // 1
console.log(counter.getValue()); // 1
```

### Prototypes & Prototypal Inheritance
```javascript
// Constructor function
function Person(name, age) {
    this.name = name;
    this.age = age;
}

Person.prototype.greet = function() {
    return `Hello, I'm ${this.name}`;
};

Person.prototype.getAge = function() {
    return this.age;
};

const person1 = new Person('John', 30);
console.log(person1.greet()); // 'Hello, I'm John'

// Inheritance
function Employee(name, age, jobTitle) {
    Person.call(this, name, age); // Call parent constructor
    this.jobTitle = jobTitle;
}

// Set up prototype chain
Employee.prototype = Object.create(Person.prototype);
Employee.prototype.constructor = Employee;

Employee.prototype.work = function() {
    return `${this.name} is working as a ${this.jobTitle}`;
};

// Object.create
const personPrototype = {
    greet() {
        return `Hello, I'm ${this.name}`;
    }
};

const person = Object.create(personPrototype);
person.name = 'John';
```

### This Context
```javascript
const obj = {
    name: 'John',
    greet: function() {
        console.log(`Hello, I'm ${this.name}`);
    },
    
    // Arrow function doesn't bind this
    arrowGreet: () => {
        console.log(`Hello, I'm ${this.name}`); // this is undefined or global
    }
};

// Method call
obj.greet(); // 'Hello, I'm John'

// Function call (this is undefined or global)
const greetFn = obj.greet;
greetFn(); // 'Hello, I'm undefined'

// Explicit binding
greetFn.call(obj); // 'Hello, I'm John'
greetFn.apply(obj); // 'Hello, I'm John'
const boundGreet = greetFn.bind(obj);
boundGreet(); // 'Hello, I'm John'

// Event handler context
button.addEventListener('click', function() {
    console.log(this); // Button element
});

button.addEventListener('click', () => {
    console.log(this); // Lexical this (often window/undefined)
});
```

### Hoisting
```javascript
// Variable hoisting
console.log(x); // undefined (not error)
var x = 5;

// Function hoisting
sayHello(); // Works!
function sayHello() {
    console.log('Hello!');
}

// let/const are hoisted but in temporal dead zone
console.log(y); // ReferenceError
let y = 10;

console.log(z); // ReferenceError
const z = 20;
```

### Call Stack & Event Loop
```javascript
// Synchronous execution
console.log('1');
console.log('2');
console.log('3');

// Asynchronous execution
console.log('1');
setTimeout(() => console.log('2'), 0);
console.log('3');
// Output: 1, 3, 2

// Promises vs setTimeout
console.log('1');
setTimeout(() => console.log('setTimeout'), 0);
Promise.resolve().then(() => console.log('Promise'));
console.log('3');
// Output: 1, 3, Promise, setTimeout
```

## Modern JavaScript Patterns

### Module Pattern
```javascript
// Revealing Module Pattern
const myModule = (function() {
    let privateVariable = 'secret';
    
    function privateMethod() {
        console.log('Private method called');
    }
    
    return {
        publicMethod: function() {
            privateMethod();
            return privateVariable;
        },
        
        setPrivateVariable: function(value) {
            privateVariable = value;
        }
    };
})();
```

### Observer Pattern
```javascript
class EventEmitter {
    constructor() {
        this.events = {};
    }
    
    on(event, callback) {
        if (!this.events[event]) {
            this.events[event] = [];
        }
        this.events[event].push(callback);
    }
    
    emit(event, data) {
        if (this.events[event]) {
            this.events[event].forEach(callback => callback(data));
        }
    }
    
    off(event, callback) {
        if (this.events[event]) {
            this.events[event] = this.events[event].filter(cb => cb !== callback);
        }
    }
}

const emitter = new EventEmitter();
emitter.on('userLogin', user => console.log(`${user} logged in`));
emitter.emit('userLogin', 'John');
```

### Factory Pattern
```javascript
function createUser(name, type) {
    const user = {
        name,
        type,
        say() {
            console.log(`Hello, I'm ${this.name}`);
        }
    };
    
    if (type === 'admin') {
        user.deleteUser = function() {
            console.log('User deleted');
        };
    }
    
    return user;
}

const admin = createUser('John', 'admin');
const regular = createUser('Jane', 'regular');
```

### Decorator Pattern
```javascript
// Method decorator
function timing(target, key, descriptor) {
    const original = descriptor.value;
    
    descriptor.value = function(...args) {
        const start = Date.now();
        const result = original.apply(this, args);
        const end = Date.now();
        console.log(`${key} took ${end - start}ms`);
        return result;
    };
    
    return descriptor;
}

// Function wrapping decorator
function memoize(fn) {
    const cache = new Map();
    return function(...args) {
        const key = JSON.stringify(args);
        if (cache.has(key)) {
            return cache.get(key);
        }
        const result = fn.apply(this, args);
        cache.set(key, result);
        return result;
    };
}

const expensiveFunction = memoize((n) => {
    console.log('Computing...');
    return n * n;
});
```

### Proxy & Reflection (ES6)
```javascript
// Basic proxy
const target = {
    name: 'John',
    age: 30
};

const proxy = new Proxy(target, {
    get(target, property) {
        console.log(`Getting property: ${property}`);
        return target[property];
    },
    
    set(target, property, value) {
        console.log(`Setting ${property} to ${value}`);
        target[property] = value;
        return true;
    },
    
    has(target, property) {
        console.log(`Checking if ${property} exists`);
        return property in target;
    }
});

console.log(proxy.name); // "Getting property: name" then "John"
proxy.age = 31; // "Setting age to 31"

// Validation proxy
function createValidatedObject(target, validators) {
    return new Proxy(target, {
        set(target, property, value) {
            if (validators[property]) {
                const isValid = validators[property](value);
                if (!isValid) {
                    throw new Error(`Invalid value for ${property}: ${value}`);
                }
            }
            target[property] = value;
            return true;
        }
    });
}

const user = createValidatedObject({}, {
    age: value => typeof value === 'number' && value >= 0,
    email: value => typeof value === 'string' && value.includes('@')
});

// Array with negative indexing
function createArray(...elements) {
    const array = [...elements];
    return new Proxy(array, {
        get(target, property) {
            if (typeof property === 'string' && /^-\d+$/.test(property)) {
                const index = target.length + parseInt(property);
                return target[index];
            }
            return target[property];
        }
    });
}

const arr = createArray(1, 2, 3, 4, 5);
console.log(arr[-1]); // 5
console.log(arr[-2]); // 4
```

### WeakMap & WeakSet (ES6)
```javascript
// WeakMap - keys must be objects, weak references
const weakMap = new WeakMap();
let obj = { name: 'John' };

weakMap.set(obj, 'some data');
console.log(weakMap.get(obj)); // 'some data'

obj = null; // Object can be garbage collected

// Private properties with WeakMap
const privateProps = new WeakMap();

class User {
    constructor(name, age) {
        this.name = name;
        privateProps.set(this, { age });
    }
    
    getAge() {
        return privateProps.get(this).age;
    }
    
    setAge(age) {
        privateProps.get(this).age = age;
    }
}

// WeakSet - only objects, weak references
const weakSet = new WeakSet();
const obj1 = {};
const obj2 = {};

weakSet.add(obj1);
weakSet.add(obj2);

console.log(weakSet.has(obj1)); // true
```

## Performance & Best Practices

### Debouncing & Throttling
```javascript
// Debounce - delay execution until after calls have stopped
function debounce(func, delay) {
    let timeoutId;
    return function(...args) {
        clearTimeout(timeoutId);
        timeoutId = setTimeout(() => func.apply(this, args), delay);
    };
}

// Throttle - execute at most once per interval
function throttle(func, interval) {
    let lastCall = 0;
    return function(...args) {
        const now = Date.now();
        if (now - lastCall >= interval) {
            lastCall = now;
            func.apply(this, args);
        }
    };
}

// Usage
const debouncedSearch = debounce(searchFunction, 300);
const throttledScroll = throttle(handleScroll, 100);

document.getElementById('search').addEventListener('input', debouncedSearch);
window.addEventListener('scroll', throttledScroll);
```

### Lazy Loading & Code Splitting
```javascript
// Dynamic imports for code splitting
async function loadModule() {
    try {
        const module = await import('./heavy-module.js');
        module.init();
    } catch (error) {
        console.error('Failed to load module:', error);
    }
}

// Conditional loading
if (userNeedsFeature) {
    import('./feature-module.js').then(module => {
        module.initFeature();
    });
}

// Intersection Observer for lazy loading
const imageObserver = new IntersectionObserver((entries) => {
    entries.forEach(entry => {
        if (entry.isIntersecting) {
            const img = entry.target;
            img.src = img.dataset.src;
            img.classList.remove('lazy');
            imageObserver.unobserve(img);
        }
    });
});

document.querySelectorAll('img[data-src]').forEach(img => {
    imageObserver.observe(img);
});
```

### Memory Management
```javascript
// Avoiding memory leaks
class ComponentManager {
    constructor() {
        this.components = new Map();
        this.eventListeners = new WeakMap();
    }
    
    addComponent(id, component) {
        this.components.set(id, component);
        
        // Store event listeners for cleanup
        const listeners = [];
        this.eventListeners.set(component, listeners);
        
        const clickHandler = () => console.log('clicked');
        component.addEventListener('click', clickHandler);
        listeners.push({ event: 'click', handler: clickHandler });
    }
    
    removeComponent(id) {
        const component = this.components.get(id);
        if (component) {
            // Clean up event listeners
            const listeners = this.eventListeners.get(component) || [];
            listeners.forEach(({ event, handler }) => {
                component.removeEventListener(event, handler);
            });
            
            this.components.delete(id);
        }
    }
}

// Object pooling for performance
class ObjectPool {
    constructor(createFn, resetFn) {
        this.createFn = createFn;
        this.resetFn = resetFn;
        this.pool = [];
    }
    
    acquire() {
        return this.pool.length > 0 
            ? this.pool.pop() 
            : this.createFn();
    }
    
    release(obj) {
        this.resetFn(obj);
        this.pool.push(obj);
    }
}

const particlePool = new ObjectPool(
    () => ({ x: 0, y: 0, vx: 0, vy: 0 }),
    (particle) => {
        particle.x = 0;
        particle.y = 0;
        particle.vx = 0;
        particle.vy = 0;
    }
);
```

### Performance Optimization Techniques
```javascript
// RequestAnimationFrame for smooth animations
function animate() {
    // Update animation state
    updatePositions();
    render();
    requestAnimationFrame(animate);
}
requestAnimationFrame(animate);

// Batch DOM operations
function efficientDOMUpdate() {
    const fragment = document.createDocumentFragment();
    
    for (let i = 0; i < 1000; i++) {
        const div = document.createElement('div');
        div.textContent = `Item ${i}`;
        fragment.appendChild(div);
    }
    
    document.body.appendChild(fragment); // Single DOM operation
}

// Virtual scrolling for large lists
class VirtualList {
    constructor(container, items, itemHeight) {
        this.container = container;
        this.items = items;
        this.itemHeight = itemHeight;
        this.visibleStart = 0;
        this.visibleEnd = 0;
        this.init();
    }
    
    init() {
        this.container.style.height = `${this.items.length * this.itemHeight}px`;
        this.container.addEventListener('scroll', this.onScroll.bind(this));
        this.render();
    }
    
    onScroll() {
        const scrollTop = this.container.scrollTop;
        const containerHeight = this.container.clientHeight;
        
        this.visibleStart = Math.floor(scrollTop / this.itemHeight);
        this.visibleEnd = Math.min(
            this.visibleStart + Math.ceil(containerHeight / this.itemHeight),
            this.items.length
        );
        
        this.render();
    }
    
    render() {
        const visibleItems = this.items.slice(this.visibleStart, this.visibleEnd);
        const offsetY = this.visibleStart * this.itemHeight;
        
        this.container.innerHTML = `
            <div style="transform: translateY(${offsetY}px)">
                ${visibleItems.map(item => `<div style="height: ${this.itemHeight}px">${item}</div>`).join('')}
            </div>
        `;
    }
}
```

### Modern JavaScript Features (ES2020+)

```javascript
// Optional chaining (?.) - ES2020
const user = {
    profile: {
        name: 'John',
        address: {
            street: '123 Main St'
        }
    }
};

console.log(user?.profile?.name); // 'John'
console.log(user?.profile?.phone?.number); // undefined
console.log(user?.nonExistent?.method?.()); // undefined

// Nullish coalescing (??) - ES2020
const config = {
    timeout: 0,
    retries: null,
    debug: false
};

const timeout = config.timeout ?? 5000; // 0 (falsy but not nullish)
const retries = config.retries ?? 3; // 3 (null is nullish)
const debug = config.debug ?? true; // false (falsy but not nullish)

// BigInt - ES2020
const hugNumber = 1234567890123456789012345678901234567890n;
const alsoHuge = BigInt('1234567890123456789012345678901234567890');

console.log(hugNumber + 1n); // Must use 'n' suffix or BigInt()
// console.log(hugNumber + 1); // TypeError: Cannot mix BigInt and other types

// Dynamic imports - ES2020
async function loadUtility() {
    const utils = await import('./utils.js');
    return utils.helper();
}

// Promise.allSettled - ES2020
const promises = [
    Promise.resolve('success'),
    Promise.reject('error'),
    Promise.resolve('another success')
];

Promise.allSettled(promises).then(results => {
    results.forEach((result, index) => {
        if (result.status === 'fulfilled') {
            console.log(`Promise ${index} succeeded:`, result.value);
        } else {
            console.log(`Promise ${index} failed:`, result.reason);
        }
    });
});

// globalThis - ES2020
const global = globalThis; // Works in all environments

// String.matchAll - ES2020
const str = 'test1test2test3';
const regex = /test(\d)/g;
const matches = [...str.matchAll(regex)];
console.log(matches); // Full match objects with groups

// Logical assignment operators - ES2021
let a = null;
let b = 0;
let c = 'hello';

a ||= 'default'; // a = a || 'default'
b &&= 10; // b = b && 10 (remains 0)
c ??= 'fallback'; // c = c ?? 'fallback' (remains 'hello')

// Numeric separators - ES2021
const million = 1_000_000;
const binary = 0b1010_0001;
const hex = 0xFF_EC_DE_5E;

// Private class fields - ES2022
class Counter {
    #count = 0; // Private field
    
    #increment() { // Private method
        this.#count++;
    }
    
    get value() {
        return this.#count;
    }
    
    tick() {
        this.#increment();
    }
}

// Top-level await - ES2022
// In modules only
const data = await fetch('/api/data');
const json = await data.json();

// Array.at() - ES2022
const arr = [1, 2, 3, 4, 5];
console.log(arr.at(-1)); // 5
console.log(arr.at(-2)); // 4

// Object.hasOwn() - ES2022
const obj = { a: 1 };
console.log(Object.hasOwn(obj, 'a')); // true (safer than hasOwnProperty)

// Error.cause - ES2022
try {
    throw new Error('Something went wrong');
} catch (err) {
    throw new Error('Failed to process', { cause: err });
}
```

### Testing Patterns
```javascript
// Simple test framework
class SimpleTest {
    constructor(description) {
        this.description = description;
        this.tests = [];
    }
    
    it(description, testFn) {
        this.tests.push({ description, testFn });
    }
    
    run() {
        console.log(`\n${this.description}`);
        let passed = 0;
        let failed = 0;
        
        this.tests.forEach(test => {
            try {
                test.testFn();
                console.log(`  ✓ ${test.description}`);
                passed++;
            } catch (error) {
                console.log(`  ✗ ${test.description}`);
                console.log(`    ${error.message}`);
                failed++;
            }
        });
        
        console.log(`\nPassed: ${passed}, Failed: ${failed}\n`);
    }
}

// Assertion helpers
const assert = {
    equal: (actual, expected) => {
        if (actual !== expected) {
            throw new Error(`Expected ${expected}, got ${actual}`);
        }
    },
    
    throws: (fn, expectedError) => {
        try {
            fn();
            throw new Error('Expected function to throw');
        } catch (error) {
            if (!expectedError || !(error instanceof expectedError)) {
                throw new Error(`Expected ${expectedError?.name || 'Error'}, got ${error.constructor.name}`);
            }
        }
    }
};

// Usage
const mathTest = new SimpleTest('Math utilities');

mathTest.it('should add numbers correctly', () => {
    assert.equal(2 + 2, 4);
});

mathTest.it('should handle division by zero', () => {
    assert.throws(() => {
        if (5 / 0 === Infinity) throw new Error('Division by zero');
    }, Error);
});

mathTest.run();
```

## Common Gotchas & Best Practices

### JavaScript Quirks
```javascript
// Type coercion surprises
console.log([] + []); // "" (empty string)
console.log([] + {}); // "[object Object]"
console.log({} + []); // 0 (in some contexts)
console.log(true + false); // 1
console.log("5" - 3); // 2
console.log("5" + 3); // "53"

// Array/Object comparison
console.log([] == []); // false (different references)
console.log({} == {}); // false (different references)

// NaN behavior
console.log(NaN === NaN); // false
console.log(Number.isNaN(NaN)); // true (preferred)
console.log(isNaN("hello")); // true (coerces to NaN)

// Floating point precision
console.log(0.1 + 0.2 === 0.3); // false
console.log(Math.abs(0.1 + 0.2 - 0.3) < Number.EPSILON); // true

// parseInt gotchas
console.log(parseInt("08")); // 8 (not octal in modern JS)
console.log(parseInt("10", 2)); // 2 (binary)
console.log(parseInt("hello123")); // NaN
console.log(parseInt("123hello")); // 123
```

### Best Practices
```javascript
// Use strict equality
if (x === y) { /* ... */ } // Preferred
if (x == y) { /* ... */ }   // Avoid

// Prefer const/let over var
const config = { /* ... */ }; // Immutable binding
let counter = 0;              // Mutable binding
// var old = 'avoid';         // Function scoped

// Use meaningful names
const userAge = 25;           // Good
const a = 25;                 // Bad

// Destructuring for cleaner code
const { name, email } = user; // Good
const name = user.name;       // Less clean
const email = user.email;

// Template literals for string interpolation
const message = `Hello, ${name}!`; // Good
const message2 = "Hello, " + name + "!"; // Verbose

// Array methods over loops
const doubled = numbers.map(n => n * 2);     // Functional
const evens = numbers.filter(n => n % 2 === 0);

// Async/await over promise chains
async function fetchUser(id) {
    try {
        const response = await fetch(`/users/${id}`);
        const user = await response.json();
        return user;
    } catch (error) {
        console.error('Failed to fetch user:', error);
        throw error;
    }
}

// Early returns to reduce nesting
function processUser(user) {
    if (!user) return null;
    if (!user.isActive) return null;
    
    // Process active user
    return {
        id: user.id,
        name: user.name.toUpperCase()
    };
}

// Use optional chaining and nullish coalescing
const userName = user?.profile?.name ?? 'Anonymous';

// Prefer composition over inheritance
const canFly = {
    fly() { console.log('Flying!'); }
};

const canSwim = {
    swim() { console.log('Swimming!'); }
};

const duck = Object.assign({}, canFly, canSwim, {
    name: 'Duck'
});
```