---
title: This is the first
summary: "It is now possible to create and read blog posts \U0001F389"
date: '2020-01-05T20:07:56+01:00'
draft: false
---
Finally I can say that my blog is good enough to start writing posts. A really exiting start of a new year and decade. 🎉

This site is still very unfinished with a lot of quirks but a goal of the site is to teach myself how to create a site that performs really well. I have already tried to make the site perform good and it scores really well on [Lighthouse](https://developers.google.com/web/tools/lighthouse/) and [WebPageTest](https://www.webpagetest.org/).

Much inspiration for how the site look and is build have been and will be taken from the following really fun and useful pages, which I really recommend taking a look at: 

* [motherfuckingwebsite.com](https://motherfuckingwebsite.com/)
* [bettermotherfuckingwebsite.com](http://bettermotherfuckingwebsite.com/)
* [perfectmotherfuckingwebsite.com](https://perfectmotherfuckingwebsite.com/)
* [thebestmotherfucking.website](http://thebestmotherfucking.website)

## What is powering this site

With those pages as basis I picked some tech that will help me accomplish this. First I wanted a static site generator and for that [Hugo](https://gohugo.io/) was chosen. I have previously worked with [Metalsmith](https://metalsmith.io/) and [Gatbsy](https://www.gatsbyjs.org/) which I both like but I wanted something as simple as Metalsmith but I knew about the speed of Hugo and I have been wanting to try it out for a while now. 

For hosting I have used [Netlify](https://www.netlify.com/) because of the simplicity to set up a site static site, which is really seamless and everything is published to a CDN. 

It could have been enough with only those two at the start and just write markdown files by hand in the Git-repo. But Netlify have conveniently created an open source CMS that integrates really well with static site gererators called [Netlify CMS](https://www.netlifycms.org/) - which also can be used without Netlify. The nice thing about it is that you do not need a database because the CMS consits only of some html, css and Javascript as a layer on top of your Markdown files in your Git-repo. The downside of that is that your git log can become very cluttered with post updates but it is possible to have it in a separate repo although that is not how I am using it.

## What is this site for

This site is here as a way for me to develop skills about programming I do not learn in my day job and as a way for me to be able to write down my thoughts on subjects related to programming. For the most part I will probably write about the development of this site, experiences from playing with new technology and of the drinking game I have been working on for the past year called [Giving Game](https://giving-game.se/) which is written using [Elixir](https://elixir-lang.org/) and [Elm](https://elm-lang.org/). So let's see what will become of this, my goal is that I will write at least a couple of more posts this year.
