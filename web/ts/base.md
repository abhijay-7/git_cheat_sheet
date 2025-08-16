# TypeScript In-Depth Cheat Sheet

## Basic Types

### Primitive Types
```typescript
// Basic types
let name: string = "John";
let age: number = 30;
let isActive: boolean = true;
let value: null = null;
let notDefined: undefined = undefined;
let id: bigint = 100n;
let uniqueId: symbol = Symbol("id");

// Type inference
let inferredString = "Hello"; // inferred as string
let inferredNumber = 42;      // inferred as number
```

### Arrays and Tuples
```typescript
// Arrays
let numbers: number[] = [1, 2, 3];
let strings: Array<string> = ["a", "b", "c"];

// Tuples
let person: [string, number] = ["John", 30];
let coordinates: [number, number, number?] = [10, 20]; // optional third element

// Named tuples
let namedTuple: [name: string, age: number] = ["John", 30];

// Rest in tuples
let restTuple: [string, ...number[]] = ["prefix", 1, 2, 3];
```

### Object Types
```typescript
// Object type annotation
let user: { name: string; age: number } = {
  name: "John",
  age: 30
};

// Optional properties
let optionalUser: { name: string; age?: number } = {
  name: "Jane"
};

// Readonly properties
let readonlyUser: { readonly id: number; name: string } = {
  id: 1,
  name: "John"
};
```

## Advanced Types

### Union and Intersection Types
```typescript
// Union types
type StringOrNumber = string | number;
let value: StringOrNumber = "hello";
value = 42;

// Intersection types
type Person = { name: string };
type Employee = { employeeId: number };
type PersonEmployee = Person & Employee;

let worker: PersonEmployee = {
  name: "John",
  employeeId: 123
};

// Discriminated unions
type Shape = 
  | { kind: "circle"; radius: number }
  | { kind: "square"; size: number }
  | { kind: "rectangle"; width: number; height: number };

function getArea(shape: Shape): number {
  switch (shape.kind) {
    case "circle":
      return Math.PI * shape.radius ** 2;
    case "square":
      return shape.size ** 2;
    case "rectangle":
      return shape.width * shape.height;
  }
}
```

### Literal Types
```typescript
// String literal types
type Direction = "up" | "down" | "left" | "right";
let move: Direction = "up";

// Numeric literal types
type DiceRoll = 1 | 2 | 3 | 4 | 5 | 6;
let roll: DiceRoll = 4;

// Boolean literal types
type Success = true;
type Failure = false;

// Template literal types
type EventName<T extends string> = `${T}Changed`;
type UserEvent = EventName<"user">; // "userChanged"
```

### Utility Types
```typescript
interface User {
  id: number;
  name: string;
  email: string;
  isActive: boolean;
}

// Partial - makes all properties optional
type PartialUser = Partial<User>;

// Required - makes all properties required
type RequiredUser = Required<PartialUser>;

// Pick - select specific properties
type UserSummary = Pick<User, "id" | "name">;

// Omit - exclude specific properties
type CreateUser = Omit<User, "id">;

// Record - create object type with specific keys and values
type UserRoles = Record<string, "admin" | "user" | "guest">;

// Exclude - exclude types from union
type NonBooleanPrimitives = Exclude<string | number | boolean, boolean>;

// Extract - extract types from union
type BooleanFromPrimitives = Extract<string | number | boolean, boolean>;

// NonNullable - exclude null and undefined
type NonNullableString = NonNullable<string | null | undefined>;

// ReturnType - get return type of function
function getUser(): User { /* ... */ }
type GetUserReturn = ReturnType<typeof getUser>; // User

// Parameters - get parameters tuple of function
type GetUserParams = Parameters<typeof getUser>; // []
```

## Functions

### Function Types
```typescript
// Function declaration
function add(a: number, b: number): number {
  return a + b;
}

// Function expression
const multiply = (a: number, b: number): number => a * b;

// Function type annotation
let operation: (x: number, y: number) => number;
operation = add;

// Optional and default parameters
function greet(name: string, greeting: string = "Hello", punctuation?: string): string {
  return `${greeting}, ${name}${punctuation || "!"}`;
}

// Rest parameters
function sum(...numbers: number[]): number {
  return numbers.reduce((acc, num) => acc + num, 0);
}

// Function overloads
function combine(a: string, b: string): string;
function combine(a: number, b: number): number;
function combine(a: string | number, b: string | number): string | number {
  if (typeof a === "string" && typeof b === "string") {
    return a + b;
  }
  if (typeof a === "number" && typeof b === "number") {
    return a + b;
  }
  throw new Error("Invalid arguments");
}
```

