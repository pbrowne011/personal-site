+++
title = "A new blog"
date = 2025-07-01T00:00:00Z
draft = false
author = "Pat"
description = "Writing more words using shorter posts"
tags = []
categories = []
images = []

# set X (formerly known as Twitter) card image

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
keywords = ["new blog"]

# if post is part of a blog series
series = []

# if you want to weight a particular post's ranking
# weight = 1

# generate table of contents
toc = false
+++

This is the first of (hopefully many) short blog posts. Inspired by reading
Chris Siebenmann's ["Wandering
Thoughts"](https://utcc.utoronto.ca/~cks/space/blog/) and Daniel Lemire's
[blog](https://lemire.me/blog/), as well as Gwern's semi-recent [pivot to
short form](https://gwern.net/blog/) and John Cook's [short
posts](https://www.johndcook.com/blog/), I've decided to write more unpolished,
random thoughts on the Internet.

I have folders of half-finished drafts that I want to get "just right" before
posting. It is likely that many of them won't see the light of day because of
this perfectionism. I want to [write
quickly](https://jsomers.net/blog/speed-matters), but that's difficult to do
when my last post is from [seven months ago](/posts/shiftreg1).

The `/blog` section will be a place for me to write about topics that are
trivial: random Emacs features I find (I just discovered `eshell` yesterday!),
interesting puzzles and problems, code snippets, and who knows what else.

### New thoughts already

Even in setting this up, I found something to write about: an issue with TOML
parsing.

I wanted to add dates to blog URLs. There will (hopefully) be more of them, and
other people seem to do this.

The static site generator I use, Hugo, has several ways to configure website
links. [Permalinks](https://gohugo.io/configuration/permalinks/) fit my use
case: they allow me to configure all blog URLs with a date, while leaving other
pages (such as About or Posts) alone.

Hugo allows users to write their site configuration file in either YAML, TOML,
or JSON. By historical accident, I set up the blog with a `config.toml` file,
though I'm much more familiar with the other two file formats at this point and
know [some](https://ruudvanasseldonk.com/2023/01/11/the-yaml-document-from-hell)
[weaknesses](https://noyaml.com/).

My configuration is a mess, so I plopped the permalinks setting somewhat
randomly:

```toml
...other settings
[permalinks]
  blog = '/:year/:month/:day/:filename/'

mainSections = ["post"]

...
```

I ran into an unreadable panic when checking the config.
```
$ hugo config
panic: runtime error: slice bounds out of range [1:0]

goroutine 1 [running]:
github.com/gohugoio/hugo/resources/page.PermalinkExpander.validate({0xc000555080?, 0x0?, 0xc000554ab0?}, {0x0?, 0x0?})
        github.com/gohugoio/hugo/resources/page/permalinks.go:182 +0x23c
github.com/gohugoio/hugo/resources/page.PermalinkExpander.parse({0xc000555080?, 0x0?, 0xc000554ab0?}, 0x8?)
        github.com/gohugoio/hugo/resources/page/permalinks.go:125 +0x136
        
...
```
Looking at the [offending
line](https://github.com/gohugoio/hugo/blob/v0.92.2/resources/page/permalinks.go#L182),
I noticed that the line being passed wasn't long enough to be indexed.

I checked if I needed another "layer" (does it have to be permalinks.page?
blog.permalinks?), or some format string was off, or if "blog" itself was
incorrect (I had just added the directory to my "content").

I tried Claude Sonnet 4, but the debugging suggestions weren't helpful. I also
did a bad job providing context: I typed the permalink lines I had added to the
config, rather than pasting the whole file.

Eventually, I just C-c C-ved the error into Google and found
[this](https://discourse.gohugo.io/t/config-permalinks-meet-panic/18479) on the
Hugo forum from zwbetz:
> In TOML you can’t write a map then write a regular key/value right after (only other maps).
>
> In other words, move your permalinks to the very bottom on your config file

I moved it to the bottom of the file and the config went back to normal. Why did
that work?

#### TOML Tables

The [v1.0.0 specification](https://toml.io/en/v1.0.0#table) describes the
"table" feature of TOML. Tables are TOML's hash table implementation. Headers
are defined by brackets (e.g., `[permalinks]`) and key-value pairs for that
table follow underneath.

In a TOML file, there is a "root table" that goes from the beginning of the file
until the first table header (or EOF). I had no conception of this root table in
my configuration, and had been throwing key-value pairs wherever they
worked. The comments and weird whitespacing I left didn't help either.

If you look at the example above again, you can see the array `mainSections =
["post"]` getting parsed incorrectly, though I didn't poke around long enough to
figure out what exactly was being parsed. Maybe if I used a debugger like
[Delve](https://github.com/go-delve/delve), this would be an easier mystery to
solve.

#### Woes of TOML

Tables highlight the main issue people point out with TOML: deeply nested data
structures.

In a k8s manifest.yaml file, the indentation describes the nesting of different
objects. While there are many footguns in describing objects in this way, the
file format makes it obvious where this nesting is. Likewise, braces (and a good
linter) reveal nesting in JSON objects.

TOML, however, requires the user to write a well-behaved file. With dotted keys,
an invisible root table, and no strict constraints on indentation, defining
nested objects becomes trickier. One must parse the various table headers and
keep track of which key-value pairs belong to each. In the other file formats,
the syntax manages this abstraction for you.

