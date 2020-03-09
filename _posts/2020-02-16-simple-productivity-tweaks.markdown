---
layout: default
title: "Simple Productivity Tweaks"
date: "2020-02-16 08:11:42 -0500"
tags: [vim]
category: [productivity]
---

Working on my tools. Simple things can make the workspace more productive. The new retina screens make default font size selection important for comfort. For me this meant setting the font size in vim/Macvim.

# How to find the current font and then make it default

From [this](https://vim.fandom.com/wiki/Change_font) you can get the current font by using `:set guifont?`. You can launch a picker to set the current font using a dialog by using `:set guifont=*`. To change it, add this to you .vimrc:

```
if has('gui_running')
  set guifont=Menlo-Regular:h14
endif
```

# Speed up bundle install

Thanks to [Phil Nash](https://philna.sh/blog/2017/06/12/speed-up-bundle-install-with-this-one-trick/) for this one:

```
bundle config --global jobs 4
```


