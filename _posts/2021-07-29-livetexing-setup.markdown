---
layout: post
title:  "My LiveTeX-ing Setup"
date:   2021-07-29 15:41:08 -0400
categories: latex
usemathjax: true
published: true
---

I've been using $\LaTeX$ for note-taking and problem set writeups for about 3 years now, and I've gradually developed my workflow for how I take notes in real time. While getting note-taking up to speed is mostly just a matter of practice, like taking math and converting it to $\LaTeX$ expressions on the fly, there are a few tricks that can make life a lot easier.

Before I get into time-saving tricks, I'll describe my distribution choice, as well as my current editor/compiler/reader setup.

### $\LaTeX$ Distribution
I do most of my liveTeX-ing on my chromebook, which is running a small debian container. The main issue with this abnd the reason I didn't aim for something like the whole TeXLive package is the sheer size, with a full install taking up anywhere from 3.2 to 5.5 GB. With my total 16 GB of storage for the whole container, this was very clearly not an option. Luckily, the existence of [TinyTeX][TinyTeX] means that I'm able to run a very bare-bones installation, and then add packages as I see fit (Which is done on the fly via a very handy python script called `texliveonfly` which automatically runs through a `.tex` file and installs any missing packages).

For my Windows computer, where storage space is not an issue, I use MikTeX, and I generally don't have to worry about any packages being missing, except for the occasional one that isn't on CTAN (`qcircuit` comes to mind).

### Editor/Compiler/Reader

As a devout Sublime Text user (and for a legitimate reason I'll discuss in a bit), I have no choice but to use **Sublime Text** as my editor of choice. It comes with plenty of features, is lightweight and responsive, and has great plugins that streamline the liveTeX-ing process (I will discuss those in another section). The main selling point is the lightweight nature, particularly when it comes to intensive syntax highlighting (which $\LaTeX$ documents inevitably run into). In fact, from my experience, the Sublime Text syntax highlighting is more responsive than the syntax highlighting used by my go-to console editor, vim. In fact, vim's syntax highlighting led to awful screen tearing and made the editor almost unusuable for any `.tex` file that required any amount of scrolling (spoiler alert: that's pretty much all of them). Even switching to non-default syntax highlighting via packages like Syntastic or ale were slow enough that editing in vim for anything more than a couple sentences was a headache. Granted, all of this was done on a pretty weak Chromebook, but it was still pretty surprising that vim was having trouble with 2 page long documents, let alone 60 page ones.

For my $\LaTeX$ compiler, I use `latexmk`, which allows for continuous compilation into a `.pdf`. Continuous compilation is a godsend, and is what I would consider the most necessary part of a liveTeX-ing setup. Specifically, the command for compilation I use is 
```
latexmk -pdf -pvc -view='none' <filename>.tex 
```
The `-pdf` command, as expected, converts the output file to a `.pdf`, and the `-pvc` enables continuous compilation. The `-view='none'` is there to stop latexmk from opening the system default pdf viewer on compilation.

For the pdf viewer, I use **SumatraPDF** on Windows, and **zathura** on debian. SumatraPDF is a great reader, and allows for tabs, which is a very handy feature. Zathura can be a bit unwieldy, but it works pretty well. In order to have it interface with reverse search in ST, I added the following to my configuration file (`~/.config/zathura/zathurarc`):
```
set synctex true
set synctex-editor-command 'subl %{input}:%{line}'
```
Now double clicking on a portion of the pdf in zathura will send my cursor in ST to the location in the $\LaTeX$ source file. While the reader is just personal preference, the readers that are in consideration should all be live-updating, which makes seeing changes very quick and easy.

### tmux Scripting

To actually open up a file and start editing, I got tired of opening up Sublime and zathura separately, and then running latexmk. Instead, I set up a tmux script that opens Sublime, zathura, and begins running a compilation command for a given file:
```bash
#!/bin/bash
# This script sets up my sublime text + latexmk + zathura live tex setup
# takes in a tex file as input, and open sublime text with that file open, as well as starting latexmk on the tex file
# It then opens up zathura with the generated pdf file
# does this all in a tmux window


# the file base name is $1
file=$1
# check if the file exists in the current directory
# this avoids bad inputs
if [ ! -f "$1.tex" ]; then
	echo "File does not exist"
	exit 1
fi

# make sure to not double open the same session
EXISTS=$(tmux list-sessions | grep $file)
# first check if the tmux session already exists
if ["$EXISTS" = ""]
then
#first start a tmux window
tmux new-session -d -s "$file"

tmux select-window -t "$file"
# open up sublime
tmux send-keys "subl $1.tex" Enter

# one window opens up latexmk
tmux split-window -h "latexmk -pdf -pvc -view='none' $file.tex"
# split window opens up zathura
tmux split-window -v "zathura $file.pdf"

# switch to the shell pane
tmux select-pane -t 0
tmux send-keys C-l
fi

# finally attach to the ready tmux session
tmux attach-session -t "$file"
```
This makes editing a file as easy as typing in `lt <filename>`.

### Snippets
Now for some actual editing tricks that I use. Since almost all of my courses that I liveTeX notes for are math and physics, there are some things that come up very often, and so it's worth making some snippets, which can then expand into expressions that are a pain to retype over and over. Examples of such snippets include `inft`, which expands into 
\\[\int_{-\infty}^{\infty}\\]
`dift`, which takes two inputs, lower and upper, and expands into
\\[\int_{\text{lower}}^{\text{upper}}\\]

Another pair that I use quite often are `td` and `pd`, which expand into
\\[\frac{d \text{upper}}{d \text{lower}} \;\; \frac{\partial \text{upper}}{\partial \text{lower}}\\]
respectively.
Another snippet that has a very specific use case is `exp`, which comes up in quantum a great deal:
\\[\langle \text{val} \rangle\\]
While these do take a little bit of time to work into muscle memory, they come in handy a great deal when trying to keep up with large expressions like 
\\[\frac{d^nf}{dx^n} = \frac{d}{dx}\left(\frac{d^{n-1}f}{dx^{n-1}}\right)\\]


### Sublime Text Plugins

One handy feature with Sublime Text is the existence of Package Control, and the number of $\LaTeX$-focused packages. The two that I recommend are **LaTeX-cwl** and **LaTeXTools**. The existence of TAB autocomplete in the form of macros is very useful. Another major feature is the existence of inline compilation preview, which continuously previews the current math mode formula as you type, without the need for saving:

{:refdef: style="text-align: center;"}
![image](/images/inline-preview.png)
{:refdef}

This makes writing long or convoluted expressions easy to check quickly, without needing to wait for `latexmk` to compile and zathura/sumatra to update.


### Closing Thoughts

Getting to the point where I could confidently sit in class and be able to keep up with the lecturer took several months of daily notetaking, but after a rough first 2 weeks, I was able to develop the muscle memory and aptitude for translating equations on the fly. The addition of snippets was a big help, and it took quite a while to fiddle with all the settings of my setup to make it work smoothly for me, but at this point I don't see myself changing anything anytime soon. LiveTeX-ing is very doable, and the additional speed with which I can write up my problem sets is a very nice bonus.


[TinyTeX]: https://yihui.org/tinytex/
