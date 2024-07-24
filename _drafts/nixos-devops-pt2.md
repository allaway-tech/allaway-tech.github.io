---
layout: post
title: Part 2 - The boring bits - NixOS DevOps
date: 2024-07-24 15:13 +0100
series: nixos-devops
categories: [NixOS, DevOps] # Can be anything
tags: [nixos,devops,flakes] # Must be lowercase
---
![The NixOS logo with other logos on top](media/posts/images/2024-07-20-nixos-devops/nix-devops-splash.png)

As you may have gathered this post is the ground work that I feel we have to cover but isn't really part of the DevOps build out. We are going to have a brief look at:
  - Buying a domain from Cloudflare; 
  - Getting a VPS and some storage from Hetzner; 
  - A brief overview of how NixOS flakes can work in the way we want;
  - Setting up the GitHub repo for backing up our flake to;
  - Flattening that VPS so we can install NixOS on it; 
  - And adding the DNS records in Clouflare for the VPS;
I fear that this maybe a fairly wordy and dry post so with no further delay lets get started.

## My local
I feel that it is a good place to start is with the laptop I am currently working on now. As I said in the previous post it is a 2014 apple macbook air laptop. It doesn't have the best spec in the world (dual core Intel® Core™ i5-4260U CPU @ 1.40GHz with 4GB of RAM) but it has coped admirably so far. It has NixOS 24.05 installed on it with KDE Plasma 6 running on Wayland. Most of this isn't important but for the fact it is running NixOS. This means that all of the tools I am using are natively installed in my OS and it makes my life a lot easier. I'm sure there are ways that you can 
