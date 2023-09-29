+++
author = "Dijkstra"
date = 2019-10-31T10:39:19Z
description = "A very little knowledge about Cloudflare's SSL feature caused too many redirect issue and took three hours to find and fix."
draft = false
slug = "cloudflare-nginx-letsencrypt-ssl-too-many-redirect"
summary = "A very little knowledge about Cloudflare's SSL feature caused too many redirect issue and took three hours to find and fix."
tags = ["nginx", "cloudflare", "letsencrypt"]
title = "Cloudflare, Nginx, Letsencrypt SSL setup. Too many redirect issue. (Silly mistake)"

+++


I know you are familiar with Cloudflare, Nginx, and Letsencrypt. In case you didn't know, **Cloudflare** does a lot of things including DNS management, DDoS protection, free SSL, caching and a lot more. **Nginx** is one of the most popular HTTP server / reverse proxy/load balancer and **Letsencrypt** let you install SSL in your server.

I am a fan of all these three technologies. I use Cloudflare a lot because why not? It is giving me protection from DDoS, free SSL, etc for free. When I was planing for creating this blog, the first cms came to my mind is WordPress. It's free, easy to maintain, tons of plugins to do stuff and cheap. But, WordPress was not the only cms that were in my mind. I already know about Ghost and I have a very soft corner for it since it is written in NodeJS. NodeJS and Nginx scales and performs better than WordPress and Apache. I could use Nginx with WordPress but ehh. Finally, I decided to use Ghost.

Bought this domain, set Cloudflare's DNS, created an EC2 Ubuntu 18.04 instance, installed all the necessary stuff and then followed Ghost's official installation instruction. I skipped the SSL part because I decided to use Cloudflare's SSL although I know very little about Cloudflare's SSL feature. All I knew was I enable it from Cloudflare's dashboard and I am good to go. In reality, it was that easy as well. But, I was facing a problem with Ghost. When I tried to add an item to the navigation menu, Ghost forcefully changing the URL from HTTPS to HTTP. I get it. Ghost doesn't know that I am using HTTPS. So, I decided to switch off Cloudflare's SSL and install SSL on my server. This is where I messed things up. **I should not have switched off Cloudflare's SSL.** There were a couple of options: **_Off, Flexible, Full, Full (Strict)._** Go to their website to know more about these options. Initially, I was using **Flexible** then switched to **Off**.

Ghost provides handy CLI to install SSL and it uses Letsencrypt. Since I was using Cloudflare's DNS, when I hit my website using HTTPS it goes to Cloudflare and Cloudflare redirect it to HTTP since I switched SSL off. Cloudflare redirects to my server using HTTP but my Nginx config redirects every request to HTTPS. It was creating a redirection cycle like the following steps.

1. Browser (Input HTTPS url to address bar).
2. Cloudflare DNS resolver. Cloudflare redirects to HTTP (Since SSL is off!).
3. My Server. My server redirect to HTTPS
4. Step 2 again.

I was like what the hell. What is happening? I kept modifying my Nginx configuration in many ways but could not make it work. Searched for the issue. I have found a lot of answers but honestly, I didn't realize that they are the same victim as mine. I thought mine was different.

After wasting two to three hours I have found a command to trace my request. Then I have found the cycle between my server and Cloudflare. I went to Cloudflare's dashboard and found my fault. Changed my SSL configuration from **Off** to **Full.** Then, it started to work as expected.

I never wasted that much time setting up an SSL certificate, EVER!

> Little knowledge is a dangerous thing.
