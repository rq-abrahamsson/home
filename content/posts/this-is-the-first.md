---
title: This is the first
summary: "It is now possible to create and read blog posts \U0001F389"
date: '2020-01-05T20:07:56+01:00'
draft: false
---
Finally I can say that my blog is good enough to start writing posts. A really exiting start of a new year and decade. 🎉

This site is still very unfinished with a lot of quirks but a goal of the site is to teach myself how to create a site that performs really well. At start measurements of the site will be made with [Lighthouse](https://developers.google.com/web/tools/lighthouse/) and [WebPageTest](https://www.webpagetest.org/).

Much inspiration for how this is and will be taken from the really fun and useful pages, which I really recommend reading through: 

* [motherfuckingwebsite.com](https://motherfuckingwebsite.com/)
* [bettermotherfuckingwebsite.com](http://bettermotherfuckingwebsite.com/)
* [perfectmotherfuckingwebsite.com](https://perfectmotherfuckingwebsite.com/)
* [thebestmotherfucking.website](http://thebestmotherfucking.website)

## What is powering this site

With that in mind I picked some tech that will help me accomplish this. First I wanted a static site generator and for that [Hugo](https://gohugo.io/) was chosen. I have previously worked with [Metalsmith](https://metalsmith.io/) and [Gatbsy](https://www.gatsbyjs.org/) which I both like but I wanted something as simple as Metalsmith but I knew about Hugo and how fast it was. 

For hosting I have used [Netlify](https://www.netlify.com/) because of the simplicity to set up a site, which is really seamless and you get some really nice features and everything is published to a CDN. 

It could have been enough there and just write markdown files by hand in the Git-repo. But Netlify have conveniently created an open source CMS that integrates really well with static site gererators called [Netlify CMS](https://www.netlifycms.org/) which also can be used without Netlify. The nice thing about it is that you do non need a database because it is only some html, css and Javascript as a layer on top of your Markdown files in your Git-repo. The downside of that is that your git log can become very cluttered but it is possible to have it in a separate repo but that is not what I have done.

## What is this site for

This site is here as a way for me to develop skills about programming I do not learn in my day job and as a way for me to be able to write down my thoughts on subjects related to programming. For the most part I will probably write about the development of this site and of the drinking game I have been working on for the past year called [Giving Game](https://giving-game.se/) which is written using [Elixir](https://elixir-lang.org/) and [Elm](https://elm-lang.org/). So let's see what will become of this, at least I hope that i will continue to write some posts.
