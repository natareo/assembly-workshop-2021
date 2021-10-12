---
title: "Arvo:  Kernel Structure"
teaching: 20
exercises: 0
questions:
- "How is the Urbit kernel structured?"
- "How do I access system operations?"
objectives:
- "Diagram the high-level architecture of Urbit."
keypoints:
- "Arvo is an event processor or state machine."
- "Events are well-defined single operations on the system state."
- "The binary executable is separated into king and serf, the system state, an event log and runtime support."
---

Urbit bills itself as a “clean-slate operating system” or “operating function”.  Now that you have some of the syntax of Hoon under your belt, we can examine what the kernel does to implement these features.  In brief, these are the selling points I make to any developer who asks, “Why bother with Urbit?”

> [Urbit] solves the hard problems of implementing a peer-to-peer network (including identity, NAT traversal, and exactly-once delivery) in the kernel so app developers can focus on business logic.
>
> The entire OS is a single pure function that provides application developers with strong guarantees: automated persistence and memory management, repeatable builds, and support for hot code reloading.

<!-- NAT = network address translation -->

My partial summary is that Urbit offers the developer:

1. Cryptographic identity.
2. Version-controlled typed filesystem.
3. Authentication primitives.
4. Persistent database.

That is, once you've done the admittedly hard shovel work of learning Urbit and building on it, you get a number of nice guarantees for “free,” or at least able to be taken for granted.

We could do worse than to start our exploration of the Urbit kernel than to quote the Whitepaper itself on the subject of Arvo:

> The fundamental unit of the Urbit kernel is an event called a `+$move`.  Arvo is primarily an event dispatcher between moves created by vanes.  Each move therefore has an address and other structured information.

Arvo is the main lifecycle function which handles these discrete events.  What initiates and handles events?  These are the _vanes_, standardized system services.  Every vane defines its own structured events (`+$move`s).  Each unique kind of structured event has a unique, frequently whimsical, name.  This can make it challenging to get used to how a particular vane behaves.

