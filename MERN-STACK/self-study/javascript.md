# Basic JavaScript Syntax Documentation

## Introduction

JavaScript is a widely-used, versatile programming language primarily employed for adding interactivity to web pages. As a client-side scripting language, it runs in the user's browser instead of on a server. This document outlines the fundamental syntax of JavaScript, serving as an essential guide for writing efficient and functional code.

## 1. Comments

JavaScript supports comments for code documentation and clarity. There are two types of comments:

- **Single-line comments**: Use `//` for comments that span a single line.
    ```js
    // This is a single-line comment
    ```

- **Multi-line comments**: Use `/* ... */` for comments that span multiple lines.
    ```js
    /*
    This is a multi-line
    comment
    */
    ```

## 2. Variables

Variables are containers for storing data values. JavaScript allows you to declare variables using the `var`, `let`, or `const` keywords:

- `var`: Declares a variable globally or locally to a function, regardless of block scope.
- `let`: Declares a block-scoped variable that can be reassigned.
- `const`: Declares a block-scoped variable that cannot be reassigned after initialization.

Example:
```js
var a = 5;
let b = 'Hello';
const c = true;
```

## 3. Data Types

JavaScript supports various data types, categorized as follows:

- **Primitive Data Types**: Number, String, Boolean, Undefined, Null
- **Composite Data Types**: Object, Array
- **Special Data Types**: Function

Example:
```js
let num = 10;
let str = "JavaScript";
let bool = true;
let arr = [1, 2, 3];
let obj = { name: "John", age: 30 };
let func = function() {
    console.log("Hello!");
};
```

## 4. Operators

JavaScript provides several operators for performing operations on variables and values. These include:

- **Arithmetic Operators**: `+`, `-`, `*`, `/`, `%`
- **Assignment Operators**: `=`, `+=`, `-=`, `*=`, `/=`
- **Comparison Operators**: `==`, `===`, `!=`, `!==`, `>`, `<`, `>=`, `<=`
- **Logical Operators**: `&&`, `||`, `!`
- **String Operator**: `+` (concatenation)
- **Conditional (Ternary) Operator**: `condition ? value1 : value2`

Example:
```js
let x = 10;
let y = 5;
let z = x + y; // Addition

let result = (x > y) ? "x is greater than y" : "x is less than or equal to y"; // Ternary Operator
```

## 5. Control Structures

JavaScript offers control structures to manage the flow of execution within a program.

- **Conditional Statements**: `if`, `else if`, `else` are used to make decisions in code.
- **Switch Statement**: Executes code based on different cases.
- **Loops**: JavaScript supports loops like `for`, `while`, and `do...while` for repeated code execution.

Example:
```js
let num = 10;

if (num > 0) {
    console.log("Positive number");
} else if (num < 0) {
    console.log("Negative number");
} else {
    console.log("Zero");
}

let i = 0;
while (i < 5) {
    console.log(i);
    i++;
}
```

## 6. Functions

Functions in JavaScript are reusable blocks of code. Functions can be defined in two ways:

- **Function Declaration**: Can be called before the function is defined in the code.
- **Function Expression**: Defines a function by assigning it to a variable.

Example:
```js
// Function Declaration
function greet(name) {
    return "Hello, " + name + "!";
}

// Function Expression
let greet = function(name) {
    return "Hello, " + name + "!";
};

console.log(greet("John"));
```

## Conclusion

This documentation covers the basic syntax of JavaScript, providing a solid foundation for writing JavaScript code. Understanding these core concepts is essential for creating dynamic and interactive web applications. As you continue to learn and practice, your grasp of JavaScript will deepen, allowing you to build more complex and efficient programs.