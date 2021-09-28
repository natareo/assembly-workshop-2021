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

> For a long time Urbit was too unstable to develop on easily, but over the past couple years that's improved greatly. Now the biggest hurdle to getting more apps on Urbit is that there's no real way to distribute them.

![A pyramid capstone from Amenemhat III](https://i.imgur.com/KoBYoNI.jpg)

What makes a regular `%gall` app into a deployed app?  In the past, anyone wanting to use a `%gall` app has needed to access a repo, copy the files down into `app/`, and manually start the app.  If something was buggy, you risked lobotomizing your ship, so it was common to set up moons to run installed agents on.  Not total bedlam, but not for the faint of heart.

Grid and `%docket` automate most of the process now for users.  Software distribution has now become a first-class service on Mars.

![](https://room.eu.com/images/contents/article28.jpg)

#### References

- [`~rovnys-ricfer`, “Third-Party Software Distribution in Urbit”](https://gist.github.com/belisarius222/fe93df670b776ebd276f9bd0d42f0b12)

##  Installing an App

We will install a remote app onto our local ship at the command line (rather than via the browser UI).

```hoon
|install ~magbel %example
```

#### Resources

- [`~palfun-foslup`, “pals”](https://github.com/Fang-/suite/tree/wip/dist/pkg/pals)


##  Releasing an App

You have seen how to install and activate an agent on your local ship.  Let's look at how to set up and deploy your own app onto the network.

The basic concept of software distribution for Urbit is that a ship has a desk with self-contained agent code and a `%docket` mark which

Broadly, speaking, desks look the same, except for some modest additions for agent registration.  The directories still obtain as follows:

- `/app` for agents
- `/gen` for generators
- `/lib` for library and helper files
- `/mar` for marks
- `/sur` for shared structures
- `/ted` for threads

These new files contain critical information to instrument the distributed software:

- `/sys/kelvin` for kernel Kelvin version, e.g. `[%zuse 420]`
- `/desk/bill` for a list of agents to run (for `%kiln`)
- `/desk/docket` for app metadata (for `%docket`)

To set this up, the first thing we need to do is create a new desk in `%clay` which will hold all of the relevant information about the app, including files and metadata.

```hoon
|merge ~zod landscape
```


This is a `$docket` mark with annotation:

```hoon
+$  href                                :: where a tile links
  $%  [%glob base=term =glob-location]  :: location of client-side data
      [%site =path]                     :: location of server-rendered frontend
  ==                                    ::
::                                      ::
+$  url   cord                          :: URL type
::                                      ::
+$  glob-location                       :: how to retrieve glob (client-side)
  $%  [%http =url]                      :: HTTP source, if applicable
      [%ames =ship]                     :: %ames source, if applicable
  ==                                    ::
::                                      ::
+$  version                             :: version of app (not Kelvin version)
  [major=@ud minor=@ud patch=@ud]       ::
::                                      ::
+$  docket                              ::
  $:  %1                                :: Docket protocol tag
      title=@t                          :: text on home screen
      info=@t                           :: long-form description
      color=@ux                         :: app tile color
      =href                             :: link to client bundle
      image=(unit url)                  :: app tile background
      =version                          :: version of app (not Kelvin version)
      website=url                       :: URL to open on click
      license=cord                      :: software release license
  ==                                    ::
```

An example `/desk/docket` file:

```hoon
```

{% include links.md %}
