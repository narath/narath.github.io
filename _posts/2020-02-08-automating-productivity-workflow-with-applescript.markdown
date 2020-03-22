---
layout: post
title: "Automating my productivity workflow with a little applescript"
date: "2020-02-08 08:28:37 -0500"
---
I spent a little time today learning applescript to automate some of my productivity workflow, and it was very empowering!

## Ideas

I keep track of my ideas in a [freeplane](https://www.freeplane.org/wiki/index.php/Home) using an [alfred](https://www.alfredapp.com/) keyword "ideas" I can either:

* quickly launch my ideas mindmap
* or with a keyword: quickly add an idea and be ready to explore it

Here's the script I am using:

```applescript
on run argv
  if (count of argv > 0) then
	set hasQuery to true
    set theQuery to item 1 of argv
  else
  	set hasQuery to false
  end if

  set ideaFile to "Macintosh HD:Users:<filepathhere>" as alias
  tell application "Finder" to open file ideaFile

  if (hasQuery) then
    tell application "System Events"
      tell process "Freeplane"
        click menu item "Goto root" of menu "Navigate" of menu bar 1
        click menu item "New child node" of menu "New node" of menu item "New Node" of menu "Edit" of menu bar 1
        keystroke theQuery
        keystroke return
      end tell
    end tell
  end if
end run
```

## Todo

I also keep track of my daily todos in a "Today!" node in my Focus mindmap. This script lets me quickly view my today items, or if I add an argument add it quickly to my list for today.

Here is that script:

```applescript
on run argv
  if (count of argv > 0) then
	set hasQuery to true
    set theQuery to item 1 of argv
  else
  	set hasQuery to false
  end if

  set theMindMap to "Macintosh HD:Users:<file_path_here>" as alias

  tell application "Finder" to open file theMindMap

  if (hasQuery) then
    tell application "System Events"
      tell process "Freeplane"
        click menu item "Goto root" of menu "Navigate" of menu bar 1
          keystroke "f" using command down
            keystroke "Today!"
            keystroke return
        click menu item "New child node" of menu "New node" of menu item "New Node" of menu "Edit" of menu bar 1
        keystroke theQuery
        keystroke return
      end tell
    end tell
  end if
end run
```

