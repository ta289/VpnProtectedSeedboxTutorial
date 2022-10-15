# Guide for setting up a self-hosted VPN-protected seedbox

A seedbox is a dedicated machine used for downloading and seeding torrents. Using a seedbox instead of torrenting on your own device has several advantages
* It doesn't compete for system resources with other software
* It's convenient for torrents where seeders only appear occasionally
* You can seed 24/7, making it easier to get a high ratio or keeping niche torrents alive
* If you torrent regularly, a seedbox might cost you less than the extra electricity cost for keeping your device on

There are different ways to go about creating one, each has their own benefits and limitations. We will only cover the last type of seedbox, since you can find plenty of guides on the first two kinds.


## Types of seedboxes

### Managed seedbox

There are many seedbox providers that host a preconfigured seedbox for you. These usually come with additional utility tools as well as dashboards for tracking your activity. However, **if you plan to use public trackers you might quickly run into trouble**. Some providers explicitely forbid public trackers in their TOS or disable them on a technical level.

### Self-hosted preconfigured seedbox

Such a seedbox needs to be installed and configured on a Virtual Private Server (VPS) by yourself. There are free products (e.g. swizzin) with an easy installation process that come with utility tools and dashboards, just like in the previous option. If you live in a torrent-friendly country and use a torrent-friendly VPS provider this is probably the best option for you.

### Self-hosted VPN-protected seedbox

In addition to installing and configuring the seedbox on a VPS, this type of seedbox also redirects all of it's traffic through a VPN. The advantage is that your VPS (and thus you yourself) won't get into trouble for torrenting activity. In addition you might get a bigger selection of torrent-friendly countries, as VPS locations are usually more limited than VPN locations.

**This is the safest option which allows you to torrent without worrying at all, but it also has the biggest limitations.** First of all you are dependent on port-forwarding through your VPN (more on that later) which makes most bundled seedbox solutions unviable. In addition to that since all of your traffic goes through a VPN, this will be your bottleneck. It might make more sense to host several small seedboxes of this type, instead of one beefy seedbox. These two factors also make it less convenient to stream directly from your seedbox on a personal device.

## Overview the network structure and why it gets complicated

Normally you interact with your VPS using it's URL/hostname and the relevant port for whatever you're doing. This is very straightforward and what most of you already are familiar with.
![classic](./Classic_VPS_Communication.drawio.png)

If you want to route all of the VPS traffic through a VPN this adds a middle layer and several complications
* You no longer interact with the IP of your VPS. Instead you go through the exit IP of the VPN server you're connected to, which is hard to look up and regularly changes
* You need to port-forward through the VPN. Those VPN providers that support port-forwarding have a strict limit on the number of ports you can be assigned
* You need to port-forward on your VPS, either through supported configurations of a software (e.g. sshd_config) or by using a general approach (e.g. via iptables)

![vpn protected variant](./VPN_protected_Communication.drawio.png)

## What you need for this setup

### VPN with support for port-forwarding
You need a VPN provider that 
* you personally trust
* supports port-forwarding
* has servers in torrent-friendly countries
* ideally offers good upload/download speeds

A good VPN comparison can be found [here](https://techlore.tech/vpn). Not all of the criteria above are included in the chart.

### VPS provider
You need a VPS provider that
* doesn't prohibit P2P traffic
* offers storage focused products
Since your bottleneck is going to be your upload/download speed forget about renting SSD machines. Go with cheap HDD storage, multiple CPU cores and some decent bandwidth. Interserver and Nexusbytes offer good value for money in this case, though do your own research since these things change quickly.


### DDNS provider or a Domain registrar with DDNS support
Dynamic DNS (DDNS) is a service that regularly checks the external IP of the server it is installed on, and updates DNS records accordingly. Once you connect your VPS to your VPN you will get kicked out of it. If you haven't properly set up DDNS at that point you can start your setup all over again.

If you don't own a domain there are free DDNS service providers. If you already use a Domain registrar, check if they offer a DDNS tool or API.

# Example setup on Ubuntu 20.04 LTS
This is a beginner-friendly minimalist setup withou a graphical Torrent client and without any media server. If you succeed setting this up feel free to dabble in how to set up the media server of your choice (fuck Plex, all my homies hate Plex) or a UI for managing torrents in your browser (e.g. rTorrent, Transmission).


# Common issues

## DDNS tool/script doesn't work

Possible causes are
* Issues determining the external IP
	* Solution: Check if you can pass an IP address manually. Some tools allow for an argument using the **-i** flag. There are [several](https://linuxnightly.com/check-external-ip-from-linux-command-line/) ways to check for it manually, one simple way would be 
```bash
  -i $(wget -4 -qO - icanhazip.com)
```
* Issues connecting to the DDNS provider
	* Solution: Your DDNS provider could be blocking incoming traffic from the datacenter that is used by your VPS provider. To check that curl the address that your DDNS tool uses on your server and your personal device and compare the results. If your DDNS provider indeed blocks that datacenter, ask them if they can whitelist your IP

## Unknown address

If you try to connect to your VPS using the hostname that is updated with your DDNS tool, you might get an error that the address is unknown.

Solution: Check if your DNS records have been updated. If they were not, you have not properly set up your DDNS. If they are, you might have to wait a bit until the Time To Live (TTL) that comes with each record change.
