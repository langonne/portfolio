---
author: ["Léo"]
title: "Portfolio Presentation"
date: "2025-09-28"
description: "First introduction to my portfolio. Why I created it, which tools I used, and how I deployed it."
summary: "First introduction to my portfolio. Why I created it, which tools I used, and how I deployed it."
tags: ["portfolio", "hugo", "nginx", "docker"]
ShowToc: true
TocOpen: false
---

# Portfolio Presentation

I had been thinking about creating a portfolio for a while, but I wasn't sure what form it should take or how to build it.
When I recently came across [Hugo](https://gohugo.io/) again, which I had seen before, I felt like giving it a try. While exploring the themes, I discovered [PaperMod](https://github.com/adityatelange/hugo-PaperMod), and thought: "Why not?" It was the perfect opportunity to discover a new tool.

This portfolio is meant to showcase my projects, going beyond simple GitHub READMEs. I want to present more technical details, share some experiences, and maybe occasionally write a few articles. I will also try to include some of my older projects.

# Installation and Deployment

I started by installing Hugo and running a few local tests. I took the PaperMod template and customized it to suit my needs.

Once the basic setup was done, I moved on to deployment.
I already own a NAS where I host several services, so I decided to deploy the site in an NGINX container via Docker Compose.

```yaml
services:
  nginx-portfolio:
    image: nginx:alpine
    container_name: nginx-portfolio
    restart: unless-stopped
    volumes:
      - $DOCKERDIR/appdata/nginx/portfolio/site:/usr/share/nginx/html
    networks:
      - socket_proxy
      - t3_proxy
```

For routing, I use my existing Traefik setup. I just had to add a few labels for everything to work automatically.

```yaml
    labels:
      - "traefik.enable=true"
      # HTTP Routers
      - "traefik.http.routers.nginx-portfolio-rtr.entrypoints=websecure"
      - "traefik.http.routers.nginx-portfolio-rtr.rule=Host(`leo.$DOMAINNAME_1`)"
      # Middlewares
      - "traefik.http.routers.nginx-portfolio-rtr.middlewares=chain-no-auth@file"
      # HTTP Services
      - "traefik.http.routers.nginx-portfolio-rtr.service=nginx-portfolio-svc"
      - "traefik.http.services.nginx-portfolio-svc.loadbalancer.server.port=80"
```

As for updating the files, for now, I'm sticking with a simple approach, after each modification, I build the site locally and then transfer the generated files to the NAS.



