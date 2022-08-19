# Lisp in 99 lines of C and how to write one yourself

In honor of the contributions made by Church and McCarthy, I wrote this project and the accompanying [article](tinylisp.pdf) to show how anyone can write a tiny Lisp interpreter in a few lines of C or in any "C-like" programming language for that matter.  I attempted to preserve the original meaning and flavor of Lisp as much as possible.  As a result, the C code in this project is strongly Lisp-like in compact form.  Despite being small, these tiny Lisp interpreters in C include 20 built-in Lisp primitives, garbage collection and REPL, which makes them a bit more practical than a toy example.  If desired, more Lisp features can be easily added with a few more lines of C as explained in my [article](tinylisp.pdf) with examples that are ready for you to try.

## Tinylisp running on a vintage Sharp PC-G850VS pocket computer

![PC-G850](lisp850.jpg)

A cool pocket computer with a built-in C compiler.  Tinylisp compiles and runs on this machine too with its native C!

## How is tinylisp so small?

Using NaN boxing and BCD boxing and some programming tricks in C.  See my [article](tinylisp.pdf) for details, examples and detailed instructions how to extend tinylisp with more features.

## Project code

Lisp in 99 lines is written in a Lisp-like functional style of structured C, lines are 55 columns wide on average and never wider than 120 columns for convenient editing.  It supports double precision floating point, has 20 built-in Lisp primitives, a REPL and a simple garbage collector.  Tail-call optimized versions are included for speed and reduced memory use.

- [tinylisp.c](src/tinylisp.c) Lisp in 99 lines of C with double precision
- [tinylisp-commented.c](src/tinylisp-commented.c) commented version in an (overly) verbose C style
- [tinylisp-opt.c](src/tinylisp-opt.c) optimized version for speed and reduced memory use
- [tinylisp-float.c](src/tinylisp-float.c) Lisp in 99 lines of C with single precision
- [tinylisp-float-opt.c](src/tinylisp-float-opt.c) optimized version with single precision
- [lisp850.c](src/lisp850.c) Lisp in 99 lines of C for the Sharp PC-G850 with BCD boxing
- [lisp850-opt.c](src/lisp850-opt.c) optimized version for speed and reduced memory use
- [common.lisp](src/common.lisp) common Lisp functions defined in tiny Lisp itself
- [list.lisp](src/list.lisp) list functions library
- [math.lisp](src/math.lisp) some Lisp math functions

To compile tinylisp:

    $ cc -o tinylisp tinylisp-opt.c

The number of cells allocated is N=1024 by default, which is only 8K of memory.  To increase memory size, increase the value of N in the code.  Then recompile tinylisp.

To install one or more optional Lisp libraries to run tinylisp, use Linux/Unix `cat`:

    $ cat common.lisp list.lisp math.lisp | ./tinylisp

But before you can do this, change the `look` function to reopen /dev/tty as explained in Section 7 of the [article](tinylisp.pdf).

## Lisp on a vintage Sharp PC-G850

