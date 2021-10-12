---
title: "Hoon Basics:  Advanced Data Structures"
teaching: 30
exercises: 15
questions:
- "How are structured data typically represented, populated, and accessed in Hoon?"
- "Which common patterns in Hoon exist?"
objectives:
- "Identify common Hoon molds:  units, sets, maps, jars, and jugs."
- "Explore the standard library’s functionality:  text parsing & processing, functional hacks, randomness, hashing, time, and so forth."
keypoints:
- "Doors generalize the notion of a gate to a gate-factory."
- "Units, sets, maps, jars, and jugs provide a principled way to access and operate on data."
---

##  Data Structures

Urbit employs a series of data structures (molds) which are either unique to the Urbit operating environment (such as a unit) or analogous to convenient forms in other languages (like a set or a map).

### Units

Every atom in Hoon is an unsigned integer, even if interpreted by an aura.  Auras do not carry prescriptive power, however, so they cannot be used to fundamentally distinguish a \texttt{NULL}-style or \texttt{NaN}-style non-result.  That is, if the result of a query is a `~`, how does one distinguish a value of zero from a non-result (missing value)?

Units mitigate the situation by acting as a type union of `~` (for no result) and a cell `[~ u=item]` containing the returned item (with face `u`).

```
++  unit
  |$  [item]
  $@(~ [~ u=item])
```

Many lookup operations return a unit precisely so you can differentiate a failed search from an answer of zero.

### Sets

Many compound data structures in Hoon are trees, maps, or sets.  Trees are the least important for these for our purposes today, so we will focus on `+$map` and `+$set`.

In computer science, a set is like a mathematical set with single-membership of values.  A set is not ordered so the results may not be ordered or stored the same as the input.

```
> (sy ~[5 4 3 2 1])
[n=5 l={} r={1 2 3 4}]
> (sy ~[8 8 8 7 7 6 5 5 4 3 2 1])
[n=6 l={8 5 7} r={1 2 3 4}]
```

For the most part, we don't worry too much about the particular structure chosen for internal representation of the set as a tree because we will use the `++in` door for interacting with elements.

A door is a kind of core more general than a gate.  In particular, rather than a default arm `$` the caller must specify which arm of the door to pull.  This produces a gate which must then be slammed on the values.  The `++in` core instruments `+$set` instances, which makes for a somewhat intuitive English-language reading of operations.

To produce a set, you can either construct it manually from a list using `++sy`, or you can insert items using the `++put` arm of `by`.

For each of the following, we assume the following set has been defined in Dojo:

```
=my-set (sy ~['a' 'B' "sea" 4 .5])
```

- Put a value in the set:

    ```
    > (~(put in my-set) ~.6)
    [n=.7.6e-44 l=[n=.5 l={} r={}] r=[n=.9.2e-44 l={} r={[i='s' t="ea"] 'a' '\04'}]]
    ```

    (Notice, in the time-honored tradition of functional languages, that `my-set` is unaltered and a copy of the values is returned.)

- Query if value is in set:

    ```
    > (~(has in my-set) 'B')
    %.y
    > (~(has in my-set) 'b')
    %.n
    ```

- Get size of set:

    ```
    > ~(wyt in my-set)
    5
    ```

- Apply gate to values:

    ```
    :: strip auras from all elements in set
    > (~(run in my-set) |=(a=* a))
    {1.084.227.584 66 [115 101 97 0] 97 4}
    ```

- Convert to list:

    ```
    > ~(tap in my-set)
    ~[.6e-45 .1.36e-43 [i='s' t="ea"] .9.2e-44 .5]
    ```

There are also a number of set-theoretical operations such as difference (`dif`), intersection (`int`), union (`uni`), etc.

### Maps

A `+$map` is a set of key–value pairs:  `(tree (pair key value))`.  (Other languages know this as a dictionary or associative array.)  The `++by` core provides map services.

To produce a map, you can either construct it manually from cells using `++my`, or you can insert items using the `++put` or `++gas` arms of `by`.  The latter is more robust and we use it here.

Map keys are commonly `@tas` words.

For each of the following, we assume the following set has been defined in Dojo:

```
=greek (~(gas by *(map @tas @t)) ~[[%alpha 'α'] [%beta 'β'] [%gamma 'γ'] [%delta 'δ'] [%epsilon 'ε'] [%zeta 'ζ'] [%eta 'η'] [%theta 'θ'] [%iota 'ι'] [%kappa 'κ'] [%lambda 'λ'] [%mu 'μ'] [%nu 'ν'] [%xi 'ξ'] [%omicron 'ο'] [%pi 'π'] [%rho 'ρ'] [%sigma 'σ'] [%tau 'τ'] [%upsilon 'υ'] [%phi 'φ'] [%chi 'χ'] [%psi 'ψ'] [%omega 'ω']])
```

