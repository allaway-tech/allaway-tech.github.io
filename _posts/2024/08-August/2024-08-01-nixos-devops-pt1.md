---
layout: post
title: NixOS DevOps
date: 2024-07-16
categories: [NixOS, DevOps] # Can be anything
tags: [nixos,devops,flakes] # Must be lowercase
---
# NixOS DevOps pt1

TODO: link to udemy course

About 18 months ago I took an excellent Udemy course by [Predrag Mijatovic](https://github.com/predmijat) called [Realworld DevOps project from start to finish](https://udemy.com/course/real-world-devops-project-from-start-to-finish). It is a very insightful walk through of building a basic DevOps platform from a basic VPS image up. Predrag's course focuses on Arch Linux and Ansible to build out the system which produces a repeatable process. I even ended up nuking my home assistant install and now it lives on Arch and was built with an Ansible playbook. This was all good until I lost that playbook when the hard drive in my laptop failed. That's right folk. Rookie move here. **Lesson 1:**

 > Always backup your ansible playbooks
 {: .prompt-warning }

For a long time I toyed around with the idea of putting Ansible Semaphore on my VPS and then rebuilding the playbook to continue managing my server but in the end I just bailed and now hand manage it. This is fine but it is now my sixth OS and a very much my least favourite[^footnote1]. For now this server is relegated to being an Arch box that I need to re-google the commands every time I interact with it (I do manage to remember `pacman -Syu` but that is about it). If I hadn't lost that playbook the I probably would have continued down the ansible route, nuked a few more boxes and wouldn't be writing this blog series now. However, I decided to go with NixOS (thanks [Jupiter Broadcasting](https://jupiterbroadcasting.com)) when I reinstalled that very crufty 2014 macbook air the previously housed that ansible playbook. For about a year I kept plodded along with it feeling slightly inconvenienced by having to install any packages I wanted in the `configuration.nix`{: .filepath} file and in all fairness if I had more time I probably would ended up putting Ubuntu on it. However, the laptop has been (mostly) stable and I have had relatively few issues using it as a daily driver and recently KDE Plasma 6 has made it a much more joyful experience. While this may not seem spectacular it has lasted longer than Ubuntu did on my HP laptop and is running more smoothly than any macOS or Windows installations I have had for this long in the past.
 
## Why is NixOS better?
 
I think there are two main reasons I have been getting better mileage from NixOS. The first is down to me being a bit of a software junkie. I have always have a billion projects on the go and need to install this package and that dependency. And then invariably I will run out of space on my drive so will go through and cull files and packages from my laptop which then leads to projects not working and me shelving them. So how is NixOS different? Well in this use case it is down to the way that Nix allows for multiple versions of the same package to be installed. Each version of a package has its own unique hash which is then reference by the other packages that need it. So what does this mean? Well if I install something that needs python 3.10 then python 3.11.6 gets added to my nixStore with and ungodly hash in its path (`/nix/store/lrd13inbd799k7q55zcibm8ybz2c4nkq-python3-3.11.6`). Now if I install something that wants python 3.12.4 then this gets added to the nixStore as well. And, yes you've guessed it, it has its own ungodly has (`/nix/store/x2nqfjj5navz0w4ish9vvvp48bc9vmkl-python3-3.12.4`). This now means those two different versions of python coexist and the two apps that want the respective version are happy to find the correct packages under those ungodly hashes. The downside of this is that your nixStore can grow very quickly so make sure you give NixOS plenty of space if you decide to install it.
 
The second factor that I feel makes NixOS perfect for this project is that it is fully declarative (like ansible). In this respect NixOS doesn't really stand above ansible but when running the entire playbook (when finished) this lowly macbook air struggled to evaluate and make the changes[^footnote2]. So far (at the point of writing this post) NixOS seems to be coping with rebuilding the flake interatively with much less ramping up of the fans.

## Let's get setup
So how are we going to do this project? If all goes to plan our stack is going to look something like this:
  - a [GitHub repository]() to store/backup our nix file (learning from ansible Lesson 1)
  - Hetzner for a VPS as it is cheap and I am in Europe so latency isn't too bad
  - Cloudflare for our dns and purchasing a domain
  - Setup acme SSL certs
  - Setup tailscale for VPN access
  - Have Docker for services (This is only for a few uses as most are already packaged for Nix)
  - Setup LXC/LXD (this is something I am still researching at the time of writing this but it seems possible)
  
Services I intend to try and host on this VPS are:
  - iredmail (LXC/LXD)
  - Zulip
  - Traefik (docker)
  - piHole (docker)
  - Nextcloud
  - Checkmk
  - Borg
  - GitLab
  - A website
  - And a backup checker
  
So strap in and follow along as I work through what I can and can't recreate in pure NixOS. And try not to commit too many API keys and secrets to my GitHub repo.

---
[^footnote1]: For those who are bored enough to be interested the line up is:<br />a. NixOS (daily driver)<br />b. macOS (work)<br />c. Ubuntu (home and VPS servers)<br />d. Windows (work)<br />e. Debian (OMV nas)<br />f. Arch Linux (Home assistant server) 

[^footnote2]: There could be some of the crufty old macOS vs a cleanish NixOS install at play here but I do think I have been using this NixOS install for longer than I had my user account on the mac.
