---
layout: post
title:      "Reflecting on React"
date:       2019-05-05 08:36:14 +0000
permalink:  reflecting_on_react
---

![](https://upload.wikimedia.org/wikipedia/commons/8/86/Lemaire_Channel_reflection.jpg)

It’s been a little while since I finished my final project and I feel that it’s time for a little reflection. Sometimes it’s easy to walk away from a project and forget about it. Well ‘forget’ is a little strong, but it is good to come back and ponder what might or could have been.

Firstly, I would say that as soon as the app fires up, there’s a huge unserialized (for the most part anyway) data fetch from the API. I couldn’t see any way around this, the data was actually intended for a component that would actually mount further down the rabbit hole (I’m using client sider routing with react-router). However, being that the render method fires before any lifecycle method (in this case componentDidMount), I had to place this in an odd location in my code.

I think perhaps I might come back one day and change some things up, well I always say that before becoming engrossed in another project… Anyway, the only other thing that comes to mind right now is that although I was very happy with the styling of my project, I wished I’d had a deeper understanding of how it works going in, indeed, of CSS generally and that is now something I’m taking care of!

Anyway, maybe I’ll be back soon with some more reflections…
