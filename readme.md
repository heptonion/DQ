# DQ: *The* Queue-Based Programming Language

_Winner of the "The Knuth Award" and "Turbo" awards from [Quirky Languages Done
 Quick](https://quirkylanguages.com/)!_

## Introduction

The only data type in DQ is the queue (whether finite or infinite), and as a
result every syntactically valid program is well-typed. (In other words, the
only errors are parse errors; it’s (supposed to be) impossible to cause an
evaluation error.)

Natural numbers are encoded as queues of empty queues.
```
dq> 0
0
dq> []
0
dq> 3
3
dq> [[], [], []]
3
```

Strings are encoded as queues of numbers.
```
dq> "helo"
helo
dq> [104, 101, 108, 111]
helo
dq> [[[], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], []], [[], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], []], [[], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], []], [[], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], [], []]]
helo
```

## Statements

There are five statements:
```
name := expr
printNum  expr
printStr  expr
printRepr expr
print     expr
```

The `printNum` statement evaluates `expr` and prints the length of the resulting
queue.

The `printStr` statement evaluates `expr`, interprets the elements of the
resulting queue as numbers (that is, calculates the length each each element),
decodes each number as a character, and prints each character.

The `printRepr` statements evaluates `expr` and prints its structure with
brackets and commas, using ε as an abbreviation for `[]`.

```
dq> printNum [[[], []], [], []]
3
dq> printStr [[[], []], [], []]
^B^@^@
dq> printRepr [[[], []], [], []]
[ε, ε], ε, ε
```

The `print` statement automatically selects one of `printNum`, `printStr`, or
`printRepr` depending on its argument.

Entering an expression at the interactive prompt is equivalent to entering
`print` followed by the expression, except the evaluation of the expression
will be limited to about one million terms.

**Note** that evaluating a queue drains it:
```
dq> x := [[], [], []]
dq> print x
3
dq> print x
0
```
```
dq> x := 4
dq> x + x
4
```
You can use the `$`, `~`, and `_` operators to use a queue multiple times
(by creating an infinite queue containing copies of the queue and dequeuing
copies as needed).
```
dq> x := $[[], [], []]
dq> _(1 ~ x)
3
dq> _(1 ~ x)
3
```

## Operations

The `+` operator concatenates two queues; specifically, `a + b` drains the
elements of `a` then drains the elements of `b`, gathering the elements in
a new queue.
```
dq> 1 + 1
2
dq> "helo, " + "world!"
helo, world!
```

The `$` operator produces an infinite queue containing copies of its argument.
```
$2 = [2, 2, 2, 2, ...]
```

The `_` operator flattens its argument, creating a new queue by draining its
first element, then its second, then its third, and so on. (In other words, it
folds its argument using the `+` operator.)
```
_[] = []
```
```
_3 = _[[], [], []]
   = ([] + []) + []
   = []
   = 0
```
```
dq> _3
0
dq> _[2, 0]
2
dq> _[2, 1]
3
dq> _[2, 1, 1]
4
```

The `~` operator zips its arguments, concatenating respective pairs of elements,
```
[a, b, c, d] ~ [x, y, z] = [a+x, b+y, c+z]
```
stopping when either argument is exhausted. As a result, it can be used to find
the minimum of two numbers.
```
dq> 7 ~ 5
5
dq> 7 ~ 11
7
```

The pattern `_(1 ~ x)` shown above can be used to repeatedly obtain `a` where
`x := $a`.
```
x = [a, a, a, a, ...]

1 ~ x = [ε] ~ [a, a, a, a, ...]
      = [ε + a]
      = [a]

_(1 ~ x) = a
```

The `*` operator is syntactic sugar; `a * b` is equivalent to `_(b ~ $a)`.
```
dq> 2 * 3
6
dq> 3 * 2
6
```
```
2 * 3 = _(2 ~ $3)
      = _([ε, ε] ~ [3, 3, ...])
      = _[ ε +     3    ,  ε +     3    ]
      = _[[] + [ε, ε, ε], [] + [ε, ε, ε]]
      = _[   [ε, ε, ε]  ,   [ε, ε, ε]   ]
      = [ε, ε, ε] + [ε, ε, ε]
      = [ε, ε, ε, ε, ε, ε]
      = 6

3 * 2 = _(3 ~ $2)
      = _([ε, ε, ε] ~ [2, 2, ...])
      = _[ ε +    2  ,  ε +    2  ,  ε +    2  ]
      = _[[] + [ε, ε], [] + [ε, ε], [] + [ε, ε]]
      = _[  [ε, ε]   ,   [ε, ε]   ,   [ε, ε]   ]
      = [ε, ε] + [ε, ε] + [ε, ε]
      = [ε, ε, ε, ε, ε, ε]
      = 6
```
By happy coincidence, multiplication between string and natural numbers works
the obvious way:
```
dq> "Helo" * 2
HeloHelo
dq> 2 * "Helo"
400
```

## Errors

Error messages are usually very helpful and sometimes very misleading.

**Note** that the interpreter is perfectly happy to let you reference a name
that's not been defined:
```
dq> printRepr wat
ε
```

## To-do

We’d like to add a “take” operator `^` as an abbreviation for the pattern we
demonstrated above.
```
^x = _(1~x)

^$x = x
```
