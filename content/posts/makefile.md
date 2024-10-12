+++
title = "Why do commands require tabs in a Makefile?"
date = 2024-03-03T00:00:00Z
draft = false
author = "Pat"
description = "An unfortuate historical example of backwards compatibility"
tags = ["unix", "computer-history"]
categories = []
images = []

# set slug if you want to change a page
slug = ""

# modify type of page (posts show up on front page)
# types can be found in mainSections
type = "post"

# if you want to display modification date
# lastmod = 2024-03-01T00:00:00Z

# redirect old URLs to new URL for post
aliases = []

# SEO keywords
keywords = ["makefile", "unix", "stuart feldman", "syntax"]

# if post is part of a blog series
series = []

# if you want to weight a particular post's ranking
# weight = 1

# generate table of contents
toc = false
+++

When I was first learning how to write a Makefile to compile a C program, one
thing that stuck me (and almost everyone who encounters it) as odd was the
requirement that each command line start with a tab character
(`'\t'`). Makefiles are used to automate repetitive tasks (such as recompiling a
C/C++ program), and have a unique syntax. They are made up of rules, which
consist of a "target", a list of "dependencies", and a series of
"commands". They look like this:

```
target: dependencies
        command
```

For example, suppose I want to automate two different tasks: recompiling my 
program after editing some of the source code, and deleting all non-source
files after editing them. I could write a Makefile that looks like this:

```
.PHONY: clean

myprogram: myprogram.c
        gcc myprogram.c -o myprogram
	
clean:
        rm *.o *~ myprogram
```

The .PHONY makes sure that when I run the shell command `make clean`, I don't
create a file named `clean`. Instead, I run the command underneath, which
removes the files I don't want in my directory. The other rule has a target
filename `myprogram`, which requires a file named `myprogram.c`. If it finds
this .c file in my directory, it will compile it using the command below. If you
want to learn more about makefiles and their syntax, a resource that I highly
recommend is [Makefile Tutorial](https://makefiletutorial.com/).

Notice how each command is indented with a tab character. The UNIX utility
`make` doesn't allow you to list a command underneath a target/dependency
combination, or to indent with a number of spaces. In the 
["tabs vs. spaces" war](https://www.youtube.com/watch?v=SsoOG6ZeyUI), `make` has
definitifely taken a side. Why is this?

The author of `make`, Stuart Feldman, has explained this debacle a few times,
including
[once to Michael Stillwell](https://beebo.org/haycorn/2015-04-20_tabs-and-makefiles.html).
Stuart explained that `make` was written in a weekend. To tokenize Makefiles, he
was using [lex](https://en.wikipedia.org/wiki/Lex_(software)), which was in
its first version at the time. He got frustrated with trying to write a pattern
for commands, and instead used a fixed pattern (`'\t'`) to indicate rules. After
a while, about a dozen people at Bell Labs were using `make`. He didn't want to
disrupt his users, and so left the "tab in column 1" rule in the UNIX command to
preserve backwards compatibility.

As he says in the email:

```
> Within a few weeks of writing Make, I already had a dozen friends who were 
> using it.
> 
> So even though I knew that "tab in column 1" was a bad idea, I didn't want
> to disrupt my user base.
> 
> So instead I wrought havoc on tens of millions.
```

If you don't want to begin your commands with `'\t'`, different versions of
`make` allow you to specify other characters. For example, 
[GNUMake](https://www.gnu.org/software/make/manual/html_node/Special-Variables.html)
allows for a special variable, `.RECIPEPREFIX`, where you can change the
character that `make` assumes is introducing a recipe line. However, from
personal experience, this isn't much help to someone who is trying to make a
binary file and keeps getting syntax errors!