![](https://pbs.twimg.com/media/FBgvOeNXsKEcp_1?format=jpg&name=large)

We focus now on the center of this diagram.  The circle in the middle represents the Nock VM function.  Wrapping around that, the Arvo subject consists of the Arvo lifecycle function, the Hoon language, and the Zuse/Lull data structures and conventions.

Arvo is essentially an event handler which can coordinate and dispatch messages between vanes as well as emit Unix `%unix` events (side effects) to the underlying (presumed Unix-compatible) host OS.  Arvo as hosted OS does not carry out any tasks specific to the machine hardware, such as memory allocation, system thread management, and hardware- or firmware-level operations.  These are left to the _king_ and _serf_, the daemon runtime processes which together run Arvo.

Arvo is architected as a state machine, the deterministic end result of the event log.  We need to briefly examine Arvo from two separate angles:

1.  Event processing engine and state machine (vane coordinator).
2.  Standard noun structure (“Arvo-shaped noun”).

### Arvo as Event Processing Engine

> A vanilla event loop scales poorly in complexity.  A system event is the trigger for a cascade of internal events; each event can schedule any number of future events.  This easily degenerates into “event spaghetti.”  Arvo has “structured events”; it imposes a stack discipline on event causality, much like imposing subroutine structure on `goto`s.

Arvo events (`+$move`s) contain metadata and data.  Arvo recognizes three types of moves:

1.  `%pass` events are forward calls from one vane to another (or back to itself, occasionally), and
2.  `%give` events are returned values and move back along the calling duct.
3.  `%unix` events communicate from Arvo to the underlying binary in such a way as to emit an external effect (an `%ames` network communication, for instance, or text input and output).

A bit more terminology:

- A _move_ consists of message data and metadata indicating what needs to happen.  A move sends an action to a location along a call stack (or _duct_).
- A _card_ is an event, or action.  Cards can have arbitrarily complicated syntax depending on the vane and message.  For instance, here is an example of a Gall card:

    ```
    [%give %fact ~[/status] [%atom !>(status.state)]]
    ```

    There is a complicated interplay of cards as seen from callers and callees, but we will largely ignore that level of detail here.

> ##  Card Types (Optional)
>
> > Each vane defines a protocol for interacting with other vanes (via Arvo) by defining four types of cards: tasks, gifts, notes, and signs.
> >
> > In other words, there are only four ways of seeing a move:
> > 1. as a request seen by the caller, which is a `note`.
> > 2. that same request as seen by the callee, a `task`.
> > 3. the response to that first request as seen by the callee, a `gift`.
> > 4. the response to the first request as seen by the caller, a `sign`.
{: .callout}

To see how an event is processed in the Dojo, type `|verb` then `+ls %`.  After pressing `Enter`, you should see something like the following:

```
["" %unix %belt /d/term/1 ~2021.10.8..21.29.25..aa8c]
["|" %pass [%dill %g] [[%deal [~per ~per] %hood %poke] /] ~[//term/1]]
["||" %give %gall [%unto %poke-ack] i=/dill t=~[//term/1]]
["||" %give %gall [%unto %fact] i=/dill t=~[//term/1]]
["|||" %give %dill %blit i=/gall/use/herm/0wHkGtE/~per/view/ t=~[/dill //term/1]]
["|||" %give %dill %blit i=/gall/use/herm/0wHkGtE/~per/view/ t=~[/dill //term/1]]
["||" %pass [%gall %g] [ [%deal [~per ~per] %dojo %poke]   /use/hood/0wHkGtE/out/~per/dojo/drum/phat/~per/dojo ] ~[/dill //term/1]]
["|||" %give %gall [%unto %poke-ack] i=/gall/use/hood/0wHkGtE/out/~per/dojo/drum/phat/~per/dojo t=~[/dill //term/1]]
["" %unix %belt /d/term/1 ~2021.10.8..21.29.26..2862]
["|" %pass [%dill %g] [[%deal [~per ~per] %hood %poke] /] ~[//term/1]]
["||" %give %gall [%unto %poke-ack] i=/dill t=~[//term/1]]
["||" %give %gall [%unto %fact] i=/dill t=~[//term/1]]
["|||" %give %dill %blit i=/gall/use/herm/0wHkGtE/~per/view/ t=~[/dill //term/1]]
["|||" %give %dill %blit i=/gall/use/herm/0wHkGtE/~per/view/ t=~[/dill //term/1]]
["||" %pass [%gall %g] [ [%deal [~per ~per] %dojo %poke]   /use/hood/0wHkGtE/out/~per/dojo/drum/phat/~per/dojo ] ~[/dill //term/1]]
["|||" %give %gall [%unto %poke-ack] i=/gall/use/hood/0wHkGtE/out/~per/dojo/drum/phat/~per/dojo t=~[/dill //term/1]]
["" %unix %belt /d/term/1 ~2021.10.8..21.29.26..f451]
["|" %pass [%dill %g] [[%deal [~per ~per] %hood %poke] /] ~[//term/1]]
["||" %give %gall [%unto %poke-ack] i=/dill t=~[//term/1]]
["||" %give %gall [%unto %fact] i=/dill t=~[//term/1]]
["|||" %give %dill %blit i=/gall/use/herm/0wHkGtE/~per/view/ t=~[/dill //term/1]]
["|||" %give %dill %blit i=/gall/use/herm/0wHkGtE/~per/view/ t=~[/dill //term/1]]
["||" %pass [%gall %g] [ [%deal [~per ~per] %dojo %poke]   /use/hood/0wHkGtE/out/~per/dojo/drum/phat/~per/dojo ] ~[/dill //term/1]]
["|||" %give %gall [%unto %poke-ack] i=/gall/use/hood/0wHkGtE/out/~per/dojo/drum/phat/~per/dojo t=~[/dill //term/1]]
["" %unix %belt /d/term/1 ~2021.10.8..21.29.28..448d]
["|" %pass [%dill %g] [[%deal [~per ~per] %hood %poke] /] ~[//term/1]]
["||" %give %gall [%unto %poke-ack] i=/dill t=~[//term/1]]
[ "||" %pass [%gall %g] [ [%deal [~per ~per] %dojo %poke]   /use/hood/0wHkGtE/out/~per/dojo/drum/phat/~per/dojo ] ~[/dill //term/1]]
[ "|||" %give %gall [%unto %poke-ack] i=/gall/use/hood/0wHkGtE/out/~per/dojo/drum/phat/~per/dojo t=~[/dill //term/1]]
[ "|||" %give %gall [%unto %fact] i=/gall/use/hood/0wHkGtE/out/~per/dojo/drum/phat/~per/dojo t=~[/dill //term/1]]
["||||" %give %gall [%unto %fact] i=/dill t=~[//term/1]]
["|||||" %give %dill %blit i=/gall/use/herm/0wHkGtE/~per/view/ t=~[/dill //term/1]]
["|||||" %give %dill %blit i=/gall/use/herm/0wHkGtE/~per/view/ t=~[/dill //term/1]]
["|||||" %give %dill %blit i=/gall/use/herm/0wHkGtE/~per/view/ t=~[/dill //term/1]]
["|||" %pass [%gall %c] [%warp /use/dojo/0wHkGtE/~per/drum_~per/hand/gen/ls] ~[/dill //term/1]]
["||||" %give %clay %writ i=/gall/use/dojo/0wHkGtE/~per/drum_~per/hand/gen/ls t=~[/dill //term/1]]
["|||||" %give %gall [%unto %fact] i=/gall/use/hood/0wHkGtE/out/~per/dojo/drum/phat/~per/dojo t=~[/dill //term/1]]
["||||||" %give %gall [%unto %fact] i=/dill t=~[//term/1]]
["|||||||" %give %dill %blit i=/gall/use/herm/0wHkGtE/~per/view/ t=~[/dill //term/1]]
["|||||" %give %gall [%unto %fact] i=/gall/use/hood/0wHkGtE/out/~per/dojo/drum/phat/~per/dojo t=~[/dill //term/1]]
["|||||" %give %gall [%unto %fact] i=/gall/use/hood/0wHkGtE/out/~per/dojo/drum/phat/~per/dojo t=~[/dill //term/1]]
> +ls %
app/ gen/ lib/ mar/ sur/ sys/ ted/ tests/
```

This involves Dill, Gall, and Clay.

There is an excellent [“move trace” tutorial](https://urbit.org/docs/arvo/tutorials/move-trace) on Urbit.org which covers this in detail.  We don't need to deeply understand this terminology to understand that events are generated by vanes, dispatched by Arvo, and resolved by vanes.

> An interrupted event never happened.  The computer is deterministic; an event is a transaction; the event log is a log of successful transactions. In a sense, replaying this log is not Turing complete. The log is an existence proof that every event within it terminates.

- [Tlon Corporation, “Move Trace”](https://urbit.org/docs/arvo/tutorials/move-trace)

### Arvo as Standard Noun Structure

Arvo defines five standard arms for vanes and the binary runtime to use:

-  `++peek` grants read-only access to a vane; this is called a _scry_.
-  `++poke` accepts `++move`s and processes them; this is the only arm that actually alters Arvo's state.
-  `++wish` accepts a core and parses it against `%zuse`, which is instrumentation for runtime access.
-  `++come` and `++load` are used in kernel upgrades, allowing Arvo to update itself in-place.

Each arm possesses this same structure, which means that as the Urbit OS kernel grows and changes the main event dispatcher can remain the same.  For instance, when the build vane `%ford` was incorporated into `%clay`, no brain surgery was needed on Arvo to make this possible and legible.  Only the affected vanes (and any calls to `%ford`) needed to change.

- [Tlon Corporation, “Arvo Tutorial”](https://urbit.org/docs/arvo/overview/)

{% include links.md %}
