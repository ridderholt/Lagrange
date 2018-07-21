---
layout: post
title: "Moving my blog"
author: "Robin Ridderholt"
categories: posts
tags: [blog]
---

For a couple of years I have been hosting my blog on [Ghost.org](https://www.ghost.org) and I have no complaints but I was paying about $7 per month and I know that others had moved their blog to [Github Pages](https://pages.github.com) which is free and I thought that it could be a fun challange to see how much effort it would be to move everything and not spend any money.

So I started researching and I made a short list of features that I wanted:
* Free
* Custom domain
* Customizable menu and design
* Markdown for writing posts
* Encrypted connection (HTTPS)
* Comments
* Be able to redirect from my old domain

## Jekyll
I knew from start that it would be fun to build and design my own blog from scratch, I also knew that it would take to long and since my design skillz aren't the best that it probably wouldn't be that nice. [Jekyll](https://jekyllrb.com/) is recommended by GitHub for GitHub Pages so I started to look at that to begin with. Jekyll is a static site generator that generates HTML from plain text files.

I quickly installed it and installed a theme (which there are a lot to choose from) which allowed me to customize the menu and some design features to make the theme a little bit more personal. With Jekyll I could get the customizable menu and design and uses markdown to write posts, and it's free.

## GitHub Pages
The documentation for GitHub Pages is very easy to read and understand so it didn't take long to get my Jekyll site up and running. GitHub Pages allows me to have a custom domain (the default domain is: username.github.io but I wanted to use my own domain) and it's free.

## Disqus
In my previous blog I had already used [Disqus](https://disqus.com/) and I'm very happy with it and since it wasn't any problem moving comments from my old domain to my new domain with Disqus I decided to continue to use it. So I could cross comments of my list, and it's free.

## Cloudflare
Since I already had my new domain name and I wanted to have my new blog served with an encrypted connection and I didn't want to pay for an SSL-certificate I turned to [Cloudflare](https://www.cloudflare.com/). They have a service in which you use their nameservers for your domain and they will encrypt the traffic (and cache content etc.) which was exactly what I needed, and it's free.

## Redirect my old blog
I wanted links to the old blog to conintue to work and the root domain isn't a problem but I also wanted deep links to work so if someone had bookmarked an old blog post they would still be able to get to that specific post but on my new blog. Such a service isn't available with my domain registrar so I thought about just writing a small application which would recieve all the request for my old domain and respond with a 301 Redirect to my new blog but that would mean that I had to host that somewhere. After a quick Google session I found [301redirect.website](https://301redirect.website/) which does exactly that, and it's free (if you provide a link to their page on your page).

## Summary
So in the end I was able to meat all my requirements for the new blog without spending any money and I learned a lot doing it.





