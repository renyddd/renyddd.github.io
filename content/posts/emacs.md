---
title: "Emacs"
date: 2021-10-23T19:55:13+08:00
draft: true
---



### Do repeat

Most Emacs commands accept a numeric argument; for most commands, this
serves as a repeat-count.  The way you give a command a repeat count
is by typing C-u and then the digits before you type the command.  If
you have a META (or EDIT or ALT) key, there is another, alternative way
to enter a numeric argument: type the digits while holding down the
META key.  We recommend learning the C-u method because it works on
any terminal.  The numeric argument is also called a "prefix argument",
because you type the argument before the command it applies to.

### Discard

If Emacs stops responding to your commands, you can stop it safely by
typing C-g.  You can use C-g to stop a command which is taking too
long to execute.

### Windows

Emacs can have several "windows", each displaying its own text.  We
will explain later on how to use multiple windows.  Right now we want
to explain how to get rid of extra windows and go back to basic
one-window editing.  It is simple:

	C-x 1	One window (i.e., kill all other windows).

### Editing

	<DEL>        Delete the character just before the cursor
	C-d   	     Delete the next character after the cursor
	
	M-<DEL>      Kill the word immediately before the cursor
	M-d	     Kill the next word after the cursor
	
	C-k	     Kill from the cursor position to end of line
	M-k	     Kill to the end of the current sentence

You can also kill a segment of text with one uniform method.  Move to
one end of that part, and type C-<SPC>.  (<SPC> is the Space bar.)
Next, mnnnove the cursor to the other end of the text you intend to kill.
As you do this, Emacs highlights the text between the cursor and the
position where you typed C-<SPC>.  Finally, type C-w.  This kills all
the text between the two positions.

#### yanking

After you have done C-y to get the most recent kill, typing
M-y replaces that yanked text with the previous kill. 

#### undo

If you make a change to the text, and then decide that it was a
mistake, you can undo the change with the undo command, C-/.

Normally, C-/ undoes the changes made by one command; if you repeat
C-/ several times in a row, each repetition undoes one more command.

Alternatively, C-x u also works exactly like C-/, but is a little less
convenient to type.

### Files

In many ways, it is as if you were editing the file itself.
However, the changes you make using Emacs do not become permanent
until you "save" the file.  This is so you can avoid leaving a
half-changed file on the system when you do not want to.  Even when
you save, Emacs leaves the original file under a changed name in case
you later decide that your changes were a mistake.

```
	C-x C-f   Find a file
```

 The file name you type appears
on the bottom line of the screen.  The bottom line is called the
minibuffer when it is used for this sort of input.

```
	C-x C-s   Save the file
```

#### Buffers

If you find a second file with C-x C-f, the first file remains
inside Emacs.Emacs stores each file's text inside an object called a "buffer".
Finding a file makes a new buffer inside Emacs.  To see a list of the
buffers that currently exist, type

	C-x C-b   List buffers

When you have several buffers, only one of them is "current" at any
time.  That buffer is the one you edit.  If you want to edit another
buffer, you need to "switch" to it.  If you want to switch to a buffer
that corresponds to a file, you can do it by visiting the file again
with C-x C-f.  But there is an easier way: use the C-x b command.
In that command, you have to type the buffer's name.

Some buffers do not correspond to files.  The buffer named
"*Buffer List*", which contains the buffer list that you made with
C-x C-b, does not have any file.

The creation or editing of the second file's
buffer has no effect on the first file's buffer.  This is very useful,
but it also means that you need a convenient way to save the first
file's buffer.  Having to switch back to that buffer, in order to save
it with C-x C-s, would be a nuisance.  So we have

	C-x s     Save some buffers

`M-<` : Move to the top of the buffer (beginning-of-buffer). With numeric argument n, move to n/10 of the way from the top.

`M->` : Move to the end of the buffer (end-of-buffer).

### eXtending

There are many, many more Emacs commands than could possibly be put
on all the control and meta characters.  Emacs gets around this with
the X (eXtend) command.  This comes in two flavors:

	C-x	Character eXtend.  Followed by one character.
	M-x	Named command eXtend.  Followed by a long name.

Named eXtended commands are commands which are used even less
frequently, or commands which are used only in certain modes.  An
example is the command replace-string, which replaces one string with
another in the buffer.  When you type M-x, Emacs prompts you at the
bottom of the screen with M-x and you should type the name of the
command; in this case, "replace-string".  Just type "repl s<TAB>" and
Emacs will complete the name. 

### Settins

#### auto save

 Emacs periodically writes an "auto save" file for each file that
you are editing.  The auto save file name has a # at the beginning and
the end; for example, if your file is named "hello.c", its auto save
file's name is "#hello.c#".  When you save the file in the normal way,
Emacs deletes its auto save file.

#### echo area

If Emacs sees that you are typing multicharacter commands slowly, it
shows them to you at the bottom of the screen in an area called the
"echo area".  The echo area contains the bottom line of the screen.

#### mode line

The line immediately above the echo area is called the "mode line".
The mode line says something like this:

```
 -:**-  TUTORIAL       63% L749    (Fundamental)
```

