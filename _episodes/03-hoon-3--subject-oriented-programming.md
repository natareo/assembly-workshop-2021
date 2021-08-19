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

For instance, when we first composed generators, we made what are called “naked generators”:  that is, they do not have access to any information outside of the base subject (Arvo, Hoon, and `%zuse`).  Other generators (such as `%say` generators, described below) can have other contextual information, including random number generators and optional arguments, passed to them in their subject.

Hoon developers frequently talk about “limbs” of the subject.  Arms describe known labeled address (with `++` luslus) which carry out computations.  Legs are limbs which store data.

![](TODO:serge-nubret)

Labelled arms are invoked as e.g. gates on data.  However, if you know the address of a particular value, you can retrieve it directly using either notation:

- Numbered limb, `+`/`:` lets you grab a specific numbered arm:  `+7:[1 2 3]`
- Lark notation, which we will skip here but is described in the docs.

Lists have an additional way to grab an element:

- Sequential entry, `&` lets you grab the _n_th item of a list:  `&1:~['one' 2 .3.0 .~4.0 ~.5]`

TODO exercise about this

- [“The Subject and its Legs”](https://urbit.org/docs/hoon/hoon-school/the-subject-and-its-legs)


##  Cores and Derived Structures

### Cores

The core is the primary nontrivial data structure of Nock:  atoms, cells, cores.  A core is defined as a cell of `[battery payload]`; in the abstract this simply divides the `battery` or code from the `payload` or data.  Cores can be thought of as similar to objects in object-oriented programming languages, but possess a completely standard structure which allows for detailed introspection and “hot-swapping” of core elements.  Everything in standard Hoon and Arvo that cannot be reduced to an atom or a cell is _de facto_ a core.  (Indeed, if one wished to separate code and data in a binary tree structure, the only other logical choice one would have available is to flip the order of `battery` and `payload`.)

Conventionally, most cores are either produced by the `|%` barcen rune or are instances of a more complex form (such as a door).  Cores are a “live” type; they are not simply holders of data but are expected to operate on data, just as it is uncommon to see a C++ object which only holds data.  (The role of a C `struct` is approximated by the `$%` buccen tagged union.)

Some terminology is in order, to be expanded on subsequently.  An _arm_ is a Hoon expression to be evaluated against the
TODO

arms
legs
Arms and legs are both \emph{limbs}.  (These must be distinguished from \emph{wings}, which are resolution paths pointing to a limb.)


### Traps

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


### Gates

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



### Doors

`|_` barcab

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


##  Molds



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

```hoon
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

\pattas-tagged text elements are extremely common in such type unions, as they afford a distinguishable human-readable categorization that is nonetheless rigorous to the machine.




\section{Generators}
\labsec{he:generators}

Generators are standalone Hoon expressions that evaluate and may produce side effects, as appropriate.  They are closely analogous to simple scripts in languages such as Bash or Python.  By using generators, one is able to develop more involved Hoon code and run it repeatedly without awkwardness.

\marginnote[2mm]{You may also see commands beginning with a \texttt{|}~symbol; these are Hood commands instead.}

To run a generator on a ship, prefix its name with \texttt{+}.  Arguments may be required or optional.

\begin{lstlisting}[style=nonumbers]
+moon TODO
\end{lstlisting}


\subsection{Naked Generators}
\labsec{he:naked}

As we start to compose generators,

A naked generator is so called because it contains no metadata for the Arvo interpreter.

\subsection{\say~generators}
\labsec{he:say}

TODO

> ##  Rolling Dice
>
> Write a \say~generator which simulates scoring a simple dice throw of $n$ six-sided dice.  That is, it should return the sum of $n$ dice as inputs.  You may reuse code from `mp0` or you may use entropy following the discussion in []().  If no number is specified, then only one die roll should be returned.
>
> Since Hoon is functional but random number generators stateful, you should use the \ptisket~rune to replace the current value in the RNG.  \tisket~is a kind of "one-effect monad," which allows you to change a single part of the subject.
>
> For instance, here is a generator that returns a list of _n_ probabilities between 0–100%.
>
> ```hoon
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
> ```hoon
> +dicethrow, =n 5
> ```
>
> (Note the comma separating optional arguments.)
{: .exercise}

{% include links.md %}
