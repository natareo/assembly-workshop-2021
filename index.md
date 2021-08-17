---
layout: lesson
root: .  # Is the only page that doesn't follow the pattern /:path/index.html
permalink: index.html  # Is the only page that doesn't follow the pattern /:path/index.html
---

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

<!-- this is an html comment -->

{% comment %} This is a comment in Liquid {% endcomment %}

> ## Prerequisites
>
> We do not anticipate your knowing Hoon in advanced, although experience with
> programming is a prequisite.
>
> You should have a fakezod, or Urbit ship not running on the network.
>
> You may set this up with a new Urbit binary:
>
> ```sh
> urbit -F zod
> ```
{: .prereq}

{% include links.md %}