On the Sharp PC-G850(V)(S) use SIO or [PocketTools](https://www.peil-partner.de/ifhe.de/sharp/) to load via audio cassette interface (CE-124 or CE-126p):

    PC:   bas2img --pc=G850VS --type=asm -l0x408 lisp850-opt.c
    PC:   bin2wav --pc=G850VS lisp850-opt.img
    G850: BASIC (PRO MODE)
    G850: BLOAD
    PC:   play lisp850-opt.wav
    G850: TEXT
    G850: Basic
    G850: Text<-Basic
    G850: BASIC (2x PRO MODE)
    G850: NEW
    G850: 2ndF TEXT (C)
    G840: G (go)

The `bas2img` option `-l0x400` adds line numbers to the C source automatically.

## 99-Liner Lisp language features

### Numbers

Double precision floating point numbers, including `inf`, `-inf` and `nan` (`nan` gives `ERR`).  Numbers may also be entered in hexadecimal `0xh...h` format.

### Symbols

Lisp symbols consist of a sequence of non-space characters, excluding `(`, `)` and quotes.  When used in a Lisp expression, a symbol is looked-up for its value, like a variable typically refers to its value.  Symbols can be '-quoted like `'foo` to use symbols literally and to pass them to functions.

### Booleans

Well, Lisp doesn't need Booleans.  An `()` empty list (called nil) is considered false and anything not `()` is considered true.  For convenience, `#t` is a symbol representing true (`#t` evaluates to itself, i.e. quoting is not needed.)

### Lists

Lists are code and data in Lisp.  Syntactically, a dot may be used for the last list element to construct a pair rather than a list.  For example, `'(1 . 2)` is a pair, whereas `'(1 2)` is a list.  By the nature of linked lists, a list after a dot creates a list, not a pair.  For example, `'(1 . (2 . ()))` is the same as `'(1 2)`.  Note that lists form a chain of pairs ending in a `()` nil.  

### Function calls

    (<function> <expr1> <expr2> ... <exprn>)

applies a function to the rest of the list of expresssions as its arguments.  The following are all built-in functions, called "primitives" and "special forms".

### Quoting and unquoting

    (quote <expr>)

protects `<expr>` from evaluation by quoting, same as `'<expr>`.  For example, `'(1 () foo (bar 7))` is a list containing unevaluated expressions protected by the quote.

    (eval <quoted-expr>)

evaluates a quoted expression and returns its value.  For example, `(eval '(+ 1 2))` is 3.

### Constructing and deconstructing pairs and lists

    (cons x y)

constructs a pair `(x . y)` for expressions `x` and `y`.  Lists are formed by chaining sevaral cons pairs, with the empty list `()` as the last `y`.  For example, `(cons 1 (cons 2 ()))` is the same as `'(1 2)`.

    (car <pair>)

returns the first part `x` of a pair `(x . y)` or list.

    (cdr <pair>)

returns the second part `y` of a pair `(x . y)`.  For lists this returns the rest of the list after the first part.

### Arithmetic

    (+ n1 n2 ... nk)
    (- n1 n2 ... nk)
    (* n1 n2 ... nk)
    (/ n1 n2 ... nk)

add, substract, multiply or divide `n1` to `nk`.  This is the same as `n1` <op> `n2` <op> ... <op> `nk`. Note that `(- 2)` is 2, not -2.  Also, at least one value `n` should be provided.  As the article suggests, change the Lisp interpreter as you like to change this.

    (int n)

returns the integer part of a number `n`.

### Logic

    (< n1 n2)

returns `#t` (true) if numbers `n1` < `n2`.  Otherwise, returns `()` (empty list means false).

    (eq? x y)

returns `#t` (true) if values `x` and `y` are identical.  Otherwise, returns `()` (empty list means false).  Numbers and symbols of the same value are always identical, but non-empty lists may or may not be identical even when their values are the same.

    (not x)

returns `#t` if `x` is not `()`.  Otherwise, returns `()` (empty list means false).

    (or x1 x2 ... xk)

returns the value of the first `x` that is not `()`.  Otherwise, returns `()` (empty list means false).  Only evaluates the `x` until the first is not `()`, i.e. the `or` is conditional.

    (and x1 x2 ... xk)

returns the value of the last `x` if all `x` are not `()`.  Otherwise, returns `()` (empty list means false).  Only evaluates the `x` until the first is `()`, i.e. the `and` is conditional.

### Conditionals

    (cond (x1 y1) (x2 y2) ... (xk yk))

returns the value of `y` corresponding to the first `x` that is not `()` (meaning not false, i.e. true.)  If an `y` is missing then `y` defaults to `()`.

    (if x y z)

if `x` is not `()` (meaning not false, i.e. true), then return `y` else return `z`.

### Lambdas

    (lambda <parameters> <expr>)

returns an anonymous function "closure" with a list of parameters and an expression as its body.  For example, `(lambda (n) (* n n))` squares its argument.  The parameters of a lambda may be a single name (not placed in a list) to pass all arguments as a named list.  For example, `(lambda args args)` returns its arguments as a list.  The pair dot may be used to indicate the rest of the arguments.  For example, `(lambda (f x . args) (f . args))` applies a function argument`f` to the arguments `args`, while ignoring `x`.  The closure includes the lexical scope of the lambda, i.e. local names defined in the outer scope can be used in the body.  For example, `(lambda (f x) (lambda args (f x . args)))` is a function that takes function `f` and argument `x` to return a [curried function](https://en.wikipedia.org/wiki/Currying).

### Globals

    (define <symbol> <expr>)

globally defines a symbol associated with the value of an expression.  If the expression is a function or a macro, then this globally defines the function or macro.

### Locals

Locals are declared with the following `let*` special form.  This form differ slightly in syntax from other Lisp and Scheme implementations, with the aim to make let-forms more intuitive to use (but you can change it in the Lisp interpreter if you like):

    (let* (v1 x1) (v2 x2) ... (vk xk) y)

evaluates `y` with a local scope of bindings for symbols `v` sequentially bound from the first to the last to the corresponding values of `x`.

## Additional Lisp primitives introduced in the [article](tinylisp.pdf)

    (assoc <quoted-symbol> <environment>)

returns the value associated with the quoted symbol in the given environment.

    (env)

returns the current environment.  When executed in the REPL, returns the global environment.

    (let (v1 x1) (v2 x2) ... (vk xk) y)

evaluates `y` with a local scope of bindings for symbols `v` simultaneously bound to the values of `x`.

    (letrec* (v1 x1) (v2 x2) ... (vk xk) y)

evaluates `y` with a local scope of recursive bindings for symbols `v` sequentually bound to the values of `x`.

    (setq <symbol> x)

assigns a globally or locally-bound symbol a new value.

    (set-car! <pair> x)
    (set-cdr! <pair> y)

assign a pair a new car or cdr value, respectively.

    (macro <parameters> <expr>)

a macro is like a function, except that it does not evaluate its arguments.  Macros typically construct Lisp code that is evaluated when the macro is expanded.

    (read)

returns the Lisp expression read from input.

    (print x1 x2 ... xk)
    (println x1 x2 ... xk)

prints the expressions.

    (trace <0|1|2>)

disables tracing (0), enables tracing (1), and enables tracing with ENTER key press (2).

    (catch <expr>)

catch exceptions in the evaluation of an expression, returns the value of the expression or `(ERR . n)` for nonzero error code `n`.

    (throw n)

throws error `n`, where `n` is a nonzero integer.

## Spoiler alert!

In "[Lisp in 1k lines of C, explained](https://github.com/Robert-van-Engelen/lisp)" I introduce another small Lisp interpreter that is largely based on tinylisp.  It shares many similarities, but has over 40 built-in Lisp primitives, strings, macros, exceptions, execution tracing, file loading, a mark-sweep/compacting garbage collector and REPL.
