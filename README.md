# Roach

Roach is a minimalist, general-purpose, bit-level programming language. It's designed to easily access and modify odd or non-byte-based amounts of bits.

This document is a proof of concept for a language not yet implemented. The standard library is not yet documented too. I seek your feedback on the core of this language.

# Comments

```ro
\ This is a single-line comment.
```

```ro
\( This is a multi-line comment.
Note that '\)' needs to be preceded by a newline to close the comment.
\)
```

# Fields

Fields are groups of contiguous bits. Every scalar value in Roach is a field. Here are the various field types:

```ro
[n]   \ n-bit type
[0]   \ void or unit type
.T    \ floating-point version of type T
-T    \ signed version of unsigned type T
T~    \ dynamic version of type T where its size is its minimum number of bits
~T    \ dynamic version of type T where its size is its maximum number of bits
m~T   \ dynamic version of type T where its size is its maximum number of bits, and m is its minimum
T~m   \ dynamic version of type T where its size is its minimum number of bits, and m is its maximum
```

The bit-size of the CPU's registers (e.g. 32-bit or 64-bit) can be accessed as `?`.

Without `-`, types are unsigned.

# Composites

Composites are values composed of other values, called elements.

Here's a 64-bit floating-point type as a composite type:

```ro
[
	[1]  \ sign
	[11] \ exponent
	[52] \ fraction
]
```

Composite types can be restricted to only having elements, or items, of one type. These composites are known as arrays.

```ro
T*n \ array type with n items of type T
T*  \ equivalent to T*0~
```

# Numerals

Numerals are of type `[?]~`. Here are some examples:

```ro
0b0101
0o777
0xf0f0f0
21
0xa.1
```

# Operators

- `-x`
- `x + y`
- `x - y`
- `x * y`
- `x ** y` (exponentiation)
- `x / y`
- `~x`
- `x | y`
- `x ^ y`
- `x & y`
- `x << y`
- `x <<< y`
- `x <-< y` (left rotation)
- `x >> y`
- `x >>> y`
- `x >-> y` (right rotation)
- `!x`
- `x || y`
- `x && y`

More specific operators are explained later.

# Character Literals

Character literals are of type `[7]~`. Here are some examples:

```ro
'🐸'
'\n'
```

## Escape Sequences

* `\\` — `\`
* `\'` — `'`
* `\"` — `"`
* `\u(N)` — The unicode character with the code from the numeral `N`
* `\0` — NUL
* `\t` — TAB
* `\n` — LF
* `\r` — CR
* `\e` — ESC

# String Literals

String literals are of type `[7]~*`. Here are some examples:

```ro
"Ribbit, MFs!"
```

[Character escape sequences](#escape-sequences) apply to string literals too.

# Lengths

```ro
###x \ number of bits in x
##x \ number of elements in x
```

Lengths are of type `[8]~`.

# Names

Names can refer to types or values.

## Declarations

Names can contain any non-whitespace character, and must either start with a non-arabic-digit character, or end in a non-arabic-digit character.

Values of non-assigned names are zeroed-out.

Here's an example:

```ro
age: [8]
traversal: [
	start: [8]
	end: [8]
	n_times: [8]
]
```

Declarations return 0.

## Assignments

Once declared, names can be assigned values like this:

```ro
age = 10
```

Names can also be defined: declared and assigned in the same expression:

```ro
age: [8] = 10
```

Assignments return the assigned value.

## Constant Names

Top-level names (ones that are not refering to elements) can refer to constants by using `:=` instead of `=`. Here's an example:

```ro
SIZE := 16
```

## Type Aliases

Type aliases can't be redefined. Here are some examples:

```ro
Byte [8]
RawText [32]*
```

# Constructors

Here's an example with a quick constructor:

```ro
me: [name:RawText age:Byte] = "ghoom" 21
```

Here's a non-declarative variation with a full constructor:

```ro
me = {
	name: RawText = "ghoom"
	age: Byte = 21
}
```

# Accessing Bits

Assuming `v` is a value of any type:

```ro
v#i     \ the bit at index i
v#(i j) \ the field slice from index i to j
```

Negative indices and using this syntax in assignments are supported.

# Accessing Elements

Assuming `c` is a composite:

```ro
c.i     \ the element at index i
c.(i j) \ the composite slice from index i to j
c.a     \ the element named a
```

Negative indices and using this syntax in assignments are supported.

# Managing Bits

Bits can be inserted into a field `f` like this:

```ro
f <- x     \ a new field where x is appended
f#i <- x   \ a new field where x is inserted before the bit at index i
(f#i) <- x \ a new field only containing the bit at index i before x
```

And bits can be deleted like this:

```ro
f!       \ the default value of f's type (it's like a cleared version)
f#i!     \ a new field where the bit at index i is deleted
f#(i j)! \ a new field where the slice from index i to j is deleted
```

# Managing Elements

Elements can be inserted into a composite `c` like this:

```ro
c <- x     \ a new composite where x is appended
c#i <- x   \ a new composite where x is inserted before the element at index i
(c#i) <- x \ a new composite only containing the element at index i before x
```

And elements can be deleted like this:

```ro
c!       \ the default value of c's type (it's like a cleared version)
c#i!     \ a new composite where the element at index i is deleted
c#(i j)! \ a new composite where the slice from index i to j is deleted
```

# Note on Deleting Bits and Elements

Use `!!` to delete bits or elements in place:

```ro
c!! \ equivalent to c = c!
```

# Conversions

Assuming `x` is the value to be converted to type `T`:

```ro
x: T \ x as a value of type T
```

Everything is convertible to everything.

# Separating Statements

When necessary, statements can be seperated with `;`.

# Blocks

Blocks are scoped groups of expressions or statements, surrounded in `{}`, that can optionally return a value `x` like this:

```ro
=> x
```

# Conditionals

```ro
x -> y |> z \ If x is not 0, return y. Otherwise, return z.
```

`|> z` is optional.

# While Loops

```ro
x ~> y \ While x is not 0, add y to the array that the loop returns.
```

# Loop Control

Breaking out of a loop is done using:

```ro
<>
```

And "continuing" to the loop's next iteration is done using:

```ro
^^
```

# Functions

This is the function type:

```ro
P=>R \ function type taking parameter of type P and returning a value of type R
```

Here's an example of a function:

```ro
factorialize = n: [8] => {
	=> !n || n == 1
	-> 1
	|> n * factorialize(n - 1)
}

