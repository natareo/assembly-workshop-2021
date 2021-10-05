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

Arvo defines a number of standard operations for each vane.  Notable among these are peeks, which grant read-only access to data, called a _scry_; and pokes, which accept moves and process them.  Pokes actually alter the agent's (and Arvo's) state (rather than just retrieve information).

We are going to widen our view a little bit as well with this agent:  we will not use the `default` arms but will define our own `NOP` defaults.  This way you will be able to see what sort of information each arm processes.

**`/app/bravo.hoon`**:

```hoon
/-  bravo
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
|_  bowl:gall
+*  this      .
    default   ~(. (default-agent this %|) bowl)
    main      ~(. +> bowl)
::
++  on-init
  ^-  (quip card _this)
  ~&  >  '%bravo initialized successfully'
  =.  state  [%0 *(list @ux)]
  `this
::
++  on-save
  ^-  vase
  !>(state)
::
++  on-load
  |=  old-state=vase
  ^-  (quip card _this)
  ~&  >  '%bravo recompiled successfully'
  `this(state !<(versioned-state old-state))
::
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
      %bravo-action
    ~&  >  %bravo-action
    =^  cards  state
    (handle-action:main !<(action:bravo vase))
    [cards this]
  ==
::
++  on-watch
  |=  =path
  `this
::
++  on-leave
  |=  =path
  `this
::
++  on-peek
  |=  =path
  *(unit (unit cage))
::
++  on-agent
  |=  [wire sign:agent:gall]
  `this
::
++  on-arvo
  |=  [=wire =sign-arvo]
  `this
::
++  on-fail
  |=  [=term =tang]
  `this
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

You should copy the structure file and mark file from `%alfa` and adapt them as appropriate for `%bravo`.  This should be a matter of copying to the appropriate path and changing any internal references.

The structure file should accommodate the following actions:

- `%push` (equivalent in effect to the old `%append-value`), `[%push value=@ux]`
- `%pop` (will remove the most recent item from the list and return it), `[%pop ~]`

We will also accommodate external scrying into the agent through the `++on-peek` arm.  Once the above works correctly, you should add in an augmented `++on-peek` arm:

```hoon
++  on-peek
  |=  pax=path
  ^-  (unit (unit cage))
  ?+    pax  (on-peek:def pax)
      [%x %hexes ~]
    ``noun+!>(hexes)
    ::
      [%x %no-result ~]
    [~ ~]
  ==
```

This arm typically accepts two kinds of scries (called _cares_):

- `%x` represents data.  `%x` scries typically return a `cage` with mark.
- `%y` represents paths.  `%y` will return a `cage` of mark `%arch` and vase type `arch`.

For this case, we only need to return data, so we will only support `%gx` scries.

Both of these results are directly accessible via `.^` dotket scries:

```hoon
.^((list @ux) %gx /=bravo=/hexes/noun)
.^(noun %gx /=bravo=/no-result/noun)
```

> ##  `unit`s, `cage`s, and `vase`s, Oh My!
>
> - A `unit` allows us to distinguish "no result" from "zero result".  Since every atom in Hoon is an unsigned integer, this allows us to tell the difference between an operation that has no possible result and an operation that succeeded but returned `~` or `0`.  A `unit` can be trivially produced from any value by prefixing a tic mark `\``.
>
> - A `vase` wraps a value in its type.  A `vase` generally results from the type spear `-:!>()`.
>
> - A `cage` is a marked vase; that is, a `vase` with additional information about its structure.  A `cage` is more or less analogous to a file in a regular filesystem.
>
> These bear the following relationship to a simple atom:
>
> ```hoon
> > !>(1)
> [#t/@ud q=1]
> > (vase !>(1))
> [#t/@ud q=1]
> > (cage `(vase !>(1)))
> [p=%$ q=[#t/@ud q=1]]
> ```
>
> (That last `cage`'s `p` means that the value is a constant.)
>
> We would be remiss to not also address `arch`:
>
> - An `arch` is basically a file directory (in `%clay`) or a list of store paths (in `%gall`).
{: .callout}

With the completion of this exercise, you have seen how to alter and query state using command-line and agent-based tools.  Next, we will take a look at other means for manipulating agent state.

{% include links.md %}
