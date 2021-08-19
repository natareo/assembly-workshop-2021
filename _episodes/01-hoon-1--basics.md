---
title: "Hoon Basics:  Runes | Basic Programs"
teaching: 30
exercises: 15
questions:
- "How are Hoon programs structured?"
- "What are runes?"
objectives:
- "Identify elements of Hoon such as runes, atoms, and cells."
- "Diagram Hoon generators into the corresponding abstract syntax tree."
- "Operate Dojo by entering Hoon commands."
- "Mount the Unix filesystem and commit changes as necessary."
- "Use built-in or provided tools (gates and generators)."
- "Compose a simple generator and load it from `gen/`."
- "Understand how to pass arguments to and results from gates."
- "Produce a generator for a mathematical calculation."
keypoints:
- "Mars (Urbit) is hermetically sealed from Earth except for system calls."
- "Hoon structures all programs as binary trees of Nock."
- "The `%clay` vane acts as a file system for Urbit."
---

Urbit specifies a virtual machine with a minimalist instruction set called Nock.  Nock is Turing-complete but far too involved to code directly in, leading to the specification of Hoon as a macro language over native Nock.  Hoon affords a clean if alien approach to software construction, relying on a system of “runes” which structure your code as a binary tree.  Programming Hoon can feel Lisp-like, but has a number of its own quirks.

Our first goal is for you to be able to

-  Identify Hoon runes and children in both inline and long-form syntax.
-  Trace a short Hoon expression to its final result.
-  Execute Hoon code within a running ship.
-  Produce output as a side effect using the `~&` sigpam rune.

To do this, we need to examine a short Hoon program to see what Hoon looks like, then compose a new program of our own.


##  Reading the Runes

For instance, consider the following Python program, which implements the Fibonacci sequence as a list:

```py
def fibonacci(n):
  i = 0
  p = 0
  q = 1
  r = []
  while (i < n):
    i = i + 1
    po = p
    p = q
    q = po + q
    r.append(q)
```

```hoon
gate  n=uint  {
  let  [i=0 p=0 q=1 r=(list uint)]
  do {
    if (i == n)  return r
  } loop {
    i ← i+1      :: increment counter
    p ← q        :: cycle value forwards
    q ← (p + q)  :: add values
    r ← [q] + r  :: append to list
  }
  match-type  r  :: enforce type restraint
  apply  flop    :: reverse the list order before returning
}
```

Compare the verbal Hoon program side-by-side with the Python program.

```hoon
|=  n=@ud
%-  flop
=+  [i=0 p=0 q=1 r=*(list @ud)]
|-  ^+  r
?:  =(i n)  r
%=  $
  i  +(i)
  p  q
  q  (add p q)
  r  [q r]
==
```

The terminology used in Urbit and Hoon is often unfamiliar.  Sometimes this means that you are dealing with a truly new concept (which overloading an older word like "subroutine" or "function" would obscure), whereas sometimes you are dealing with an internal aspect that doesn't really map well to other systems.  The strangeness can be frustrating.  The strangeness can make concepts fresh again.  You'll experience both sentiments during this workshop.

Each rune accepts at least one child, except for `!!` zapzap, crash.  Most runes accept a definite number of children, but a few can accept a variable number; these use `==` tistis or `--` hephep digraphs to indicate termination of the running series.

Let's run this Fibonacci program in Urbit.  You will need to start a fakezod, at which point we have two options:

1. The Dojo REPL, which offers some convenient shortcuts to modify the subject for subsequent commands.
2. A tight loop of text editor and running a “generator”.

**Dojo REPL**

To input this program directly into Dojo, we will use a shortcut to name this code; in Hoon-speak we say we give it a face.  You should copy and paste the program rather than typing it out:

```hoon
=fib |=  n=@ud
%-  flop
=+  [i=0 p=0 q=1 r=*(list @ud)]
|-  ^+  r
?:  =(i n)  r
%=  $
  i  +(i)
  p  q
  q  (add p q)
  r  [q r]
==
```

Notice that the Dojo parses your code for compatibility in real time.  This makes the typing a bit slow and janky but means that it is fairly difficult to input invalid code or syntax errors.  (Nothing is impossible, of course!)  If at any point you are typing and nothing appears, it is because you aren't following a rule of valid Hoon.