- NN% indicates your current position in the buffer text; it
  means that NN percent of the buffer is above the top of the screen.

- The L and digits indicate position in another way: they give the
  current line number of point.

- The part of the mode line inside the parentheses is to tell you what
  editing modes you are in.  The default mode is Fundamental which is
  what you are using now.  It is an example of a "major mode".

Emacs has many different major modes.  Some of them are meant for
editing different languages and/or kinds of text, such as Lisp mode,
Text mode, etc.  At any time one and only one major mode is active,
and its name can always be found in the mode line just where
"Fundamental" is now.

Each major mode is the name of an extended command, which is how you can
switch to that mode.  For example, M-x fundamental-mode is a command to
switch to Fundamental mode.

To view documentation on your current major mode, type C-h m.

Major modes are called major because there are also minor modes.
Minor modes are not alternatives to the major modes, just minor
modifications of them.  Each minor mode can be turned on or off by
itself, independent of all other minor modes, and independent of your
major mode.  So you can use no minor modes, or one minor mode, or any
combination of several minor modes.

### Searching

The Emacs search command is "incremental".  This means that the
search happens while you type in the string to search for.

The command to initiate a search is C-s for forward search, and C-r
for reverse search.

### Windows

One of the nice features of Emacs is that you can display more than
one window on the screen at the same time.  (Note that Emacs uses the
term "frames"--described in the next section--for what some other
applications call "windows".)

```
C-x 2 which splits the screen into two windows.
C-M-v to scroll the bottom window.
C-x o ("o" for "other") to move the cursor to the bottom window.
```

C-M-v is an example of a CONTROL-META character. 



### Ref

https://book.emacs-china.org/#orgheadline1

https://www.cnblogs.com/Open_Source/archive/2011/07/17/2108747.html



## Spacemacs

>  Press `SPC h T v` (that is, the `spacebar` followed by `h`, `T` and `v`) to familiarize yourself with modal editing.

### 问题解决

#### 1 字体

Cannot find any of the specified fonts (Source Code Pro)! Font settings may not be correct.  

https://github.com/adobe-fonts/source-code-pro/releases/tag/2.038R-ro/1.058R-it/1.018R-VAR

ttf 文件解压后，可点击进行安装或者 cp：

```bash
cp -a source-code-pro-release/TTF/* ~/Library/Fonts
```

#### 2 添加自定义配置

Spacemacs divides its configuration into self-contained units called `configuration layers`. By default Spacemacs uses a dotfile called `~/.spacemacs` to control which layers to load. For example to get python support, simply add `python` to the list of `dotspacemacs-configuration-layers` in your `~/.spacemacs`. 

A configuration layer is a directory containing at least a `packages.el` file which defines and configures packages to be downloaded from Emacs package repositories using the `package.el` built-in feature of Emacs.

If you already have your own `Emacs` configuration you can move it to your own layer. The following command creates a layer in the `private` directory:

```
SPC SPC configuration-layer/create-layer RET
```

Any configuration layers you create must be explicitly loaded in `~/.spacemacs`.

To load some configuration layers using the variable `dotspacemacs-configuration-layers`:

```
;; List of configuration layers to load.
dotspacemacs-configuration-layers '(auto-completion smex)
```

At anytime you can apply the changes made to the dotfile or layers without restarting Spacemacs by pressing `SPC f e R`.

> https://www.spacemacs.org/doc/DOCUMENTATION.html#dotfile-configuration

For instance, [RMS](https://www.spacemacs.org/doc/DOCUMENTATION.html#thank-you) can add his private configuration layer like this:

```
(setq-default dotspacemacs-configuration-layers
  '(
    ;; other layers
    ;; rms layer added at the end of the list
    rms
  ))
```

#### 4 editing style

Spacemacs key bindings use a leader key which is by default bound to `SPC` (space bar) in `vim`

It is also possible to search for specific key bindings by pressing:

```
SPC ?
```

#### 5 evil-org--populate-base-bindings: Symbol’s function definition is void: evil-redirect-digit-argument

> https://github.com/syl20bnr/spacemacs/issues/14303

Deleting the `.emacs/elpa` directory inside and restarting spacemacs did the trick.

#### 6 mirror

> https://mirrors.tuna.tsinghua.edu.cn/help/elpa/

### Packages

#### Neotree

> https://develop.spacemacs.org/layers/+filetree/neotree/README.html

To toggle the `NeoTree` buffer, press `SPC f t` 

To select the NeoTree window, press `SPC 0`

#### Git

> https://develop.spacemacs.org/layers/+source-control/git/README.html

https://magit.vc/

#### shell

> https://develop.spacemacs.org/layers/+tools/shell/README.html

#### org-mode

> https://orgmode.org/worg/orgcard.html

M-x org-info  To read the on-line documentation try

C-RET (`org-insert-heading-respect-content`) ;; Insert a new heading at the end of the current subtree.

或者可通过 M-x 来键入 org-insert 来获取其快捷键的提示。

https://orgmode.org/manual/Structure-Editing.html#Structure-Editing

M-LEFT (`org-do-promote`)

M-RIGHT (`org-do-demote`)



with blog: https://ox-hugo.scripter.co/t m
