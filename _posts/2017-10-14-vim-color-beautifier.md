---
layout: post
title: "Vim color beautifier like Source Insight"
categories: vim, build-server
author: "Quyen Le Van"
---

In the previous post about [Making Source-Insight-like Vim](/2017/08/26/making-source-insight-like-vim/), we already knew how to make Vim become Source Insight by using some useful plugins. This is the second article in the series about making Source-Insight-like Vim.

Today we will beautify the layout color for syntax coloring when working with a specific source code language. I will focus on the C language because it's my favorite one.

We will study how Vim detects language syntaxes and color them. Then we can write our own color schema. The [vimcolor](http://vimcolors.com/) website has a large collection of Vim color schema, you can choose one and use it instantly.

<!-- more -->

Here is my Source Insight Window with my favorite color schema. We will make a Vim color schema as same as this one:

![Source Insight Window](/images/2017-08-26-source-insight-window.gif "Source Insight Window")


## Terminal Emulator
---
`Vim` uses information about the terminal you are using to fill the screen and recognize what keys you hit. There is also `gVim`, which is Vim with a built-in GUI. Functionally there is no difference between them. There are few addition operations supported by `gVim` like:
* More font and better text rendering support in gvim.
* Supports a much wider range of colors (RGB), while the terminal only supports 256 colors.
* `gVim` has additional menu and tool bars which `Vim` lacks.

> A computer terminal is an electronic or electromechanical hardware device that is used for entering data into, and displaying data from, a computer or a computing system.

Currently I use `Vim` in a terminal emulator name [XShell](https://www.netsarang.com/products/xsh_overview.html), where I connect to my host through SSH connection. There are many free and well-known terminal emulators such as [teraterm](https://ttssh2.osdn.jp), [Putty](http://www.putty.org/)...

> A terminal emulator, terminal application, or term, is a program that emulates a video terminal within some other display architecture. A terminal window allows the user access to a text terminal and all its applications such as command-line interfaces (CLI) and text user interface (TUI) applications. These may be running either on the same machine or on a different one via telnet, ssh, or dial-up.

These terminal emulators support the standard terminal emulators named `xterm`. Many Linux distro use `xterm` over the others because it's very lightweight. It just takes under 1MB memory. You can check your current terminal by check the `$TERM` variable. You can also see the color map which it supports by `msgcat` utility.

```
$ echo $TERM
xterm-256color

$ msgcat --color=test
```

You can try to change your terminal to `vt100` then open `Vim`. Oops!!! You will see Vim in black and white color because `vt100` does not support color.

```
$ export TERM=vt100
$ msgcat --color=test
$ vi
```

## 256 colors in Vim
---
Now we understand about terminal emulator which is environment our `Vim` will work on. The maximum color that it can support is 256 colors. Of course, it doesn't look better than `gVim`, which can support true color (16M colors). Remember that `xterm` just supports 16 colors, so in case we want to enlarge color map, we need to change your `$TERM` to `xterm-256color`.

You can enable it automatically by adding these lines to your `~/.profile`:
```
if [ -e /usr/share/terminfo/x/xterm-256color ]; then
    export TERM='xterm-256color'
else
    export TERM='xterm-color'
fi
```

You may need to enable 256 colors in vim by putting this your `.vimrc` but if the terminfo file is correct, `Vim` will ask the terminal for that value. 
```
set t_Co=256
```

## Vim color methodology
---
Everything is ready. Now we will find out how we can mark color for our function name, variable, #define... in `Vim`. The answer is `syntax` and `highliting`.

This graph helps you to understand the whole picture before we dig on it.
```
                                                               Syntax Group
     +-----+                                                   (Constants, Comment...)
     |---  |                                                     +----------+
     |--   | main.c                                              | +---------+
     |-    |                                                     +-|--------+| ......................
     +-----+                                                       +---------+                      .
        +                                                             ^                             .
        |    Open in Vim                                              .                             .
        |                                                             .                             .
        v                                                             .                             v
 +--------------+          +-----------------+ Load "c.vim" +--------------+ Load        +----------------------+
 | :filetype on | Type "C" | :syntax on      | syntax file  | Analyze file | colorscheme |:hi Comment term=bold |
 | Detect type  |+-------->| Load syntax for |+------------>| into syntax  |+----------->|Hightligh syntax group|
 | of the file  |          | detected type   |              | group        |             |                      |
 +--------------+          +-----------------+              +--------------+             +----------------------+
```

### Syntax
The `syntax` and `highlighting` commands for one language are normally stored in a syntax file. The name convention is: `{name}.vim`.  Where `{name}` is the name of the language, or an abbreviation, for example: c.vim, cpp.vim, perl.vim... Upon loading a file, `Vim` finds the relevant `syntax` file after detecting its file type using `filetype` option. `Vim` find syntax file in the `$VIMRUNTIME/syntax/` directory.

You can make your own syntax files, or if you are mostly satisfied with an existing syntax file, but would like to add a few items or change the highlighting, you can write your syntax file and locate it in `$HOME/.vim/after/syntax/c/`, file name must end with `.vim`.

Here is an example of a user defined syntax (name `cUserFunction`) for defining the C function name. You can study detail about writing syntax file from the vim help about [syntax define](http://vimdoc.sourceforge.net/htmldoc/syntax.html#:syn-define).
```
syn match cUserFunction "\<\h\w*\>\(\s\|\n\)*("me=e-1 contains=cType,cDelimiter,cDefine
```

After that we can use the name `cUserFunction` for highlighting color, says: we want to make the function name bold and green, we just need to set `cUserFunction` those properties. We will study about it in the next section.

One more thing, very interesting, `Vim` defines the syntax group name is to be used for syntax items that match the same kind of thing. These are then linked to a highlight group that specifies the color. A syntax group name doesn't specify any color or attributes itself. `Vim` suggests the preferred name for syntax groups that are common for many languages. For example:

```
*Comment    any comment

*Constant   any constant
 String     a string constant: "this is a string"
 Character  a character constant: 'c', '\n'
 Number     a number constant: 234, 0xff
 Boolean    a boolean constant: TRUE, false
 Float      a floating point constant: 2.3e10

*Identifier any variable name
 Function   function name (also: methods for classes)
```
The names marked with * are the preferred groups; the others are minor groups. For the preferred groups, the "syntax.vim" file contains default highlighting. The minor groups are linked to the preferred groups, so they get the same highlighting.

Well! Currently we understood about `syntax`, how they are defined and grouped. In the next section, we will study how `Vim` color them by its method named `highlighting`.

### Highlighting
Basically, we can use the above `syntax group` names for highlighting, using this format: `:hi[ghlight] {group-name} {key}={arg}`. For example, make comment bold:

```
:hi Comment cterm=bold
``` 
Note that all settings that are not included remain the same, only the specified field is used, and settings are merged with previous ones. So, the result is like this single command has been used:
```
:hi Comment term=bold cterm=bold ctermfg=Cyan guifg=#80a0ff gui=bold
```

You can see, we have option for `term`,`cterm` and `gui` which is based on we are using `gVim` or normal `Vim` on the terminal emulation as we told in the above section. As I said, we just want to focus on the `xterm`, here is the color term, or `cterm` options.

We can put these `highlight` commands in a `colorscheme` file and load it after that. Simple make a file `$HOME/.vim/colors/{name}.vim` and load it by:

```
:colo[rscheme] {name}
```

You can study more from the `Vim` help about [highlighting](http://vimdoc.sourceforge.net/htmldoc/syntax.html#:highlight).

## Make things happen
---
The default `syntax` for C language is very basic, it does not classify the function name correctly and more. Hence, we need to append some C syntax extra named [C Syntax Extensions](https://vim.sourceforge.io/scripts/script.php?script_id=3064) by Mikhail Wolfson.

Here is my Source-Insight-like color schema:
![](http://c1.staticflickr.com/5/4491/23838713888_87647cbf85_b.jpg "My Source-Insight-like vim color schema")

> **Did you know?**
> 
> You can set up this vim color schema easily with my plugin named [jovi](https://github.com/quyenlv/jovi).

## Reference
1. <https://stackoverflow.com/questions/9832660/why-dont-most-vim-color-schemes-look-as-nice-as-the-screenshot-when-i-use-them>
1. <https://en.wikipedia.org/wiki/Terminal_emulator>
1. <http://vimdoc.sourceforge.net/htmldoc/syntax.html#:highlight>
1. <https://en.wikipedia.org/wiki/Computer_terminal>
1. <https://askubuntu.com/questions/862531/what-is-the-difference-between-gvim-and-vim>
1. <https://askubuntu.com/questions/14284/why-is-a-virtual-terminal-virtual-and-what-why-where-is-the-real-terminal>
1. <http://www.aboutlinux.info/2005/10/configuring-xterm-in-linux.html>
1. <https://askubuntu.com/questions/2995/how-is-the-default-term-shell-variable-value-set>
1. <http://vim.wikia.com/wiki/256_colors_in_vim>
