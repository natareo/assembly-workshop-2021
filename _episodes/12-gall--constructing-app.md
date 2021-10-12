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
> _This entire section is a team exercise.  You should work in a GitHub repository to which you all have access.  [Import](https://github.com/new/import) (don't fork) the `https://github.com/sigilante/gall-delta` repo to one of your team's accounts and give the other members collaborator access to the repo (in [Settings](https://docs.github.com/en/account-and-profile/setting-up-and-managing-your-github-user-account/managing-user-account-settings/permission-levels-for-a-user-account-repository))._
>
> Your team's objective is to update the `%delta` agent to do the following:
>
> 1. Accept `PUT` messages of JSON (via `curl`, for instance).  The message should contain a target ship.
> 2. Upon receipt of a JSON request, extract the target ship and send a poke.
> 3. Upon receipt of a poke, increment a counter for that ship and display the current count.  (E.g., `~wes: 5`)
>
> ### Notes
>
> - While it is possible to set up a remote Urbit testnet, for our purposes it is simpler to test the agent using two (or more) fakezods on a single computer.
>
>     Of course, since you can only have _one_ `~zod`, you should pick another ship (galaxy, star, or planet).  Use `@p` on any unsigned integer for inspiration, then boot a new clean ship and backup as on Day One.
>
> - You don't need to worry about using `graph-store`, accessing Landscape, or any of the particular value-adds of those libraries.  Focus on making a plain-vanilla CLI `%gall` agent.
>
> - You can distinguish output lines visually using the `>`/`>>`/`>>>` syntax for `~&`:
>
>     ```
>     ~&  >  "log"        :: blue (log)
>     ~&  >>  "warning"   :: yellow (warning)
>     ~&  >>>  "error"    :: red (error)
>
> - You should store the sending ship and counter as a `map`.  Use the `++by` door to work with `map`s.
>
> - Most of the arms can be left alone as defaults:  for instance, there is no subscription in this model.
>
> - JSON objects can be parsed using `++de-json:html` and `++dejs`.  However, you need a parser for `++dejs` to work, and a parser requires a basic JSON schema.
>
>     ```
>     ++  req-parser-req                  :: parser for PUT request
>       %-  ot:dejs-soft:format
>       :~  [%ship so:dejs-soft:format]   :: only bother extracting these
>           [%json so:dejs-soft:format]   ::   two keys from the JSON
>       ==
>     ::
>     :: ...
>     ::
>     =/  myjson  '{"ship":"zod","action":"poke","app":"delta","json":"~nec"}","mark":"noun"}'
>     =/  payload  (de-json:html `@t`+511:req)
>     ?~  payload  !!
>     =/  payload-array  u:+.payload
>     =/  st  u:+:(req-parser-ot payload-array)
>     =/  source-t  (trip -.st)
>     =/  source
>       ?:  =('~' (snag 0 source-t))  u:+:`(unit @p)`(slaw %p (crip source-t))
>       u:+:`(unit @p)`(slaw %p (crip (weld "~" source-t)))
>     =/  target-t  (trip +.st)
>     =/  target
>       ?:  =('~' (snag 0 target-t))  u:+:`(unit @p)`(slaw %p (crip target-t))
>       u:+:`(unit @p)`(slaw %p (crip (weld "~" target-t)))
>     ```
>
> - To send a JSON object to the Urbit ship with a `PUT` request, use `curl`:
>
>     ```sh
>     curl \
>     --header "Content-Type: application/json" \
>     --request PUT \
>     --data '{"ship":"zod","action":"poke","app":"delta","json":"~nec","mark":"noun"}' \
>     http://localhost:8080/~checkAuth
>     ```
{: .exercise}

> ## Secure Messages
>
> For this example, we are omitting obtaining an authentication token and providing a cookie with each `curl` action.  In production, you will commonly use the `++require-authorization:app:server` gate to make sure that any outside calls are in fact authorized.
>
> There is a good example of this usage in `~timluc-miptev`'s [`%gall` tutorial](https://github.com/timlucmiptev/gall-guide/blob/master/example-code/app/gall-test2.hoon#L66) and in the [“`%eyre` External API Reference” docs](https://urbit.org/docs/arvo/eyre/external-api-ref).
{: .callout}

{% include links.md %}
