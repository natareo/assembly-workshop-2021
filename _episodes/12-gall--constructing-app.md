---
title: "`%gall`:  Constructing a Novel App"
teaching: 0
exercises: 45
questions:
- "How can I build a `%gall` app starting from a minimal working example?"
objectives:
- "Produce a new `%gall` app with specified behavior."
keypoints:
- "A `%gall` app can be readily produced to a standard of operability."
---

> ##  Building `%delta` to Count Pokes
>
> _This entire section is a team exercise.  You should work in a GitHub repository to which you all have access and employ basic pair programming to talk through your solution process._
>
> Your team's objective is to update the `%charlie` agent to a new `%delta` agent, which does the following:
>
> 1. Accept remote pokes from another ship.  Pokes should be either increment or decrement requests.
> 2. Upon receipt of an increment request, increment a count in a map of `@p`→`@ud` for each ship.  Add the key if necessary.
> 3. Upon receipt of a decrement request, decrement the corresponding counter.
> 4. In either case, print the resulting value on the local ship and send a poke to the remote ship to make it print the result as well.
>
> > ##  Hints
> >
> > - While it is possible to set up a remote Urbit testnet, for our purposes it is simpler to test the agent using two (or more) fakezods on a single computer.  (You can also use several comets on the live network.)
> >
> >     Of course, since you can only have _one_ `~zod`, you should pick another ship (galaxy, star, or planet).  Use `@p` on any unsigned integer for inspiration, then boot a new clean ship and backup as on Day One.
> >
> > - You can distinguish output lines visually using the `>`/`>>`/`>>>` syntax for `~&`:
> >
> >     ```
> >     ~&  >  "log"        :: blue (log)
> >     ~&  >>  "warning"   :: yellow (warning)
> >     ~&  >>>  "error"    :: red (error)
> >     ```
> >
> > - You should store the sending ship and counter as a `map`.  Use the `++by` door to work with `map`s.
> >
> > - Most of the arms can be left alone as defaults:  for instance, there is no subscription in this model.
> {: .solution}
{: .challenge}

{% include links.md %}
