---
layout: default
title: "Scripting in Freeplane"
date: "2020-02-21 12:34:58 -0500"
categories: [productivity]
tags: [freeplane, mindmap, groovy]
---

As part of my daily routine, there are some more fixed elements of a day that are always part of my day. These can be thought of as chores, or as habits. I had intended this to talk mostly about the chores and not the habits, but thinking of it now, it is probably worth thinking a little more about the chores and extending them to habits, since while not pleasant, they often are associated with larger good goals which are worth keeping in my mind.

The chores I wanted added to my daily routine are:

* email wrangling
* inbasket (managing patient followup tasks in the less than modern emr messaging interface)
* email wrapup = email wrangling PM

I was very inspired by Justin DiRose's article on [managing startup and shutdown routines](https://inside.omnifocus.com/justin-dirose), and actually used it to help me establish some very helpful routines in 2020.

I wanted to make sure I could do this in my productivity system with Freeplane so I used a Groovy script and a hotkey to make this more effortless. The side-effect of this is that is has also forced me to declare my "daily chores" and has now helped me to think more about what I actually do need to do each day.

# Automating in Freeplane with Groovy

* [Groovy Syntax](http://groovy-lang.org/syntax.html)
* [Freeplane API](https://www.freeplane.org/doc/api/)
* [Freeplane Scripting Wiki](https://www.freeplane.org/wiki/index.php/Scripting)

For this I:

1. Created a "daily.groovy" script
2. Created a shortcut to run that script


```groovy
node.createChild("correspondence AM 1h");
node.createChild("inbasket 30m");
node.createChild("correspondence PM 30m");

node.createChild("-- stretch");
node.createChild("-- nice to have");
node.createChild("-- new");
```

# MacVim, Finder and Folders

I had often missed the ability to quickly create a text file in the current drive that I was browsing. Today I discovered that if I alt-click on the folder in the folder bar of Finder > Service > MacVim buffer here, then I can easily use MacVim to create files in that folder - super!


