+++
date = '2025-01-10T13:07:23+01:00'
title = 'How to Publish Your Website Using Hugo and Survive to the Process'
author = "Felipe Torres González"
description = "Some hints about publishing a website using Hugo"
tags = ['hugo']
categories = ['nubecita']
+++

When I decided to start this blog, I considered two main options: [Wordpress](https://wordpress.com/) and [Hugo](https://gohugo.io). I have some experience with the former, as I already host some websites at [Nubecita](https://nubecita.eu), and I had a personal page before using Wordpress (a simple profile page).

However, I wanted something easy and simple. After all, my purpose for the site was to add some information about myself and a few blog posts. So why bother with all the overhead of Wordpress?

Let's walk through some of the tips I wish I'd known before.

## Make a List With Your Needs

It's important to have a clear idea of what you want your website to do.

Do you just need a simple profile page? Or do you want to become a blogger? Do you intend to write in more than one language? These are important questions because not all Hugo themes offer all the features you might need.

So, it's important to have a quick list of your needs as a starting point, as this will affect your next decision, which is choosing a theme.

## Select a Theme According To Your Needs

One of the first decisions you need to take when you aim to deploy a website using Hugo is choosing a theme.

There are some great themes out there, and you can find them on this [page](https://themes.gohugo.io/). But there's no standard set of features that all themes should have. I really liked the look of one theme, but it didn't have some features I needed.

When you're starting out, the last thing you'd want to do is modify a theme, or write one. So you need to find a theme that has as many of the features on your initial list as possible. Then you can start from there.

I made the mistake of trying to configure the website with all the features I wanted right from the start. It's unlikely you'll find a theme that covers all your needs. So choose a theme, start from something, and build your page step by step.

## Deploy The Example Website

Sometimes, you don't realise a theme doesn't work as expected or meet your needs until you actually deploy it. And when I mean deploy,  I mean to put it on a web server so that people can access it through the Internet.

Why is that? I spent a lot of time working on my website and when I hosted it, I realised the theme had troubles with CORS. After a bit of search, it seemed that it was something the author was already aware of, but she claimed to have solved it. Could I be bothered opening an issue in GitHub or trying to fix it myself? Not at that stage. The theme was nice, but not perfect, and I rather choose another theme than spending to much time fixing it.

So my best advice is: go to your themes's project, and find the demo site. Most themes do include such thing. Deploy it by yourself, and test it. That would be the best way to go before taking the final decision which theme to choose.

## Automate Your Deployment

It might seem like overkill, but trust me, it saves you loads of time. What do you prefer: using the site every time you want to publish content, or letting a script do the job for you? If you want the second option but are worried about the difficulty, don't worry! Just read my post about it:

<a href="/posts/website-deployment" class="button inline">Automated Website Deployment</a>

