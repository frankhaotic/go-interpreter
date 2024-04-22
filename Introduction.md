---
resources:
  - "[[Writing an Interpreter in Go]]"
---

Interpreters take in code and produce something; characters are fed in and become meaningful.

Though interpreters vary, a fundamental attribute is that they take source code and evaluate it without producing some visible intermediate result that can later be executed, in contrast to [[Compilers]] which product output in another language that the underlying system can understand (c++ compiles to assembly, for example)

Some interpreters are small and don't even parse the input, skipping straight to interpreting the input source code. Others are highly optimised and use advanced [[Parsing|parsing]] and evaluation techniques. Some compile into an internal representation of "[[Bytecode]]" and evaluate this. Other more advanced techniques are [[Just-in-time Compiler]]s which compile the input "just-in-time" into native machine code which *then* gets executed.

This book focuses on building a "[[Tree-walking Interpreter]]" interpreter which will:
- [[Parse the source code]]
- [[Build an Abstract Syntax Tree]] ([[Abstract Syntax Tree]])
- [[Evaluating the Abstract Syntax Tree]]

### What does it mean to "create" a programming language
To create a language is to build an interpreter or a compiler. Without one, the language is merely a specification of potential language features. It's the *environment* in which that language can run that defines its functional existence.

### Monkey feature list

- C-like syntax
- Variable bindings
- Integers and booleans
- Arithmetic expressions
- Built-in functions
- First-class and higher-order functions
- Closures
- A string data structure
- An array data structure
- A hash data structure

The language will be capable of a standard experience familiar to a C/Javascript developer, including recursive function calls:

```monkey

let fibonacci = fn(x) {
    if(x == 0) {
        0
    } else {
        if (x == 1) {
            1
        } else {
            fibonacci(x - 1) + fibonacci(x - 2)
        }
    }
}

```

### Parts of the interpreter

- The [[Lexer]]
- The [[Parser]]
- The [[Abstract Syntax Tree]]
- The [[Internal Object System]]
- The [[Evaluator]]