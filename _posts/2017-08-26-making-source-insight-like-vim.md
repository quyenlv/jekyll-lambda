---
layout: post
title: "Making Source-Insight-like Vim"
categories: build-server vim
author: "Quyen Le Van"
---

The fact is Source Insight is an attractive program editor and analyzer. I used it for 4 years. The reason that I did not use cscope from the beginning, is Source Insight easier to use. When you are too familiar with Window, it's very hard to use the terminal, instead of GUI.

But the problem is, Source Insight is a commercial software. Currently I am using an crack version :D But the more I love Vim and open source software, the more I think I MUST leave Source Insight. Source-Insight-like Vim is my best choice for now.

Therefore, I will do study about Vim plugins which can help me to make Vim become Source Insight software, for free. We will step by step build it up from scratch. Here is our road map:

* Building Source-Insight-like Vim skeleton
* Vim color beautifier
* Vim Ninja with shortcuts
* How I use Vim for real projects

This is the first article of this series, specifically focusing on making the Source-Insight-like skeleton in Vim.

Here is our target:

<!-- more -->

![Source Insight Window](/images/2017-08-26-source-insight-window.gif "Source Insight Window")

## Source Insight User Interface Analysis
---
There are four main windows:
1. Symbol window, on the left top side, shows symbols defined in the file.
1. Source file window, on the middle top side, the main window for editing and viewing code.
1. Project window, on the right top side, lists all the files in the project, and it allows you to open files quickly; regardless of what directory they are in.
1. Context window, on the bottom side, automatically shows declarations and other information while you click on things or type.

## Making Source-Insight-like Vim skeleton
---
One of the things that makes vim great is that it can be extended through plugins. There are plugins for more than you would expect. Of course, there are awesome plugins can work as same as above Source Insight windows in Vim. I will separate into two parts: the first part is to make the skeleton and the second part is to make up it (including color, shortcuts...).

Excluding the `Source File Window`, because it is the Vim's origin feature, the remaining three Source Insight windows can be handled by these plugins:

### Taglist - The Symbol Window
In order to show the source code symbols, we need a tool call the `exuberant ctags` to generates an index (or tag) file of language objects found in source files that allows these items to be quickly and easily located by a text editor or other utility. A tag signifies a language object for which an index entry is available (or, alternatively, the index entry created for that object).

Then we need a Vim plugin, called `Tag List`, to show symbols in Vim using that generated tag file. Remember that the taglist plugin relies on the exuberant ctags utility to dynamically generate the tag listing. The exuberant ctags utility MUST be installed in your system to use this plugin.

```

      +-------+
    +-|-----+ |         +---------+           +---------+         +---------+
  +-|-|----+| |         |         |  generate |  tags   | import  | Taglist |
  | | +-------+ ------->|  ctags  |---------->|  file   |-------->| plugin  |
  | +-------+           |         |           |         |         |         |
  +--------+            +---------+           +---------+         +---------+
  Source code

```

1. Install the `exuberant ctags` utility:
```
sudo apt-get install exuberant-ctags
```
1. Install the Taglist plugin using Vundle:
```
Bundle 'taglist.vim'
```

Now you can generate the tag file using ctags in two ways:
* Navigate into your project directory and run ctags in your shell:
```
ctags -R --exclude=.git --exclude=log --exclude=wutils --c++-kinds=+p --fields=+iaS --extra=+q *
```
Then a file name `tags` will be generated in the same directory.
* Mapping a shortcut in vim to run ctags directory in Vim when you press `CTRL-F12`. You just need to adding this line in the `$HOME/.vimrc` file.
```
map <C-F12> :!/usr/bin/ctags -R --exclude=.git --exclude=log --exclude=wutils --c++-kinds=+p --fields=+iaS --extra=+q *<CR>
```

Once we do have a Ctags index generated, we can use it for navigation in Vim. Vim by default looks for a tags file in your current directory and in the directory of the current file:
```
set tags?
tags=./tags,./TAGS,tags,TAGS
```

There are several different ways you can navigate your code base using tags:

