---
layout: post
comments: true
title:  "Decision tree solution to the Monty Hall problem"
date:   2017-03-18 21:18:00 +0100
categories: problems
---

The [Monty Hall problem](https://en.wikipedia.org/wiki/Monty_Hall_problem) was first famously solved by Marilyn vos Savant in 1990, and infamously argued by many intellects. The premise is roughly as follows: 

> There are three doors: behind two are goats, and behind one is a car. You will receive the object behind the one you open. You must first choose one door, and the host (who knows the contents of all three doors) will then open *one of the two remaining doors* and show you a goat. You then can choose to open your original choice, or switch your choice to the remaining door. Are you more likely to get the car if you switch?

<!--excerpt-->

The answer is **yes**. You are more likely to get a car if you switch your choice to the remaining door. The answer and the reasoning behind it stumped me as much as anyone else, but here is my proof of the solution by decision tree.


![Monty Hall decision tree]({{site.baseurl}}/assets/post-images/2017-03-18.png "Monty Hall decision tree")

