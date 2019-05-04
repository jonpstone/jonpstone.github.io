---
layout: post
title:      "Something in the Pipeline"
date:       2019-05-04 08:52:01 +0000
permalink:  something_in_the_pipeline
---

![](https://cdn-images-1.medium.com/max/800/1*yJMJMQNLuqim5G1DL7pyFg.jpeg)

So I was introduced to the asset pipeline at the end of the basic JavaScript section with Flatiron School. Despite modern front end frameworks rising in prominence, it’s still important to understand what it is and some fundamentals. Here, I’m going to try and press home, what, in my opinion, is the most important thing to remember about the asset pipeline. First, however, how is it defined? Ruby on Rails documentation states.

The asset pipeline provides a framework to concatenate and minify or compress JavaScript and CSS assets. It also adds the ability to write these assets in other languages and pre-processors such as CoffeeScript, Sass, and ERB. It allows assets in your application to be automatically combined with assets from other gems.
I would say that crucially, the most important thing is that when referencing your assets in a manifest location in the assets folder, like so…

```
//= require jquery
//= require myJsFile
```

Notice how I don’t need to specify a file path or an extension type? Welp, Rails will scour your directory to find a match! You don’t need to be more exacting than what’s above. What’s crucial though, in my opinion, is to remember that you can place files anywhere and reference them in your manifest. This above all else gives you a tremendous amount of flexibility when developing in Rails!