factorialize 5 \=> 120
```

All names in the parameter are accessible inside the function. `..` is the argument (useful for unnamed parameters).

Functions names are overloadable.

## Calls

Function calls are straightforward:

```ro
f   \ Call f with no argument (defaulting to 0).
f x \ Call f with argument x.
```

Use `(f)` to get the function instead of calling it.

## Coroutines

Coroutines are functions that use one of these constructs:

```ro
=>> x   \ Yield x.
=<< f   \ Yield to coroutine f with no arguments.
=<< f x \ Yield to coroutine f and pass argument x if its the first time yielding to it.
```

The following constructs are unlocked for coroutines:

```ro
^f   \ Asynchronously call coroutine f.
^f x \ Asynchronously call coroutine f with argument x.
```

Asynchronous calls return a value of type `[done:[1] value:T]` where `T` is the return type, and `done` is whether the function finally returned.

# For Loops

```ro
x: c ~> y \ For each return value of iterator call c, named x, add y to the array that the loop returns.
```

# Pointers

Here's the pointer type expression:

```ro
@T \ type of pointers to values of type T
```

Dereferencing a pointer `p` is done like this:

```ro
@p
```

# Generics

A generic is a composite with ambiguous types to be specified later.

Let's say we're remaking the `Map` type from the standard module *maps*:

```ro
Map <K V>[
	keys: K*
	values: V*
]
```

This is how we would select a variation of it:

```ro
Map<RawText RawText>
```

Generic selection isn't required if the types can be infered. For example:

```ro
Optional <T>[
	defined: [1]
	value: T
]

byte: [8] = 7
optional_byte: Optional = (1 byte)
```

# Command Line Arguments

`$` is an array of type `[7]~**` containing the command line arguments.

# Slash Functions

Slash functions are built-in constructs that are used as functions.

# Modules

Assuming *regex* is the name of a standard module, and `"user.ro"` is a path to a module, this is how you include them:

```ro
+regex
+"user.ro"
```

You can always use '/' in paths, regardless of the operating system. Assuming *x* is the name of a disk in Windows (e.g. C), the root path '/x' is 'x:' in Windows.

# Machine Code

Machine code can be inserted like this:

```ro
/x
```

...where `x` is the machine code.

# Detecting the Underlying ABI

`??`, a field of type `[?]`, can be one of:

1. Windows
1. Linux
1. Apple
1. FreeBSD
1. Solaris
1. HP-UX