---
layout: post
date: 2016-11-05 17:34:06
title: Exploring Elm
published: true
---

It's no secret that I enjoy functional programming, so when I heard about [elm](http://elm-lang.org) I had to give it a try. Elm bills itself as "A delightful language for reliable webapps", and while it leaves some things to be desired, it mostly lives up to this mantra.

If you've ever used React, you'll probably be familiar with the high-level concepts in elm design. Facebook emphasizes the use of the Flux architecture for its applications, providing developers with clear, unidirectional dataflow. Once this was picked up, people quickly built on this foundation to isolate their state into large immutable chunks using libraries like immutable.js. Coupled with redux, this effectively turns your web-app into a giant state machine, with actions providing updates that are fed into a "reducer".

Well what does this have to do with Elm? Elm applications are built around something very similar, aptly named the Elm architecture.[^1]

[^1]: Evan Czaplicki, the creator of Elm, has emphasized simple and direct naming in elm-land. This is often in direct contrast with the punny or even obtuse names of javascript libraries. Instead you'll see things like "elm-package" (the package manager), "elm-make" (the build tool), and "elm-html" (who would have guessed, the elm html library).


## A Quick Tour

elm example?
