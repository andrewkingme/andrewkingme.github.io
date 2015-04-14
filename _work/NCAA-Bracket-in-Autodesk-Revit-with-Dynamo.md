---
layout: article
title: "NCAA Bracket in Autodesk Revit with Dynamo"
date: 2015-03-21 04:02:00
cover: /img/work/NCAA-Bracket-in-Autodesk-Revit-with-Dynamo-Cover.png
section: work
tags:
  - all
  - visualprogramming
  - revit
  - dynamo
---

A recent [tweet by @pix3lot](https://twitter.com/pix3lot/status/577532781426798592) inspired me to make a 2015 NCAA bracket using the open source graphical programming engine [Dynamo](http://dynamobim.com). I use Revit daily at my job and I was already looking for an excuse to improve my knowledge of Dynamo within Revit. I can't think of a better way to dive in!

<!--more-->

Since @pix3lot only posted a [screen capture of his final bracket](https://twitter.com/pix3lot/status/577532781426798592), I needed to reverse engineer the "NCAA Picker" custom node (while adding some magic of my own). I multiplied the rank of each team by a random number from 0 to 1 and selected the lower number as the winner. This method was simple enough for a Dynamo n00b and also allowed for reasonable upsets.

![NCAA Bracket in Autodesk Revit with Dynamo](/img/work/NCAA-Bracket-in-Autodesk-Revit-with-Dynamo-002.png)
*Visual Programming for the NCAA Picker Custom Node*

At the time of this post, the first round of the tournament is coming to a close. I entered two pools with two unique brackets. I'll update this post with my results at the end of the tournament.

![NCAA Bracket in Autodesk Revit with Dynamo](/img/work/NCAA-Bracket-in-Autodesk-Revit-with-Dynamo-001.png)
*My NCAA Bracket in Dynamo for Revit*

---

**Update 2015-04-13:**

I won! (and I lost.) I submitted two brackets this year, each to a pool of approximately 12 people.

After the first round, Bracket 1 was performing great! The upset factor was working, having picked a 14 Georgia State upset over 3 Baylor and a 11 UCLA upset over 6 SMU. Out of the 32 picks in the first round, Dynamo picked 25 correctly. I was in first place! Unfortunately, Bracket 1 fell apart in the second and third rounds, ultimately leading to a last-place bracket.

Bracket 2 started out slowly (middle of the pack) but improved throughout the tournament. Dynamo correctly picked 3 of the 4 Final Four and Duke as the winner, winning the pool!
