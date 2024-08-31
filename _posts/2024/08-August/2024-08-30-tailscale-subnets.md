---
layout: post
title: Local access with tailscale subnet routing
date: 2024-08-30
categories: [Tailscale, Networking] # Can be anything
tags: [tailscale,networking,linux] # Must be lowercase
---
Of late I have seen a couple of posts on Reddit where people have been asking why they cannot access a tailscale node via the local network if it is part of a subnet that is already being routed to their tailnet. I myself have found this problem in the past. Normally people (including me) recommend running the following commands `tailscale up --reset` and then `tailscale up` without the `--accept-routes` flag. This make it work but if we want to have that node able to access resources in the other subnets that are being routed on the tailnet we have to make the decision between local or remote access. After this small spate of people asking (and not really getting a proper explanation as to why it doesn't work) I decided to take a bit of a further look at it.

I would you preface this post with a disclaimer that I am neither a network nor a Linux professional. I would class myself as an advanced amateur and this is just me documenting my findings. I also know that the font on the pictures I have included is quite small but at time we have a lot of data on the screen that we need to see.

## Setting up the test lab
To test this I used terraform to spin up a lab on Hetzner. Using cloudinit I joined Ron and Hermione to my tailnet with a pre-authenticated key. This is the relevant section from terraform that describes the lab ([TL;DR below](#tldr))[^footnote1]:
```terraform
resource "hcloud_network" "Hogwarts" {
  name     = "Hogwarts"
  ip_range = "10.0.0.0/16"
}

resource "hcloud_network_subnet" "Hogwarts-subnet" {
  type         = "cloud"
  network_id   = hcloud_network.Hogwarts.id
  network_zone = "eu-central"
  ip_range     = "10.0.1.0/24"
}

resource "hcloud_network" "Durmstrang" {
  name     = "Durmstrang"
  ip_range = "10.1.0.0/16"
}

resource "hcloud_network_subnet" "Durmstrang-subnet" {
  type         = "cloud"
  network_id   = hcloud_network.Durmstrang.id
  network_zone = "eu-central"
  ip_range     = "10.1.1.0/24"
}

resource "hcloud_server" "Harry" {
  name        = "Harry"
  server_type = "cx22"
  image       = "ubuntu-20.04"
  location    = "nbg1"
  ssh_keys = [
    "demo@allaway.tech",
    "demo@dolphin"
  ]

  network {
    network_id = hcloud_network.Hogwarts.id
  }

  depends_on = [
    hcloud_network_subnet.Hogwarts-subnet
  ]
}


resource "hcloud_server" "Hermione" {
  name        = "Hermione"
  server_type = "cx22"
  image       = "ubuntu-20.04"
  location    = "nbg1"
  ssh_keys = [
    "demo@allaway.tech",
    "demo@dolphin"
  ]

  network {
    network_id = hcloud_network.Hogwarts.id
  }

  network {
    network_id = hcloud_network.Durmstrang.id
  }

  user_data = templatefile("hetzner.tftpl", { tailscale_auth_key = var.tailscale_auth_key })

  depends_on = [
    hcloud_network_subnet.Hogwarts-subnet,
    hcloud_network_subnet.Durmstrang-subnet
  ]
}


resource "hcloud_server" "Ron" {
  name        = "Ron"
  server_type = "cx22"
  image       = "ubuntu-20.04"
  location    = "nbg1"
  ssh_keys = [
    "demo@allaway.tech",
    "demo@dolphin"
  ]

  network {
    network_id = hcloud_network.Hogwarts.id
  }

  user_data = templatefile("hetzner.tftpl", { tailscale_auth_key = var.tailscale_auth_key })

  depends_on = [
    hcloud_network_subnet.Hogwarts-subnet
  ]
}

resource "hcloud_server" "Krum" {
  name        = "Krum"
  server_type = "cx22"
  image       = "ubuntu-20.04"
  location    = "nbg1"
  ssh_keys = [
    "demo@allaway.tech",
    "demo@dolphin"
  ]

  network {
    network_id = hcloud_network.Durmstrang.id
  }

  depends_on = [
    hcloud_network_subnet.Durmstrang-subnet
  ]
}
```

### TL;DR
  - Network 1 (Hogwarts) - 10.0.1.0/24
  - Network 2 (Durmstrang) - 10.1.1.0/24
  - VPS 1 (Harry) connected to Hogwarts no tailscale
  - VPS 2 (Ron) connected to Hogwarts tailscale installed
  - VPS 3 (Hermione) connected to Hogwarts and Durmstrang tailscale installed (this will be our subnet router)
  - VPS 4 (Krum) connected to Durmstrang no tailscale

All VPS's are running Ubuntu 20.04

### The non-tailscale results
Just to establish the state of the networks before we enable the subnet router on Hermione and --accept-routes on Ron here is a ping between all nodes of the network.

![Pings between all four nodes in the lab](media/posts/images/2024-09-01-tailscale-subnets/tailscale1.png)

As you can see from these the network is behaving as we expect. All nodes can communicate with each other across the Hogwarts LAN. Hermione and Krum can also talk in Durmstrang LAN.

## Enabling tailscale routes
So lets run `tailscale up --advertise-routes "10.0.1.0/24,10.1.1.0/24" --ssh --accept-routes` on Hermione and `tailscale up --accept-routes --ssh` on Ron. And then repeat the ping test.

![Setting Hermione up as a subnet router and Ron to accept routes](media/posts/images/2024-09-01-tailscale-subnets/tailscale3.png)

So now we can see that Harry can no longer talk to Ron and (this is the first time I had noticed this) Hermione also can not talk to Ron.

Just an aside. The `--ssh` isn't necessary but as I had already set the flag in the cloudinit script tailscale complains if I don't use it.

So why does the network break? If we have a look at the output of tracepath both before and after the subnets are enable it show us why.

![The results of the command tracepath for Ron before and after routes were enabled on the tailnet](media/posts/images/2024-09-01-tailscale-subnets/tailscale12.png)

As we can see Ron believes that the route back to Harry has changed from him being able to talk directly to Harry to having to go through Hermione (sounds familiar from The Goblet of Fire?). This means we have an asymetric network path causing the packets to get lost as other devices are not aware that Harry and Ron are trying to talk so discard the traffic rather than forwarding it on.

## So how can we fix this? 

After a lot of digging on the internet I found this blog post from 2021 by tailscale [The long wondrous life of a Tailscale packet](https://tailscale.com/blog/2021-05-life-of-a-packet). In this post the briefly mention that tailscale uses routing table 52 to do its magic. So if we go and look at routing table 52 it looks something like this.

![Routing table 52 used by tailscale](media/posts/images/2024-09-01-tailscale-subnets/tailscale4.png)

Here we can see that the route to the subnet 10.0.1.0/24 is being overridden to use the device tailscale0. This is why our packets are leaving through the "wrong" interface and getting lost. After some research it appears that the routing table that has the highest priority it the local one.

 > If you break this table you are likely to break your server's network as this table is used by the kernel.
 {: .prompt-warning }
 
## The Fix
The command for adding routes to the local table on Ron follow the pattern `ip route add <network address>/<subnet CIDR> via <gateway address> table local` so for my test lab that is `ip route add 10.0.1.0/24 via 10.0.0.1 table local`. The easiest way to find the correct gateway address is to run the tracepath command before applying the routes on the tailnet. This way you can just copy the address that you know already works. We can also run the exact same command on Hermione to also restore the local routes there.

Once again performing the ping tests we can see the following:

![Successful pings around the test lab](media/posts/images/2024-09-01-tailscale-subnets/tailscale11.png)

As you can see Krum can still only talk to Hermione. Harry can talk to both Ron and Hermione. And both Hermione and Ron can talk to everyone.

---
[^footnote1]: If you would like to follow along and try this for yourself then here is a referral link for Hetzner worth $20 https://hetzner.cloud/?ref=KcIFXUNR2aM4<br />The terraform configuration files are located [here](https://github.com/allaway-tech/tailscale-subnet-terraform).
