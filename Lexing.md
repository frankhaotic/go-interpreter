![[Writing an Interpreter in Go#^ba1e50]]

The first transformation from source code into tokens is called "[[Lexical Analysis]]", or "Lexing", and it's performed by the "Lexer" (or "tokeniser" or "scanner").

A [[Token]] is a small categorisable data structure which is fed into the parser, which performs the second transformation into the [[Abstract Syntax Tree]]

![[Writing an Interpreter in Go#^7eb662]]

A production-ready lexer may also attach the line number, column number and filename to a token so that it can output helpful error messages in the parsing stage; instead of `error: expected semicolon token` it can output `error: expected semicolon token. line 42, column 23, program.monkey`.

### Defining our token

```monkey

let five = 5;
let ten = 10;

let add = fn(x, y) = {
    x + y;
}

let result = add(five, ten);

```

First we identify all of the potential tokens that we see within a block of text:
- numbers (5, 10)
- variable names (x, y, add, result)
- keywords (let, fn)
- special characters ('(', ')', '{', '}', '=', ',', ';')
 
### Comments on code

`TokenType` is defined as a string which allows us to use many different values as `TokenType`s. Many other advantages - including easy debugging as a string value (just print that fucker).

In the `lexer_test.go` we define an *anonymous structure* with fields to hold data specific to each test case:

```go
tests := []struct {
    expectedType    token.TokenType
    expectedLiteral string
}{
    {token.ASSIGN, "="},
    {token.PLUS, "+"},
    {token.LPAREN, "("},
    {token.RPAREN, ")"},
    {token.LBRACE, "{"},
    {token.RBRACE, "}"},
    {token.COMMA, ","},
    {token.SEMICOLON, ";"},
    {token.EOF, ""},
}
```

In the assertion phase of the tests we use Go's for-loop syntax:

```go

for i, tt := range tests {

    // stuff

}

```

This code does the following (ChatGPT):
- `i, tt := range tests`: This statement uses Go's range keyword to iterate over the tests slice. The range tests returns two values for each iteration:
    - `i` is the index of the current element in the slice.
    - `tt` is a **copy of the element at that index in the tests slice.** Each element is expected to be a struct containing expectedType and expectedLiteral, used to define test expectations for the lexer.

The `New()` function is defined in the `lexer` package after we create our definition of the `Lexer` [[struct]]. It's annotated as returning an instance of a Lexer. (what is at the address of the returned pointer, which is of type Lexer?)

> [!NOTE] ChatGPT explanation
> 1. **Function Declaration**: `func New(input string) *Lexer`, this declares a function named New that takes one argument, input, of type string. The function returns a pointer to a Lexer struct, indicated by the `*Lexer` return type.
> 3. **Initialization of a Lexer Instance**: `l := &Lexer{input: input}`, this line creates a new Lexer struct with the field input set to the value passed to the function. The & operator is used to create a pointer to the struct, rather than the struct itself. This means l is a pointer to the Lexer struct, not the struct itself.
> 4. **Return Value:** `return l`, this line returns the pointer l. Since l is of type `*Lexer` (a pointer to a Lexer), the return type matches the function declaration.

The & operator is used to get the memory address of a variable. It creates a pointer pointing to the variable. For example, if lexer is a variable of type Lexer, then &lexer would be a pointer to lexer, and its type would be *Lexer (the type which is a pointer to a Lexer).

When the \* is used before a pointer variable, it dereferences that pointer. That is, it accesses the value stored at the pointer's address. For instance, if l is a pointer of type *Lexer, using *l accesses the Lexer object that l points to.

```go

func main() {
	// lexerPointer is of type *Lexer
    lexerPointer := New("example input")  
    
	// dereference to get the actual Lexer struct
    // now actualLexer is of type Lexer and holds the actual struct value
    actualLexer := *lexerPointer          
}
```

- `lexerPointer` holds the memory address of the `Lexer` instance.
- `actualLexer` holds the actual `Lexer` object by dereferencing `lexerPointer`.

### Questions
What's the significance of defining a separate type of `TokenType` instead of just using a `string` in the `Token` definition?
- I suppose that it allows us to only specify certains strings that qualify?