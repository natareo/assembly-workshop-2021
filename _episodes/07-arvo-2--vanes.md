---
title: "Arvo:  Vane Roles"
teaching: 20
exercises: 0
questions:
- "What services do Arvo vanes provide?"
- "How do I access system operations?"
objectives:
- "Understand architecture of the `%ames` network and communications."
- "Access file data from `%clay`."
- "Access arbitrary revisions of a file through `%clay`."
- "Manipulate remote data through `%eyre`/`%iris`."
- "Provide external commands to Arvo through `%khan`."
keypoints:
- "Vanes communicate by means of events."
- "`%ames` provides network communications including peer-to-peer events."
- "`%clay` instruments a typed version-controlled filesystem."
- "`%eyre` and `%iris` together offer client and server operations."
- "`%khan` is the external control plane."
---

##  Arvo Vanes

Each vane has a characteristic structure which identifies it as a vane to Arvo and allows it to handle moves consistently.  Most operations are one of three things:

1. A _scry_ or request for data (`++peek` or `++scry` or `++on-peek`).
2. An update (`++poke`).
3. An evaluation (of a core) (`++wish`).

In order to orient yourself around the kinds of things the Urbit OS does, it is worth a brief tour of the Arvo vanes.  Each vane offers a particular system service.  These are sometimes represented schematically as the kernel layer around the Arvo state machine:

![](https://media.urbit.org/site/understanding-urbit/technical-overview/technical-overview-kernel@2x.png)

### `%ames`, A Network

In a sense, `%ames` is the operative definition of an Urbit ship on the network.  That is, from outside of one's own urbit, the only specification that must be hewed to is that `%ames` behaves a certain way in response to events.  (Of course, without a fully operational ship behind the `%ames` receiver not much would happen.)

`%ames` implements a system expecting—and delivering—guaranteed one-time delivery.  This derives from an observation in the Whitepaper:

> There is a categorical difference between a bus, which transports commands, and a network, which transports packets. You can drop a packet but not a command; a packet is a fact and a command is an order. To send commands over a wire is unforgivable: you’ve turned your network into a bus. Buses are great, but networks are magic.
>
> Facts are inherently idempotent; learning fact X twice is the same as learning it once. You can drop a packet, because you can ignore a fact. Orders are inherently sequential; if you get two commands to do thing X, you do thing X twice.

`%ames` communicates using the UDP protocol.  Scries into `%ames` typically locate ship and peer information, protocol version, ship state, etc.


### `%behn`, A Timer

`%behn` is a simple vane that promises to emit events after—but never before—their timestamp.  This is used as a wake-up timer for many deferred events.  `%behn` maintains an event handler and a state.

As the shortest vane, we commend `%behn` to the student as an excellent subject for a first dive into the structure of a vane.

`%behn` scries retrieve timers, timestamps, next timer to fire, etc.


### `%clay`, A File System

`%clay` is one of the most significant vanes in Arvo.  `%clay` is a global-namespace typed version-control filesystem, meaning that it can 1) refer to any value on any ship (although it may not be able to _access_ said value); 2) has a type for each noun it holds; and 3) retains a full history of the file system.



For instance, to read a local file, use `++warp`:  TODO

```
~tinnus-napbus
3:57 PM
what is the correct way to read a file on a remote ship? I've tried both warp and werp and I'm not getting a response, just messages in the target about clay something something indirect and then I get crash on fragment errors sometimes
with an %x care
~rovnys-ricfer
4:02 PM
warp should work, I think
make sure the file is actually there
i.e. can you do the same warp on the local ship?
```

\begin{itemize}
  \item  \texttt{aeon} is
  \item  \texttt{arch} is
  \item  \texttt{care} is the `%clay` ~submode, defined in \lull.
  \item  \texttt{desk} is
  \item  \texttt{dome} is
  mark cast file
\end{itemize}


\subsection{Data Model}
\labsec{kr:c:data}

paths

merges

marks

\subsection{Scrying into `%clay` }
\labsec{kr:c:scry}

`%clay` ~has the most sophisticated scry taxonomy.

