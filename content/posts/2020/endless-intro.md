---
title: "Endless Intro"
date: 2020-12-29T01:18:15+01:00
categories: ["endless"]
tags: ["code", "golang", "annoyances"]
draft: false
---
Usually I have somewhere between 10 to 30 tabs open in my browser, mostly stuff to read at a later time. On its one this is really an annoyance, but this pattern repeats (in extreme) on my iPad Pro and my iPhone. Currently my phone has about 65 tabs active. At some point I was fed up with this and decided to do something about this. I saw this as an exercise in my coding skills and at the same time solve something that bothered me.

Given the fact that the state and amount of open tabs seems to be endless I named the solution "Endless".

Currently it's nothing more that a CLI tool that stores an URL in a flat file database structure implemented in Golang. Endless uses Boltdb as a backend. Features that I'm planning on implementing are:
- Read status
- Long term archiving
- Random item

The "Random item" feature is the most interesting for me since my lists tend to get overtly long and leave me in an analysis paralisis state.

You can find the project on my Github profile: [Endless](https://github.com/qjvtenkroode/endless)
