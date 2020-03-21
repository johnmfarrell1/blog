---
layout: post
title:  "Using Jekyll for static websites"
date:   2020-01-30 11:34:20
category: Technology
tags: [Ruby, Jekyll]
image: /assets/images/royal_canal_bridge.jpg
---
For those who have never heard of [Jekyll](https://jekyllrb.com/), Jekyll is a simple static website development framework build in Ruby.
It uses a bunch of existing tools and bundles them together to make it easier to create a static website, perfect for blogs or static company websites.
It predominately uses [markdown](https://daringfireball.net/projects/markdown/) syntax for articles and the [liquid](https://github.com/Shopify/liquid/wiki) templating engine with HTML and CSS - no databases.
Jekylls [philosophy](https://jekyllrb.com/philosophy/) is simple...
> Jekyll does what you tell it to do — no more, no less. It doesn't try to outsmart users by making bold assumptions, nor does it burden them with needless complexity and configuration. Put simply, Jekyll gets out of your way and allows you to concentrate on what truly matters: your content.

Jekyll is in fact the engine behind Github Pages.

Originally when I had the idea to do a blog, I wanted it to be written in something I could somewhat play around with but without it getting in the way of writing blog content.
I had considered using Rails, but Rails is a far to much for a simple static blog. I thought it might be interesting to use
ReactJS, but given that I'm only just beginning to dip my toes in React, I didnt want it to be getting in the way of producing content.
Sinatra was another option, but most of the examples of blogs running on Sinatra seemed to use a database which I wanted to avoid.
Enter Jekyll, which is the perfect middle ground. And, given I wanted to sit it on Github Pages, its the perfect tool for the job.

# Getting started with Jekyll
Like any framework, Jekyll gives you generators that allow you to create blank websites that you can build on.
It requires bundler also, so you need to install that too.
The following commands will generate a completely blank jekyll website and fire it up locally

```
gem install jekyll bundler
jekyll new my-website
cd my-website
bundle exec jekyll serve
```

Navigate to http://localhost:4000 and you should see your Jekyll site running there ready for development.
I'd recommend generating a blank website like this and just play around with it to get familiar with the structure of the project and how things work.
There's also plenty of documentation on the official [Jekyll page](https://jekyllrb.com/docs/)

# Creating a static website/blog
You could start from a blank slate with Jekyll and take the time to build up a nice looking blog, but the great thing about Jekyll is that there are loads of great, ready to use projects out there already.
Using an existing project means you can focus more on content rather than on the specifics of getting the blog created.

Some examples

* [Helium](https://github.com/heliumjk/heliumjk.github.io)
* [Modern Blog](https://github.com/inded/Jekyll_modern-blog)
* [Modern Blog 2](https://github.com/Open-SL/Jekyll-Modern-Blog)
* [Mediator](https://github.com/dirkfabisch/mediator) (This blog is a modified version of this)
* [Minimal Resume](https://github.com/murraco/jekyll-theme-minimal-resume)
* [Cause](https://github.com/CloudCannon/cause-jekyll-template)
* [Photorama](https://github.com/sunbliss/photorama)
* [Spectral](https://github.com/arkadianriver/spectral)
* [Jekyll-Uno](https://github.com/joshgerdes/jekyll-uno)
* [Creative Theme](https://github.com/volny/creative-theme-jekyll)
* [Grayscale](https://github.com/jeromelachaud/grayscale-theme)

# Hosting your blog/static website
The obvious place to start would be [Github pages](https://pages.github.com/). 
Its completely free to host a static web site on Github pages directly from your source code repository.
