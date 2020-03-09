---
layout: default
title: "More automation please"
date: "2020-02-10 09:25:49 -0500"
---

So when you develop some useful automation, then when it breaks... you spend time fixing it.

Not the way I expected to start my Monday, but I learned a lot more about applescript and vim-gist.

# Applescript and Java processes

Fascinatingly, what was working yesterday, but clearly not fully real-world tested, was interacting with java applications. Java applications (like Freeplane) often have a process name (in the System Events list) of JavaAppLauncher.

You can use this script to get all the running processes:

```
tell application "Finder"
	get name of every process
end tell
```

So in order to interact with your java application it is probably more reliable to access use the bundle identifier:

```
plutil -p /Applications/Freeplane.app/Contents/Info.plist | grep BundleIdentifier
```

For Freeplane this turns out to be: "CFBundleIdentifier" => "org.freeplane.core"

So in your Applescript you can more reliably get to the java application using code like this:

```
tell application "System Events"
  set process_name to (name of (application processes where bundle identifier is "org.freeplane.core") as string)
  tell process process_name
    -- do cool things here
  end tell
end tell
```

# Automation scripts and version control

I do think version control for automation scripts is a really great idea. I'm using [gists](https://gist.github/com) for these since you can now easily store multiple files in a single gist (I keep related workflows in a single gist), and am using [vim-gist](https://github.com/mattn/vim-gist) for this. This now supports multiple files within a gist (see this [issue](https://github.com/mattn/vim-gist/issues/56)), but you need to add a single line to your .vimrc:

```
let g:gist_get_multiplefile = 1
```

And then it works beautifully - sweet. If only my Alfred could use gists for the workflow source!

# If PluginInstall asks for your github username, you probably entered the incorrect github title

This has happened more than once. When installing the applescript syntax, the github repo is "vim-scripts/applescript.vim". I entered this as Plugin 'vim-scripts/applescript' (without the .vim at the end) and it asked me for my github username. This is a red flag that the github repo name is incorrect since *for public repos this should not be needed*. Adding back the full name, and it installs beautifully

# Alfred has a lovely script debugger

https://www.alfredapp.com/help/workflows/advanced/debugger/ is actually really helpful.

# Finally, handling ^M in files in vim

This is due to the line endings for scripts (in this case copying from Alfred's editor). To change them to proper newlines in vim, just use:

```
:%s:^M:\r:g
```

Note: don't just copy the above, you need to use ctrl-v ctrl-m to enter that correctly. \r = the newline character for replacements in vim.





