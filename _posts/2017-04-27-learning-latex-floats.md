---
layout: post
title:  "Learning LaTeX: floats"
date:   2017-04-27 21:18:00 +0100
categories: academics
---

Up until now, I have written all my papers, reports and documents using Microsoft Word. Granted, I have used the IEEE or some other helpful Word templates on occasion, but yes, I have re-worked references and renamed figures many times, manually. Find-and-replace is very manual, as is typing numbers.

Last week my supervisor suggested I invest a few hours in learning LaTeX and exploring some kind of reference management system. A little exploration and some helpful tips from classmates, and I’ve committed to a workflow mostly involving Google Scholar Libraries, LaTeX, and BibTeX. This ensures that I have a collection of all the useful papers I’ve found, their BibTeX citations, and an easy way to automatically reference them when writing.

LaTeX tries to automatically make things look nice. This is great, except when you can’t find your figures and tables and they appear in sections far away from where you intended them to be. **A little understanding of LaTeX and floating goes a long way.**

Basically, tables and figures by default float to where they will look nice. Even if you specify you want a table here using the [h] parameter, LaTeX and you may disagree on where here is. This is because LaTeX follows a standard set of typographic rules to minimise wasted whitespace and to completely fill each page with text/figures/tables.

Here are some options if you don’t like where your figures float.

+ You can force a page break (whitespace until the end of the page) by:

```latex
\newpage or \clearpage
```

+ Force tables or figures to show exactly where you place them:

```latex
\usepackage{float}
\restylefloat{figure}
...
\begin{figure} [H]
```

+ Allow figures to float, but not past a certain barrier:


```latex
\usepackage{placeins}
...
\FloatBarrier
```

+ Use a combination of placement options (gives LaTeX more flexibility to apply its own floating rules):

```latex
\begin{figure}[!htbp]
```

[!] — when possible, ignore LaTeX placement rules such as floating objects per page, whitespace allowed, text/figure ratios, etc.  <br />
[h] — means place it here, but LaTeX will only do this if there is enough remaining space on the page.  <br />
[t][b] — place it at the top or bottom of the page.  <br />
[p] — place it on a page filled with only figures and tables.




