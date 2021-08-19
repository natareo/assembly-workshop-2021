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

```hoon
0b1101.1001
`@ud`0b1101.1001  :: yields 217
`@ux`0b1101.1001  :: yields 0xd9
```

> ##  Auras
>
> Try the following auras.  See if you can figure out how each one is behaving.
>
> ```hoon
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

```hoon
&((lth a b) (|((gte b c) d)))
```

The Hoon standard library, largely in defined in `%zuse`, further defines bitwise operations, arithmetic for both integers and floating-point values (half-width, single-precision, double-precision, and quadruple-precision), string operations, and more.

### Cells

All structures in Nock are binary trees; thus also Hoon.  This can occasionally lead to some awkward addressing when composing tetchy library code segments that need to interface with many different kinds of gates, but by and large is an extremely helpful discipline of thought.

TODO
binary tree format
flattened representation/convention

Hoon values are addressed as elements in a binary tree.

TODO

Finally, the most general mold is \texttt{*} which simply matches any noun—and thus anything in Hoon at all.


> ##  Hoon as Nock Macro
>
> The point of employing Hoon is, of course, that Hoon compiles to Nock.  Rather than even say _compile_, however, we should really just say Hoon is a _macro_ of Nock.  Each Hoon rune, data structure, and effect corresponds to a well-defined Nock primitive form.  We may say that Hoon is to Nock as C is to assembler, except that the Hoon-to-Nock transformation is completely specified and portable.  Hoon is ultimately defined in terms of Nock; many Hoon runes are defined in terms of other more fundamental Hoon runes, but all runes parse unambiguously to Nock expressions.
>
> Hoon expands on Nock primarily through the introduction of metadata
>
> Each Hoon rune has an unambiguous mapping to a Nock representation.  Furthermore, each rune has a well-defined binary tree structure and produces a similarly well-structured abstract syntax tree (AST).  As we systematically introduce runes, we will expand on what this means in each case:  for now, let's examine two runes without regard for their role.
>
> > `|=` bartis produces a _gate_ or function.  Every gate has the same shape, which means certain assumptions about data access and availability can be made.
> >
> > ```hoon
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

> ##  Aura Conversion Generator
> TODO
{: .exercise}


##  Some Data Structures

### Lists

TODO

A list is a binary tree which branches rightwards.

A _lest_ is a special case of list:  one guaranteed to be non-null.  (Certain operations require the stronger guarantee that a list has some content.)

### Text

Hoon recognizes two basic text types:  the _cord_ or `@t` and the `tape`.  Cords are single atoms containing the text as UTF-8 bytes interpreted as a single stacked number.  Tapes are lists of individual one-element cords.  (Lists are null-terminated, and thus so are tapes.)

Cords are useful as a compact primary storage and data transfer format, but frequently parsing and processing involves converting the text into tape format.  There are more utilities for handling tapes, as they are already broken up in a legible manner.

For instance, `++trip` converts a cord to a tape; `++crip` does the opposite.

```hoon
++  trip
  |=  a=@  ^-  tape
  ?:  =(0 (met 3 a))  ~
  [^-(@ta (end 3 1 a)) $(a (rsh 3 1 a))]
```

`++met`, `++end`, and `++rsh` are bitwise manipulation gates.

```hoon
++  crip  |=(a=tape `@t`(rap 3 a))}
```

`++rap` assembles the list interpreted as cords with block size of $2^3$ (in this case.

All text in Urbit is UTF-8 (and typically just 8-bit ASCII).  The `@c` UTF-32 aura is only used by the keyboard vane `%dill` and Hood (the Dojo terminal agent).

> ## Accessing String Elements
>
> A list is a binary tree which branches rightwards.  Thus a tape also is a binary tree.
>
> ![](TODO)
>
> There are two primary ways to access elements:
>
> - Direct indexing with `&` or `+`.
> - The `++snag` gate, `(snag 5 mytape)`.
>
> TODO
>
{: .exercise}

> ## Converting Between Cords and Tapes
>
> `++crip` and `++trip`
{: .exercise}

{% include links.md %}