Now whenever we want to run the gate, we can use a Lisp-like syntax to operate on a value:

```hoon
(fib 15)
```

**Generator**

The foregoing method works reasonably well when testing short snippets out, but is impractical and doesn't scale well.  What we need to be able to do is put the code into the running ship with a name attached so we can locate it, build it, and evaluate it.  However, Mars doesn't know about Earth:  we can't directly edit the code in Mars, but instead have to edit on Earth and then synchronize to Mars.  What does this look like?

1. In the fakezod, run `|mount %` to mount the `%clay` filesystem to the Unix host filesystem.
2. Open a text editor and paste the program.  Save this file in `~fakezod/home/gen/` as `fib.hoon`.
3. In the fakezod, tell the runtime to synchronize Earth-side changes back into `%clay`:  `|commit %home`.
4. Run the generator in the Dojo as `+fib 15`.

Note the different syntax.  In the first case, we have a face in the operating context or _subject_ and we can invoke it directly as a function.  In the latter case, we have to tell Dojo that there is a text file somewhere with a given name, and that it should locate it, build it, pass in the arguments, and evaluate it.

TODO write a new generator from scratch which uses `~&`
TODO chart out runes and children as AST

##  Irregular Forms

Many runes in common currency are not written in their regular form (tall or wide), but rather using syntactic sugar as an irregular form.

For instance, `%-` is most frequently written using parentheses `()` which permits a Lisp-like calling syntax:

```hoon
(add 1 2)
```

is equivalent to

```hoon
%-  add  [1 2]
```

which in turn is also equivalent to

```hoon
%-(add [1 2])
```

All three mode of expression are encountered in production code.  Developers balance expressiveness, comprehensibility, and pattern-matching in deciding how lapidary to compose a runic expression.  Many extremely common patterns were soon subsumed by a sugar rune.

Hoon parses to an abstract syntax tree (AST), which includes cleaning up all of the sugar syntax and non-primitive runes.  To see the AST of any given Hoon expression, use `!,` zapcom.

```hoon
> !,(*hoon =/(n 4 +(n)))
[%tsfs p=term=%n q=[%sand p=%ud q=4] r=[%dtls p=[%wing p=~[%n]]]]
```


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

> ### Auras
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

The Hoon standard library, largely in \zuse, further defines bitwise operations, arithmetic for both integers and floating-point values (half-width, single-precision, double-precision, and quadruple-precision), string operations, and more.  These are introduced incidentally as necessary and listed in more detail in Appendix~\ref{stdlib}.

\subsection{Cells}
\labsec{cells}

Just as all structures in Nock are binary trees, so too with Hoon.  This can occasionally lead to some awkward addressing when composing tetchy library code segments that need to interface with many different kinds of gates, but by and large is an extremely helpful discipline of thought.


binary tree format
flattened representation/convention

\marginnote[2mm]{Binary trees are explained in more detail in Section~\ref{nock0}.}

Hoon values are addressed as elements in a binary tree.


Finally, the most general mold is \texttt{*} which simply matches any noun—and thus anything in Hoon at all.

\section{Hoon as Nock Macro}
\labsec{macro}

The point of employing Hoon is, of course, that Hoon compiles to Nock.  Rather than even say \emph{compile}, however, we should really just say Hoon is a \emph{macro} of Nock.  Each Hoon rune, data structure, and effect corresponds to a well-defined Nock primitive form.  We may say that Hoon is to Nock as C is to assembler, except that the Hoon-to-Nock transformation is completely specified and portable.  Hoon is ultimately defined in terms of Nock; many Hoon runes are defined in terms of other more fundamental Hoon runes, but all runes parse unambiguously to Nock expressions.


Hoon expands on Nock primarily through the introduction of metadata


Each Hoon rune has an unambiguous mapping to a Nock representation.  Furthermore, each rune has a well-defined binary tree structure and produces a similarly well-structured abstract syntax tree (AST).  As we systematically introduce runes, we will expand on what this means in each case:  for now, let's examine two runes without regard for their role.

