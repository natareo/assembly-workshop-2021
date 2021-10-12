---
title: "Hoon Basics:  Data Structures"
teaching: 20
exercises: 10
questions:
- "How does Hoon represent data values and structures?"
- "How can I compose and run multi-line programs?"
objectives:
- "Apply auras to transform an atom."
- "Identify common Hoon molds, such as lists and tapes."
- "Produce a generator to convert a value between auras."
- "Identify common Hoon patterns: atoms, cells, cores, faces, and traps."
keypoints:
- "All types in Urbit are unsigned integers `@ud`."
- "Molds provide a structured type system for Hoon."
- "All data in Hoon are binary trees; thus, common patterns arise again and again."
---

##  Nouns

All values in Urbit are _nouns_, meaning either atoms or cells.  An _atom_ is an unsigned integer.  A _cell_ is a pair of nouns.

### Atoms

An atom is an unsigned integer of any size.  (It may be fruitful for you to think of an atom as being like a Gödel number or a binary machine value.)  Since all values are ultimately (collections of) integers, we need a way to tell different "kinds" (or applications) of integers apart.

Atoms have tags called _auras_ which can be used coercively or noncoercively to represent single-valued data.  In other words, an aura is a bit of metadata Hoon attaches to a value which tells Urbit how you intend to use a number.  The default aura for a value is `@ud`, unsigned decimal, but of course there are many more.  Aura operations are extremely convenient for converting between representations.  They are also used to enforce type constraints on atoms in expressions and gates.

For instance, to a machine there is no fundamental difference between binary `0x11011001`, decimal `217`, and hexadecimal `0xD9`.  A human coder recognizes them as different encoding schemes and associates tacit information with each:  an assembler instruction, an integer value, a memory address.  Hoon offers two ways of designating values with auras:  either directly by the input formatting of the number (such as \texttt{0b1101.1001}) or using the irregular syntax ``@``:

```
0b1101.1001
`@ud`0b1101.1001  :: yields 217
`@ux`0b1101.1001  :: yields 0xd9
```

> ##  Auras
>
> Try the following auras.  See if you can figure out how each one is behaving.
>
> ```
> `@ud`0x1001.1111
> `@ub`0x1001.1111
> `@ux`0x1001.1111
> `@p`0x1001.1111
>
> `@ud`.1
> `@ux`.1
> `@ub`.1
> `@sb`.1
>
> `@p`0b1111.0000.1111.0000
>
> `@ta`'hello'
> `@ud`'hello'
> `@ux`'hello'
> `@uc`'hello'
> `@sb`'hello'
> `@rs`'hello'
> ```
{: .challenge}

The atom/aura system represents all simple data types in Hoon:  dates, floating-point numbers, text strings, Bitcoin addresses, and so forth.  Each value is represented in least-significant byte (LSB) order; for instance, a text string may be deconstructed as follows:

| Urbit |
| ----- |
| `'Urbit'` |
| `0b111.0100.0110.1001.0110.0010.0111.0010.0101.0101` |
| `0b111.0100` `0b110.1001` `0b110.0010` `0b111.0010` `0b101.0101` |
| `116` `105` `98` `114` `85` (ASCII characters) |
| `t` `i` `b` `r` `U` |

![](./img/atom-cord.png)

Note in the above that leading zeroes are always stripped.  Since each atom is an integer, there is no way to distinguish `0` from `00` from `000` etc.

In this vein, it's worth remembering that Dojo automatically parses any typed input and disallows invalid representations.  This can lead to confusion until you are accustomed to the type signatures; for instance, try to type `0b0001` into Dojo.

### Operators

Hoon has no primitive operators.  Instead, aura-specific functions or _gates_ are used to evaluate one or more atoms to produce basic arithmetic results.  Gate names are conventionally prefixed with `++` luslus which designates them as _arms_ of a _core_.  (More on this terminology in a future section.)  Some gates operate on any input atom auras, while others enforce strict requirements on the types they will accept.  Gates are commonly invoked using a Lisp-like syntax and a reverse-Polish notation (RPN), with the operator first followed by the first and second (and following) operands.

| Operation | Function | Example |
| --------- | -------- | ------- |
| Addition  | `++add`  | `(add 1 2)` → `3` |
| Subtraction | `++sub` | `(sub 4 3)` → `1` |
| Multiplication | `++mul` | `(mul 5 6)` → `30` |
| Division  | `++div`  | `(div 8 2)` → `4` |
| Modulus/Remainder | `++mod` | `(mod 12 7)` → `5` |

Following Nock's lead, Hoon uses loobeans (`0` = `true`) rather than booleans for logical operations.  Loobeans are written `%.y` for `true`, `0`, and `%.n` for `false`, `1`.

