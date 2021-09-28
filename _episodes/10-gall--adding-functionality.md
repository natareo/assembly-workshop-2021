---
title: "`%gall`:  Adding Functionality"
teaching: 30
exercises: 15
questions:
- "How can I build a `%gall` app which operates on data?"
objectives:
- "Manage internal `%gall` state."
- "Scry for needed information."
keypoints:
- "A `%gall` app can be outfitted with a helper core to provide necessary operations."
---

Generally speaking, the Urbit data flow model is _reactive_, meaning that rather than poll for updates periodically one _subscribes_ to a data source which notifies all subscribers when a change occurs.

Arvo defines a number of standard operations for each vane.  Notable among these are `++peek`, which grants read-only access to data, called a _scry_; and `++poke`, which accepts moves and processes them.  Pokes actually alter Arvo's state (rather than just retrieve information).

We are going to widen our view a little bit as well with this agent:  we will not use the `default` arms but will define our own NOP defaults.  This way you will be able to see what sort of information each arm processes.

**`app/grub.hoon`**:

```hoon
/+  dbug
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
%-  agent:dbug
=|  state-0
=*  state  -
^-  agent:gall
=<
|_  bowl:gall
+*  this      .
    default   ~(. (default-agent this %|) bowl)
    main      ~(. +> bowl)
::
++  on-init
  `this
::
++  on-save
  !>(state)
::
++  on-load
  |=  =old-state=vase
  `this(state !<(@ old-state-vase))
::
++  on-poke
  |=  [=mark =vase]
  ~&  >  state=state
  ~&  got-poked-with-data=mark
  =.  state  +(state)
  `this
::
++  on-watch
  |=  path
  `this
::
++  on-leave
  |=  path
  `this
::
++  on-peek
  |=  path
  *(unit (unit cage))
::
++  on-agent
  |=  [wire sign:agent:gall]
  `this
::
++  on-arvo
  |=  [wire sign-arvo]
  `this
::
++  on-fail
  |=  [term tang]
  `this
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

{% include links.md %}
