---
title: "`%gall`:  A Minimal Working Example"
teaching: 20
exercises: 10
questions:
- "What are the structure and expectations of a `%gall` app?"
objectives:
- "Understand how `%gall` instruments an app."
- "Produce a basic `%gall` app."
keypoints:
- "A `%gall` app expects certain arms to be present to handle and emit events."
---

##  “It was six men of Indostan,/To learning much inclined”

On the first day, we talked about bones and trunks and ivory.  Today we're going to meet an elephant.

![](../img/elephant-noloop.gif)

Unfortunately, there's not really a good way to modulate the sudden jump in complexity we're encountering now.  You can't really build an airplane from just a wing:  you have to leap from "wing" to "airplane" in one go.

![](https://media1.giphy.com/media/10QGikPE4pIRwY/giphy.gif)

We will proceed at first by simply providing examples of fully-formed `%gall` agents and discussing their structure and salient features.  As our objective today is relatively modest—being able to write and deploy a simple app—this should suffice to whet your appetite.

##  A Minimal Working Example

Before we do anything substantial with `%gall`, however, we are simply going to look at a minimal working example.  This is the equivalent of a `pass` statement, it does nothing and talks to no one, whistling in the dark.

**`app/egg.hoon`**

```hoon
/-  graph-store
/+  default-agent, dbug
|%
+$  versioned-state
  $%  state-0
  ==
::
+$  state-0
  $:  [%0 hexes=`(list @ux)`]
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
  ~&  >  '%egg initialized successfully'
  =.  state  [%0 *(list @ux)]
  `this(state prev)
  ==
++  on-save   on-save:default
++  on-load   on-load:default
++  on-poke
  |=  [=mark =vase]
  ^-  (quip card _this)
  =^  cards  state
    ?+    mark  (on-poke:default mark vase)
    ::
      %take-action
    (handle-action:main !<(action vase))
++  on-arvo   on-arvo:default
++  on-watch  on-watch:default
++  on-leave  on-leave:default
++  on-peek   on-peek:default
++  on-agent  on-agent:default
++  on-fail   on-fail:default
--
|_  =bowl:gall
++  handle-action
  |=  =action
  ^-  (quip card _state)
  ?-    -.action
    ::
      %append-value
    =.  hexes.state  (weld hexes.state value.action)
    :_  state
    ~[[%give %fact ~[/hexes] [%atom !>(hexes.state)]]]
  ==
--
```

- What do you recognize here?  What is unfamiliar?

Install this agent by copying in these files:

```sh
cp -r gall-egg/* zod/home
```

Run this

```hoon
:egg &take-action ~[%append-value 0xdead.beef]
```

At each point, you can check the internal state using the `debug` subject wrapper:

```hoon
:egg +dbug
```

Any nontrivial app needs to define some shared files, which is one of the reasons this is an elephant.  In particular, a shared structure in `sur/` and a mark in `mar/` are required to handle data transactions.

The shared structure defines common molds like actions and expected structural definitions (like tagged unions or associative arrays).

**`sur/egg.hoon`**

```hoon
|%
+$  action
  $%  [%append-value value=@ud]
  ==
--
```

Most marks are straightforward or can be developed by glancing

**`mar/action/egg.hoon`**

```hoon
/-  egg
|_  action=action:egg
++  grab
  |%
  ++  noun  action:egg
  --
++  grow
  |%
  ++  noun  action
  --
++  grad  %noun
--
```

Exercise:

- Examine `sur/file-server.hoon`
- Examine `mar/json.hoon`

Every Gall agent is a door with two components in its subject:

1. `bowl:gall` for Gall-standard tools and data structures
2. App state information

The `bowl` is a collection of information which renders the agent legible to Arvo, such as providing the subscriptions:

```hoon
++  bowl              ::  standard app state
  $:  $:  our=ship    ::  host
          src=ship    ::  guest
          dap=term    ::  agent
      ==              ::
      $:  wex=boat    ::  outgoing subscriptions
          sup=bitt    ::  incoming subscriptions
      ==              ::
      $:  act=@ud     ::  change number
          eny=@uvJ    ::  entropy
          now=@da     ::  current time
          byk=beak    ::  load source
  ==  ==
```

For instance, the incoming subscriptions are a map from the `duct` (or `(list path)`) to a particular path on a particular ship.

```hoon
+$  bitt  (map duct (pair ship path))
```

The `duct` is the main construct for tracking information.  (Think back to our discussion of scrying:  this is the same concept in new clothes.)  The `path` or `wire` (same thing) bears a characteristic structure for each vane.  For instance, a directory listing from `%clay` is a simple path into `/c` with a tag `y` indicating the type of request:

```hoon
> `path`[%cy /===/sys/vane]
/cy/~sev/home/~2021.9.17..17.14.19..3635/sys/vane
> `(list @t)`[%cy /===/sys/vane]
<|cy ~sev home ~2021.9.17..17.15.48..8a3f sys vane|>
> `(list @ta)`[%cy /===/sys/vane]
/cy/~sev/home/~2021.9.17..17.15.58..ccd9/sys/vane
> `(list @tas)`[%cy /===/sys/vane]
~[%cy %~sev %home %~2021.9.17..17.16.01..a51e %sys %vane]
```

(This `path` can be directly executed with `.^` dotket as follows:  `.^(arch %cy /===/sys/vane)`.)

A `%gall` `path` could look like this:

```hoon
> `path`[%gx /=settings-store=/has-entry/urbit-agent-permissions/'http://localhost:3000'/noun]
```

So this is _called_ a `path` but it's really a complete package of request type, agent information, and data with metadata.

> ##  Agents and Apps
>
> The terminology for userspace has not yet completely solified.  I strive to use “agent” to refer to a particular running instance, like a “container” in Docker, whereas an “app” is more like a Docker “image”, the archetypal instance.  However, I will frequently and inadvertently use “agent” in synecdoche to refer to apps.
{: .callout}

{% include links.md %}
