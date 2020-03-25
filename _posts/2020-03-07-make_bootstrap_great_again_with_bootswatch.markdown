---
layout: post
comments: true
title:  "Make Bootstrap great again with Bootswatch"
date:   2020-03-07 16:21:15
category: Technology
tags: [Bootstrap, Webpack, Bootswatch, Rails, React]
image: /assets/images/bamberg_altenburg_castle.jpg
image2: /assets/images/bamberg_altenburg_castle_mobile.jpg
---
[Bootstrap](https://getbootstrap.com/) gives you a good foundation for laying out your web application via their responsive grid system as well as a bunch of useful components to use.
The problem with it is that it doesnt really look great. Well, it's not so much that it doesnt look great, I guess its that it looks very... _bootstrappy_.
It especially doesnt look great when its up against all the advancements in HTML5 and CSS/CSS3 over the last few years. Maybe Bootstrap 5 will change this (apparently dropping [later this year](https://mdbootstrap.com/bootstrap-5/)).

One way to get some extra millage out of bootstrap in the looks department while still taking advantage of its benefits, is to write some custom CSS.
This can be tedious and time consuming, and having worked on a project where we went down this route, it turned into a bit of a mess to maintain over time.

A much quicker, less cumbersome, alternative is to look to something like [Bootswatch](https://bootswatch.com/), which gives you a bunch of different templates for your bootstrap driven application.

# Setting up Bootswatch

Setting up Bootswatch is very straight forward in a Webpack driven application. Simply put, its just a matter of installing it via Yarn and then importing it where you import bootstrap in the first place.

You can install Bootswatch via Yarn as follows:

```
yarn add bootswatch
```

Once installed, its simply a matter of importing the styles. There's a bunch of them available to choose from on the [Bootswatch](https://bootswatch.com/) site. 
In a Rails 6 application you can simply add 2 lines of code either side of your bootstrap import.

```scss
// app/javascripts/stylesheets/application.scss
@import "~bootswatch/dist/[INSERT THEME NAME HERE]/variables";

@import 'node_modules/bootstrap/scss/bootstrap'; // existing bootstrap import

@import "~bootswatch/dist/[INSERT THEME NAME HERE]/bootswatch";
```

If your using Reach, its even easier. It can be imported as follows (in App.js):

```javascript
// App.js

import React from 'react';
import logo from './logo.svg';
import 'bootswatch/dist/[INSERT_THEME_NAME_HERE]/bootstrap.min.css'; // Add this
import './App.css';
```

Thats pretty much it.

![Darkly Bootswatch]({{ site.url }}/assets/article_images/2020-03-07-make_bootstrap_great_again_with_bootswatch/darkly_screenshot.png)
![Sketchy Bootswatch]({{ site.url }}/assets/article_images/2020-03-07-make_bootstrap_great_again_with_bootswatch/sketchy_screenshot.png)


If you'd like to play around with this in a Rails 6 environment, I have created a blank [Rails 6 template repo](https://github.com/johnmfarrell1/rails6-bootstrap4) complete with Bootstrap and Bootswatch as well as a default welcome index page with some components.