\begin{table}[h!]
  \begin{center}
    \caption{`%clay` ~\dotket~Calls.}
    \label{ha:clay}
    \begin{tabular}{lll}
      Symbol & Meaning & Example \\
      \hline \\
      \texttt{\%a} & Expose file build into namespace. & \texttt{\dotket (vase \%ca /\~zod/home/1/lib/generators/hoon} \\
      \texttt{\%b} & Expose mark build into namespace. & \\
      \texttt{\%c} & Expose cast build into namespace. & \\
      \texttt{\%d} & diff; ask clay to validate .noun as .mark & \\
      \texttt{\%e} & & \\
      \texttt{\%f} & & \\
      \texttt{\%p} & Get the permissions that apply at path. & \\
      \texttt{\%r} & Get the data at a node (like \texttt{\%x}) and wrap it in a vase. & \\
      \texttt{\%s} & produce yaki or blob for given tako or lobe & \\
      \texttt{\%t} & produce the list of paths within a yaki with :pax as prefix & \\
      \texttt{\%u} & Check for existence of a node at aeon. & \\
      \texttt{\%v} & Get the desk state at a specified aeon. & \\
      \texttt{\%w} & Get all cases referring to the same revision as the given case. & \\
      \texttt{\%x} & Get the data at a node. & \\
      \texttt{\%y} & Get the \texttt{arch} (directory listing) at a node. & \\
      \texttt{\%z} & Get a recursive hash of a node and its children. & \\
    \end{tabular}
  \end{center}
\end{table}

% https://github.com/urbit/urbit/blob/master/pkg/arvo/sys/vane/gall.hoon#L1538
% https://urbit.org/blog/ford-fusion/

Now that you have seen a little more of how vanes work, take a gander at the `+ls` and `+cat` generators.  (These are the most complicated generators we'll see.)

```hoon
> +cat %/gen/ls/hoon
/~per/home/~2021.10.8..19.11.55..0a8a/gen/ls/hoon
::  LiSt directory subnodes
::
::::  /hoon/ls/gen
  ::
/?    310
/+    show-dir
::
::::
  ::
~&  %
:-  %say
|=  [^ [arg=path ~] vane=?(%g %c)]
=+  lon=.^(arch (cat 3 vane %y) arg)
tang+[?~(dir.lon leaf+"~" (show-dir vane arg dir.lon))]~
```

```hoon
> +cat %/gen/cat/hoon
/~per/home/~2021.10.8..19.06.15..bc83/gen/cat/hoon
::  ConCATenate file listings
::
::::  /hoon/cat/gen
  ::
/?    310
/+    pretty-file, show-dir
::
::::
  ::
:-  %say
|=  [^ [arg=(list path)] vane=?(%g %c)]
=-  tang+(flop `tang`(zing -))
%+  turn  arg
|=  pax=path
^-  tang
=+  ark=.^(arch (cat 3 vane %y) pax)
?^  fil.ark
  ?:  =(%sched -:(flop pax))
    [>.^((map @da cord) (cat 3 vane %x) pax)<]~
  [leaf+(spud pax) (pretty-file .^(noun (cat 3 vane %x) pax))]
?-     dir.ark                                          ::  handle ambiguity
    ~
  [rose+[" " `~]^~[leaf+"~" (smyt pax)]]~
::
    [[@t ~] ~ ~]
  $(pax (welp pax /[p.n.dir.ark]))
