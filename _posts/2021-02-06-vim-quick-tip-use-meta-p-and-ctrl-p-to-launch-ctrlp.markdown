---
layout: post
title: "Vim quick tip: use cmd-p and ctrl-p to launch CtrlP"
date: "2021-02-06 14:32:26 -0500"
---

I've really gotten into using [Obsidian](https://obisidian.md). It also has a wonderful fuzzy search ala SublimeText with Cmd-P.

Here is what you need to do to make this work in MacVim as well.

## Turn off Cmd-P launching Print in MacVim

In ~/.gvimrc

```
macmenu File.Print key=<nop>
```

Thanks to this [StackOverflow post](https://stackoverflow.com/questions/23351782/how-to-map-cmdp-to-ctrlp-without-actually-modifiying-cmd-from-root)

## Then in your .vimrc you can use this

```  
" Map Meta-P to CtrlP as well
" Note: you also have to set in ~/.gvimrc `macmenu File.Print key=<nop>
map <D-p> <C-p>
```
Restart MacVim and enjoy!:

```

