---
layout: post
comments: true
title:  "Displaying images and plotting stuff with matplotlib.pyplot"
date:   2017-03-19 21:18:00 +0100
categories: programming
---

The [matplotlib.pyplot](http://matplotlib.org/api/pyplot_api.html) library is my go-to for easy generation of graphs, charts, histograms and anything that can be plotted, and recently I have also been using it lots for viewing images.

The only hitch is, I can never remember the syntax for basic, common things like displaying more than one plot at a time on a graph, or showing a graph with multiple lines and a matching legend box. I confess to having googled “pyplot subplots” a few too many times before writing up my own functions which do the job and are easy to use and remember.

<!--excerpt-->

I’m terrible at open-sourcing code, but am working on making my non-academic work available. You can view, fork, clone or anything my little library [here on GitHub](https://github.com/ysbecca/imagepy-toolkit). Here are a few examples of what it does at the moment. Displaying images in an N by N grid:

![Displaying images in a 5x5 grid]({{site.baseurl}}/assets/post-images/2017-03-19-a.png "Displaying images in a 5x5 grid")

For easy displaying of small patches or windows, there’s also a built-in show_patches().

![As many little patches as you like…]({{site.baseurl}}/assets/post-images/2017-03-19-b.png "As many little patches as you like")

The [matplotlib.pyplot legends API](http://matplotlib.org/users/legend_guide.html) is straightforward enough, but for those who would rather not write the same six lines of code every time you need a graph with a legend, there’s this:

![Plot a graph with multiple lines and a simple legend]({{site.baseurl}}/assets/post-images/2017-03-19-c.png "Plot a graph with multiple lines and a simple legend")

That's it for now, more to follow.