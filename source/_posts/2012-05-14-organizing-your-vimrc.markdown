---
layout: post
title: "Organizing your .vimrc"
date: 2012-05-14 18:06
comments: true
categories: [software]
---

Most .vimrc files that I’ve seen are quite long and unwieldy, often extending
to upwards of 300 lines of code. The .vimrc becomes a mess of code as it is
built up over the years, just a few lines at a time. Many people try to
resolve this issue by breaking the .vimrc down into sections, often with the
help of comments or fold markers. This makes it much easier to read, but it
isn’t a perfect solution. I still find myself having to scroll around the
.vimrc to find the line of I want to change or to find a place I can insert
a new line of vimscript.

There’s a much better solution for organizing the .vimrc, thanks to the
ability to source other vimscripts from within a vimscript. I’ve broken up my
own .vimrc file into 8 smaller files, and I include them into my main .vimrc
file by using the source function. Here’s what it looks like:

{% codeblock lang:vim %}
set nocompatible
behave xterm
filetype plugin indent on
syntax on

source $HOME/.vim/vimrc/filetypes.vim
source $HOME/.vim/vimrc/looks.vim
source $HOME/.vim/vimrc/mappings.vim
source $HOME/.vim/vimrc/misc.vim
source $HOME/.vim/vimrc/plugin_configs.vim
source $HOME/.vim/vimrc/plugins.vim
source $HOME/.vim/vimrc/settings.vim
source $HOME/.vim/vimrc/spelling.vim

" Source a local vimrc if it exists
if filereadable(expand("$HOME/.vimrc.local"))
  source $HOME/.vimrc.local
endif
{% endcodeblock %}

Now each of my files are much smaller and easier to manage! It’s also quite
simple to navigate to each of these files. I’ve set up <leader>; to open up
a tab with the .vimrc, and then I can use vim’s handy gf action to open up the
file given by the filename under the cursor. I’ve included a nice way to allow
for local .vimrc additions as well.

If you’re interested, check out my vimfiles on github here:
https://github.com/thenovices/dotfiles
