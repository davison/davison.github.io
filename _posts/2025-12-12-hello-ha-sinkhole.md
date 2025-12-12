---
layout: post
title: HA Sinkhole - High Availability DNS Without the Headache
abstract: I've been running Pi-hole for years now. If you're not familiar with it, it's a DNS-based ad blocker that sits on your network and intercepts requests for known advertising and tracking domains, returning nothing instead of letting them load. It's brilliant - browse the web without ads, stop smart TVs phoning home, block tracker domains on mobile apps. Once you've experienced an ad-free network, there's no going back. But there was always this nagging issue.. what happens when the Pi goes down?
comments: true
tags: privacy smarthome
---

I've been running [Pi-hole](https://github.com/pi-hole/pi-hole) for years now. If you're not familiar with it, it's a DNS-based ad blocker that sits on your network and intercepts requests for known advertising and tracking domains, returning nothing instead of letting them load. It's brilliant - browse the web without ads, stop smart TVs phoning home, block tracker domains on mobile apps. Once you've experienced an ad-free network, there's no going back. But there was always this nagging issue.. what happens when the Pi goes down?

You can mitigate this somewhat by configuring a secondary DNS server in your DHCP settings, but that just means some queries leak through to whatever fallback you've set and you can't guarantee DNS client behaviour anyway, they could slow down a lot trying to contact the defunct server before trying the new one, and then do the same again on the next request. You can try to hack together some keepalived configuration with gravity-sync between two Pi-holes, following one of the various community guides, but it's fragile, unsupported by the Pi-hole project, and troublesome to maintain.

What I really wanted was proper high availability - multiple DNS nodes sharing a virtual IP address with automatic failover. If one node dies, another picks up the load instantly. No client reconfiguration needed, no service interruption, no leaked queries to unfiltered DNS servers.

The problem is that Pi-hole wasn't designed with HA in mind. It's a monolithic system doing multiple jobs (optional DHCP, DNS, blocklist management, metrics) which makes it difficult to split across nodes in a way that actually works reliably.

So I built something different.

## Enter ha-sinkhole

