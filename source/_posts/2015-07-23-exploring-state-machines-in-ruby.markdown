---
layout: post
title: "Exploring State Diagrams"
date: 2015-07-23 09:56:22 -0400
comments: true
categories: modeling
---

Part of the Flatiron School web development curriculum is a month-long Project Mode, so we as instructors advise the students on modeling their data. We always have the students map out their models and associations before beginning to code, but there are tools besides basic flowcharts that can help a project simplify its data. One of those tools is the __state diagram__.

## What is a State Diagram?

[Wikipedia](https://en.wikipedia.org/wiki/State_diagram) gives an overview of state diagrams:

> State diagrams are used to give an abstract description of the behavior of a system. This behavior is analyzed and represented in series of events, that could occur in one or more possible states.

Some more research leads to the fact that state diagrams are representations of state machines. Vaidehi Joshi wrote two excellent blog posts that explain [what state machines are](http://vaidehijoshi.github.io/blog/2015/03/17/a-machine-state-of-mind-part-1-understanding-state-machines/), as well as [how to implement them in Ruby](http://vaidehijoshi.github.io/blog/2015/03/24/a-machine-state-of-mind-part-2-implementing-state-machines/). In short:

> At the risk of sounding a bit philosophical, it all boils down to actions that are taken, and the reasons we take certain actions. State machines are how we keep track of different events, and control the flow between those events.

## First State Steps

In order to get a bit more practice modeling data, I'm going to create a state diagram to represent a Movie object. From [Mitch Boyer](https://twitter.com/mrmitchboyer), I received this list of production states:

> - Development: concept, develop screenplay, build core team, funding 
- Pre-Production: refine script, build crew, casting, location scouting, contracts, etc.
- Production: shooting the movie
- Post-Production: edit, music/sound mix, VFX, color correct
- Distribution: festival circuit, VOD services (video on demand), etc.

That's a lot of information! Time to break it down. I'll start with a simple action – putting each of the five production states into nodes:

<center>{% img http://i.imgur.com/c2T47Yt.png 200 %}</center>

## Where Can We Go From Here?

Rather than try to list all possible transitions between production states in a Movie, I'll attack one node at a time. From the first node (Development), I can move forward one state (Pre-Production). From the second node (Pre-Production), I can move forward one state (Production) or backward one state (Development). I can start to see a pattern – at any point, I can move onto the next state or return to the state that came before. At any point, I can start over and return to the drawing board (Development).

Represented with flowchart arrows, my diagram now looks like this:

<center>{% img http://i.imgur.com/sxHFqBI.png 300 %}</center>

## ...And How Do We Get There?

The next step is to figure out what actions need to be taken to move from one state onto the next one. For me, it helps to think of each state as having an entry gate – what conditions need to be fulfilled for admission to a state?

Let's take the first two nodes – Development and Pre-Production. According to our list, the development stage consists of developing a concept, developing screenplay, building core team, and funding the project. It's impossible to move on _without_ funding and a developed idea; those are the requirements. So if I call an action `develop_and_fund`, my Movie should be ready for the Pre-Production state.

I can figure out my entry gates for each step forward:

<center>{% img http://i.imgur.com/MKIBCe4.png 400 %}</center>

Naming the backwards steps is a little trickier. For example, at what point is a project no longer in Post-Production, but rather in Production again? When we need more footage. The final state diagram looks like this:

<center>{% img http://i.imgur.com/iDkNdzl.png 500 %}</center>

## Why Use State Diagrams?

1. __Simplicity__. State diagrams allow us to represent the way that objects evolve as certain actions occur. Since information is represented visually, we can more easily point out its flaws – for example, we might decide that a Movie _cannot_ move from Production back to Pre-Production, because the Production state will deplete some of the movie's funding. We may need to rerun `develop_and_fund` before we can return to the Pre-Production state.

2. __Communication__. State diagrams can help us communicate the way systems will be laid out to non-technical team members.

3. __Organizaiton__. Since state machines can only exist in one state at a time, we can easily sort our Movies by progress. Furthermore, we know that API calls to each different state will return mutually exclusive sets of objects.

Hope you've enjoyed this simple example!

<center>{% img http://media.giphy.com/media/WxxsVAJLSBsFa/giphy.gif %}</center>