---
author: ["Léo"]
title: "Présentation du portfolio"
date: "2025-09-28"
description: "Première introduction à mon portfolio. Pourquoi je l'ai créé, avec quels outils et comment je l'ai déployé."
summary: "Première introduction à mon portfolio. Pourquoi je l'ai créé, avec quels outils et comment je l'ai déployé."
tags: ["portfolio", "hugo", "nginx", "docker"]
ShowToc: true
TocOpen: false
---

# Présentation du portfolio

Cela faisait un moment que j'avais en tête l'idée de créer un portfolio, mais je ne savais pas vraiment quelle forme lui donner ni comment le réaliser.
En retombant récemment sur [Hugo](https://gohugo.io/), que j'avais déjà aperçu auparavant, j'ai eu envie d'essayer. En explorant les thèmes, je suis tombé sur [PaperMod](https://github.com/adityatelange/hugo-PaperMod), et je me suis dit : "Pourquoi pas ?". C'était l'occasion parfaite de découvrir un nouvel outil.

Ce portfolio a donc pour but de rassembler mes projets, en allant plus loin que de simples README sur GitHub. J'aimerais y présenter davantage de détails techniques, partager certaines expériences, et peut-être écrire ponctuellement quelques articles. J'essaierai également d'y intégrer certains de mes anciens projets.

# Installation et déploiement

J'ai commencé par installer Hugo et effectuer quelques tests en local. J'ai repris la template de PaperMod et l'ai personnalisée pour qu'elle corresponde à mes besoins.

Une fois la configuration de base terminée, je me suis attaqué au déploiement.
Je possède déjà un NAS sur lequel j'héberge plusieurs services. J'ai donc décidé de déployer le site dans un conteneur NGINX via Docker Compose.

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

Pour le routage, j'utilise mon installation Traefik qui est déjà opérationnel. Il m'a simplement suffi d'ajouter quelques labels pour que tout fonctionne automatiquement.

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

Concernant la mise à jour des fichiers, je reste pour l'instant sur une approche simple, après chaque modification, je build le site en local puis je transfère les fichiers générés sur le NAS.