### Generic Functions
```typescript
// Generic function
function identity<T>(arg: T): T {
  return arg;
}

// Multiple type parameters
function pair<T, U>(first: T, second: U): [T, U] {
  return [first, second];
}

// Generic constraints
interface Lengthwise {
  length: number;
}

function logLength<T extends Lengthwise>(arg: T): T {
  console.log(arg.length);
  return arg;
}

// Conditional types in functions
type ApiResponse<T> = T extends string 
  ? { message: T } 
  : { data: T };

function handleResponse<T>(input: T): ApiResponse<T> {
  if (typeof input === "string") {
    return { message: input } as ApiResponse<T>;
  }
  return { data: input } as ApiResponse<T>;
}
```

## Interfaces and Classes

### Interfaces
```typescript
interface Animal {
  name: string;
  age: number;
  makeSound(): void;
}

// Interface inheritance
interface Dog extends Animal {
  breed: string;
  wagTail(): void;
}

// Multiple inheritance
interface Pet {
  owner: string;
}

interface DomesticDog extends Dog, Pet {
  isHouseTrained: boolean;
}

// Interface merging
interface User {
  name: string;
}

interface User {
  age: number;
}

// Now User has both name and age

// Index signatures
interface StringDictionary {
  [key: string]: string;
}

interface NumberDictionary {
  [key: string]: number;
  length: number; // OK, length is a number
  // name: string; // Error, name is not a number
}

// Call signatures
interface SearchFunc {
  (source: string, subString: string): boolean;
}

// Construct signatures
interface ClockConstructor {
  new (hour: number, minute: number): ClockInterface;
}

interface ClockInterface {
  tick(): void;
}
```

### Classes
```typescript
class Vehicle {
  // Property declarations
  protected engine: string;
  private _speed: number = 0;
  public readonly manufacturer: string;

  constructor(engine: string, manufacturer: string) {
    this.engine = engine;
    this.manufacturer = manufacturer;
  }

  // Getter and setter
  get speed(): number {
    return this._speed;
  }

  set speed(value: number) {
    if (value >= 0) {
      this._speed = value;
    }
  }

  // Methods
  public start(): void {
    console.log("Starting engine...");
  }

  protected accelerate(): void {
    this._speed += 10;
  }
}

// Class inheritance
class Car extends Vehicle {
  private doors: number;

  constructor(engine: string, manufacturer: string, doors: number) {
    super(engine, manufacturer);
    this.doors = doors;
  }

  // Method override
  public start(): void {
    console.log("Starting car...");
    super.start();
  }

  public boost(): void {
    this.accelerate(); // Can access protected method
  }
}

// Abstract classes
abstract class Shape {
  abstract area(): number;
  
  display(): void {
    console.log(`Area: ${this.area()}`);
  }
}

class Circle extends Shape {
  constructor(private radius: number) {
    super();
  }

  area(): number {
    return Math.PI * this.radius ** 2;
  }
}

// Static members
class MathUtils {
  static readonly PI = 3.14159;
  
  static circleArea(radius: number): number {
    return this.PI * radius ** 2;
  }
}
```

## Generics

### Generic Types
```typescript
// Generic interfaces
interface Container<T> {
  value: T;
  getValue(): T;
  setValue(value: T): void;
}

// Generic classes
class GenericNumber<T> {
  zeroValue: T;
  add: (x: T, y: T) => T;

  constructor(zeroValue: T, addFn: (x: T, y: T) => T) {
    this.zeroValue = zeroValue;
    this.add = addFn;
  }
}

// Generic constraints
interface Identifiable {
  id: number;
}

class EntityStore<T extends Identifiable> {
  private entities: T[] = [];

  add(entity: T): void {
    this.entities.push(entity);
  }

  findById(id: number): T | undefined {
    return this.entities.find(entity => entity.id === id);
  }
}

// Keyof constraints
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}

// Conditional types
type NonNullable<T> = T extends null | undefined ? never : T;

type FunctionPropertyNames<T> = {
  [K in keyof T]: T[K] extends Function ? K : never;
}[keyof T];

type FunctionProperties<T> = Pick<T, FunctionPropertyNames<T>>;

// Mapped types
type Readonly<T> = {
  readonly [P in keyof T]: T[P];
};

type Optional<T> = {
  [P in keyof T]?: T[P];
};

type Nullable<T> = {
  [P in keyof T]: T[P] | null;
};
```

## Type Guards and Assertions

