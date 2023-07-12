# Lexing and Parsing

The very first thig the compiler does is take the source code and turn it into something that is more conveniently to work with than strings.

Lexing takes the source code and turns it into stream of tokens. For example, the following code:

```
pub fun main() {
    println("hello world")
}
```

is turned into the tokens, `pub`, `fun`, `main`, `(`, `)`, `{`, `println`, `(`, etc. The lexer lives in `ry_lexer` crate.

Parsing then takes the stream of tokens and turns it into an Abstract Syntax Tree (AST). AST is a data structure that represents relations between the tokens. The parser lives in `ry_parser`.

The crate defines `Parse` and `OptionallyParse` traits. These correspond to types that can convert streams of tokens into AST:

```rs
pub trait Parse
where
    Self: Sized,
{
    type Output;

    fn parse(&self, state: &mut ParseState<'_, '_, '_>) -> Self::Output;
}
```

```rs
pub trait OptionallyParse
where
    Self: Sized,
{
    type Output;

    fn optionally_parse(&self, state: &mut ParseState<'_, '_, '_>) -> Self::Output;
}
```

The difference between `Parse` and `OptionallyParse` is that `OptionallyParse` checks if it can go on. If it can't, it returns some kind of default value.

Here you might have already noticed `ParseState`. `ParseState` serves as: token iterator, parsing context (diagnostics and lexer state).

To parse the code above, we can use `parse_module`, which creates a `ParseState` and parses the code:

```rs
let interner = Interner::default();
let diagnostics = vec![];

let ast = parse_module(source, &mut interner, &mut diagnostics);
println!("{}", serialize_ast(&ast, &interner));
```

And this is the execution flow:

```
     read_and_parse_module() -> parse_module()
                                        |
    <ItemsParser as Parse>::parse()   <-|
            |
            |-> <ItemParser as Parse>::parse() -> ...
```
