---
title: "`%gall`:  Interfacing with a Client"
teaching: 60
exercises: 30
questions:
- "How can I build a `%gall` app which operates on data?"
objectives:
- "Produce an intermediate `%gall` app."
- "Understand how `%gall` interfaces with an external client like Landscape."
- "Retrieve and store information in `graph-store`."
- "Apply prior learning to understand how Clock works in Landscape."
keypoints:
- "A `%gall` app can talk to a user interface client."
---

##  Communications

![](https://i.pinimg.com/originals/19/b4/d0/19b4d0df3b42cae635a32cb2daf69dc4.gif)

https://urbit.org/docs/userspace/graph-store/sample-application-overview

We need to examine all of the ways a `%gall` app can communicate with the outside world.  Recall that an agent has ten arms:

```hoon
|_  =bowl:gall
++  on-init
++  on-save
++  on-load
++  on-arvo
++  on-peek
++  on-poke
++  on-watch
++  on-leave
++  on-agent
++  on-fail
--
```

Arvo alone interacts with several of these:

```hoon
++  on-init
++  on-save
++  on-load
++  on-arvo
```

The `++on-agent` and `++on-fail` arms are called in certain circumstances (i.e. as an update to a subscription to another agent or cleanup after a `%poke` crash).  We can leave them as boilerplate for now.

If you are _exposing_ information, you can do so via a peek (`++on-peek`), a response to a poke (`++on-poke`), or a subscription (`++on-watch`).  (`++on-leave` handles cleanup after a terminated subscription.)  These are the main ways that the Urbit API protocol (formerly Airlock) interacts with an agent on a ship.  We'll focus on these.

### `++on-peek` Scry

A scry

### `++on-poke` Scry

A poke

### `++on-watch` Scry

A subscription is a data-reactive

###  The agent

**`/app/pupa.hoon`**:

https://urbit.org/docs/userspace/graph-store/sample-application-overview

- [“External API Reference”](https://urbit.org/docs/arvo/eyre/external-api-ref)

##  `graph-store`

Many Gall apps now use `graph-store`, a backend data storage format and database that both provides internally consistent data and external API communications endpoints.

`graph-store` handles data access perms at the hook level, not at the store level.

https://gist.github.com/matildepark/268c758c079d6cf83fd1541b2430ff7f

{% include links.md %}