\begin{enumerate}
  \item  \pbartis~produces a \emph{gate} or function.  Every gate has the same shape, which means certain assumptions about data access and availability can be made.

    \begin{lstlisting}[language=hoon,
                       style=nonumbers]
::  XOR two binary atoms
|=  [a=@ub b=@ub]
`@ub`(mix a b)
    \end{lstlisting}

    maps to the Nock code

    \begin{lstlisting}[language=hoon,
                       style=nonumbers]
[8 [1 0 0] [1 8 [9 1.494 0 4.095] 9 2 10 [6 [0 28] 0 29] 0 2] 0 1]
    \end{lstlisting}

    This Nock code is fully annotated in Example~\ref{ex:nock:xor}.

  \item  TODO
\end{enumerate}


We call Hoon's data type specifications \emph{molds}.  Molds are more general than atoms and cells, but these form particular cases.  Hoon uses molds as a way of matching Nock tree structures (including Hoon metadata tags such as auras).

\section{Subject-Oriented Programming}

TODO
arms legs basics

\marginnote[2mm]{
Be careful to not confuse \irrtis, which evaluates to \dottis, with the various \wut~runes like \wuttis.
}

\section{Key Data Structures}
\labsec{he:structures}

\subsection{Lists}
\labsec{he:lists}


Lests

\subsection{Text}
\labsec{text}

\marginnote[2mm]{Both cords and tapes are casually referred to as strings.}

Hoon recognizes two basic text types:  the \emph{cord} or \patt~and the \emph{tape}.  Cords are single atoms containing the text as UTF-8 bytes interpreted as a single stacked number.  Tapes are lists of individual one-element cords.

\marginnote[2mm]{Lists are null-terminated, and thus so are tapes.}

Cords are useful as a compact primary storage and data transfer format, but frequently parsing and processing involves converting the text into tape format.  There are more utilities for handling tapes, as they are already broken up in a legible manner.

\begin{lstlisting}[language=hoon,
                   style=nonumbers]
++  trip
  |=  a=@  ^-  tape
  ?:  =(0 (met 3 a))  ~
  [^-(@ta (end 3 1 a)) $(a (rsh 3 1 a))]
\end{lstlisting}


For instance, \texttt{trip} converts a cord to a tape; \texttt{crip} does the opposite.

%\marginnote[2mm]{
%\begin{lstlisting}
%++  trip
%  |=  a/@  ^-  tape
%  ?:  =(0 (met 3 a))  ~
%  [^-(@ta (end 3 1 a)) $(a (rsh 3 1 a))]
%\end{lstlisting}
%\texttt{++met}, \texttt{++end}, and \texttt{++rsh} are bitwise manipulation gates.
%}

\marginnote[2mm]{
%\begin{lstlisting}
\texttt{++  crip  |=(a=tape `@t`(rap 3 a))}
%\end{lstlisting}

\texttt{++rap} assembles the list interpreted as cords with block size of $2^3$ (in this case).
}


All text in Urbit is UTF-8 (and typically just 8-bit ASCII).  The \patc~UTF-32 aura is only used by \dill and Hood (the Dojo terminal agent).


\subsection{Cores and Derived Structures}
\labsec{he:cores-derived}

\subsubsection{Cores}
\labsec{he:core}

The core is the primary nontrivial data structure of Nock:  atoms, cells, cores.  A core is defined as a cell of \texttt{battery payload}; in the abstract this simply divides the \texttt{battery} or code from the \texttt{payload} or data.  Cores can be thought of as similar to objects in object-oriented programming languages, but possess a completely standard structure which allows for detailed introspection and “hot-swapping” of core elements.  Everything in standard Hoon and Arvo that cannot be reduced to an atom or a cell is \emph{de facto} a core.  (Indeed, if one wished to separate code and data in a structure, the only other logical choice one would have available is to flip the order of battery and payload.)

Conventionally, most cores are either produced by the \pbarcen~rune or are instances of a more complex form (such as a door).  Cores are a “live” type; they are not simply holders of data but are expected to operate on data, just as it is uncommon to see a C++ object which only holds data.  (The role of a C \texttt{struct} is approximated by the \buccen~tagged union.)

Some terminology is in order, to be expanded on subsequently.  An \emph{arm} is a Hoon expression to be evaluated against the
arms
legs
Arms and legs are both \emph{limbs}.  (These must be distinguished from \emph{wings}, which are resolution paths pointing to a limb.)


