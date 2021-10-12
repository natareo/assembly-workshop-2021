---
title: "Hoon Basics:  Libraries"
teaching: 20
exercises: 10
questions:
- "How can I import code into a Hoon program?"
objectives:
- "Access built-in libraries."
- "Create a library in `lib/` and utilize it using `/+`."
- "Access a library in the Dojo with `-build-file`."
- "Compose Hoon that adheres to the Tlon Hoon style guide."
keypoints:
- "Libraries are code which can be built and included in the subject of downstream code."
---

##  Accessing Built-In Library Cores

Hoon provides a wrapped subject:  Arvo wraps `%zuse` wraps `hoon.hoon`.  These subject components are immediately available to you when you run any program.  In addition to these, some contexts expose the `[our now eny]` pattern of ship-bound knowledge:  the current ship identity `our`, the system time `now`, and a source of entropy `eny`.  (This, incidentally, is why the boot process is necessary for each instance including fakezods.)

If you need more functionality than these, you can import a library using the Ford `/` fas runes; specifically,

- `/-  foo, *bar, baz=qux` imports a file from the sur directory (`*` pinned with no face, `=` with specified face)
- `/+  foo, *bar, baz=qux` imports a file from the lib directory (* pinned with no face, = with specified face)

You can also directly build a file in Dojo with the thread `-build-file`; this is frequently more convenient for interactive testing.

```
=foo -build-file %/lib/foo/hoon
```

At this point, most operations are first-class features of the Hoon subject.  For instance, to render a string with URL-compatible codes:

```
=mytape "Parallax_(Star_Trek:_Voyager)"
(weld "https://en.wikipedia.org/wiki/" (en-urlt:html mytape))
```

### Useful Libraries

Check the contents of the `/lib` directory with the `+ls` generator:

```
> +ls %/lib
  agentio/hoon
  aqua-azimuth/hoon
  aqua-vane-thread/hoon
  azimuth/hoon
  ...
```

View the contents of a file with the `+cat` generator:

```
> +cat %/lib/ethereum/hoon
```

(No syntax highlighting with `+cat`, alas!)

You'll see many of these employed in the Gall agents we compose tomorrow.

> ## Libraries in Use
>
> Scan through the generators in `%/gen` and see which libraries are used and how they are imported.
{: .challenge}

There are some notable omissions still which you can rectify for yourself:

- [ordered maps of sets](https://github.com/tirrel-corp/urbit/commit/29c5ad4ebaf42edff1ff227eca3c09214d4d9943) (see the next lesson for motivation)
- [`lazytrig.hoon`](https://github.com/sigilante/lazytrig) provides transcendental functions (unjetted, Hoon-only).
- string parsing is still a bit rough (DIY from fundamental components)


### Hoon Style

Dogmatically good Hoon style has evolved over the years, and the current Arvo kernel is a palimpsest of styles.

> ##  Deducing Style
>
> Examine several source code files.  Enumerate some principles of good Hoon style that you infer from these.
{: .challenge}

Generally speaking, Hoon prefers short variable names with loose mnemonics, left-aligned rune branches, sparse commentary, and preference of whichever irregular form is most expressive in-context.

> Hoon’s position on layout is: so long as your code is (a) correctly commented, (b) free from blank or overlong lines, (c) parses, and (d) looks good, it’s good layout.

We do need to distinguish two common forms that you've already encountered _in vivo_:

- Wide form fits on a single line.
- Tall form uses multiple lines.

Perfect is the enemy of good-enough, particularly when you are getting started.  Focus on writing working code with clear intent before getting hung up on formatting.

- [Tlon Corporation, “Hoon Style Guide”](https://urbit.org/docs/hoon/reference/style)


{% include links.md %}
