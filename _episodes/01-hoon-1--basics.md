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

What we don't need to do in this workshop is motivate Urbit adoption:  you're invested enough to be here, which means that you perceive some unique benefits to working on the platform.  We are going to focus on the _how_ and to a lesser extent the _why_ of programming on Urbit.

Our first goal today is for you to be able to

-  Identify Hoon runes and children in both inline and long-form syntax.
-  Trace a short Hoon expression to its final result.
-  Execute Hoon code within a running ship.
-  Produce output as a side effect.

To do this, we need to examine a short Hoon program to see what Hoon looks like, then compose a new program of our own.


##  Creating a Fakezod

We will set up a fakezod and a backup.  Since an Urbit ship has a persistent session (as perceived by itself), it is frequently helpful to toss out a lobotomized pier and start anew.  Because this process takes a while the first time we do it locally,

1. Navigate to a new directory `/home/username/urbit`.  (If you work somewhere else, you are completely responsible for transposing anything we do that depends on directory paths.)
2. Download the pill, or pre-packaged Urbit OS configuration:  `wget https://bootstrap.urbit.org/urbit-v1.5.pill`.
3. Start a new fakezod in this directory with `urbit -F zod -B urbit-v1.5.pill`.  The boot sequence will begin.  Let this run in the background while we talk.
4. When this is done, press `Ctrl`+`D` to exit the Urbit session.
5. Copy the fakezod to a local backup:  `cp -r zod zod-ist-tot`.


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

This is a _targum_ of Hoon rephrased into English-like pseudocode.  (Very early on there was a standard version of Hoon not unlike this, although it's not been used for many years now.)

```
func  n=uint  {
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
  call  flop     :: reverse the list order before returning
}
```

Compare the verbal Hoon program side-by-side with the Python program.

```
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

The terminology used in Urbit and Hoon is often unfamiliar.  Sometimes this means that you are dealing with a truly new concept (which would be obscured by overloading an older word like "subroutine" or "function"), whereas sometimes you are dealing with an internal aspect that doesn't really map well to other systems.  The strangeness can be frustrating.  The strangeness can make concepts fresh again.  You'll experience both sentiments during this workshop.

Each rune accepts zero or more children.  Most runes accept a definite number of children, but a few can accept a variable number; these use `==` tistis or `--` hephep digraphs to indicate termination of the running series.  We separate adjacent children by `gap`s, or sequences of whitespace longer than a single space `ace`.

Let's run this Fibonacci program in Urbit.  You will need to start a fakezod, at which point we have two options:

1. The Dojo REPL, which offers some convenient shortcuts to modify the subject for subsequent commands.
2. A tight loop of text editor and running a “generator”.

**Dojo REPL**

To input this program directly into Dojo, we will use a shortcut to name this code; in Hoon-speak we say we give it a face.  You should copy and paste the program rather than typing it out:

```
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

```
(fib 15)
```

**Generator**

The foregoing method works reasonably well when testing short snippets out, but is impractical and doesn't scale well.  What we need to be able to do is put the code into the running ship with a name attached so we can locate it, build it, and evaluate it.

If we `cd` into the ship's pier in Unix and `ls` the directory contents, by default we see nothing.  With `ls -l`, a `.urb/` directory containing the ship's configuration and contents in obfuscated format becomes visible.  This directory is not interpretable by us now, so we leave it until a later discussion of the Urbit binary.  Mars doesn't know about Earth:  we can't directly edit the code in Mars, but instead have to edit on Earth and then synchronize to Mars.

1. In the fakezod, run `|mount %` to mount the `%clay` filesystem to the Unix host filesystem.  (It's actually helpful to do this before you make a backup, or to go ahead and do it pre-emptively in the backup.)
2. Open a text editor and paste the program.  Save this file in `zod/home/gen/` as `fib.hoon`.
3. In the fakezod, tell the runtime to synchronize Earth-side (Unix-side) changes back into `%clay`:  `|commit %home`.
4. Run the generator in the Dojo as `+fib 15`.

Note the different syntax.  In the first case, we have a face in the operating context or _subject_ and we can invoke it directly as a function.  In the latter case, we have to tell Dojo that there is a text file somewhere with a given name, and that it should locate it, build it, pass in the arguments, and evaluate it.

### Charting Rune Children

Let us read the runes by charting out the children of each rune vertically:

```
|=
  n=@ud
  %-
    flop
    =+
      [i=0 p=0 q=1 r=*(list @ud)]
      |-
        ^+
          r
          ?:
            =(i n)
            r
            %=
              $
              i
              +(i)
              p
              q
              q
              (add p q)
              r
              [q r]
            ==
```

### A Generator

Let us compose a short generator from scratch.

> ## Project Euler Problem #1
>
> If we list all the natural numbers below 10 that are multiples of 3 or 5, we get 3, 5, 6 and 9. The sum of these multiples is 23.
>
> Find the sum of all the multiples of 3 or 5 below 1000.
{: .exercise}

Conceptually, we need to build a gate which accepts some number `n` and yields the sum of the values that meet this criterion.

If we start from the Fibonacci sequence gate, we can build a list of acceptable values, then figure out how to sum the values in the list.  Let's start by building the list then, and introducing new functions or runes if we need them.

1. Copy `fib.hoon` to a new file.
2. Scan down, top to bottom, and see what needs to be altered or removed.
3. Test by `|commit %home` then `+p1 10`

```
|=  n=@ud                   :: gate accepts a single @ud argument
%-  roll  :_  add
=+  [i=3 m=0 r=*(list @ud)]
|-  ^+  r
?:  =(i n)  r
?:  =((mod i 3) 0)
  %=  $
    i  +(i)
    r  [i r]
  ==
?:  =((mod i 5) 0)
  %=  $
    i  +(i)
    r  [i r]
  ==
%=  $
  i  +(i)
==
```

Once you can put this together, the next step is to sum the values together.  Hoon is a functional language, which means that it prefers to express set operations.  In this case, `++roll` will help us.  Add this as the second line:

```
%-  roll  :_  add
```

(As a standalone, `++roll` works like this:

```
(roll `(list @ud)`~[9 6 5 3] add)
```

What is `:_` colcab doing here?)

<!-- https://davis68.github.io/martian-computing/lessons/lesson08-subject-oriented-programming.html for barcol example -->

##  Irregular Forms

Many runes in common currency are not written in their regular form (tall or wide), but rather using syntactic sugar as an irregular form.

For instance, `%-` is most frequently written using parentheses `()` which permits a Lisp-like calling syntax:

```
(add 1 2)
```

is equivalent to

```
%-  add  [1 2]
```

which in turn is also equivalent to

```
%-(add [1 2])
```

All three mode of expression are encountered in production code.  Developers balance expressiveness, comprehensibility, and pattern-matching in deciding how lapidary to compose a runic expression.  Many extremely common patterns were soon subsumed by a sugar rune.

> ## Abstract Syntax Tree (Optional)
>
> Hoon parses to an abstract syntax tree (AST), which includes cleaning up all of the sugar syntax and non-primitive runes.  To see the AST of any given Hoon expression, use `!,` zapcom, against `*hoon`.
>
> ```
> > !,(*hoon =/(n 4 +(n)))
> [%tsfs p=term=%n q=[%sand p=%ud q=4] r=[%dtls p=[%wing p=~[%n]]]]
> ```
{: .callout}

{% include links.md %}