\subsubsection{Traps}
\labsec{he:trap}

The trap creates the basic looping mechanism in Hoon, a special instance of a core which is capable of concisely recursing into itself.  The trap is formally a core with one arm named \buc, and it is either created with \pbarhep~(for instant evaluation) or \pbardot~(for deferred evaluation).

In practice, traps are used almost exclusively as recursion points, much like loops in an imperative language.  For instance, the following program counts from 5 down to 1, emitting output via \psigpam~at each iteration, then return \sig.

\begin{lstlisting}[language=hoon,
                   style=nonumbers]
=/  count  5
|-
  ?:  =(count 0)  ~
~&  count
$(count (dec count))
\end{lstlisting}

\marginnote[2mm]{
The \texttt{\$()} notation is shorthand for \pcentis, which evaluates a wing given a set of changes.
}
The final line \texttt{\$(count (dec count))} serves to modify the subject (at \texttt{count}) then to pull the \buc~arm again.  In practice the tree unrolls as follows, with indentation indicating code running “inside” of another rune.

\begin{lstlisting}[language=hoon,
                   style=nonumbers]
=/  count  5
|-
  ?:  =(count 0)  ~
  ~&  count
  =/  count  (dec count)
  ?:  =(count 0)
  TODO think about how to unroll this in a visually pleasing manner that isn't
  a total lie
$(count (dec count))
\end{lstlisting}


recursive


\subsubsection{Gates}
\labsec{he:gate}

Similar to how a trap is a core with one arm, a gate is a core with one arm and a sample.  This means that new per-invocation data are available in the gate's subject.


A gate can recurse into itself like a trap when necessary.  In this example, the gate accepts a value \texttt{n} (the gate's \texttt{spec}) and applies an arm (implicitly named \buc) which carries out the calculation.  (Say it with me:  \texttt{sample}/\texttt{payload}.)  When the \texttt{\$()} recursion point is reached, the entire \buc~arm is re-evaluated with the specified change of $\texttt{n} \rightarrow \texttt{n} - 1$.

\begin{lstlisting}[language=hoon,
                   caption={Calculating a factorial using recursion.  (Example from Tlon documentation.)}]
|=  n=@ud             :: accept a single value n for n!
?:  =(n 1)  1         :: check n ≟ 1; if so, return 1
(mul n $(n (dec n)))  :: multiply n times the product of this arm w/ n-1
\end{lstlisting}

That is, when one reads this program, one reads it falling into two components:

\begin{lstlisting}[language=hoon,
                   style=nonumbers]
|=  n=@ud             :: accept a single value n for n!
?:  =(n 1)  1         :: check n ≟ 1; if so, return 1
(mul n $(n (dec n)))  :: multiply n times the product of this arm w/ n-1
\end{lstlisting}



\subsubsection{Doors}
\labsec{he:door}

\barcab

The \pbarcen~rune produces a dry core, thus all arms it contains are dry.

The \pbarpat~rune produces a wet core, so it can contain wet arms.

Doors are superficially similar to gates, but calling an arm in a door is more general:  the calling convention for a gate replaces the sample then pulls the \buc~arm, while the calling convention for a door replaces the sample then pulls whichever arm has been requested.

\marginnote[2mm]{
Doors and other cores frequently include custom type definitions; these are discussed below in Section~\ref{he:mold}.
}
Since \dot~refers to the subject TODO:expand, this yields the ability to manipulate the sample without calling any arm directly.  The following examples illustrate:

\begin{lstlisting}[language=hoon,
                   style=nonumbers]
> (add [1 5]})              :: call a gate
6
> ~($ add [1 5])            :: call the $ arm in the door
6
> ~(. add [1 5])            :: update the sample (arguments) but don't evaluate
<1.otf [[@ud @ud] <45.xig 1.pnw %140>]>
> =<  $  ~(. add [1 5])     :: call the $ arm of the door with updated sample
6
> =/  addd  ~(. add [1 5])  :: do the same thing but in two steps
  =<  $  addd
6
\end{lstlisting}