- Put a value in the map:

    ```
    (~(put by greek) %digamma 'ϝ')
    ```

    (Notice, in the time-honored tradition of functional languages, that `my-set` is unaltered and a copy of the values is returned.)

- Query if value is in set:

    ```
    > (~(has by greek) %alpha)
    %.y
    > (~(has by greek) %betta)
    %.n
    ```

- Get size of set:

    ```
    > ~(wyt by greek)
    24
    ```

- Get value by key:

    ```
    > (~(get by greek) %delta)
    [~ 'δ']
    ```

    (What mold is this return value?  Why?)

- Apply gate to values:

    ```
    > (~(run by greek) |=(a=@t `@ud`a))
    ```

- Get list of keys:

    ```
    ~(key by greek)
    ```

- Get list of values:

    ```
    ~(val by greek)
    ```

    (Note that these are not in the same order.)

- Convert to list of pairs:

    ```
    ~(tap by greek)
    ```

Many aspects of Hoon, in particular the parser, have their own characteristic data structures, but this should serve to get you started reading and retrieving data from structured sources.

- [Hoon School, “Trees, Sets, and Maps”](https://urbit.org/docs/hoon/hoon-school/trees-sets-and-maps)

> ##  Jars and Jugs (Optional)
>
> A couple of compound structures are also used frequently that have their own terminology:
>
> - `jar` is a map of lists.
>
>     ```
>     ++  jar  |$  [key value]  (map key (list value))
>     ```
>
> - `jug` is a map of sets.
>
>     ```
>     ++  jug  |$  [key value]  (map key (set value))
>     ```
{: .callout}


##  Standard Library

The Hoon standard library is split between `sys/hoon.hoon` and `sys/zuse.hoon`.  Since Hoon follows the Lisp dictum that all code is data, the standard library functions can be quite helpful in operating on all sorts of inputs.  We will focus on a few particular aspects:  text parsing & processing, randomness, hashing, and time.

### Text Parsing and Processing

Most of the text parsing library is built to process Hoon itself, and a variety of rule parsers and analyzers have been built up to this end.  However, we are interested in the much simpler case of loading a structured text string and converting it to some sort of internal Hoon representation.

#### Urbit Types from `@t`

Frequently, one receives plaintext data from some source (like an HTTP `PUT` request) and needs to convert it to a Hoon-compatible atom format.  Alternatively, one needs to convert from a raw Hoon atom into a text representation.  `++slaw` and `++scot` provide this functionality:

```
> (slaw %p '~dopzod')
[~ 4.608]
> (scot %p 1.000)
~.~wanlyn
```

- `++slaw` converts a given `@t` `cord` by a designated aura into a `unit` of that aura.
- `++scot` converts a given value of any type into a target `@ta` representation of that value as a given aura.  (See also `++scow` to convert to a `tape`.)

#### JSON Structured Data

JSON is a frequently-employed data representation standard that organizes data into series of lists and key-value pairs (or maps).

```json
{
  "name": "John",
  "age": 30,
  "car": null
}
```

Hoon has built-in support for parsing and exporting JSON-styled data.  **THIS IS WHERE THINGS GET WEIRD.**  Frequently when working in Hoon, it becomes necessary for you the developer to think in terms of _structures_ rather than _representations_.  JSON is frequently thought of as a generic text representation of data, and while it is acknowledged that there are some different but compatible representations possible (such as by manipulating whitespace), in general JSON _is_ the text.  Not so for Hoon.  It's a very Lisp-y move to consider the data as _more fundamental_ than the instantiation as one particular type of text, and thus there being multiple valid representations of the same data, some quite different from each other.

The foregoing example becomes in Hoon:

```
[~
  [%o p={
      [p='car'
       q=~
      ]
      [p='name'
       q=[%s p='John']
      ]
      [p='age'
       q=[%n p=~.30]
      ]
    }
  ]
]
```

Note the type tags on the information components, and that the `map` is unordered.

Hoon parses JSON in two passes:  first, the JSON is converted in the raw to a tagged data structure using `++de-json:html`.  After this, schema-specific parsers are used to deconstruct the tagged data into usable components with `++dejs:format`.  _This gets a little complicated_, and we'll revisit it again tomorrow when we need it.

The first part is relatively straightforward.  Given a `cord` (`@t`) of JSON data, parse it to a tagged `json` data structure:

```
> =a '{"name":"John", "age":30, "car":null}'
> a
'{"name":"John", "age":30, "car":null}'
> (de-json:html a)
[~ [%o p={[p='car' q=~] [p='name' q=[%s p='John']] [p='age' q=[%n p=~.30]]}]]
```

JSON is a structured data type, however, so one cannot generically process it into particular values.  One must know and account for the expected structure.

We have to build a parser to extract expected values (and ignore others).

```
> =parser %-  ot:dejs-soft:format
  :~  [%name so:dejs-soft:format]
      [%age no:dejs-soft:format]
      [%car (mu so):dejs-soft:format]  :: allows for unit result incl. null
  ==
> (parser u:+:(de-json:html a))
```

The `dejs-soft` library fails gracefully (as opposed to `dejs`).  It can parse, among others, the following types.  (Wrap with `++mu` if the quantity is optional.)

- `da`:  UTC date
- `ne`:  number as real
- `ni`:  number as integer
- `so`:  string as cord
- `ul`:  null

JSON parsing code often ends up rather involved.

- [~timluc-miptev, “Call from Outside:  JSON & `chanel.js`”](https://github.com/timlucmiptev/gall-guide/blob/master/chanel.md)
- [“Fetch JSON”](https://urbit.org/docs/userspace/threads/examples/get-json)

### Randomness

We encountered random numbers previously when dealing with the six-sided dice problem.

The entropy `eny` is generated from [`/dev/random`](https://en.wikipedia.org/wiki//dev/random), itself seeded by factors such as keyboard typing latency.

For instance, to grab a particular entry at random from a list, you can convert `eny` to a valid key and then retrieve:

```
(snag (~(rad og eny) 5) (gulf 1 5))
(snag (~(rad og eny) 5) `(list @p)`~[~wanzod ~marzod ~binzod ~litzod ~samzod])
```

The `og` core provides a number of methods:

- `++rad` for random in range
- `++raw` for random binary bits

The main thing to keep in mind when using the RNG is that you must update the internal state.  Since Hoon is a functional programming language without side effects, you have to pin the modified state as the old state's name to continue to use it.

```
:-  %say
|=  [[* eny=@uv *] [n=@ud ~] ~]
  :-  %noun
  =/  values  `(list @ud)`~
  =/  count  0
  =/  rng  ~(. og eny)
  |-  ^-  (list @ud)
    ?:  =(count n)  values
    =^  r  rng  (rads:rng 100)
  $(count +(count), values (weld values ~[(add r 1)]))
```

### Time

Hoon supports a native date-time aura, `@da` (absolute, relative to “the beginning of time”, `~292277024401-.1.1` = January 1, 292,277,024,401 B.C.), and `@dr` (relative).  This is called the _atomic date_ in the docs.

There is also a parsed time format `tarp`

```
> *tarp
[d=0 h=0 m=0 s=0 f=~]
```

and a parsed date format `date`

```
> *date
[[a=%.y y=0] m=0 t=[d=0 h=0 m=0 s=0 f=~]]
```

which are convenient for representations and interconversions.


### Numerical Conversions

Since every value is at heart an unsigned decimal integer, there is a well-formed binary representation for each value.  This determines the way that the system treats text characters and floating-point values, for instance, and if not careful one can quickly run into absurdity:

```
> (add 1 .-1)
3.212.836.865
```

If we specify the aura, we get a glimpse into what is happening:

```
> `@rs`(add 1 .-1)
.-1.0000001
```

That is, there is a binary representation of `.-1` to which the value `1` has been added:

```
> `@ub`.-1
0b1011.1111.1000.0000.0000.0000.0000.0000
> `@ub`(add .-1 1)
0b1011.1111.1000.0000.0000.0000.0000.0001
```

Things can get much worse, too!

```
> (add .1 .1)
2.130.706.432
> `@rs`(add .1 .1)
.1.7014118e38
```

Auras have a characteristic core that enables one to consistently work with their values.  For instance, for floating-point mathematics, one should use the `rs` core:

```
> (add:rs .1 .1)
.2
```

To convert between numbers, don't do a simple aura cast, which won't have the desired effect most of the time:

```
> `@rs`1.000
.1.401e-42
> `@rs`0
.0
```

Instead, employ the correct conversion routine:

```
> (sun:rs 5)
.5
```


##  Wrapping Up the First Day

At this point, you've been drinking from the firehose and it may have hurt a little.

<iframe src='https://gfycat.com/ifr/AthleticFatHedgehog' frameborder='0' scrolling='no' allowfullscreen width='640' height='524'></iframe><p> <a href="https://gfycat.com/athleticfathedgehog">via Gfycat</a></p>

Our objective for today was to start you thinking in terms of the data structures of idiomatic Hoon.  I want you to absorb as much of this as you can tonight and then tomorrow we will venture into the operating function, userspace, and software deployment, all of which you should be equipped to grapple with now.

If you are interested in exploring some single-purpose generators, I suggest these:

- [Matt Newport, `planetppm.hoon`](https://gist.github.com/mattnewport/f3eaa71c3f71cafe83da541cf3c052ae), a raytracer written in Hoon.
- [N E Davis, `julia`](https://github.com/sigilante/julia), a fractal generator written in Hoon.

{% include links.md %}
