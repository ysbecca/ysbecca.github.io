---
layout: post
comments: true
title:  "Jekyll with GitHub hosting"
date:   2017-04-29 21:18:00 +0100
categories: programming
tags:
- jekyll
- web-development
- github
---

WordPress is a stable, well-documented, powerful blogging platform. I've developed WordPress sites, so I've seen firsthand the wide range of things it can do. But [Jekyll](https://jekyllrb.com) has been piquing my curiousity for quite some time. This post will cover how I customised a few things in my clean installation of Jekyll. [GitHub Pages](https://pages.github.com) gets credit for the free, seamless hosting.

<!--excerpt-->

<h4>Overriding templates and post previews</h4>

I made a few little tweaks. Firstly, I wanted to customise the footer. With Jekyll, you can view all the code from the theme (minima by default) by doing **```subl $(bundle show minima)```** with your choice of text editor (mine is [Sublime](https://www.sublimetext.com/3)), and then overwrite it by creating the same files in the same layout in your site folder. I ended up with the following additions to my folder&#58;

```
 ├── _includes
 │   ├── disqus_comments.html
 │   └── footer.html
 ├── _layouts
 │   ├── default.html
 │   ├── home.html
 │   └── post.html
 ```

Secondly, I wanted to show a little preview of each post on the index page, not just the title. Credits go to [this blog](https://wesleytsai.io/2015/07/06/create-post-previews-for-jekyll-blogs/) for outlining this easy way to do it.

*Note&#58; due to an error where Kramdown is parsing my inline ERB, I have replaced all double {} by [].*
*_layouts/home.html*

```html
[[ post.excerpt ]]
[% if post.content contains site.excerpt_separator %]
  <a href="[[ site.baseurl ]][[ post.url ]]">Read more</a>
[% endif %]
```

*_config.yml*
```YAML
excerpt_separator: "<!--excerpt-->"
```
And finally, in my posts:

*YYYY-MM-DD-post-title.md*
```markdown
Excerpt-text
<!--excerpt-->
Continued-post-text
```

<h4>Enabling comments with Disqus</h4>

Because Jekyll has no database, adding comments is easiest using [Disqus](https://disqus.com/), a popular online comment system. I followed the Disqus instructions but when loading my blog post pages, saw the "**We were unable to load Disqus**" error. First, I checked all the URL variables in the JavaScript. I caught an easy error where the URL was relative, not absolute. I added the beginning 'https://' as follows.

```html+erb
s.src = 'https://[[ site.disqus_shortname ]].disqus.com/embed.js';
```

Now inspecting the JS in the loaded page showed correct URL's, but the same error appeared. I played around with ```this.page.identifier``` and ```this.page.title``` in the disqus_config variable.

```
var disqus_config = function () {
      this.page.url = '[[ page.url | absolute_url ]]';
      this.page.identifier = '[[ site.disqus_shortname ]]';
      this.page.title = '[[ site.disqus_shortname ]]';
    };
```
This still returned the same error. Digging around in the Chrome console gave evidence that the issue was still related to this function. Eventually I completely deleted the variable and function altogether, and mysteriously, it works!

<h4>Adding SEO</h4>

A minor issue for my small blog, but I added the ```jekyll-seo-tag``` gem anyways. A quick ```gem install jekyll-seo-tag``` to get the gem, ```jekyll-seo-tag``` added to the _config.yml file, then ```[% seo %]``` added to the header file before the closing ```</head>``` tag.

<h4>Setting up assets and templates</h4>

Jekyll imposes very little structure, giving you the freedom to define your own workflow. I wanted an organised way to add assets, as well as store post ideas and post templates. I placed the assets as following, with the post images matching the _posts formatting, and an extra folder for images associated with static parts of the site.

```
 ├── assets
 │   ├── post-images
 │   ├───────2017-03-19-a.png
 │   ├───────2017-03-19-b.png
 │   ├───────2017-04-05-a.png
 │   ├── static-images
 │   ├───────me.png
 │   └───────email-icon.png
 ```
I did the same, but with hidden folders, for my post drafts and templates.

```
 ├── .misc
 │   ├── .templates
 │   ├───────YYYY-MM-DD-cs-post-template.md
 │   ├───────YYYY-MM-DD-cooking-post-template.md
 │   ├── .drafts
 │   ├───────tensorflow-slim-vs-learn.md
 │   └───────tumeric-turkey-chili.md
 ```

<h4>My reasons for choosing Jekyll</h4>

**Control**

Firstly, I must note that at least for now, I don't want to pay for my blog. This makes my comparison different than for those who are using WordPress with paid hosting. WordPress provides free hosting but the free hosting does not allow you to touch any code. You're limited to using their beautiful, hefty, sometimes laggy interface. As a former web developer, I'm well acquainted to WordPress's layout so finding my way around was not an issue.

Jekyll on the other hand, hosted for free by GitHub Pages, allows you to do everything that hosting WordPress on a paid plan does -- customising everything about your site. All the site code is on your local machine. Everything, including layout, theme, pages, posts, config -- is in your hands. You essentially build your own site. You make it, you break it. I'm a big fan of Ruby on Rails, and Jekyll is like a slimmed-down, static, single-purpose version of Rails.

**Concept**

I like to use the mouse as little as possible. ***A completely static blog where you write in Markdown and ruby and update the site simply with a Github commit and push...*** as a developer, this just says it all. It's not as beautiful out-of-the-box as WordPress, but sometimes less is more.

**Ease of use**

WordPress initially wins the ease of use battle. While getting vanilla Jekyll running did take 10 minutes, making the tweaks I needed to make my blog do what my WordPress version had (customising layout, adding post previews, enabling comments via disqus, importing my old blog posts) took about 2 hours. Now that I've figured out the fundamentals of Jekyll, however, it's definitely winning...



<h4>Other interesting Jekyll vs. WordPress comparisons</h4>

1. [overall comparison with pros and cons for both](https://www.slant.co/versus/999/1006/~wordpress_vs_jekyll)
2. [the owner of a high-traffic, lagging, malware-prone blog moves to Jekyll](https://www.sitepoint.com/wordpress-vs-jekyll-might-want-make-switch/)
3. [a developer chooses Jekyll for flexibility and control](http://progur.com/2016/08/jekyll-vs-wordpress.html)



