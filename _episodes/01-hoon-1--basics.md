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

The foregoing method works reasonably well when testing short snippets out, but is impractical and doesn't scale well.  What we need to be able to do is put the code into the running ship with a name attached so we can locate it, build it, and evaluate it.  If we `cd` into the ship's pier in Unix and `ls` the directory contents, by default we see nothing.  With `ls -l`, a `.urb/` directory containing the ship's configuration and contents in obfuscated format becomes visible.  This directory is not interpretable by us now, so we leave it until a later discussion of the Urbit binary.  Mars doesn't know about Earth:  we can't directly edit the code in Mars, but instead have to edit on Earth and then synchronize to Mars.

1. In the fakezod, run `|mount %` to mount the `%clay` filesystem to the Unix host filesystem.
2. Open a text editor and paste the program.  Save this file in `~fakezod/home/gen/` as `fib.hoon`.
3. In the fakezod, tell the runtime to synchronize Earth-side changes back into `%clay`:  `|commit %home`.
4. Run the generator in the Dojo as `+fib 15`.

Note the different syntax.  In the first case, we have a face in the operating context or _subject_ and we can invoke it directly as a function.  In the latter case, we have to tell Dojo that there is a text file somewhere with a given name, and that it should locate it, build it, pass in the arguments, and evaluate it.

TODO write a new generator from scratch which uses `~&`
TODO chart out runes and children as AST

\subsection{Running Developer Code on an Urbit Ship}
\labsec{he:running}

Since the Urbit file system, called \clay, is independent of the Unix file system on which it is hosted, you must commit your Unix-side code into your pier.


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

{% include links.md %}