### Type Guards
```typescript
// typeof type guard
function padLeft(value: string, padding: string | number) {
  if (typeof padding === "number") {
    return Array(padding + 1).join(" ") + value;
  }
  if (typeof padding === "string") {
    return padding + value;
  }
  throw new Error(`Expected string or number, got '${padding}'.`);
}

// instanceof type guard
class Fish {
  swim() { console.log("Swimming..."); }
}

class Bird {
  fly() { console.log("Flying..."); }
}

function move(animal: Fish | Bird) {
  if (animal instanceof Fish) {
    animal.swim();
  } else {
    animal.fly();
  }
}

// Custom type guards
interface Cat {
  meow(): void;
}

interface Dog {
  bark(): void;
}

function isCat(pet: Cat | Dog): pet is Cat {
  return (pet as Cat).meow !== undefined;
}

function makeSound(pet: Cat | Dog) {
  if (isCat(pet)) {
    pet.meow();
  } else {
    pet.bark();
  }
}

// Nullish type guard
function processValue(value: string | null | undefined) {
  if (value != null) { // checks for both null and undefined
    console.log(value.toUpperCase());
  }
}
```

### Type Assertions
```typescript
// Angle bracket syntax
let someValue: unknown = "this is a string";
let strLength: number = (<string>someValue).length;

// As syntax (preferred)
let strLength2: number = (someValue as string).length;

// Non-null assertion operator
function processEntity(entity?: { id: number; name: string }) {
  // We know entity is not null/undefined here
  console.log(entity!.name);
}

// Const assertions
let colors = ["red", "green", "blue"] as const;
// colors is now readonly ["red", "green", "blue"]

let config = {
  apiUrl: "https://api.example.com",
  timeout: 5000
} as const;
// All properties are now readonly
```

## Modules and Namespaces

### ES6 Modules
```typescript
// math.ts
export function add(a: number, b: number): number {
  return a + b;
}

export function subtract(a: number, b: number): number {
  return a - b;
}

export default function multiply(a: number, b: number): number {
  return a * b;
}

export const PI = 3.14159;

// main.ts
import multiply, { add, subtract, PI } from './math';
import * as MathUtils from './math';

// Re-exports
export { add, subtract } from './math';
export { default as multiply } from './math';
```

### Namespaces
```typescript
namespace Geometry {
  export interface Point {
    x: number;
    y: number;
  }

  export function distance(p1: Point, p2: Point): number {
    return Math.sqrt((p2.x - p1.x) ** 2 + (p2.y - p1.y) ** 2);
  }

  export namespace Shapes {
    export class Circle {
      constructor(public center: Point, public radius: number) {}
    }
  }
}

// Usage
let point1: Geometry.Point = { x: 0, y: 0 };
let point2: Geometry.Point = { x: 3, y: 4 };
let dist = Geometry.distance(point1, point2);
let circle = new Geometry.Shapes.Circle(point1, 5);
```

## Declaration Files and Ambient Declarations

### Declaration Files
```typescript
// types.d.ts
declare module "my-library" {
  export function doSomething(value: string): number;
  export interface Config {
    apiKey: string;
    debug?: boolean;
  }
}

// Global declarations
declare global {
  interface Window {
    myGlobalFunction(): void;
  }
  
  var MY_GLOBAL_VAR: string;
}

// Ambient modules
declare module "*.json" {
  const value: any;
  export default value;
}

declare module "*.css" {
  const classes: { [key: string]: string };
  export default classes;
}
```

### Triple-slash Directives
```typescript
/// <reference path="types.d.ts" />
/// <reference types="node" />
/// <reference lib="dom" />
```

## Advanced Patterns

### Decorators
```typescript
// Class decorator
function sealed(constructor: Function) {
  Object.seal(constructor);
  Object.seal(constructor.prototype);
}

// Method decorator
function enumerable(value: boolean) {
  return function (target: any, propertyKey: string, descriptor: PropertyDescriptor) {
    descriptor.enumerable = value;
  };
}

// Property decorator
function format(formatString: string) {
  return function (target: any, propertyKey: string): any {
    let value = target[propertyKey];

    const getter = () => {
      return `${formatString} ${value}`;
    };

    const setter = (newVal: string) => {
      value = newVal;
    };

    return {
      get: getter,
      set: setter,
      enumerable: true,
      configurable: true,
    };
  };
}

@sealed
class Greeter {
  @format("Hello")
  greeting: string;

  constructor(message: string) {
    this.greeting = message;
  }

  @enumerable(false)
  greet() {
    return "Hello, " + this.greeting;
  }
}
```

