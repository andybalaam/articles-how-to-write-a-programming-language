# How to write a programming language - Part 1, The Lexer

A programming language interpreter is a program that converts text (source
code) into behaviour.  Because it's a program that operates on other programs,
it can sound complicated - even something that shouldn't be attempted by mere
mortals - but really, programming languages are relatively simple programs,
often much simpler than the programs they are used to write.

![](programming-language-noexe.svg)

In this series we will be writing an interpreter for our own programming
language, called Cell.

Most programming languages are designed to make life easy for someone: usually
that person is the programmer who will be writing programs in the language.
Different languages have different designs based on the needs of that person -
for example, Python is designed to make code easy to read, and Rust is designed
to make it easy to avoid certain kinds of common mistakes.

Cell is unusual because it is designed to make life easy for us: the people who
are writing it.  Lots of things about its design mean that we can write less
code when we are implementing it.  Sometimes, this will make life a bit more
difficult for the person writing programs in Cell.  We will have to live with
that: Cell is a toy language, not a hardened tool.

First, let's look at an example of a program written in Cell.

## A Cell Program

This program shows how to make variables and call functions in Cell:

    x = 3;
    y = x + 2;
    print(y);

When you run it, this program will print out the value of y, which is 5.

Cell programs should be relatively familiar to people who have used a
curly-brace language like C, C++ or Java, and also takes inspiration from
dynamic languages like Python and Ruby.  (In fact, under the covers, the
language Cell looks most like is Lisp, but its syntax is different.)

## How does a programming language work?

Most programming languages are built from several parts: the lexer takes in
the source code and converts it into tokens, the parser understands the
structure described by the tokens and builds them into a syntax tree, and then
the evaluator uses the syntax tree to decide what to do*.

* Some languages have other steps - for example, compiled languages convert
  the syntax tree into an executable file instead of actually doing things,
  and the syntax tree can be transformed in various ways, for example to make
  the program run more quickly (optimisation).

## The Lexer

In this article we will look at the lexer, which takes in text (source code)
and transforms it into tokens.  Tokens are things like a number, or a name.

In Cell, the types of tokens are:

* Numbers: 12, 4.2
* Strings: "foo", 'bar'
* Symbols: baz, qux_Quux
* Operators: +, -
* Other punctuation: (, }
