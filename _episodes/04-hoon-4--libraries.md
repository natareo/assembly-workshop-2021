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

Hoon provides a wrapped subject:  Arvo wraps `%zuse`/`%lull` wraps `hoon.hoon`.

how to import
w/o a face `*`

Many operations are first-class features of the Hoon subject.  For instance,

```hoon
(en-urlt:html mytape)
```

json etc.

TODO

##  




alternatively you can (en-urlt:html your-tape) if you want standard %20 style encoding

However, because I am showing you stylistically excellent Hoon does not mean that you should start composing this way right now.  You should aim to produce `C` code as your first product (frequently by way of `F` and `D`), except with more comments.  Perfect is the enemy of good-enough, particularly when you are getting started.




{% include links.md %}
