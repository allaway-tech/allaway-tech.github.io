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
  - And adding the DNS records in Cloudflare for the VPS;
I fear that this maybe a fairly wordy and dry post so with no further delay lets get started.

## My local system
I feel that it is a good place to start is with the laptop I am currently working on now. As I said in the previous post it is a 2014 apple macbook air laptop. It doesn't have the best spec in the world (dual core Intel® Core™ i5-4260U CPU @ 1.40GHz with 4GB of RAM) but it has coped admirably so far. It has NixOS 24.05 installed on it with KDE Plasma 6 running on Wayland. Most of this isn't important but for the fact it is running NixOS. This means that all of the tools I am using are natively installed in my OS and it makes my life a lot easier. I'm sure there are ways that you can do this on other platforms but I have never tried. If someone manages to do it chuck it in the comments and I will add a link to it.

## Buying a domain

## Hetzner
### Buying a VPS

### Buying storage

## NixOS flakes
The [NixOS (unofficial) wiki](https://nixos.wiki/wiki/flakes) has the following description of flakes:

`Nix flakes provide a standard way to write Nix expressions (and therefore packages) whose dependencies are version-pinned in a lock file, improving reproducibility of Nix installations. The experimental nix CLI lets you evaluate or build an expression contained within a flake, install a derivation from a flake into an User Environment, and operate on flake outputs much like the original nix-{build,eval,...} commands would.`

Clear as mud? great lets move on.

Realistically the part we are interested in is the part about the dependencies being version-pinned and the reproduciblity. This means that we can duplicate this installation on other VPS's or hardware with only updates to the hardware specific parts of the flake.

## Version control
### Setup git on the local machine

### Link the repo to GitHub

## Flattening the VPS

## Adding DNS records to Cloudflare