1. Basically, use the `:tag` command to jump to a tag by name. This comes with tab completion and positions your cursor straight on the line of the tag definition (such as where a constant is defined).
1. Using the `CTRL-]` command you can jump to a tag under your cursor. This is nice when you come across a class name in your code and want to jump to it. Position your cursor on the name, press `CTRL-]` and there you go. Using the `CTRL-t` to jump back.
1. Using the tag stack, which is a list of tag-based locations you have visited. After following some tags, you can trace back through your history with :pop or `CTRL-T` and forward with :tag (each take an optional count). To see your history, you can use :tags. This works much like Vim’s regular `CTRL-I` and `CTRL-O` commands for the jump list, but the tag stack only contains, well, tags.
1. From the vim editor, execute `:TlistOpen` as shown above, which opens the tag list window with the tags of the current file as shown in the figure below. By clicking on the function name in the left panel, you would be able to go to the definition of the functions.


### NERD Tree - The Project Window
The NERDTree is a file system explorer for the Vim editor. Using this plugin, users can visually browse complex directory hierarchies, quickly open files for reading or editing, and perform basic file system operations.

Install it using Vundle:
```
Bundle 'scrooloose/nerdtree'
```

You can open it in Vim by `:NERDTreeToggle`

### SrcExpl - The Context Window
SrcExpl (Source Explorer) is a source code explorer that provides context for the currently selected keyword by displaying the function or type definition or declaration in a separate window. This plugin aims to recreate the context window available in the IDE known as "Source Insight". This plugin also works based on tags.

```

      +-------+
    +-|-----+ |         +---------+           +---------+         +---------+
  +-|-|----+| |         |         |  generate |  tags   | import  | Taglist |
  | | +-------+ ------->|  ctags  |---------->|  file   |-------->| plugin  |
  | +-------+           |         |           |         |         |         |
  +--------+            +---------+           +---------+         +---------+
  Source code                                      |
                                                   |      import  +---------+
                                                   +------------->| SrcExpl |
                                                                  | plugin  |
                                                                  |         |
                                                                  +---------+

```

From the vim editor, execute `:SrcExplToggle`, which opens the source explorer window at the bottom side of the Vim window. Then, whenever you put the cursor on a function name in the Normal mode, its definition will show on the Source Explorer window a moment later. As soon as you do the 'double-click' operation using your mouse onto the Source Explorer window which had appeared on the bottom of (G)Vim, the definition and its context will be shown on the editor window.(Just like Source Insight does). Besides, multi-definitions' listing and jumping works like the Source Insight too.

Finally, combine all of them into a shortcut. You can press `F12` to toggle these plugins.

```
:noremap <F12> :TlistToggle<CR>:NERDTreeToggle<CR>:SrcExplToggle<CR><c-w><c-l>
```

You made it!!!
```
 +----------------------------------------------------------------------------+
 | Terminal                                                                   |
 +----------------+-----------------------------------------+-----------------+
 |-demo.c---------|                                         |-/home/myprj/----|
 |function        | 1 void foo(void)     /* function 1 */   ||~ src/          |
 |  foo           | 2 {                                     || `-demo.c       |
 |  bar           | 3 }                                     |`-tags           |
 |                | 4 void bar(void)     /* function 2 */   |                 |
 |~ .----------.  | 5 {                                     |~ .-----------.  |
 |~ | Tag List |\ | 6 }        .-------------.              |~ | NERD Tree |\ |
 |~ .----------. ||~           | Edit Window |\             |~ .-----------. ||
 |~ \___________\||~           .-------------. |            |~ \____________\||
 |~               |~           \______________\|            |~                |
 +-__Tag_List__---+-demo.c----------------------------------+-_NERD_tree_-----+
 |Source Explorer v5.3           .-----------------.                          |
 |~                              | Source Explorer |\                         |
 |~                              .-----------------. |                        |
 |~                              \__________________\|                        |
 |-Source_Explorer------------------------------------------------------------|
 |:                                                                           |
 +----------------------------------------------------------------------------+
```
Here is the screenshots you may have:

![](http://i.imgur.com/bbGVO.jpg)


> **Did you know?**
> 
> All above things can be made with a plugin named [Trinity](https://github.com/wesleyche/Trinity).

## Reference
1. <https://github.com/geekan/source-insight-vim>
1. <http://ctags.sourceforge.net/>
1. <http://vim-taglist.sourceforge.net/manual.html>
1. <https://www.sourceinsight.com/doc/v3/ae1143746.htm>
1. <http://arjanvandergaag.nl/blog/combining-vim-and-ctags.html>
1. <http://www.thegeekstuff.com/2009/04/ctags-taglist-vi-vim-editor-as-sourece-code-browser/>
1. <https://shapeshed.com/vim-netrw/>
1. <https://github.com/wesleyche/SrcExpl/blob/master/doc/srcexpl.txt>
