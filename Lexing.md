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

(2024-04-22 18:11)
We create the `readChar()` method to help the lexer read the `ch` character in `position`

(link to image - pg 14)

After adding helper methods to read through the characters of the source code, we need to convert them into tokens:

```go
func (l *Lexer) NextToken() token.Token {
	var tok token.Token

	switch l.ch {
	case '=':
		tok = newToken(token.ASSIGN, l.ch)
	case ';':
		tok = newToken(token.SEMICOLON, l.ch)
	case '(':
		tok = newToken(token.LPAREN, l.ch)
	case ')':
		tok = newToken(token.RPAREN, l.ch)
	case ',':
		tok = newToken(token.COMMA, l.ch)
	case '+':
		tok = newToken(token.PLUS, l.ch)
	case '{':
		tok = newToken(token.LBRACE, l.ch)
	case '}':
		tok = newToken(token.RBRACE, l.ch)
	case 0:
		tok.Literal = ""
		tok.Type = token.EOF
	}

	l.readChar()
	return tok
}

func newToken(tokenType token.TokenType, ch byte) token.Token {
	return token.Token{Type: tokenType, Literal: string(ch)}
}

```

All we're doing here is reading through the source code and returning the appropriate `token.Token` according to what we've defined in the `token.go` file.

Returning to the test, we change the input to a block of legitimate-looking source code.

> [!NOTE] Note on [[Testing]]
> We're looking to test the behaviour of the code - here we're moving between testing and implementation, and using each to drive the other. 
>
> This change breaks on **identifiers**, **keywords** and **numbers**.
>
> After the first set of identifiers were implemented, we change the input to something more closely resembling the `monkey` language in its final form - we'll use these test to determine if our implementation is [[On Spec]] (*see: [[Design by Contract]]*).

We need to change the `Lexer` so that it can recognise the new identifiers, keywords and numbers that we're passing into it.

```go
func (l *Lexer) NextToken() token.Token {
    var tok token.Token

    switch l.ch {
    case '=':
	tok = newToken(token.ASSIGN, l.ch)
    case ';':
	tok = newToken(token.SEMICOLON, l.ch)
    case '(':
	tok = newToken(token.LPAREN, l.ch)
    case ')':
	tok = newToken(token.RPAREN, l.ch)
    case ',':
	tok = newToken(token.COMMA, l.ch)
    case '+':
	tok = newToken(token.PLUS, l.ch)
    case '{':
	tok = newToken(token.LBRACE, l.ch)
    case '}':
	tok = newToken(token.RBRACE, l.ch)
    case 0:
	tok.Literal = ""
	tok.Type = token.EOF
    default:
	if isLetter(l.ch) {
		tok.Literal = l.readIdentifier()
		return tok
	} else {
		tok = newToken(token.ILLEGAL, l.ch)
	}
    }

    l.readChar()
    return tok
}

func (l *Lexer) readIdentifier() string {
    position := l.position

    for isLetter(l.ch) {
	l.readChar()
    }

    return l.input[position:l.position]
}

func isLetter(ch byte) bool {
    return 'a' <= ch && ch <= 'z' || 'A' <= ch && ch <= 'Z' || ch == '_'
}
```

Add code to handle identifiers in the source code; if the current `input` is a character, but isn't a recognised token, then we read the text with `readIdentifier()`, and when it's completed we return the identifier in an array.

To account for this we need to a way to differentiate between identifiers and protected keywords; we add this to `Token`, as in future we might decide to add more primitive tokens to the language:

```go
var keywords = map[string]TokenType{
    "fn":  FUNCTION,
    "let": LET,
}

func LookupIdent(ident string) TokenType {
    // passing the `ident` string into the table, we'll return anything that
    // successfully return from out of it 
    if tok, ok := keywords[ident]; ok {
	return tok
    }

    // ...otherwise we return the IDENT constant
    return IDENT
}
```

> [!NOTE]- On code snippets
> I don't think it's really so productive to copy and paste everything that I'm adding to source code in here, so I'll try to stop adding things which aren't so special from here on out. ==The concepts from page-to-page aren't difficult, but the end-result is complex, so it's difficult to appreciate what deserves noting down.==

Continuing on, I added methods to skip whitespace and parse the remaining tokens in the source input.

(2024-04-23 13:47) - starting section 1.4

This section focusses on extending the token set and lexer functionality. We'll add support for the following operators: ==, !, !=, -, /, \*, \<, \>, and keywords: `true`, `false`, `if`, `else`, and `return`.

Starting with the operators, we add these new tokens to the token.go file, and update the lexer to recognise these new types. Update the tests to account for these too.

Adding the remaining keywords is also made simple by the existing `keywords` table and parsing methods on the lexer.

We now need to account for tokens composed of multiple characters - `==` and `!-`. To build this out we introduce the `peekChar()` method, which will allow us to peek ahead at upcoming char values from the input - similar to the `readChar()` method, but without incrementing `l.position` and `l.readPosition`.

> [!NOTE]- Lexer standards on peeking
> Most languages implementing peeking, but languages differ by the amount of peeking necessary to correctly tokenise the input.

Currently the lexer recognises `==` as 2 separate `ASSIGN` tokens, so using `peekChar()` we look ahead to see what the next character is:

```go
func (l *Lexer) peekChar() byte {
    if l.readPosition >= len(l.input) {
	return 0
    } else {
	return l.input[l.readPosition]
    }
}

// [...]
case '=':
    if l.peekChar() == '=' {
	ch := l.ch
	l.readChar()
	literal := string(ch) + string(l.ch)
	tok = token.Token{Type: token.EQ, Literal: literal}
    } else {
	tok = newToken(token.ASSIGN, l.ch)
    }
// [...]
case '!':
    if l.peekChar() == '=' {
	ch := l.ch
	l.readChar()
	literal := string(ch) + string(l.ch)
	tok = token.Token{Type: token.NOT_EQ, Literal: literal}
    } else {
	tok = newToken(token.BANG, l.ch)
    }
// [...]
```

We assign the current character to `ch` and read the next character, advancing the position for the next case of the lexer.

