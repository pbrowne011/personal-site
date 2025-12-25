+++
title = "Belichick's library"
date = 2025-12-24T00:00:00Z
draft = false
author = "Pat"
description = "About Bill Belichick's football library  and my search page for it"
tags = []
categories = []
images = []

# set X (formerly known as Twitter) card image
twimg = "https://upload.wikimedia.org/wikipedia/commons/9/9a/Bill_Belichick_2019_%28cropped%29.jpg"

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
keywords = ["Bill Belichick", "football", "football books", "USNA",
"Naval Academy"]

# if you want to weight a particular post's ranking
# weight = 1

# generate table of contents
toc = false
+++

*[Back to catalogue](/belichick/)*

## About This Page

This is intended to be an outline of a (future) blog post explaining the page,
how I made it, and anything else interesting about the process of creating
it. Possible section headers:

- Info on collection. From articles, press releases, interviews, ...
- Statistics: categories, dates, keywords, ...
- Links that I chose to include, why, and something about each
- ISBNs and how they came to be
- [Fuzzy string
  searching](https://en.wikipedia.org/wiki/Approximate_string_matching), how the
  webpage searches for books and handles typos and related words
- Thoughts on data cleaning and use of AI in this process


### Search Links

Each book has links to four sites.

- **WC (WorldCat)** - Search global library holdings to find a copy near
  you. Found using OCLC (Open Computer Library Center) catalog.
- **IA (Internet Archive)** - May have digitized copies for free online
  reading. The Internet Archive is its own digital library website for books,
  movies, and dated copies of websites.
- **AA (Anna's Archive)** - Aggregates multiple sources for digital
  copies. Anna's Archive is a search engine for shadow libraries that is a
  successor to sites such as Z-Library, Sci-Hub, and Library Genesis (LibGen).
- **BF (BookFinder)** - Search used book sellers to purchase a copy. BookFinder
  aggregates new and used copies from tens of thousands of booksellers,
  including sites such as Amazon, AbeBooks, and eBay.

It is recommended that you create accounts on WorldCat and IA to get maximum
usage out of the sites. An account is often required to view all available
copies of a book in libraries on WorldCat, as well as to check out a book and/or
download a PDF of it on IA.

You can create an anonymous account on Anna's Archive, but I don't see the use
in having one. It currently operates in a legal grey area; for that reason, I
may remove links to AA from the search page at some point. BookFinder does not
have accounts (if it did, it would be your Amazon account, since it is owned by
AbeBooks).

Other sites may be added in the future. Potential candidates:
- **Goodreads**. Not included because, if Belichick has it, why would you need a
  second opinion?
- **Amazon/AbeBooks**. Currently covered by BookFinder.
- **ISBN DB**. Most of these books do not have an ISBN (International Standard
  Book Number).
- **Kalamazoo Public Library**, or **Western Michigan's library**. Maybe in the
  future.
- **Google Books**. Probably at some point, I didn't think of it at first.

### On Data Quality

I still have much work to do on improving data quality. Many books are missing
author name(s) and/or publish dates. Titles are incorrect. The sheet from USNA
used abbreviations (such as "FB" for "football") and misspelled words.

For any corrections that you find, please send to pjbrowne011 [at] gmail [dot]
com. To make it easier for me to update the entry, please send each corrected
entry in the following JSON format:

```json
{
    "title": "",
    "author": "",
    "publisher": "",
    "year": 0,
}
```

Fill in the quotation marks with the strings for each. Update the year to the
correct number (no quotation marks). I will likely update the requested
information format as time goes on too. If you have multiple, bookend all of the
books (objects) with brackets `[` `]`, as it is now a list of more than one
book.

With all of this said, even with perfect data, there will be mismatches between
this site's data for a book and how it is represented on each external site. It
is up to you to fiddle with search terms until you find the resource you are
looking for. **Do not give up easily**. Try fixing obvious typos or guessing at
words, removing some, adding others, searching only author name, whatever it
takes. I suspect that every book on this site is findable on at least one
external website.


---

[Back to catalogue](/belichick/)
