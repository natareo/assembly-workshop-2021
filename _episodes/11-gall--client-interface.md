---
title: "`%gall`:  Interfacing with a Client"
teaching: 60
exercises: 30
questions:
- "How can I build a `%gall` app which operates on data?"
objectives:
- "Produce an intermediate `%gall` app."
- "Understand how `%gall` interfaces with an external client."
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

A scry represents a direct look into the agent state using the Nock `.^` dotket operator.

Only local scries are permitted.

### `++on-poke` Request

A poke initiates some kind of well-defined action by an agent.  Typically this either triggers an event (such as `charlie` and `bravo`'s modification of `hexes`) or requests a data return of some kind.

Remote pokes are allowed (and common for single-instance requests).

### `++on-watch` Subscription

A subscription is a data-reactive standing request for changes.  For instance, one can watch a database agent for any changes to the database.  Whenever a change occurs, the agent notifies all subscribers, who then act as they should in the event of a message being received (e.g. from a particular ship).

Remote subscriptions are in common use.

###  The agent

`%charlie` is yet another upgrade of `%bravo` which allows remote ships to poke each other and push hex values to or pop hex values from each others' `hexes`:

**`/app/charlie.hoon`**:

```hoon
/-  charlie
/+  default-agent, dbug
|%
+$  versioned-state
  $%  state-0
  ==
::
+$  state-0
  $:  [%0 hexes=(list @ux)]
  ==
::
+$  card  card:agent:gall
::
--
%-  agent:dbug
=|  state-0
=*  state  -
^-  agent:gall
=<
|_  =bowl:gall
+*  this      .
    default   ~(. (default-agent this %|) bowl)
    main      ~(. +> bowl)
::
++  on-init
  ^-  (quip card _this)
  ~&  >  '%charlie initialized successfully'
  =.  state  [%0 *(list @ux)]
  `this
++  on-save   on-save:default
++  on-load   on-load:default
++  on-poke
  |=  [=mark =vase]
  ^-  (quip card _this)
  ?+    mark  (on-poke:default mark vase)
      %noun
    ?+    q.vase  (on-poke:default mark vase)
        %print-state
      ~&  >>  state
      ~&  >>>  bowl  `this
    ==
    ::
      %charlie-action
    ~&  >  %charlie-action
    =^  cards  state
    (handle-action:main !<(action:charlie vase))
    [cards this]
  ==
++  on-arvo   on-arvo:default
++  on-watch  on-watch:default
++  on-leave  on-leave:default
++  on-peek
  |=  =path
  ^-  (unit (unit cage))
  ?+    path  (on-peek:default path)
      [%x %hexes ~]
    ``noun+!>(hexes)
  ==
++  on-agent  on-agent:default
++  on-fail   on-fail:default
--
|_  =bowl:gall
++  handle-action
  |=  =action:bravo
  ^-  (quip card _state)
  ?-    -.action
    ::
      %push
    =.  hexes.state  (weld hexes.state ~[value.action])
    ~&  >>  hexes.state
    :_  state
    ~[[%give %fact ~[/hexes] [%atom !>(hexes.state)]]]
    ::
      %pop
    =.  hexes.state  (snip hexes.state)
    ~&  >>  hexes.state
    :_  state
    ~[[%give %fact ~[/hexes] [%atom !>(hexes.state)]]]
  ==
--
```

https://github.com/urbit/docs/pull/892/files
https://urbit.org/docs/userspace/graph-store/sample-application-overview

- [“External API Reference”](https://urbit.org/docs/arvo/eyre/external-api-ref)

> ##  Permissions on Mars (Optional)
>
> Many Gall apps use Graph Store, a backend data storage format and database that both provides internally consistent data and external API communications endpoints.
>
> To understand Graph Store, think in terms of the Urbit data permissions model:
>
> - A _store_ is a local database.
> - A _hook_ is a permissions broker for the database.  They request and return information after negotiating access with remote agents.
> - A _view_ is a data aggregator which parses JSON objects for external clients such as Landscape.
>
> Graph Store handles data access perms at the hook level, not at the store level.
>
> TODO
>
> https://gist.github.com/matildepark/268c758c079d6cf83fd1541b2430ff7f
{: .callout}

{% include links.md %}