### Mixins
```typescript
// Mixin pattern
type Constructor = new (...args: any[]) => {};

function Timestamped<TBase extends Constructor>(Base: TBase) {
  return class extends Base {
    timestamp = Date.now();
  };
}

function Activatable<TBase extends Constructor>(Base: TBase) {
  return class extends Base {
    isActivated = false;

    activate() {
      this.isActivated = true;
    }

    deactivate() {
      this.isActivated = false;
    }
  };
}

class User {
  name = "";
}

const TimestampedUser = Timestamped(User);
const TimestampedActivatableUser = Timestamped(Activatable(User));
```

## TypeScript Configuration

### tsconfig.json
```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "commonjs",
    "lib": ["ES2020", "DOM"],
    "outDir": "./dist",
    "rootDir": "./src",
    "strict": true,
    "noImplicitAny": true,
    "strictNullChecks": true,
    "strictFunctionTypes": true,
    "noImplicitReturns": true,
    "noImplicitThis": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "exactOptionalPropertyTypes": true,
    "moduleResolution": "node",
    "baseUrl": "./",
    "paths": {
      "@/*": ["src/*"],
      "@utils/*": ["src/utils/*"]
    },
    "esModuleInterop": true,
    "allowSyntheticDefaultImports": true,
    "experimentalDecorators": true,
    "emitDecoratorMetadata": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "resolveJsonModule": true
  },
  "include": [
    "src/**/*"
  ],
  "exclude": [
    "node_modules",
    "dist",
    "**/*.test.ts"
  ]
}
```

## Compiler API and Advanced Usage

### Compiler Options
```typescript
// Programmatic usage
import * as ts from "typescript";

const source = `
  function greet(name: string): string {
    return "Hello, " + name;
  }
`;

const result = ts.transpile(source, {
  module: ts.ModuleKind.CommonJS,
  target: ts.ScriptTarget.ES2015
});

console.log(result);
```

### JSDoc Support
```typescript
/**
 * Calculates the area of a rectangle
 * @param width - The width of the rectangle
 * @param height - The height of the rectangle
 * @returns The area of the rectangle
 * @example
 * ```typescript
 * const area = calculateArea(5, 3); // 15
 * ```
 */
function calculateArea(width: number, height: number): number {
  return width * height;
}

// JSDoc type annotations (for JS files)
/**
 * @typedef {Object} User
 * @property {string} name
 * @property {number} age
 */

/**
 * @param {User} user
 * @returns {string}
 */
function getUserInfo(user) {
  return `${user.name} (${user.age})`;
}
```

## Best Practices and Common Patterns

### Error Handling
```typescript
// Result pattern
type Result<T, E = Error> = 
  | { success: true; data: T }
  | { success: false; error: E };

function parseJSON<T>(json: string): Result<T> {
  try {
    const data = JSON.parse(json);
    return { success: true, data };
  } catch (error) {
    return { success: false, error: error as Error };
  }
}

// Option pattern
type Option<T> = T | null;

function findUser(id: number): Option<User> {
  // Implementation
  return null; // or user object
}

// Custom error types
class ValidationError extends Error {
  constructor(
    message: string,
    public field: string
  ) {
    super(message);
    this.name = "ValidationError";
  }
}
```

### Performance Tips
```typescript
// Use const assertions for immutable data
const themes = ["light", "dark"] as const;
type Theme = typeof themes[number]; // "light" | "dark"

// Prefer interface over type for object shapes
interface User {
  id: number;
  name: string;
}

// Use unknown instead of any
function processValue(value: unknown): string {
  if (typeof value === "string") {
    return value;
  }
  return String(value);
}

// Lazy evaluation with getter functions
type LazyValue<T> = () => T;

function createExpensiveComputation(): LazyValue<number> {
  return () => {
    // Expensive computation here
    return Math.random() * 1000;
  };
}
```

## CLI Commands

### TypeScript Compiler
```bash
# Install TypeScript globally
npm install -g typescript

# Compile TypeScript file
tsc app.ts

# Compile with options
tsc app.ts --target es2015 --module commonjs

# Watch mode
tsc --watch

# Compile project
tsc -p tsconfig.json

# Initialize tsconfig.json
tsc --init

# Check types without emitting
tsc --noEmit

# Show compiler version
tsc --version
```

### Common NPM Scripts
```json
{
  "scripts": {
    "build": "tsc",
    "build:watch": "tsc --watch",
    "build:prod": "tsc --build --clean",
    "type-check": "tsc --noEmit",
    "type-check:watch": "tsc --noEmit --watch"
  }
}
```

This cheat sheet covers the essential TypeScript concepts from basic types to advanced patterns. Keep it handy for quick reference during development!