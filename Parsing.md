#### 2.1 Parsers
> [!Definition] Parser
> Software that takes input data and builds a data structure such as a [[Parse Tree]] or [[Abstract Syntax Tree]], giving a structural representation of the input, checking for correct syntax in the process.
>
> The parser is often preceded by a separate lexical analyser, which creates tokens from the sequence of input characters.

A parser turns its input into a data structure that represents the input.

(link to the javascript image)

Fundamentally, the process that `JSON.parse()` applies to the input string is the same as our monkey parser will apply to something like `if((5 + 2 * 3) == 91) { return computeStuff(input1, input2) }`, it's just not necessarily intuitively obvious how one might construct a hierarchical data structure out of this.

The process of parsing is also called "syntactic analysis," as we can't avoid analysing the input, checking that it conforms to the expected structure.

#### 2.2 Parser generators

Examples: yacc, bison, ANTLR

Function: When fed with a formal description of a language, produce parsers as their output. This output can then be compiled/interpreted and itself fed with source code as input to produce a syntax tree.

Formats: [[Context-Free Grammar]]

As it happens, parsing is a well understood aspect of [[Computer Science]] - parsers are well suited to being automatically generated.

(link to highlight on page 31)

#### 2.3 Writing a parser

2 main strategies - top-down parsing or bottom-up parsing. There are different forms for each; "recursive descent parsing", "Early parsing", "predictive parsing" are variations of top-down parsing.

We'll build a "top-down operator precendence" parser, named a [[Pratt Parser]] after its inventor [[Vaughan Pratt]].

We'll start by parsing `let` and `return` statements.

#### 2.4 Parsing `let` statements

```monkey
let x = 5;
let y = 10;
let foobar = add(5, 5);
let barfoo = 5 * 5 / 10 + 18 - add(5, 5) + multiply(124);
let anotherName = barfoo;
```

These are `let` statements in monkey, and bind a value to the given name. To parse let statements correctly means that the parser products an [[Abstract Syntax Tree]] which accurately represents the information in the original let statement.

> [!Definition] Statements & Expressions
> "Before we go on, a few words about the difference between statements and expressions are needed. *Expressions* produce values, statements don't. 
>
> let x = 5 doesn't produce a value, whereas 5 does (the value it produces is 5). 
> A return 5; statement doesn't produce a value, but add(5, 5) does.
>
> This distinction - expressions produce values, statements don't - changes depending on who you ask, but it's good enough for our needs."

Depending on the language and their designers, different things can be expressions - in Monkey a lot of things are expressions, including [[Function Literal|function literals]].

We'll start work on the ast now, in `ast/ast.go`.

(link to statement on page 33 about the types that we're creating in here)