| Operation | Function | Example |
| --------- | -------- | ------- |
| Greater than, > | `++gth` | `(gth 5 6)` → `%.n` |
| Greater than or equal to, ≥ | `++gte` | `(gte 5 6)` → `%.n` |
| Less than, < | `++lth` | `(lth 5 6)` → `%.y` |
| Less than or equal to, ≤ | `++lte` | `(lte 5 6)` → `%.y` |
| Equals, = | `=` | `=(5 5)` → `%.y` |
| Logical `AND`, ∧ | `&` | `&(%.y %.n)` → `%.n` |
| Logical `OR`, ∨ | `|` | `|(%.y %.n)` → `%.y` |
| Logical `NOT`, ¬ | `!` | `!%.y` → `%.n` |

Since all operations are explicitly invoked Lisp-style within nested parentheses, there is no need for explicit operator precedence rules.

$$
(a < b) \land ((b \geq c) \lor d)
$$

```
&((lth a b) (|((gte b c) d)))
```

The Hoon standard library, largely in defined in `%zuse`, further defines bitwise operations, arithmetic for both integers and floating-point values (half-width, single-precision, double-precision, and quadruple-precision), string operations, and more.

### Cells

All structures in Nock are binary trees; thus also Hoon.  This can occasionally lead to some awkward addressing when composing tetchy library code segments that need to interface with many different kinds of gates, but by and large is an extremely helpful discipline of thought.

![](./img/binary-tree.png)

For instance, a list of characters (or `tape`, one of Hoon's string types) can be addressed directly by the rightward-branching cell addresses or via a convenience notation (which is more intuitive).  (We'll see more of this in a moment when we discuss text data structures.)

![](./img/binary-tree-tape.png)

```
> +1:"hello"
i="hello"
> +2:"hello"
i='h'
> +3:"hello"
t="ello"
> +4:"hello"
dojo: hoon expression failed
> +5:"hello"
dojo: hoon expression failed
> +6:"hello"
i='e'
> +7:"hello"
t="llo"
> +15:"hello"
t="l"
> +16:"hello"
t="lo"
> +30:"hello"
t="l"
> +31:"hello"
t="o"
> +62:"hello"
i='o'
> +63:"hello"
t=""
> +127:"hello"
dojo: hoon expression failed
>
>
>
> &1:"hello"
i='h'
> &2:"hello"
i='e'
> &3:"hello"
i='l'
> &4:"hello"
i='l'
> &5:"hello"
i='o'
```

> ## Cell Construction
>
> Produce a cell representation of the fruit tree.  You may use the following Unicode strings as `@t` `cord`s:
>
> ![](img/binary-tree-fruit.png)
>
> - '🍇'
> - '🍌'
> - '🍉'
> - '🍏'
> - '🍋'
> - '🍑'
> - '🍊'
> - '🍍'
> - '🍒'
>
> Use `~` as a placeholder for an empty node when necessary.
>
> > ### Solution
> >
> > ```py
> > [[[['🍏' ~] '🍇'] [[~ ['🍑' [~ '🍍']]] '🍌']] [['🍉' [~ '🍋']] [~ [~ ['🍊' ['🍒' ~]]]]]]
> > ```
> {: .solution}
{: .exercise}

More common than direct numerical addressing (of either form), however, is _lark addressing_, which is a quirky but legible shorthand.  The head and tail of each cell are selected by alternating `+`/`-` and `<`/`>` pairs, which is readable once you know what you're looking at.

```
> =hello "hello"
> -.hello
i='h'
> +.hello
t="ello"
> -<.hello
dojo: hoon expression failed
> ->.hello
dojo: hoon expression failed
> +<.hello
i='e'
> +>.hello
t="llo"
> +>-.hello
i='l'
> +>+.hello
t="lo"
> +>+<.hello
i='l'
> +>+>.hello
t="o"
> +>+>-.hello
i='o'
> +>+>+.hello
t=""
```

Finally, the most general mold is \texttt{*} which simply matches any noun—and thus anything in Hoon at all.  The `*` applied to a value yields the _bunt_, or default empty definition.

```
> *@ud
0
> *@ux
0x0
> *add
0
> *mul
1
```

> ##  Hoon as Nock Macro (Optional)
>
> The point of employing Hoon is, of course, that Hoon compiles to Nock.  Rather than even say _compile_, however, we should really just say Hoon is a _macro_ of Nock.  Each Hoon rune, data structure, and effect corresponds to a well-defined Nock primitive form.  We may say that Hoon is to Nock as C is to assembler, except that the Hoon-to-Nock transformation is completely specified and portable.  Hoon is ultimately defined in terms of Nock; many Hoon runes are defined in terms of other more fundamental Hoon runes, but all runes parse unambiguously to Nock expressions.
>
> Hoon also expands on Nock through the introduction of metadata, such as auras and core annotations.  The Hoon compiler enforces conventions to aid the programmer.
>
> Each Hoon rune has an unambiguous mapping to a Nock representation.  Furthermore, each rune has a well-defined binary tree structure and produces a similarly well-structured abstract syntax tree (AST).  As we systematically introduce runes, we will expand on what this means in each case:  for now, let's examine a rune and its Nock equivalent.
>
> > `|=` bartis produces a _gate_ or function.  Every gate has the same shape, which means certain assumptions about data access and availability can be made.
> >
> > ```
> > ::  XOR two binary atoms
> > |=  [a=@ub b=@ub]
> > `@ub`(mix a b)
> > ```
> >
> > maps to the Nock code
> >
> > ```nock
> > [8 [1 0 0] [1 8 [9 1.494 0 4.095] 9 2 10 [6 [0 28] 0 29] 0 2] 0 1]
> > ```
> {: .source}
>
>
> We call Hoon's data type specifications \emph{molds}.  Molds are more general than atoms and cells, but these form particular cases.  Hoon uses molds as a way of matching Nock tree structures (including Hoon metadata tags such as auras).
{: .callout}