At this point, we need to step back and contextualize the power afforded by the use of cores.  In another language, such as C or Python, we specify a behavior but have relatively little insight into the instantiation effected by the compiler:

\begin{lstlisting}[language=python,
                   caption={Python loop to sum the length of several strings.}]
metals = ['gold', 'iron', 'lead', 'zinc']
total = 0
for metal in metals:
    total = total + len(metal)
\end{lstlisting}

\begin{lstlisting}[language=jvmis,
                   caption={Python bytecode equivalent}]
1          0 LOAD_CONST               0 ('gold')
           2 LOAD_CONST               1 ('iron')
           4 LOAD_CONST               2 ('lead')
           6 LOAD_CONST               3 ('zinc')
           8 BUILD_LIST               4
          10 STORE_NAME               0 (metals)

2         12 LOAD_CONST               4 (0)
          14 STORE_NAME               1 (total)

3         16 LOAD_NAME                0 (metals)
          18 GET_ITER
     >>   20 FOR_ITER                16 (to 38)
          22 STORE_NAME               2 (metal)

4         24 LOAD_NAME                1 (total)
          26 LOAD_NAME                3 (len)
          28 LOAD_NAME                2 (metal)
          30 CALL_FUNCTION            1
          32 BINARY_ADD
          34 STORE_NAME               1 (total)
          36 JUMP_ABSOLUTE           20
     >>   38 LOAD_CONST               5 (None)
          40 RETURN_VALUE
\end{lstlisting}

Many popular programming languages specify \emph{behavior} rather than \emph{implementation}.  By specifying Hoon as a macro language over Nock, the Urbit developers collapse this distinction.

For Hoon (like Lisp), the structure of execution is always front-and-center, or at least only thinly disguised.  When one creates a core with a given behavior, one can immediately envision the shape of the underlying Nock.  This affords one immense power to craft efficient, effective, graceful programs.

\begin{lstlisting}[language=hoon,
                   caption={Hoon trap to sum the length of several tapes.}]
=/  metals  `(list tape)`~["gold" "iron" "lead" "zinc"]
=/  count  0
=/  total  0
|-
  ?:  =(count (lent metals))  total
  =/  metal  `tape`(snag count metals)
$(total (add total (lent metal)), count +(count))
\end{lstlisting}

\begin{lstlisting}[language=hoon,
                   caption={Nock equivalent}]
\end{lstlisting}

\subsection{Molds}
\labsec{he:mold}



Most software programs require—or at least are greatly simplified by—custom type definitions.  These are customarily defined with a \barcen~core preceding the \barcab~door definition.  We commonly see three runes supporting this structure:

\begin{itemize}
  \item  \plusbuc~creates a type constructor arm to define and validate type definitions.
  \item  \pbuccen~creates a collection of named values (type members).
  \item  \pbucwut~defines a union, a set validating membership across a defined collection of items.  (This is similar to a \texttt{typedef} or \texttt{enum} in C-related languages.)
\end{itemize}

To illustrate these, we serially consider several ways to define a vehicle.  In the first, we employ only \lusbuc~to capture key vehicle characteristics.  Using only \lusbuc, it's hard to say much of interest:

% TODO check all of these

\begin{lstlisting}[style=nonumbers]
+$  vehicle  tape               :: vehicle identification number
\end{lstlisting}

By permitting collections of named type values with \buccen, we can produce more complicated structures:

\begin{lstlisting}[style=nonumbers]
+$  vehicle
  $%  vin=tape                  :: vehicle identification number
      owner=tape                :: car owner's name
      license=tape              :: license plate
  ==
\end{lstlisting}

Type definition arms can rely on other type definition arms available in the subject:

\begin{lstlisting}[style=nonumbers]
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
\end{lstlisting}

\marginnote[2mm]{ \\ \\ \\ \\ \\ \\ \\ \\ \\ \\ \\ \\
In general, Hoon style does not require you to be careful about masking variable names in the subject (using the same name for the value as the mold).  This rarely introduces surprising bugs but is typically contextually apparent to the developer.
}
Finally, by introducing unions with \bucwut, a type definition arm can validate possible values:

\begin{lstlisting}[style=nonumbers]
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
\end{lstlisting}

