---
title: "Force Gpg to Use Terminal for Password"
date: 2022-02-13T23:32:21+01:00
categories: ["TIL"]
tags: ["gpg", "git"]
draft: true
---
A minor annoyance is when I'm working on my iPad pro and using Blink and tmux to jump into my playground and do some work and I need to commit something to git and the GPG window will pop up on my displays in my office. I sign all my commits but I'm always battling the GPG password prompt since it will never show up where I want it to...

A quick fix that worked for me:
```
DISPLAY="" DBUS_SESSION_BUS_ADDRESS="/dev/null" git commit -m "adjusted consul service names for registry and traefik test jobs"
```
