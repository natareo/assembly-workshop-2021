#   Urbit for Developers
##  N E Davis (`~lagrev-nocfep`) · [Assembly 2021](http://assembly.urbit.org/)

![](https://pbs.twimg.com/media/E8c4z59WEAEWZ0a?format=jpg&name=medium)


##  Objectives

1. Program literately using the Hoon language, including source code conventions and interoperability.
2. Explain and navigate the schematics and technical implementation of the Urbit OS kernel (Arvo and vanes).
3. Construct novel userspace apps to run on the Urbit OS platform (Gall, Landscape).
4. Sign and distribute working apps through an Urbit-backed software development service.

We will _not_ deal with Azimuth, parsing, many aspects of generators, event logs, runtime issues, and many other aspects of fully grokking Urbit.


##  Audience

Software developers in attendance at Assembly in Austin, Texas.  These all have skill in some programming language platform; we expect two categories of participants:

1. Those with little to no experience with software development in Hoon and on the Urbit platform.
2. Those with Hoon School-tier exposure to the Hoon programming language.


##  Outline

- Hoon (_The Hoon section aims to teach the ~25 most common runes, the core pattern, and code building._)
  - Basic runes
  - Irregular forms
  - Data structures (nouns, atoms & auras, cells, lists)
  - Subject-oriented programming
  - Libraries
  - Advanced data structures (cores, maps, molds)
  - Unit tests & building code
- Arvo (_The Arvo section aims to teach enough of Arvo to know how events are processed and what tools are available._)
  - Kernel structure
  - Key vanes
  - API, scrying, etc.
- `%gall` (_The `%gall` section aims to teach how a `%gall` app is structured and how to compose a new app and deploy it._)
  - Minimal working example
  - Adding functionality through a helper core
  - Deep dive:  `chat-cli` (excursus on `%shoe`/`%sole`)
  - Interfacing with a client
  - Deep dive:  Clock (Landscape)
  - Constructing a novel app
  - Deploying & installing software

Both days have about 4½ hours maximum.  The first day should aim at reaching Arvo content so the second day can focus on `%gall`.  Since it can be difficult to gauge time in a new workshop, some parts (like unit testing and `chat-cli`) should be able to be skipped for time if necessary.  If we assume some facility with Hoon already on the part of attendees, this problem is slightly simplified.

We will provide supplementary and reference materials derived from the documentation and the Fall 2020 Urbit graduate seminar.

The event should be hands-on.  In particular, if we can figure out a rapid way of running "interactive" Hoon code (maybe talking to a comet), this will be invaluable.  We will also need to consider the heterogeneity of development environments.


##  Detailed Outline

_(Work in progress; notably, breaks and hands-on components are not yet laid out.)_

1. Hoon (3¼ hours)
    1. Runes & basic programs (45 min)
        1. Operate Dojo by entering Hoon commands.
        2. Mount the Unix filesystem and commit changes as necessary.
        3. Use built-in or provided tools (gates and generators).
        4. Identify elements of Hoon such as runes, atoms, and cells.
        5. Diagram Hoon generators into the corresponding abstract syntax tree.
        6. Compose a simple generator and load it from `gen/`.
        7. Understand how to pass arguments to and results from gates.
        8. Produce a generator for a mathematical calculation.
    2. Data structures (30 min)
        1. Apply auras to transform an atom.
        2. Identify common Hoon molds, such as lists and tapes.
        3. Produce a generator to convert a value between auras.
        4. Identify common Hoon patterns: atoms, cells, cores, faces, and traps.
    3. Subject-oriented programming & generators (45 min)
        1. Explain what a “subject-oriented language” means.
        2. Create a `%say` generator.
        3. Review known runes in context of highest-frequency, highest-impact runes.
    4. Libraries (30 min)
        1. Create a library in `lib/` and utilize it using `/+`.
        2. Compose Hoon that adheres to the Tlon Hoon style guide.
    5. Advanced data structures (45 min)
        1. Identify common Hoon patterns: batteries, and doors, arms, wings, and legs.
        2. Identify common Hoon molds: units, sets, maps, jars, and jugs.
        3. Explore the standard library’s functionality: text parsing & processing, functional hacks, randomness, hashing, time, and so forth.
    6. \[Optional:  Unit tests (30 min)]
        1. Write unit tests for a library.
2. Arvo (1¼ hours)
    1. Kernel structure (20 min)
        1. Diagram the high-level architecture of Urbit.
    2. Vane roles (20 min) (high-level overview of operations of `%ames`, `%clay`, `%eyre`/`%iris`, `%gall`, and possibly `%khan`)
        1. Understand architecture of the `%ames` network and communications.
        2. Access file data from `%clay`.
        3. Access arbitrary revisions of a file through `%clay`.
        4. Manipulate remote data through `%eyre`/`%iris`.
        5. Provide external commands to Arvo through `%khan`.
    3. API, scrying, etc. (35 min)
        1. Enumerate functionality of the Urbit API.
        2. Scry for local and remote data.
        3. Use subscriptions to access data.
3. `%gall` (4¼ hours)
    1. MWE (30 min)
        1. Understand how static `%gall` instruments an app.
        2. Produce a basic `%gall` app.
    2. Adding functionality (45 min)
        1. Manage internal `%gall` state.
        2. Scry for needed information.
    3. \[Optional:  `chat-cli` (45 min)]
        1. Apply `%sole` and `%shoe` to operate a command-line interface app.
    4. Interfacing with a client (60 min)
        1. Produce an intermediate `%gall` app.
        2. Understand how `%gall` interfaces with an external client like Landscape.
        3. Retrieve and store information in `graph-store`.
    5. Deep dive:  Clock (30 min)
        1. Apply prior learning to understand how Clock works in Landscape.  \[or Weather]
    6. Constructing a novel app (45 min)
        1. Produce (with assistance) a new `%gall` app with specified behavior.
    7. Deploying & installing software (45 min)
        1. Sign and deploy the app.
        2. Install the app on another ship (moon, comet).


##  Operations

The most effective learning is mimetic:  it either directly demands productivity or engages the mirror neurons.  Most of the workshop should be hands-on, either as a live-coding exercise following the instructor directly or as guided exercises.  This is fairly demanding to administer (in case anything goes wrong on a learner's workstation), so we need _at least_ one helper per 25 participants.

If students are working on their own machines, we can either expect them to run a fakezod directly or run one on the cloud.

The rhythm of teaching should be 15–20 minutes of instruction, 5 minutes of exercise, 5 minutes of break, scaled fractally as needed.  Breaks should become slightly more frequent as the afternoon wanes.


##  Resources

- [The Carpentries, “Curriculum Development Handbook”](https://cdh.carpentries.org/guiding-principles.html)
- [The Carpentries, “Lesson Development Handbook”](https://docs.carpentries.org/topic_folders/lesson_development/index.html)