\pattas-tagged text elements are extremely common in such type unions, as they afford a distinguishable human-readable categorization that is nonetheless rigorous to the machine.


\subsection{Data Structures}
\labsec{he:data-structures}

\subsubsection{Units}
\labsec{he:unit}

Every atom in Hoon is an unsigned integer, even if interpreted by an aura.  Auras do not carry prescriptive power, however, so they cannot be used to fundamentally distinguish a \texttt{NULL}-style or \texttt{NaN}-style non-result.  That is, if one receives a \sig~back from a query, how does one distinguish a value of zero from a non-result (missing value)?

Units mitigate this situation by acting as a type union of \sig~(for no result) and a cell \texttt{[\textasciitilde~u=item]} containing the returned item with face \texttt{u}.

\begin{lstlisting}[style=nonumbers]
++  unit
  |$  [item]
  $@(~ [~ u=item])
\end{lstlisting}


\subsubsection{Maps}
\labsec{he:map}

\subsubsection{Sets}
\labsec{he:set}

\subsubsection{Trees}
\labsec{he:tree}

\subsubsection{Dimes}
\labsec{he:dime}


\section{Generators}
\labsec{he:generators}

Generators are standalone Hoon expressions that evaluate and may produce side effects, as appropriate.  They are closely analogous to simple scripts in languages such as Bash or Python.  By using generators, one is able to develop more involved Hoon code and run it repeatedly without awkwardness.

\marginnote[2mm]{You may also see commands beginning with a \texttt{|}~symbol; these are Hood commands instead.}

To run a generator on a ship, prefix its name with \texttt{+}.  Arguments may be required or optional.

\begin{lstlisting}[style=nonumbers]
+moon TODO
\end{lstlisting}

\subsection{Running Developer Code on an Urbit Ship}
\labsec{he:running}

Since the Urbit file system, called \clay, is independent of the Unix file system on which it is hosted, you must commit your Unix-side code into your pier.

If we \texttt{cd} into the ship's pier in Unix and \texttt{ls} the directory contents, by default we see nothing.  With \texttt{ls -l}, a \texttt{.urb/} directory containing the ship's configuration and contents in obfuscated format becomes visible.  This directory is not interpretable by us now, so we leave it until a later discussion of the Urbit binary.  To move files into \clay, we must synchronize Urbit and Unix together.  We initiate this inside of the running ship; run the Urbit system command:

\begin{lstlisting}[style=nonumbers]
|mount %
\end{lstlisting}

where `%` represents the current (home) path in \clay.  Unix-side, run \texttt{ls} again and a \texttt{home/} directory appears with a number of children:  \texttt{app/}, \texttt{gen/}, \texttt{lib/}, \texttt{mar/}, and so forth.  This is the internal esoteric structure of \clay~made manifest to Unix.

\marginnote[2mm]{\clay~implements several \emph{desks}, which are like branches in version control systems; the most important of these are \chome~and \ckids.}
Generally speaking, we compose \emph{generators}, which are short Hoon scripts.  These are created in or copied into the \texttt{home/mar/} directory, and then must be synchronized with Urbit's \clay.  Commit the change:

\begin{lstlisting}[style=nonumbers]
|commit %home
\end{lstlisting}

and the generator is now available within \clay.

\subsection{Naked Generators}
\labsec{he:naked}

As we start to compose generators,

A naked generator is so called because it contains no metadata for the Arvo interpreter.

\subsection{\say~generators}
\labsec{he:say}


---
`++dicethrow`

Write a \say~generator which simulates scoring a simple dice throw of $n$ six-sided dice.  That is, it should return the sum of $n$ dice as inputs.  You may reuse code from `mp0` or you may use entropy following the discussion in []().  If no number is specified, then only one die roll should be returned.

Since Hoon is functional but random number generators stateful, you should use the \ptisket~rune to replace the current value in the RNG.  \tisket~is a kind of "one-effect monad," which allows you to change a single part of the subject.

For instance, here is a generator that returns a list of _n_ probabilities between 0–100%.

\begin{lstlisting}
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
\end{lstlisting}

Your command to run this generator in the Dojo should look like this:

```hoon
+dicethrow, =n 5
```

(Note the comma separating optional arguments.)

{% include links.md %}