::
    *
  =-  [palm+[": " ``~]^-]~
  :~  rose+[" " `~]^~[leaf+"*" (smyt pax)]
      `tank`(show-dir vane pax dir.ark)
  ==
==
```

### `++ford` , A Build System

`++ford` builds code (either from the Dojo, from a library, from an app, etc.). `++ford` used to be a standalone vane [but was integrated into `%clay` in 2020](https://urbit.org/blog/ford-fusion/).

`++ford` provides a number of runes for building and importing code into a subject:

- `/-`:  Import a structure file from `sur`.
- `/+`:  Import a library file from `lib`.
- `/=`:  Import a user-specified file.
- `/*`:  Import the contents of a file converted by given mark.

Importing with `*` removes the face (i.e. imports directly into the namespace), while `foo=bar` renames the face.


### Marks and conversions

A mark is a rule for a data structure.  It's sort of like a file extension for `%clay`.  `%clay` also maintains rules for mapping that data structure (such as `%json`) to another (like `%txt`).  Any `%clay` path includes the mark to use on that file—it's not really a file extension, per se, it's an interpretive rule!  Marks live in `mar/` and have a standard core structure.

For instance, the mark for `%json` lives at `mar/json.hoon` and reads as:

```hoon
::
::::  /hoon/json/mar
  ::
/?    310
  ::
::::  compute
  ::
=,  eyre
=,  format
=,  html
|_  jon=json
::
++  grow                                                ::  convert to
  |%
  ++  mime  [/application/json (as-octs:mimes -:txt)]   ::  convert to %mime
  ++  txt   [(crip (en-json jon))]~
  --
++  grab
  |%                                                    ::  convert from
  ++  mime  |=([p=mite q=octs] (fall (rush (@t q.q) apex:de-json) *json))
  ++  noun  json                                        ::  clam from %noun
  ++  numb  numb:enjs
  ++  time  time:enjs
  --
++  grad  %mime
--
```

Marks are used to:

1. Convert between marks (`tube`s).
2. Diff, patch, and merge for `%clay`'s revision control operations.
3. Validate untyped nouns.

Each mark has three arms:

1. `++grow` converts to the mark (first attempt to convert).
2. `++grab` converts from the mark (second attempt to convert).
3. `++grad` is used to `++diff`, `++pact`, `++join`, and `++mash` the noun.

`%gall` uses marks to validate and manipulate the data values being carried by pokes and scries.

- [“Ford Fusion”](https://urbit.org/blog/ford-fusion)

> ### Build a Mark (Optional)
>
> [A `csv` file](https://datatracker.ietf.org/doc/html/rfc4180) (comma-seperated value or common seperator of values) contains tabular information with entry fields across lines and records spanning down.
>
> ```csv
> Duration,Pulse,Maxpulse,Calories
> 60,110,130,409.1
> 60,117,145,479.0
> 60,103,135,340.0
> 45,109,175,282.4
> 45,117,148,406.0
> 60,102,127,300.0
> 60,110,136,374.0
> 45,104,134,253.3
> 30,109,133,195.1
> ```
>
> Compose a mark capable of conversion from a CSV file to a plain-text file (and vice versa).
>
> The `++grad` arm can be copied from the hoon mark, since we are not concerned with preserving CSV integrity.
{: .exercise}

### `%dill` , A Terminal Driver

`%dill` handles keypress events generated from the keyboard or telnet.  This includes the state of the terminal window (size, shape, etc.) and keystroke-by-keystroke events.  `%dill` scrys are unusual, in that they are typically only necessary for fine-grained Arvo control of the display.  Even command-line apps instrumented with `%shoe` do not call into `%dill` commonly.  The only instance of use in the current Arvo kernel is in Herm, the terminal session manager.

### `%eyre` and `%iris`, Server and Client Vanes

`%eyre` handles HTTP requests from clients.  For instance, `%eyre` handles session cookies for the browser.  `%eyre` is also the main interface for `%gall` agents to the outside world.  Channels are defined as pipelines from external HTTP clients to `%eyre` as a thin layer over `%gall` agents, and back again.

`%iris` handles HTTP requests from servers.  For instance, `%iris` can fetch remote HTTP resources (`HTTP GET` command).

- [“`%eyre` Tutorial”](https://urbit.org/docs/arvo/eyre/eyre)
- [“`%iris` Tutorial”](https://urbit.org/docs/arvo/iris/iris)
- [“External API Reference”](https://urbit.org/docs/arvo/eyre/external-api-ref)

### `%jael` , Secretkeeper

`%jael` keeps secrets, the cryptographic keys that make it possible to securely control your Urbit.  Among other cryptographic facts, `%jael` keeps track of the following:

- Subscription to `%azimuth-tracker`, the current state of the Azimuth PKI.
- Initial public and private keys for the ship.
- Public keys of all galaxies.
- Record of Ethereum registration for Azimuth.

`%jael` weighs in as one of the shorter vanes, but is critical to Urbit as a secure network-first operating system.  `%jael` is in fact the first vane loaded after `%dill` when bootstrapping Arvo on a new instance.

- [“`%jael` API Reference”](https://urbit.org/docs/arvo/jael/jael-api/)

![](https://miro.medium.com/max/1838/1*GW2fC3gQxz97B45EFkNA6Q.png)

{% include links.md %}
