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
FIXME

Arvo defines a number of standard operations for each vane.  Notable among these are `++peek`, which grants read-only access to data, called a _scry_; and `++poke`, which accepts moves and processes them.  Pokes actually alter Arvo's state (rather than just retrieve information).

Generally speaking, the Urbit data flow model is _reactive_, meaning that rather than poll for updates periodically one _subscribes_ to a data source which notifies all subscribers when a change occurs.

{% include links.md %}