[`ha-sinkhole`](https://github.com/davison/ha-sinkhole) does one thing and does it well: highly available DNS blocking. It doesn't try to be your DHCP server or provide a fancy web UI (though metrics are available if you want them). It just focuses on keeping DNS resolution and ad blocking working, even when nodes fail.

The architecture is pretty straightforward. Each DNS node runs four containers:

* **[dns-resolver](https://github.com/davison/ha-sinkhole/tree/main/dns-resolver)** - built on [CoreDNS](https://coredns.io/), handles actual DNS queries and blocklist filtering
* **[blocklist-updater](https://github.com/davison/ha-sinkhole/tree/main/blocklist-updater)** - fetches and consolidates blocklists from various sources daily
* **[vip-manager](https://github.com/davison/ha-sinkhole/tree/main/vip-manager)** - uses [keepalived](https://www.keepalived.org/) to manage the shared virtual IP between nodes
* **[stats-collector](https://github.com/davison/ha-sinkhole/tree/main/stats-collector)** - scrapes metrics and ships them to Grafana (optional)

All of this (except the vip manager) runs rootless using [podman](https://podman.io/) and is managed via systemd. The blocklist updater and dns-resolver share a volume for the consolidated blocklist file. When the file changes, CoreDNS automatically reloads it. Simple.

The clever bit is the VIP management. Each node monitors the health of its DNS service and participates in elections to determine which node should hold the virtual IP. If the primary node goes down or the DNS service fails, another node assumes the VIP within a couple of seconds. Your clients don't notice anything because they're all configured to use the VIP address - they don't care which physical node is answering.

## Why Not Just Use Pi-hole in HA?

Fair question. The Pi-hole project is excellent at what it does but the design choices that make Pi-hole simple and user-friendly also make it challenging to run in a proper HA configuration:

* It's not designed for distributed deployments
* Gravity-sync (the community HA solution) is brittle and unsupported
* The integrated DHCP server complicates failover if you're using it
* Multiple services tightly coupled make independent failure domains difficult

`ha-sinkhole` makes different trade-offs. It separates concerns - your router or other dedicated server handles DHCP, the `ha-sinkhole` DNS nodes handle resolution and blocking. Each container does one job. This makes the system more composable and much easier to make truly highly available.

## Getting Started

The [installation](https://github.com/davison/ha-sinkhole#-quick-start-guide) is deliberately simple. You need:

* Two or more Linux machines (VMs, Pis, bare metal, cloud instances - doesn't matter) to deploy `ha-sinkhole` components to
* SSH key authentication configured
* Passwordless sudo on target nodes
* Podman or Docker on your local machine
* `ssh-agent` with your key added to it on the local machine

Create an inventory file specifying your nodes and the VIP they'll share:
```yaml
dns_nodes:
  vars:
    ansible_user: pi
    vip: 192.168.0.53
    vrrp_secret: your_secret_here
  
  hosts:
    dns1:
      ansible_host: 192.168.0.1
    dns2:
      ansible_host: 192.168.0.2
```

Then run the [installer](https://github.com/davison/ha-sinkhole/tree/main/installer):
```bash
curl -sL https://bit.ly/ha-install | bash
```

The installer (which is just an [Ansible](https://docs.ansible.com/) playbook in a container) will configure both nodes in parallel. A couple of minutes later, you've got a working HA DNS setup. Point your DNS clients at the VIP and you're done.

If a node goes down, the VIP moves to another node automatically. Bring it back up and it rejoins the cluster. Make configuration changes by editing the inventory file and re-running the installer - it only applies the delta.

## Configuration Options

The [defaults](https://github.com/davison/ha-sinkhole/blob/main/installer/inventory.example.yaml) are sensible, but you can configure quite a bit:

* **Upstream DNS servers** - defaults to [Quad1](https://1.1.1.1/) and [Quad9](https://www.quad9.net/), but use whatever you prefer
* **Local domain resolution** - forward queries for your local network to your router
* **Blocklists** - defaults to [Steven Black's hosts file](https://github.com/StevenBlack/hosts), but you can add as many as you want
* **Local host overrides** - define specific host-to-IP mappings
* **Trusted networks** - restrict which networks can query your DNS

If you have a preferred primary node (maybe one machine has better hardware), you can configure it to start as `MASTER` with higher priority. Otherwise all nodes start as `BACKUP` and elect a primary based on their configuration.

## Metrics and Monitoring

By default, DNS metrics are collected but not shipped anywhere. If you want visibility into what's happening, you can [configure the stats-collector](https://github.com/davison/ha-sinkhole/tree/main/stats-collector#cloud-metrics) to push to a [Grafana Cloud](https://grafana.com/) account (they have a genuinely useful free tier). Configure three variables in your inventory:

* `prometheus_endpoint` - the push endpoint URL
* `prometheus_user` - your account number / user
* `prometheus_api_token` - a generated token with write permissions

The installer creates a podman secret from the token and systemd mounts it securely at runtime. It's never visible on disk or in environment variables.

There's a [dashboard JSON file](https://github.com/davison/ha-sinkhole/blob/main/.files/grafana-dashboard.json) in the repo you can import to get started. Having it in Grafana Cloud means you can check on things from anywhere, though obviously that does introduce a cloud dependency if you're trying to avoid those. Alternatively, install [prometheus](https://hub.docker.com/r/prom/prometheus/) and [grafana](https://hub.docker.com/r/grafana/grafana) yourself locally and adjust the 3 prmoetheus values in inventory accordingly (only the endpoint URL is required).

## What's Different from Pi-hole?

I should be clear: this isn't trying to replace Pi-hole entirely. Pi-hole has a beautiful web interface for managing things, built-in DHCP if you need it, and a huge community. If you're running a single node and that works for you, stick with Pi-hole.

But if you need:

* True high availability with automatic failover
* Containers with systemd control
* The ability to treat your DNS infrastructure as code

... then `ha-sinkhole` might be a better fit.

The configuration is all declarative - you define what you want in the inventory file and the installer makes it so. Need to add another node? Add it to the inventory and run the installer. Want to change the VIP? Edit the inventory and run the installer. It's idempotent, so you can run it as many times as you like.

## Under the Hood

If you're curious about the implementation:

* **CoreDNS** is doing the heavy lifting for DNS resolution. It's fast, well-maintained, and extremely configurable.
* **Keepalived** manages the VIP with VRRP protocol. It's battle-tested infrastructure that's been around for decades.
* **Grafana Alloy** handles metrics collection and forwarding (if you want metrics).
* **Podman quadlets** integrate containers with systemd, so everything behaves like native services.

The VIP manager runs as root (it has to, for network interface manipulation), but everything else runs rootless. DNS queries arrive on port 53 at the VIP, but `nftables` rules transparently rewrite them to port 1053 where the rootless CoreDNS container is listening. It's significantly more secure than running DNS as root.

Health checks run every few seconds. If the DNS service becomes unhealthy, keepalived moves the VIP to another node even if the container is still running. This catches issues like CoreDNS getting wedged or the blocklist becoming corrupted.

## Room for Improvement

This is definitely a "scratching my own itch" project. There are things I'd like to add:

* More sophisticated blocklist management
* Temporary overrides or additions of custom domains to (un)block
* Query logging options
* Better documentation for edge cases

But it's working for me so far at this early stage, my entire network now relies on `ha-sinkhole` and my old and lately very idle pi-hole will soon be decommissioned. I can reboot sinkhole nodes for OS updates or handle temporary hardware glitches (even swap out all the hardware) without anyone noticing. It just works.

## Give It a Try

If you're keen to have a critical service like DNS and ad-blocking with HA support, or you just want to play with container-based infrastructure, give it a shot. The installation is quick enough that you can have it running in ten minutes and tear it all down cleanly just as fast if you hate it.

The code is on GitHub at [github.com/davison/ha-sinkhole](https://github.com/davison/ha-sinkhole). It's MIT licensed, so safe to deploy in home, SoHo or commercial environments. [Contributions are welcome](https://github.com/davison/ha-sinkhole/blob/main/CONTRIBUTING.md) - there are many different ways to get involved and help improve it if you're interested.

And if you do try it, I'd really love to hear how it goes. Star the repo if you find it useful, open issues if things break, and definitely let me know if you have ideas for improvements.
