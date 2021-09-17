---
title: "`%gall`:  Constructing a Novel App"
teaching: 15
exercises: 30
questions:
- "How can I deploy a release-worthy `%gall` app?"
- "How can I authenticate my app?"
objectives:
- "Sign and deploy the app."
- "Install the app on another ship (planet, moon, or comet)."
keypoints:
- "The Urbit software distribution service affords a straightforward way to deploy, update, and remove `%gall` apps."
---

##  Installing an App

![A pyramid capstone from Amenemhat III](https://i.imgur.com/KoBYoNI.jpg)

What makes a regular `%gall` app into a deployed app?  In the past, anyone wanting to use a `%gall` app has needed to access a repo, copy the files down into `app/`, and manually start the app.  If something was buggy, you risked lobotomizing your ship, so it was common to set up moons to run installed agents on.  Not total bedlam, but not for the faint of heart.

Grid and `%docket` lead to you being able to automate most of the process now.  Software distribution has become a first-class service in Urbit.


##  Releasing an App

You have seen how to install and activate an agent on your local ship.  Let's look at how to set up and deploy your own app onto the network.

The first thing we need to do is create a new desk in `%clay` which will hold all of the relevant information about the app, including files and metadata.

```hoon
|
```

```hoon
|install ~magbel %example
```

{% include links.md %}
