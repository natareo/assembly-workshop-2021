---
title: "Arvo:  API & Scrying"
teaching: 20
exercises: 15
questions:
- "How do I access data locally?"
- "How do I access data remotely?"
objectives:
- "Update and access the file system."
- "Scry for local and remote data."
- "Use subscriptions to access data."
- "Enumerate functionality of the Urbit API."
keypoints:
- "The main data operations in Urbit are:  pokes, scries, and subscriptions."
- "A poke is like an HTTP `PUT` request for data."
- "A scry is like an HTTP `GET` retrievel of data."
- "A subscription is a reactive request for data updates."
---

Data tend to live in one of two places in Urbit:

1. `%clay` holds files (typically source files).
2. `%gall` holds data stores (of all kinds:  metadata, data, etc.).

Other data are frequently stored off-ship, such as in an S3 bucket.


##  Filesystem Operations

TODO

- [Tlon Corporation, “Filesystem User's Manual”](https://urbit.org/using/os/filesystem)


##  Userspace Operations

The main data operations in Urbit userspace are:  scries, pokes, and subscriptions.

- A scry is like an HTTP `GET` retrievel of data.
- A poke is like an HTTP `PUT` request for data.
- A subscription is a reactive request for data updates.

##  `++peek`/Scrying

We've talked at some length about data access through scrying, but how does one actually do this?  Most of the time these events are generated as a matter of course through `%gall` agents as moves which Arvo automatically handles, but there is a lightweight way to access them as well:  `.^` dotket.

A `.^` dotket call accepts a type `p` for the result of the call and a compound `path` which indicates where the call goes and other information.

```
::  Query %clay for a list of files at the current path.
.^(arch %cy %)
```

The first letter of the second element (`@tas`) indicates the destination vane of the move.  The second letter of the query is a mode flag.  Most vanes only recognize a `%x` mode, but `%clay` has a very sophisticated array of calls, including:

- `%a` for file builds
- `%b` for mark builds
- `%c` for cast builds (mark conversion gate)
- `%w` for version number
- `%x` for data
- `%y` for list of children
- `%z` for subtree

Some examples:

```
TODO one each for the scry types of %clay, %gall

::  Build a conversion mark from JSON to txt.
.^(tube:clay %cc /~zod/home/1/json/txt)

::  Query %clay for a list of files at the current path.
.^(arch %cy %)

::  Ask for a file from several hours ago.
.^(arch %cy /(scot %p our)/home/(scot %da (sub now ~h5)))
```


##  `++poke`/Poking

##  Subscriptions

As mentioned previously, Urbit prefers a reactive data flow model in which when a value or data store is updated it notifies all subscribers (rather than having them poll periodically).  Internal subscriptions, or subscriptions between ships, take the form of

- `%clay` subscriptions notify subscribers when the filesystem changes.
- `%gall` subscriptions notify subscribers when applications change.
- `%jael` subscriptions notify subscribers when Urbit ID information changes.

> A subscription is a stream of events from a publisher to a subscriber, where
>
> 1. the publisher has indicated interest in that stream,
> 2. any update sent to one subscriber is sent to all subscribers on the same stream, and
> 3. when the publisher has a new update ready,
>
> they send it immediately to their subscribers instead of waiting for a later request.

channels

https://urbit.org/docs/arvo/concepts/subscriptions

##  Urbit API

Besides the _Urbit internal API_ (which includes the standard arms of vanes and the semantics of moves), there is also the _Urbit external API_, which handles Mars ⇄ Earth communications.  This is accomplished through the intermediary of `%eyre` which supports two-way channels to external agents and applications.

- [“`%eyre` External API Reference”](https://urbit.org/docs/arvo/eyre/external-api-ref)

{% include links.md %}
