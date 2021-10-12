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

**`/app/alfa.hoon`**

```
/-  alfa
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
  ~&  >  '%alfa initialized successfully'
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
      %alfa-action
    ~&  >  %alfa-action
    =^  cards  state
    (handle-action:main !<(action:alfa vase))
    [cards this]
  ==
++  on-arvo   on-arvo:default
++  on-watch  on-watch:default
++  on-leave  on-leave:default
++  on-peek   on-peek:default
++  on-agent  on-agent:default
++  on-fail   on-fail:default
--
|_  =bowl:gall
++  handle-action
  |=  =action:alfa
  ^-  (quip card _state)
  ?-    -.action
    ::
      %append-value
    =.  hexes.state  (weld hexes.state ~[value.action])
    ~&  >>  hexes.state
    :_  state
    ~[[%give %fact ~[/hexes] [%atom !>(hexes.state)]]]
  ==
--
```

- What do you recognize here?  What is unfamiliar?

Install this agent by copying in these files:

```sh
cp -r src/gall-alfa/* zod/home
```

(We'll use a shell script introduced a bit later on to assist with this.)

Until a couple of weeks ago, the syntax to start a `%gall` agent was `|start`.  As of Grid's release, this has switched to `|rein`.  Run this by itself to see what it expects as input arguments:

```
dojo> |rein
>   dojo: nest-need
[ [now=@da eny=@uvJ bec=[p=@p q=@tas r=?([%da p=@da] [%tas p=@tas] [%ud p=@ud])]]
  [desk=@tas arg=it([?(%.y %.n) @tas])]
  liv=?(%.y %.n)
]
>   dojo: nest-have
[ [now=@da eny=@uvJ bec=[p=@p q=@tas r=?([%da p=@da] [%tas p=@tas] [%ud p=@ud])]]
  %~
  liv=?(%.y %.n)
]
```

Run the following:

```
|rein %bitcoin [& %btc-provider]
|rein %echo [& alfa]
:alfa &alfa-action append-value+0xdead.beef
:alfa %print-state
```

At each point, you can check the internal state using the `debug` subject wrapper:

```
:alfa +dbug
```

Any nontrivial app needs to define some shared files, which is one of the reasons this is an elephant.  In particular, a shared structure in `/sur` and a mark in `/mar` are required to handle data transactions.

The shared structure defines common molds like actions and expected structural definitions (e.g. as tagged unions or associative arrays).

**`/sur/alfa.hoon`**

```
|%
+$  action
  $%  [%append-value value=@ux]
  ==
--
```

Most marks are straightforward or can be developed by glancing at others.

**`/mar/alfa/action.hoon`**

```
/-  alfa
|_  =action:alfa
++  grab
  |%
  ++  noun  action:alfa
  --
++  grow
  |%
  ++  noun  action
  --
++  grad  %noun
--
```

> ##  Exercise
>
> - Examine `/sur/dns.hoon` (basic) or `/sur/bitcoin.hoon` (advanced).
> - Examine `/mar/json.hoon`.
{: .challenge}

> ## A Shell Script
>
> Place this shell script into your root working directory and use it to update each `%gall` agent:
>
> ```sh
> #! /bin/sh
> mkdir -p $1/home/mar/$2
> yes | cp $2/src/mar/action.hoon $1/home/mar/$2
> yes | cp $2/src/sur/$2.hoon $1/home/sur
> yes | cp $2/src/app/$2.hoon $1/home/app
> ```
>
> Usage:
>
> ```sh
> ./copy-in.sh zod alfa
> ```
>
> Note that this assumes you have already run `|mount %` and that you run `|commit %home` after each Unix-side update.
{: .callout}

### How It Works

Every Gall agent is a door with two components in its subject:

1. `bowl:gall` for Gall-standard tools and data structures
2. App state information

The `bowl` is a collection of information which renders the agent legible to Arvo, such as providing the subscriptions:

```
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

```
+$  bitt  (map duct (pair ship path))
```

The `duct` is the main construct for tracking information.  (Think back to our discussion of scrying:  this is the same concept in new clothes.)  The `path` or `wire` (same thing) bears a characteristic structure for each vane.  For instance, a directory listing from `%clay` is a simple path into `/c` with a tag `y` indicating the type of request:

```
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

```
> `path`[%gx /=settings-store=/has-entry/urbit-agent-permissions/'http://localhost:3000'/noun]
```

So this is _called_ a `path` but it's really a complete package of request type, agent information, and data with metadata.

> ##  Agents v. Apps
>
> The terminology for userspace has not yet completely solified.  I strive to use “agent” to refer to a particular running instance, like a “container” in Docker, whereas an “app” is more like a Docker “image”, the archetypal instance.  However, I will frequently and inadvertently use “agent” in synecdoche to refer to apps.
{: .callout}

{% include links.md %}