##  Some Data Structures

### Lists

A list is a binary tree which branches rightwards.  The tape, which we saw earlier, is a special case of this applying to single characters.

A `lest` is a special case of list:  one guaranteed to be non-null.  (Certain operations require the stronger guarantee that a list has some content.)

### Text

Hoon recognizes two basic text types:  the _cord_ or `@t` and the `tape`.  Cords are single atoms containing the text as UTF-8 bytes interpreted as a single stacked number.  Tapes are lists of individual one-element cords.  (Lists are null-terminated, and thus so are tapes.)

For instance, a list of characters (or `tape`, one of Hoon's string types) can be addressed directly by the rightward-branching cell addresses or via a convenience notation (which is more intuitive).

![](./img/binary-tree-tape.png)

```
> +1:"hello"
i="hello"
> +2:"hello"
i='h'
> +3:"hello"
t="ello"
> +4:"hello"
dojo: hoon expression failed
> +5:"hello"
dojo: hoon expression failed
> +6:"hello"
i='e'
> +7:"hello"
t="llo"
> +15:"hello"
t="l"
> +16:"hello"
t="lo"
> +30:"hello"
t="l"
> +31:"hello"
t="o"
> +62:"hello"
i='o'
> +63:"hello"
t=""
> +127:"hello"
dojo: hoon expression failed
```

Lists have an additional way to grab an element:

- Sequential entry, `&` lets you grab the _n_th item of a list:  `&1:~['one' 2 .3.0 .~4.0 ~.5]`

```
> &1:"hello"
i='h'
> &2:"hello"
i='e'
> &3:"hello"
i='l'
> &4:"hello"
i='l'
> &5:"hello"
i='o'
```

In contrast to tapes, cords store the ASCII values in a single arbitrary integer in LSB order.

![](./img/atom-cord.png)

(More properly, it's done in UTF-8:  here's Cherokee.)

```
> 'ᏣᎳᎩ'
'ᏣᎳᎩ'
> `@ux`'ᏣᎳᎩ'
0xa9.8ee1.b38e.e1a3.8fe1
> `@ub`'ᏣᎳᎩ'
0b1010.1001.1000.1110.1110.0001.1011.0011.1000.1110.1110.0001.1010.0011.1000.1111.1110.0001
> :: 0xa9.8ee1  0xb3.8ee1  0xa3.8fe1
```

Cords are useful as a compact primary storage and data transfer format, but frequently parsing and processing involves converting the text into tape format.  There are more utilities for handling tapes, as they are already broken up in a legible manner.

For instance, `++trip` converts a cord to a tape; `++crip` does the opposite.

```
++  trip
  |=  a=@  ^-  tape
  ?:  =(0 (met 3 a))  ~
  [^-(@ta (end 3 1 a)) $(a (rsh 3 1 a))]
```

`++met`, `++end`, and `++rsh` are bitwise manipulation gates.  Basically they chop up a cord into $2^3 = 8$-bit slices as the elements of a list.

```
++  crip
  |=  a=tape  ^-  @t
  (rap 3 a)
```

`++rap` assembles the list interpreted as cords with block size of $2^3 = 8$ (in this case).

> ## Unicode in Urbit
>
> All text in Urbit is UTF-8 (and typically just 8-bit ASCII).  The `@c` UTF-32 aura is only used by the keyboard vane `%dill` and Hood (the Dojo terminal agent).
{: .callout}

> ## Sudan Function
>
> Implement the [Sudan function](https://en.wikipedia.org/wiki/Sudan_function) in Hoon.
>
> $$
> \begin{array}{lcl}
> F_0 (x, y) & = x+y \\
> F_{n+1} (x, 0) & = x & \text{if } n \ge 0 \\
> F_{n+1} (x, y+1) & = F_n (F_{n+1} (x, y), F_{n+1} (x, y) + y + 1) & \text{if } n\ge 0 \\
> \end{array}
> $$
>
> > ```
> > ++  sudan
> >   |=  [n=@ud x=@ud y=@ud]  ^-  @ud
> >   ?:  =(n 0)  (add x y)
> >   ?:  =(y 0)  x
> >   $(n (dec n), x $(n n, x x, y (dec y)), y (add y $(n n, x x, y (dec y))))
> > ```
> {: .solution}
{: .challenge}

{% include links.md %}
