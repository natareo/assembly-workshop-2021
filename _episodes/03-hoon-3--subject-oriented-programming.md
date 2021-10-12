---
title: "Hoon Basics:  Subject-Oriented Programming"
teaching: 30
exercises: 15
questions:
- "How are Hoon programs structured?"
- "What are runes?"
objectives:
- "Explain what a “subject-oriented language” means."
- "Identify common Hoon patterns: batteries, and doors, arms, wings, and legs."
- "Create a `%say` generator."
- "Review known runes in context of highest-frequency, highest-impact runes."
keypoints:
- "Generators are Dojo's way of importing simple standalone programs."
- "The subject consists of the context and arguments to a program."
---

##  Subject-Oriented Programming

Urbit adopts an innovative programming paradigm called “subject-oriented programming.”  By and large, Hoon (and Nock) is a functional programming language in that
However, Hoon also very carefully bounds the known context of any part of the program as the _subject_.  Basically, the subject is the noun against which any arbitrary Hoon code is evaluated.

For instance, when we first composed generators, we made what are called “naked generators”:  that is, they do not have access to any information outside of the base subject (Arvo, Hoon, and `%zuse`) and their sample (arguments).  Other generators (such as `%say` generators, described below) can have more contextual information, including random number generators and optional arguments, passed to them to form part of their subject.

Hoon developers frequently talk about “limbs” of the subject.  Arms describe known labeled address (with `++` luslus) which carry out computations.  Legs are limbs which store data.

