---
layout: post
title: "Deploying Ghost"
date: 2024-02-07 14:30:00 +0100
categories: blog
author: <author_id> 
published: true
---

### Why Ghost
For a long time, Wordpress was the preferred choice for a CMS/publishing solution due to its large ecosystem, extensive install base, and the support it received from numerous providers. However, it has often been viewed as a bit of a behemoth, requiring an AMP server (Apache, MySQL, and PHP).
I also explored the possibility of using static pages with Hugo or Jekyll, hosted on either Cloudflare or Github pages (both of which function similarly). However, my proficiency with these solutions has diminished over the years, and I no longer have the patience to grapple with them. All of this made Ghost a good fit for me.
Ghost is an open-source, headless CMS (back-end system that separates content creation from its presentation) built on Node.js.

<img src="/assets/images/giphy.gif" alt="I have a plan">



### My original plan
I found a helpful guide on [Noted](https://noted.lol/self-host-ghost/) that provided step-by-step instructions on how to self-host Ghost. A follow up guide provided instructions on how to expose [Ghost](https://noted.lol/cloudflare-tunnel-and-zero-trust/) to the internet using Cloudflare Zero Trust.
I started by creating a debian virtual machine on my ESXI host called Tuchanka (bonus points if you catch the referance). It's running on an [Intel Skull Canyon NUC (NUC8i7HNK)](https://www.intel.com/content/www/us/en/products/sku/126141/intel-nuc-kit-nuc8i7hnk/specifications.html). Following Noted's instructions, I deployed Docker and Portainer, and then attempted to deploy Ghost.
This is where I stumbled into my first hurdle, it would not work properly. After some debugging I traced the issue to problems with MySQL. Ultimately, I switched to MariaDB, which resolved the issue on the first attempt. However, it’s worth noting that Ghost does not recommend this solution for production environments.
This is the docker-compose file I used. You can also find it as [Gist](https://gist.github.com/nechered/e066424ff3a18354ca4c264d321809cc).

```
version: "3.3"
services:
  ghost:
    image: ghost:latest
    restart: always
    ports:
      - "2368:2368"
    depends_on:
      - db
    environment:
      url: http://domain
      database__client: mysql
      database__connection__host: db
      database__connection__user: ghost
      database__connection__password: ghostdbpass
      database__connection__database: ghostdb
    volumes:
      - /home/ghost/content:/var/lib/ghost/content

  db:
    image: mariadb:latest
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: [PASSWORD]
      MYSQL_USER: ghost
      MYSQL_PASSWORD: ghostdbpass
      MYSQL_DATABASE: ghostdb
    volumes:
      - /home/ghost/mysql:/var/lib/mysql
```

At this point Ghost was running on http on my local server. Now it was time to expose Ghost to the internet, and maybe get an "s" in http.
[Cloudflare Zero Trust](https://www.cloudflare.com/en-gb/plans/zero-trust-services/) offers a free tier that enables you to expose your internal infrastructure to the internet. It also allows you to set access policies for your domain, subdomain, or individual pages. Cloudflare provides a straightforward setup guide to assist users in the process and you can use docker to run the secure tunnel.
If you require access to your internal infrastructure and prefer not to deal with the complexities of a VPN, Cloudflare Zero Trust is the choice I recommend.
Eventually, I began experiencing stability issues. Interestingly, these were not related to Cloudflare, but were instead tied to my database setup.. Might be the reason that Ghost does not recommend this solution. In the end I decided to migrate to another hosting solution.

<img src="/assets/images/giphy2.gif" alt="Horrible mistake">


### What I actually ended up doing
First I took some inspiration from [Molly White](https://www.citationneeded.news/substack-to-self-hosted-ghost/) and tried DigitalOcean (DO), however their 1-click install failed multiple times. I was using a small server (or droplet as DO calls them), but time after time it failed. After five tries I just gave up on DO.
Although I could have used Azure, where I already run a Ghost instance for my project ([perspektivet.news](https://perspektivet.news/)), I decided to explore other options.
That’s when I discovered [PikaPods](https://www.pikapods.com/). They offer 1-click installs on Open Source web apps at a very reasonable price, and even provide a 5 USD credit. Within minutes, I had Ghost running on my own domain. All for the low low price of 1.90 USD (I choose more storage and memory).

<img src="/assets/images/picapods.png" alt="Picapods">
11
