---
title: "Next Steps"
teaching: 15
exercises: 0
questions:
- "What should I do next to contribute to the Urbit community?"
objectives:
- "Install the app on another ship (planet, moon, or comet)."
keypoints:
- "The Urbit software distribution service affords a straightforward way to deploy, update, and remove `%gall` apps."
---

##  What Else is There?

We have had a whirlwind tour of developing basic `%gall` agents for Urbit.  We have necessarily had to leave out some important topics, including:

1. Graph Store operations and data format
2. Endpoint operations using `%eyre`
3. Detailed JSON production and parsing
4. Remote subscriptions and kicks
5. Front-end development (à la Landscape)
6. Threading for transient data requests
7. Composition of store/hook/view arrangement

For a demonstration of a few of these, I suggest you examine [`dcSpark/authenticate-with-urbit-id`](https://github.com/dcSpark/authenticate-with-urbit-id), which is a small agent that demonstrates parsing JSON input, subscribing to Graph Store, and some other simple elements of `%gall` agents.

For a more complex single-purpose example including front-end development, you should examine [`yosoyubik/canvas`](https://github.com/yosoyubik/canvas), a peer-to-peer drawing app with a browser frontend and an Urbit backend.

To learn more about threads, read about [Spider](https://urbit.org/docs/userspace/threads/overview) in the docs and examine `ted/example-fetch.hoon` in your own Urbit.  Threading is particular useful for handling complicated IO outside of an agent without compromising the agent's internal state.

##  Joining the Urbit Developer Community

### Contributing

The Urbit ecosystem is primarily developed by Tlon Corporation, a few companies (Urbit.Live, Tirrel, dcSpark, and some hosting companies), and an army of open-source contributors.  The types of projects which you can contribute to include:

1. End-user applications (“userspace”, `%gall`)\
2. Operating system functionality (Arvo & vanes)
3. Runtime (king/serf, jets)

As the Urbit developer community grows, there will be opportunities as well for various types of services, ranging from service hosts to star and galaxy owners.

The [Tlon contributor's guide](https://github.com/urbit/urbit/blob/master/CONTRIBUTING.md) largely focuses on kernel contributor discipline, but provides a good framework for approaching development of any part of the ecosystem.

If you have not participated in an open-source project before, please check these resources as well:

- [Gregory V. Wilson, “Joining a Project”](https://third-bit.com/2021/03/30/joining-a-project/) (good advice from my mentor)
- [Zara Cooper, “Getting started with contributing to open source”](https://stackoverflow.blog/2020/08/03/getting-started-with-contributing-to-open-source/) (more generic advice from SO)
- [Philip Monk `~wisdev-wicryt`, “Urbit Precepts”](https://urbit.org/blog/precepts) (the philosophy underlying Urbit's architectural decisions)

### Main Groups

Some community-facing groups which you can use to learn more about programming and developing in Urbit include:

- `~hiddev-dannut/new-hooniverse` (Hooniverse, beginner-oriented)
- `~littel-wolfur/the-forge` (The Forge, general development)
- `~dasfeb/smol-computers` (Smol Computers, running on non-x86 hardware)
- `~pindet-timmut/urbitcoin-cash` (BTC)

### Urbit Foundation Opportunities

The Urbit Foundation maintains an active developer outreach including:

1. Bounties
2. Apprenticeships
3. Grants

Talk to Josh Lehman `~wolref-podlex` for more information.

### Resources

-   [*The Architecture of Open-Source Applications*, vv. 1–2](https://aosabook.org/en/index.html)
-   [*Code Complete*](https://www.oreilly.com/library/view/code-complete-second/0735619670/)
-   [*The Pragmatic Programmer*](https://pragprog.com/titles/tpp20/the-pragmatic-programmer-20th-anniversary-edition/)
-   [*Working with Legacy Code*](https://www.oreilly.com/library/view/working-effectively-with/0131177052/)
-   [*Design Patterns for Humans*](https://github.com/kamranahmedse/design-patterns-for-humans)

{% include links.md %}