![](https://davis68.github.io/martian-computing/img/08-nubret.png)

> ## Addressing Redux
>
> Labelled arms are invoked as e.g. gates on data.  However, if you know the address of a particular value in a limb, you can retrieve it directly using either notation:
>
> - Numbered limb, `+`/`:` lets you grab a specific numbered arm:  `+7:[1 2 3]`
> - Lark notation, `+`/`-`/`<`/`>` lets you select by relative address:  `+>:[1 2 3]`
>
> (One challenge of navigation in the current Dojo is that Urbit developer tools return details like hashes of arms rather than supervening labels.)
{: .callout}

> ## Addressing the Fruit Tree
>
> Produce the numeric and lark-notated equivalent addresses for each of the following nodes in the binary fruit tree:
>
> ![A fruit tree](https://raw.githubusercontent.com/natareo/assembly-workshop-2021/gh-pages/img/binary-tree-fruit.png)
>
> - 🍇
> - 🍌
> - 🍉
> - 🍏
> - 🍋
> - 🍑
> - 🍊
> - 🍍
> - 🍒
>
> > ### Solution
> >
> > - 🍇 `9` or `-<+`
> > - 🍌 `11` or `->+`
> > - 🍉 `12` or `+<-`
> > - 🍏 `16` or `-<-<`
> > - 🍋 `27` or `+<+>`
> > - 🍑 `42` or `->->-`
> > - 🍊 `62` or `+>+>-`
> > - 🍍 `87` or `->->+>`  # heuristic for these mathematically
> > - 🍒 `126` or `+>+>+-`
> {: .solution}
{: .exercise}

- [“The Subject and its Legs”](https://urbit.org/docs/hoon/hoon-school/the-subject-and-its-legs)


##  Cores and Derived Structures

### Cores

The core is the primary nontrivial data structure of Nock:  atoms, cells, cores.  A core is defined as a cell of `[battery payload]`; in the abstract this simply divides the `battery` or code from the `payload` or data.  Cores can be thought of as similar to objects in object-oriented programming languages, but possess a completely standard structure which allows for detailed introspection and “hot-swapping” of core elements.  Everything in standard Hoon and Arvo that cannot be reduced to an atom or a cell is _de facto_ a core.  (Indeed, if one wished to separate code and data in a binary tree structure, the only other logical choice one would have available is to flip the order of `battery` and `payload`.)

In short a `battery` is a collection of arms and a payload is a collection of data, possibly from various sources.  (At this point the “leg” terminology breaks down a bit, and in practice mostly people talk about “arms.”)

Conventionally, most cores are either produced by the `|%` barcen rune or are instances of a more complex form (such as a door).  Cores are a “live” type; they are not simply holders of data but are expected to operate on data, just as it is uncommon to see a C++ object which only holds data.  (The role of a C `struct` is approximated by the `$%` buccen tagged union.)

Some terminology is in order, to be expanded on subsequently:

- An [_arm_](https://urbit.org/docs/glossary/arm) is a Hoon expression to be evaluated against the core subject (i.e. its parent core is its subject).
- A [_leg_](https://urbit.org/docs/hoon/hoon-school/the-subject-and-its-legs) is a
- A [_limb_](https://urbit.org/docs/hoon/reference/limbs/limb) is either an arm or a leg:  formally, “an attribute of a subject.”  A limb is resolved with
- A [_wing_](https://urbit.org/docs/hoon/reference/limbs/wing) is a resolution path pointing to a limb.  It's a search path, like ``

Altogether, this yields a rich introspective framework for accessing and manipulating cores.  We won't do a lot with it, but if you are interested, look up core genericity and variadicity in the docs.

> ## Shadowing Names (Optional)
>
> In any programming paradigm, good names are valuable and collisions are likely.  In Hoon, if you need to catch an outer-context label that has the same name as an inner-context value, use `^` ket to skip the depth-first match.
>
> ```
> ^^json
> ```
{: .callout}

> ## Limb Resolution Paths
>
> While [the docs on limbs](https://urbit.org/docs/hoon/reference/limbs/limb) contain a wealth of information on how limbs are resolved by the Hoon compiler, it is worth addressing in brief the two common resolution tools you will encounter today:  `.` dot and `:` col.
>
> - `.` dot resolves the wing path into the current subject.
> - `:` col resolves the wing path with the right-hand-side as the subject.
{: .callout}

> ## Operators (Optional)
>
> There's a lot going on with addressing.  In practice, it's typically readable but subtleties abound.
>
> - `+` lus is the _slot operator_.
> - `&` pam is the _head_ of a cell.
> - `|` bar is the _tail_ of a cell.
> - `.` dot is the wing resolution path into the subject (using Nock Zero).
>
>   `=/(foo [a=3 b=4] b.foo)` is `4`, because one adds `foo=[a=3 b=4]` to the subject.
>
> - `:` col is the right-associative search path into the right-hand side _as_ subject (using Nock Seven).  (This is also the irregular syntax of [`+<` tisgal](https://urbit.org/docs/hoon/reference/rune/tis#-tisgal).)
>
>    `=/(foo [a=3 b=4] b:foo)` is `4`, because one computes the wing `foo` against the subject beneath the `=/` and then using that as the subject for the computing the wing `b`.
>
> - `+` lus/`-` hep/`<` gal/`>` gar are the lark notation symbols.
{: .callout}

- [“Arms and Cores”](https://urbit.org/docs/hoon/hoon-school/arms-and-cores)

### Traps

The [trap](https://urbit.org/docs/glossary/trap) creates the basic looping mechanism in Hoon, a special instance of a core which is capable of concisely recursing into itself.  The trap is formally a core with one arm named `$` buc, and it is either created with `|-` barhep (for instant evaluation) or `|.` bardot (for deferred evaluation).

In practice, traps are used almost exclusively as recursion points, much like loops in an imperative language.  For instance, the following program counts from 5 down to 1, emitting output via `~&` sigpam at each iteration, then return `~` sig.

```
=/  count  5
|-
  ?:  =(count 0)  ~
~&  count
$(count (dec count))
```

The final line `$(count (dec count))` serves to modify the subject (at `count`) then to pull the `$` buc arm again.  In practice the tree unrolls as follows, with indentation indicating code running “inside” of another rune.


### Gates

Similar to how a trap is a core with one arm, a gate is a core with one arm and a sample.  This means that new per-invocation data are available in the gate's subject.

A gate is a core `[battery payload]` with one arm `$` buc in the battery and a `payload` consisting of `[sample context]`; thus, `[$ [sample context]]`.  We've encountered gates in generators before (which Dojo knows how to connect), but very frequently they are implemented as arms in a larger core.

When you invoke a gate using `%-` cenhep (what is the irregular form?), the `$` buc arm is pulled and the `sample` is passed in to it.

Gates—and other cores—have a standard structure, which renders them valid objects of introspection.  Indeed, Lisp-style brain surgery on cores is not uncommon, although we will by and large not need it in this workshop.

```
$:add
$:mul
$:arch
```

A trap isn't the only way to recurse in Hoon.  In fact, one sees the default `$` buc arm invoked directly in many cases to recalculate an entire gate recursively.  Consider, for instance, this implementation of the factorial:

```
|=  n=@ud
?:  =(n 1)
  1
(mul n $(n (dec n)))
```

In this case, the `$` buc arm is directly invoked in the `++mul` itself.  When the `$()` recursion point is reached, the entire `$` buc arm is re-evaluated with the specified change of `n` → `n-1`.

That is, when one reads this program, one reads it falling into two components:

```
|=  n=@ud             :: accept a single value n for n!
?:  =(n 1)  1         :: check n ≟ 1; if so, return 1
(mul n $(n (dec n)))  :: multiply n times the product of this arm w/ n-1
```

- [“Gates”](https://urbit.org/docs/hoon/hoon-school/gates)

### Doors

A gate is a particular instance of a core which is “automatically executable.”  A door is a more general instance of a gate:  really, it's a gatemaker.  It produces gates as necessary.

When using a gate, the calling convention replaces the `sample` then pulls the `$` buc arm.  In contrast, when using a door, the calling convention replaces the `sample` then pulls which arm has been requested.

For instance, the random number generator uses the system entropy `eny` to produce a random number in the range 1–100:

```
(~(rad og eny) 1.000)
```

To build a door, use the `|_` barcab rune to produce a core with multiple arms (and no default `$` buc arm).  A running program agent is a door, so we will work more with these incidentally tomorrow.

This is part of the digit-to-cord display door:

```
++  ne
  |_  tig=@
  ++  d  (add tig '0')
  ++  x  ?:((gte tig 10) (add tig 87) d)
  ++  v  ?:((gte tig 10) (add tig 87) d)
  ++  w  ?:(=(tig 63) '~' ?:(=(tig 62) '-' ?:((gte tig 36) (add tig 29) x)))
  --
```

`++ne` is used as follows:

```
`@t`~(d ne 7)   :: decimal digit as cord
`@t`~(x ne 14)  :: hexadecimal digit as cord
`@t`~(v ne 25)  :: base-32 digit as cord
`@t`~(w ne 52)  :: base-64 digit as cord
```

> ## Custom Types
>
> Doors and other cores frequently include custom type definitions; these are discussed below in “Molds”.
{: .callout}

> ## Manipulating the Sample Directly (Optional)
>
> Since `.` dot refers to the subject, this yields the ability to manipulate the sample without calling any arm directly.  The following examples illustrate:
>
> ```
> (add [1 5])               :: call a gate
> 6
> ~($ add [1 5])            :: call the $ arm in the door
> 6
> ~(. add [1 5])            :: update the sample (arguments) but don't evaluate
> <1.otf [[@ud @ud] <45.xig 1.pnw %140>]>
> =<  $  ~(. add [1 5])     :: call the $ arm of the door with updated sample
> 6
> =/  addd  ~(. add [1 5])  :: do the same thing but in two steps
>   =<  $  addd
> 6
> ```
{: .callout}

> ## Code as Specification (Optional)
>
> At this point, we need to step back and contextualize the power afforded by the use of cores.  In another language, such as C or Python, we specify a behavior but have relatively little insight into the instantiation effected by the compiler.
>
> Python as written in the interpreter:
>
> ```py
> metals = ['gold', 'iron', 'lead', 'zinc']
> total = 0
> for metal in metals:
>     total = total + len(metal)
> ```
>
> Python as compiled bytecode:
>
> ```asm
> 1          0 LOAD_CONST               0 ('gold')
>            2 LOAD_CONST               1 ('iron')
>            4 LOAD_CONST               2 ('lead')
>            6 LOAD_CONST               3 ('zinc')
>            8 BUILD_LIST               4
>           10 STORE_NAME               0 (metals)
>
> 2         12 LOAD_CONST               4 (0)
>           14 STORE_NAME               1 (total)
>
> 3         16 LOAD_NAME                0 (metals)
>           18 GET_ITER
>        20 FOR_ITER                16 (to 38)
>           22 STORE_NAME               2 (metal)
>
> 4         24 LOAD_NAME                1 (total)
>           26 LOAD_NAME                3 (len)
>           28 LOAD_NAME                2 (metal)
>           30 CALL_FUNCTION            1
>           32 BINARY_ADD
>           34 STORE_NAME               1 (total)
>           36 JUMP_ABSOLUTE           20
>        38 LOAD_CONST               5 (None)
>           40 RETURN_VALUE
> ```
>
> Many popular programming languages specify _behavior_ rather than _implementation_.  By specifying Hoon as a macro language over Nock, the Urbit developers collapse this distinction.
>
> For Hoon (like Lisp), the structure of execution is always front-and-center, or at least only thinly disguised.  When one creates a core with a given behavior, one can immediately envision the shape of the underlying Nock.  This affords one immense power to craft efficient, effective, graceful programs.
>
> ```
> =/  metals  `(list tape)`~["gold" "iron" "lead" "zinc"]
> |=  [metals=(list tape)]
> =/  count  0
> =/  total  0
> |-
>   ?:  =(count (lent metals))  total
>   =/  metal  `tape`(snag count metals)
> $(total (add total (lent metal)), count +(count))
> ```
>
> ```
> !=  |=  [metals=(list tape)]  =/  count  0  =/  total  0  |-  ?:  =(count (lent metals))  total  =/  metal  `tape`(snag count metals)  $(total (add total (lent metal)), count +(count))
> [ 8
>   [1 0]
>   [ 1
>     8
>     [1 0]
>     8
>     [1 0]
>     8
>     [ 1
>       6
>       [5 [0 14] 8 [9 343 0 32.767] 9 2 10 [6 0 126] 0 2]
>       [0 6]
>       8
>       [8 [9 20 0 32.767] 9 2 10 [6 [0 30] 0 126] 0 2]
>       9
>       2
>       10
>       [14 4 0 30]
>       10
>       [ 6
>         8
>         [9 36 0 131.071]
>         9
>         2
>         10
>         [6 [0 30] 7 [0 3] 8 [9 343 0 65.535] 9 2 10 [6 0 6] 0 2]
>         0
>         2
>       ]
>       0
>       3
>     ]
>     9
>     2
>     0
>     1
>   ]
>   0
>   1
> ]
> ```


##  Molds

Classically, computation was unified by an early and elegant conception of code-as-data, particularly in Lisp but hearkening back to Gödel.  Nock knows only about unsigned integers and mechanically applies rules over structures consisting only of unsigned integer values and cells (together, “nouns”).  However, for human-legibility we would like more sophisticated type annotation, whether coercive or purely in metadata.

The aura is a particular example of a _mold_, the type enforcement mechanism in Hoon.  A mold is a specific type definition, customarily defined with a `|%` core.  We commonly see three runes supporting this structure:

- `+$` lusbuc creates a type constructor arm to define and validate type definitions.
- `$%` buccen creates a collection of named values (type members).
- `$=` bucwut defines a union, a set validating membership across a defined collection of items.  (This is similar to a `typedef` or `enum` in C-related languages.)

To illustrate these, we consider several ways to define a vehicle.  In the first, we employ only `+$` lusbuc to capture key vehicle characteristics.  Using only lusbuc, it's hard to say much of interest:

```
+$  vehicle  tape               :: vehicle identification number
```

By permitting collections of named type values with `$%` buccen, we can produce more complicated structures:

```
+$  vehicle
  $%  vin=tape                  :: vehicle identification number
      owner=tape                :: car owner's name
      license=tape              :: license plate
  ==
```

Type definition arms can rely on other type definition arms available in the subject:

```
+$  vehicle
  $%  vin=tape                  :: vehicle identification number
      owner=tape                :: car owner's name
      license=tape              :: license plate
      kind=kind                 :: vehicle manufacture details
  ==
+$  kind
  $%  make=tape                 :: vehicle make
      model=tape                :: vehicle model
      year=@da                  :: nominal year of manufacture (use Jan 1)
  ==
```

> ## Masking Variables
>
> In general, Hoon style does not require you to be careful about masking variable names in the subject (using the same name for the value as the mold).  This rarely introduces surprising bugs but is typically contextually apparent to the developer.  Check the note on “Shadowing Variables” above for more details.
{: .callout}

Finally, by introducing unions with `$?` bucwut, a type definition arm can validate possible values:

```
+$  vehicle
  $%  vin=tape                  :: vehicle identification number
      owner=tape                :: car owner's name
      license=tape              :: license plate
      kind=kind                 :: vehicle manufacture details
  ==
+$  kind
  $%  make=tape                 :: vehicle make
      model=tape                :: vehicle model
      year=@da                  :: nominal year of manufacture (use Jan 1)
  ==
+$  make                        :: permitted vehicle makes
  ?(%acura %chrysler %delorean %dodge %jeep %tesla %toyota)
```

`@tas`-tagged text elements are extremely common in such type unions, as they afford a human-legible categorization that is nonetheless rigorous to the machine.  (This is a like a `typedef` and constant combined, in that it has only the types in the union.)


##  Generators

Generators are standalone Hoon expressions that evaluate and may produce side effects, as appropriate.  They are closely analogous to simple scripts in languages such as Bash or Python.  By using generators, one is able to develop more involved Hoon code and run it repeatedly without awkwardness.  Put another way, a generator is a nonpersistent computation:  it maps an input to an output.

(You will also see commands beginning with a `|` symbol; these are `%hood` commands instead, the CLI agent for Dojo.)

To run a generator on a ship, prefix its name with `+` lus.  Arguments may be required or optional.

```
+trouble
```

```
:: Only for a real ship.
+moon
+moon ~rinset-lapter-sampel-palnet
```

### Naked Generators

The simplest generator is a simple map of input to output without even a broader subject.  We've used these already, as with `+fib`.

A naked generator is so called because it contains no metadata for the Arvo interpreter.  Its subject is simply the standard Arvo/`%zuse`/Hoon stack, and its sample is a simple single noun.  (Since a noun can be a cell, you can sneak in more than one argument.)  Naked generators are nonpersistent computations, thus naked generators are typically straightforward calculators or system queries.

### `%say` Generators

More interesting for most cases are `%say` generators, which can include more information in their sample.  (Dojo knows how to handle these as standard cases because they are tagged with `%say` in the return cell.)

`%say` generators do know about Arvo and the subject and are able to leverage information from and about the operating system in performing their calculations.

A basic `%say` generator looks like this:

```
:-  %say
|=  *
:-  %noun
(sub 1.000 1)
```

- `:-` composes a cell.
- `%` in front of text indicates a `@tas`-style constant.  Here, this is a type annotation for the handler evaluating the generator.
- `*` is a mold matching any data type, atom or cell.  Since the sample is unused, there's no point in restricting it.

This generator can accept any input (`*`) or none at all.  It returns, in any case, `999`.

To match a particular mold, you can specify from this table, with atoms expanding to the right as auras.

| Shorthand | Mold |
| --------- | ---- |
|    `*`    | noun |
|    `@`    | atom |
|    `^`    | cell |
|    `?`    | loobean |
|    `~`    | null |

The generator itself consists of a cell `[%say hoon]`, where `hoon` is the rest of the code.  The `%say` metadata tag indicates to Arvo what the expected structure of the generator is _qua_ `%say` generator.

In general, a `%say` generator doesn't need a sample (input arguments) to complete:  Arvo can elide that if necessary.  More generally, though, a `%say` generator is useful any time a calculation needs to depend on user input or system parameters (beyond the static system library).

The maximalist `sample` is a 3-tuple:  `[[now eny beak] ~[unnamed arguments] ~[named arguments]]`.

> `now` is the current time.  `eny` is 512 bits of entropy for seeding random number generators.  `beak` contains the current ship, desk, and case.

How do we leave things out?

> Any of those pieces of data could be omitted by replacing part of the noun with * rather than giving them faces. For example, `[now=@da * bec=beak]` if we didn't want eny, or `[* * bec=beak]` if we only wanted `beak`.

### Now

In Dojo, you can always produce the current time as an atom using `now`.  This is a Dojo convenience, however, and we need to bind `now` to a face if we want to use it inside of a generator.

Time in Urbit will be covered in Zuse.

### Entropy

![](repo:./img/07-header-apollo-2.png){: width=100%}

What is _entropy_?  [Computer entropy](https://en.wikipedia.org/wiki/Entropy_%28computing%29) is a hardware or behavior-based collection of device-independent randomness.  For instance, "The Linux kernel generates entropy from keyboard timings, mouse movements, and IDE timings and makes the random character data available to other operating system processes through the special files `/dev/random` and `/dev/urandom`."

For instance, run `cat /dev/random` on a Linux box and observe the output.  You'll need to run `Ctrl`+`C` to exit to the prompt.  Run it again, and again.  You'll see that the store of entropy diminishes rather quickly because it is thrown away once it is used.

(And you thought that random number generators just used the time as a seed!)

### Beak

>Paths begin with a piece of data called a `beak`. A beak is formally a `(p=ship q=desk r=case)`; it has three components, and might look like `/~dozbud-namsep/home/11`.

You can get this information in Dojo by typing `%`.

##  Other Arguments

The full sample prototype for a `%say` generator looks like `[[now, eny, beak] [unnamed arguments] [named arguments]]`.

You see a similar pattern in languages like Python, which permits (required) unnamed arguments before named "keyword arguments".

### Unnamed Arguments

By "unnamed" arguments, we really mean _required_ arguments; that is, arguments without defaults.  We stub out information we don't want with the empty noun `*`:

```
|=  [* [a=@ud b=@ud c=@ud ~] ~]
(add (mul a b) c)
```

(You can use this in Dojo as well:

```
=f |=  [* [a=@ud b=@ud c=@ud ~] ~]
(add (mul a b) c)
(f [* ~[1 2 3] ~])
```

.)

Note that we retain the terminating `~` since the expected sample is a list.

### Named Arguments

We can incorporate optional arguments although without default values (i.e., the default value is always type-appropriate `~`).

```
|=  [* ~ [val=@ud ~]]
(add val 2)
```

To use it (saved as `gen/gate.hoon` and `|commit`ed):

```
+g =val 4
```

Since the default value is `~`, if you are testing for the presence of named arguments you should test against that value.

Note that, in all of these cases, you are writing a gate `|=` bartis which accepts `[* * ~]` or the like as sample.  Dojo (and Arvo generally) recognizes that `%say` generators have a special format and parse the command-line form into appropriate form for the gate itself.

- Reading: [Tlon Corporation, "Generators"](https://urbit.org/docs/tutorials/hoon/generators/), sections "%say Generators", "%say generators with arguments", "Arguments without a cell"

### Worked Examples

> ##  Rolling Dice
>
> Write a \say~generator which simulates scoring a simple dice throw of $n$ six-sided dice.  That is, it should return the sum of $n$ dice as inputs.  You may reuse code from `mp0` or you may use entropy following the discussion in []().  If no number is specified, then only one die roll should be returned.
>
> Since Hoon is functional but random number generators stateful, you should use the \ptisket~rune to replace the current value in the RNG.  \tisket~is a kind of "one-effect monad," which allows you to change a single part of the subject.
>
> For instance, here is a generator that returns a list of _n_ probabilities between 0–100%.
>
> ```
> :-  %say
> |=  [[* eny=@uv *] [n=@ud ~] ~]
>   :-  %noun
>   =/  values  `(list @ud)`~
>   =/  count  0
>   =/  rng  ~(. og eny)
>   |-  ^-  (list @ud)
>     ?:  =(count n)  values
>     =^  r  rng  (rads:rng 100)
>   $(count +(count), values (weld values ~[(add r 1)]))
> ```
>
> Your command to run this generator in the Dojo should look like this:
>
> ```
> +dicethrow, =n 5
> ```
>
> (Note the comma separating optional arguments.)
>
> - **Stretch goal**:  Modify this to reproduce the behavior of `n` 6-sided dice added together.
{: .exercise}

> ## Prime Sieve
>
> The Sieve of Eratosthenes is a classic (if relatively inefficient) way to produce a list of prime numbers.  Save this as a file `gen/primes.hoon`, sync it, and run it as `+primes 100`.  (Be careful not to use too large a number—use `Ctrl`+`C` to interrupt evaluation!)
>
> ```
> :-  %say
> |=  [[* eny=@uv *] [n=@ud ~] ~]
> :-  %noun
> =<
> (siev n)
> |%
> ::
> :: Decompose into prime factors in ascending order.
> ::
> ++  prime-factors
>   |=  n=@ud
>   %-  sorter
>   ?:  =(n 1)  ~[n 1]
>   =+  [i=0 m=n primes=(primes-to-n n) factors=*(list @ud)]
>   |-  ^+  factors
>   ?:  =(i (lent primes))
>     [factors]
>   ?:  =(0 (mod m (snag i primes)))
>     $(factors [`@ud`(snag i primes) factors], m (div m (snag i primes)), i i)
>   $(factors factors, m m, i +(i))
> ::
> :: Find prime factors in ascending order.
> ::
> ++  primes-to-n
>   |=  n=@ud
>   %-  dezero
>   ?:  =(n 1)  ~[~]
>   ?:  =(n 2)  ~[2]
>   ?:  =(n 3)  ~[3]
>   =+  [i=0 cands=(siev (div n 2)) factors=*(list @ud)]
>   |-  ^+  factors
>   ?:  =(i (lent cands))
>     ?:  =(0 (lent (dezero factors)))
>       ~[n]
>     factors
>   $(factors [`@ud`(filter cands n i) factors], i +(i))
> ::
> :: Strip off matching modulo-zero components, (mod n factor)
> ::
> ++  filter
>   |*  [cands=(list) n=@ud i=@ud]
>   ?:  =((mod n (snag i `(list @ud)`cands)) 0)
>     [(snag i `(list @ud)`cands)]
>   ~
> ::  Find primes by the sieve of Eratosthenes
> ++  siev
>   |=  n=@ud
>   %-  dezero
>   =+  [i=2 end=n primes=(gulf 2 n)]
>   |-  ^+  primes
>   ?:  (gth i n)
>     [primes]
>   $(primes [(clear (sub i 2) i primes)], i +(i))
> :: wrapper to remove zeroes after sorting
> ++  dezero
>   |=  seq=(list @)
>   =+  [ser=(sort seq lth)]
>   `(list @)`(skim `(list @)`ser pos)
> ++  pos
>   |=  a=@
>   (gth a 0)
> :: wrapper sort---does NOT remove duplicates
> ++  sorter
>   |=  seq=(list @)
>   (sort seq lth)
> :: replace element of c at index a with item b
> ++  nick
>   |*  [[a=@ b=*] c=(list @)]
>   (weld (scag a c) [b (slag +(a) c)])
> :: zero out elements of c starting at a modulo b (but omitting a)
> ++  clear
>   |*  [a=@ud b=@ud c=(list)]
>   =+  [j=(add a b) jay=(lent c)]
>   |-  ^+  c
>   ?:  (gth j jay)
>     [c]
>   $(c [(nick [j 0] c)], j (add j b))
> --
> ```
{: .exercise}

> ## Documentation Examples
>
> The following Hoon Workbook examples walk you line-by-line through several `%say` generators of increasing complexity.
>
> The traffic light example is furthermore an excellent prelude to our entrée to Gall.
>
> - Reading: [Tlon Corporation, "Hoon Workbook:  Digits"](https://urbit.org/docs/tutorials/hoon/workbook/digits/)
> - Reading: [Tlon Corporation, "Hoon Workbook:  Magic 8-Ball"](https://urbit.org/docs/tutorials/hoon/workbook/eightball/)
> - Reading: [Tlon Corporation, "Hoon Workbook:  Traffic Light"](https://urbit.org/docs/tutorials/hoon/workbook/traffic-light/)
{: .exercise}

> ## `%ask` Generators (Optional Content)
>
> `%ask` generators assume some interactivity with the user.  They are less commonly encountered in Arvo since at this point many developers prefer to write whole `%gall` apps.
>
> For instance, here is the generator to retrieve your `+code` for web login.  (At this point, focus on the _structure_ not the _content_ of this generator.)
>
> ```
> ::  Helm: query or reset login code for web
> ::
> ::::  /hoon/code/hood/gen
>   ::
> /?    310
> /-  *sole
> /+  *generators
> :-  %ask
> |=  $:  [now=@da eny=@uvJ bec=beak]
>         [arg=?(~ [%reset ~]) ~]
>     ==
> =*  our  p.bec
> ^-  (sole-result [%helm-code ?(~ %reset)])
> ?~  arg
>   =/  code=tape
>     %+  slag  1
>     %+  scow  %p
>     .^(@p %j /(scot %p our)/code/(scot %da now)/(scot %p our))
>   =/  step=tape
>     %+  scow  %ud
>     .^(@ud %j /(scot %p our)/step/(scot %da now)/(scot %p our))
>   ::
>   %+  print  'use |code %reset to invalidate this and generate a new code'
>   %+  print  leaf+(weld "current step=" step)
>   %+  print  leaf+code
>   (produce [%helm-code ~])
> ::
> ?>  =(%reset -.arg)
> %+  print  'continue?'
> %+  print  'warning: resetting your code closes all web sessions'
> %+  prompt
>   [%& %project "y/n: "]
> %+  parse
>   ;~  pose
>     (cold %.y (mask "yY"))
>     (cold %.n (mask "nN"))
>   ==
> |=  reset=?
> ?.  reset
>   no-product
> (produce [%helm-code %reset])
> ```
>
> Look for the following:
>
> - `/?` Kelvin version pin
> - `?~` null check
> - `?>` assertion
> - `.^` vane call to `%j` = `%jael`
>
> `%helm` is the CLI app.  `%sole` is a CLI library.
{. :callout}

{% include links.md %}
