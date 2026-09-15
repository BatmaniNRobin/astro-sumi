---
title: "My Self-Hosting Setup"
description: "A brief overview of my homelab and self-hosting setup."
pubDate: 2026-09-15
updatedDate: 2026-09-15 # optional
tags: ["devops", "Lifestyle", "tech"]
draft: false # drafts are visible in dev, dropped from builds
heroImage: ./cover.jpg # optional, optimised through astro:assets
heroImageAlt: "…"
---

## Why Self Host?

Self hosting and Open-Source Software (OSS) allows you the benefit of controlling your privacy and maintaining your own data. 

I prefer OSS and self-hosting since I can control where my data and services live. If I’m dependent on a service, then I want to decide when I upgrade it, including any changes in terms of service affecting my use. If the service no longer serves my interest or maintained, then I can decide how to migrate my data and dependencies rather than being in the hands of large corporations.

I’m still dependent on several services I can’t self host or aren’t OSS, but my hope is to minimize the rug pulls I’m susceptible to while meeting my needs. Once I’m no longer experimenting with a service and relying on it, I try to make regular contributions. I hope once you’re dependent on a service, you do the same to keep the service maintained, instead of looking at OSS as the “free” or “cheap” solution.

## My Machines

### Future

NAS - DS923+ or better

Compute - Minisforum MS-03 or better

### Current

Old Gaming Laptop - MSI GT72 6QD - using proxmox

## Home Server

As it stands currently I am using my old gaming laptop, using Proxmox OS, as both my NAS and compute server. 

### Services

Immich - for backing up my photos and maintaining full availability on my phone

Jellyfin - to host my media, movies, tv shows, music

Caddy - acts as my reverse proxy, assigning domain names and generating Let’s Encrypt certificates

Proxmox Backup Server - backup of all my proxmox LXC containers

vaultwarden - self-hosted bitwarden password manager. benefit of self-hosting this vs their free-tier cloud version is getting to use TOTP and authenticator all built in

your-spotify - i LOVE my own data and listen to a ton of music, only right to cross these interests

nextcloud - after using this for a bit not sure if i’ll keep it but it was meant to replace google calendar for me

uptime kuma - for alerts and service uptime monitoring

homepage - browser homepage to all my services with widgets for one dashboard metrics

homeassistant - for smart home automations - mainly zigbee devices for me currently

pihole - DNS sinkhole/ad-blocker

gaming / minecraft servers via docker - For hosting my friends and I’s game servers for maximum resources and always-on availability

### Networking

Currently I’m leveraging Tailscale from trusted devices to access all of these services when away from home. So long as the server is online and the network is available, everything remains available due to MagicDNS. Since I also leverage a DNS sinkhole via Pi-Hole, I benefit from large-scale ad-blocking by remaining connected frequently. If I’m ever not connected, I have Proton VPN available.

### Future Changes

- bezelle - docker stats
- more alerts - prometheus, grafana
- homeassistant automations
- gaming servers
- samba - fileshare system
- portainer - management via web ui
- self-hosted LLM - for privacy, unlimited tokens, and no commercial cooling / eco damage. uses existing home cooling system

### Future Improvements

The only issue i’m facing currently is that my dedicated GPU a 970M on this old laptop is not being used. For some reason proxmox couldn’t boot when getting installed without switching the GPU off via a physical button on the device. However, I believe a few apps can benefit from utilizing this so I’m looking into dual-booting the laptop and leveraging MSI SCM to re-enable the device for the OS to recognize it.

## conclusions

Being in tech is super cool :)