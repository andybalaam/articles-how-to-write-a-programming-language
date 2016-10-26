# How to write a programming language - Part 3, The Evaluator

This is the third part of our series on writing a programming language.  In
part one we broke up the code into chunks like numbers, strings and symbols
(lexing), and in part two we assembled those parts into a tree structure
(parsing).  Now we are ready to start understanding and evaluating the parts
of that tree structure to produce values and behaviour.

By the end of this article you will have seen all the most important parts
of a programming language, and be ready to write your own!

![](programming-language-parts.svg)

## Recap - lexing and parsing

The lexer and parser take some text like this:

    print( x + 2 );

and break it into parts, and then assemble it into a tree structure like this:

    ("call",
        ("symbol", "print"),
        [
            ("operation",
                "+",
                ("symbol", "x"),
                ("number", "2")
            )
        ]
    )

Cell is written in Python and uses Python tuples to hold all the structures it
represents.  Each tuple contains a string representing its type as the first
element, and then other information in the other elements of the tuple.

So far we have seen tuples representing tokens coming out of the lexer, and
tuples representing syntax trees coming out of the parser.  This time we will
see more tuples, representing values that have been worked out by the
evaluator.

## The evaluator calculates values

![](evaluator.svg)

The evaluator starts at the leaves of the syntax tree and calculates the values
of the leaves, then combines together leaves and branches until it ends up with
a single value.  On the way it may have produced some side effects such as
printing something out.

![](values.svg)

## Scope

Before we look at how the evaluator works out values, we must look at the idea
of scope.  Scope describes what names can be seen where in our code.  Cell,
like almost all modern programming languages, uses "lexical" scope, which means
the values you can see are dictated by the position of the code in the text on
the screen.

So, for example, this snippet of Cell code:

    x = "World!";
    myfn = {
        x = "Hello,";
        print( x );
    };
    myfn();
    print( x );

prints:

    Hello,
    World

because the value of x inside the function myfn is set to "Hello," within
the function definition, but it reverts to "World!" in code that is outside
that block.

This more complicated example:

    outerfn = {
        x = 12;
        innerfn = {
            print(x);
        };
        innerfn;
    };

    thing = outerfn();
    thing();

prints "12" because the function innerfn carries the values it knows about with
it, meaning that when we call the function returned by outerfn(), which is
actually innerfn because that is what is returned by outerfn when we call it,
it runs the "print(x)" line and it still knows what "x" is.  Functions that are
carrying their values with them are called Closures, and the set of values
that is passed around is called an Environment.  Environments are key to the
way the evaluator works.

## Environments

An environment is a namespace that holds all the symbols that are defined in
your program.  As illustrated in figure 3, each piece of code operates inside
a local environment (such as the current function) but can also access
symbols from outer environments (such as an outer function) and the global
environment, that contains important symbols such as the "if" function.

![](environments.svg)

This structure is provided in Cell by a class called Env.  It takes a parent
environment as a constructor argument, which it holds in self.parent.  To
look up a name we call the get() method:

    class Env:
        # ...
        def get(self, name):
            if name in self.items:
                return self.items[name]
            elif self.parent is not None:
                return self.parent.get(name)
            else:
                return None

This method checks whether a symbol is defined locally, and if not, it asks
the parent environment, and gives up when it gets to the global environment,
which has None for its self.parent value.

Defining a symbol means calling set(), which is simpler:

    class Env:
        # ...
        def set(self, name, value):
            self.items[name] = value

So newly defined symbols are always defined in the local environment, and
don't leak out into wider scopes.

## The Evaluator

Listing 1 shows the main logic of the evaluator.  You can see the full code
at https://github.com/andybalaam/cell/blob/master/pycell/eval_.py , but here
we can see the main structure is very similar to the code we saw in the
previous two parts - a large if-elif block responding differently to the
various possible structures.

    def eval_expr(expr, env):
        typ = expr[0]
        if typ == "number":
            return ("number", float(expr[1]))
        elif typ == "string":
            return ("string", expr[1])
        elif typ == "none":
            return ("none",)
        elif typ == "operation":
            return _operation(expr, env)
        elif typ == "symbol":
            name = expr[1]
            ret = env.get(name)
            if ret is None:
                raise Exception("Unknown symbol '%s'." % name)
            else:
                return ret
        elif typ == "assignment":
            var_name = expr[1][1]
            val = eval_expr(expr[2], env)
            env.set(var_name, val)
            return val
        elif typ == "call":
            return _function_call(expr, env)
        elif typ == "function":
            return ("function", expr[1], expr[2], Env(env))
        else:
            raise Exception("Unknown expression type: " + str(expr))

In the evaluator, the eval_expr() function takes in an expression to evaluate,
and the environment in which to work.  Without an environment we can't do
anything since we don't know where to look up the symbols that are being used
in the code.  The environment is an instance of the Env class we saw earlier,
and the expression is a Python tuple representing part of a syntax tree.

The first part of the tuple tells us the type of syntax tree section we have.
We place this into a variable called typ, and use it in the if block.

If we are evaluating a number, we use Python's float() function to convert the
string form that was captured in the lexer token (as expr[1], the second value
in the tuple) into a Python number.  This means we can do arithmetic with it
later if we need to, and illustrates the fact that the evaluator is where the
textual and structural forms of the code are converted in "meaning" such as
finding actual numbers and looking up the values of symbols.

If we find a string, we have very little to do, since the the value of a string
looks identical to its form as a lexer token - i.e. it is a tuple of two
values, the first of which is "string" and the second is the contents of that
string.

## Summary

#Parsing is an odd programming task, because we want to handle it piece by
#piece, but we sometimes need to soak up several tokens before we know what
#we are dealing with, and we need to produce a nested structure as our output.
#By using recursion (calling next_expression from inside itself) we can get
#the nested structure almost for free.
#
#The code we looked at here is more complicated than the lexer we saw in the
#last article, but I think you'll agree there is no magic here.  The whole of
#Cell's parser is just 81 lines of code (including empty lines).  You can find
#it on Cell's GitHub site at https://github.com/andybalaam/cell along with more
#explanations (including some videos).
#
#Next time, we'll get to the real point: we'll look at the evaluator, which
#takes in the nice structured syntax tree produced by the parser and actually
#does things, turning our code into behaviour.
